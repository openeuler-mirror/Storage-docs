# CMA Folio 用户指南

## 1. CMA 基础知识

### 1.1 什么是 CMA

**CMA（Contiguous Memory Allocator，连续内存分配器）** 是 Linux 内核中的一种内存管理机制，用于分配**物理地址连续**的大块内存。

在 Linux 系统中，内存通常由 **Buddy Allocator（伙伴系统）** 管理。Buddy Allocator 擅长分配 4KB ~ 4MB 的物理页，但随着系统运行时间增长，物理内存会逐渐碎片化，难以分配出大块的连续物理内存。

CMA 通过在内核启动时**预留**一片连续的物理内存区域，在需要时提供给需要连续内存的模块使用。

### 1.2 为什么需要 CMA

需要物理连续内存的典型场景：

| 场景 | 为什么需要连续内存 |
|------|------------------|
| **DMA 设备** | 许多 DMA 控制器只能访问连续的物理地址范围 |
| **GPU 显存** | GPU 驱动通常需要连续的物理内存做 buffer |
| **摄像头/视频编解码** | V4L2、编解码器需要连续 buffer |
| **大页（THP / HugeTLB）** | 大页需要连续的物理内存 |
| **CMA Folio** | 从连续区域分配 large folio，减少 TLB miss |

### 1.3 CMA 的工作原理

CMA 的核心工作流程：

```text
启动阶段（Boot）                    运行阶段（Runtime）
┌──────────────────┐               ┌────────────────────────┐
│ memblock 预留    │──────────────►│ CMA 区域作为 buddy     │
│ 连续物理内存     │               │ 的 MIGRATE_CMA 页面    │
└──────────────────┘               │ 可被 buddy 借用        │
                                   └────────────────────────┘
                                              │
                                   ┌──────────▼──────────┐
                                   │ cma_alloc() 需要时   │
                                   │ alloc_contig_range() │
                                   │ 回收并分配连续内存    │
                                   └─────────────────────┘
```

**关键机制**：

1. **预留内存**：内核启动时通过 `cma_declare_contiguous()` 或命令行 `cma=` 从 memblock 预留一片连续物理内存
2. **MIGRATE_CMA 类型**：预留的内存被标记为 `MIGRATE_CMA`，可以参与 buddy 分配，也可以被回收
3. **按需回收**：当驱动或子系统调用 `cma_alloc()` 时，`alloc_contig_range()` 会隔离并迁移该区域内的页面，形成连续的空闲块
4. **释放回 CMA**：`cma_release()` 将连续内存释放回 CMA 区域

### 1.4 CMA 与传统 Buddy Allocator 的区别

| 特性 | Buddy Allocator | CMA |
|------|----------------|-----|
| 分配粒度 | 4KB ~ 4MB（受 MAX_ORDER 限制） | 可分配数 MB ~ 数 GB |
| 连续性 | 不保证物理连续 | 保证物理连续 |
| 碎片化敏感度 | 运行越久越难分配大块 | 通过预留避免碎片化 |
| 内存利用率 | 高（按需分配） | 预留内存可能被借用，有开销 |
| 典型用户 | 通用内核分配 | 设备驱动、THP、CMA Folio |

---

## 2. CMA Folio 简介

### 2.1 什么是 Folio

**Folio** 是 Linux 内核中用于表示**一组连续物理页**的数据结构，是 compound page 的抽象。

- 一个 Folio 可以包含多个 page（如 2MB THP 包含 32 个 64KB 页）
- Folio 简化了大页的生命周期管理
- Page cache、THP、shmem 等子系统都使用 Folio

内核中引入了**large folio**能力，使得folio可以支持许多不同的粒度，较大粒度的large folio(MB级别，GB级别)十分依赖连续内存。

### 2.2 什么是 CMA Folio

**CMA Folio** 是 CMA 机制与 Folio 大页的结合：

> 从 CMA 预留的连续物理内存区域中分配 large folio，用于匿名内存和 shmem/tmpfs 的大页分配。

**核心思想**：

- 传统匿名内存/shmem内存使用 buddy 分配 4KB/64KB 小页
- CMA Folio 直接从连续内存区域分配 2MB / 32MB / 512MB 的大 folio
- 减少页表项数量，降低 TLB miss，提升 IO 和内存访问性能

### 2.3 CMA Folio 的优势

| 优势 | 说明 |
|------|------|
| **更大的 folio** | 可分配 PMD-order（如64K页表下的512MB）的大页
| **减少 TLB miss** | 大页覆盖更大的虚拟地址范围，减少页表遍历 |
| **提升 IO 吞吐** | page cache 使用大页时，访问传输更高效|
| **降低元数据开销** | 单个 folio 替代多个 page，减少 `struct page` 和 rmap 开销 |

### 2.4 适用架构

CMA Folio 主要针对 **arm64 64K page** 系统设计：

- arm64 64K page 下，PMD-order folio 可达 **512MB**
- 64K page 的 TLB 效率更高，大页收益更明显
- 当前 Kconfig 限制 `CONFIG_CMA_FOLIO` 依赖于 `ARM64`

---

## 3. 启用与配置

### 3.1 编译时配置

确保内核配置开启以下选项：

```bash
CONFIG_CMA=y
CONFIG_CMA_FOLIO=y
CONFIG_TRANSPARENT_HUGEPAGE=y
CONFIG_NUMA=y          # 如果使用 NUMA 策略
```

`CONFIG_CMA_FOLIO` 的依赖关系：

```bash
config CMA_FOLIO
    bool "CMA based folio allocation"
    depends on CMA && TRANSPARENT_HUGEPAGE
    depends on ARM64
```

### 3.2 启动参数配置

通过 `folio_cma=` 内核启动参数预留 CMA Folio 内存：

```bash
# 每个 NUMA 节点预留 512MB
folio_cma=512M

# 为指定节点预留（节点 0: 512M，节点 1: 1G）
follio_cma=0:512M,1:1G

```

**参数格式**：

- 纯大小：`folio_cma=512M`（应用到所有 online 节点）
- 节点指定：`folio_cma=<nid>:<size>`
- 逗号分隔多个节点

**常用组合示例**：

```bash
# 推荐：总内存的 10% ~ 25% 用于 CMA Folio
folio_cma=4G transparent_hugepage=always
```

### 3.3 运行时控制

CMA Folio 是 **per-task opt-in** 机制，默认不开启。需要通过 `/proc/<pid>/folio_cma_enabled` 为每个进程启用：

```bash
# 为当前 shell 启用
sudo sh -c 'echo 1 > /proc/self/folio_cma_enabled'

# 为指定进程启用（例如 pid 1234）
sudo sh -c 'echo 1 > /proc/1234/folio_cma_enabled'

# 查看状态
cat /proc/self/folio_cma_enabled
```

**子进程继承**：

- 子进程不继承父进程的folio_cma_enabled设置
- 可通过 `/proc/<pid>/folio_cma_enabled` 随时关闭

---

## 4. 使用场景

### 4.1 匿名内存加速

对于存在大量匿名内存的工作负载，CMA Folio 可以确保匿名内存用上thp：

```bash
# 启用 CMA Folio 后，通过mmap+madvise申请内存
echo 1 > /proc/self/folio_cma_enabled
```

**适用场景**：

- 大块匿名内存使用的场景
- 需要确保较低TLB miss率以提升性能

### 4.2 shmem/tmpfs THP

tmpfs 上的文件也可以使用 CMA Folio：

```bash
# 挂载 tmpfs 并启用 THP
mount -t tmpfs -o size=10G,huge=always none /mnt/tmpfs

# 在启用 folio_cma 的进程中写入 tmpfs
echo 1 > /proc/self/folio_cma_enabled
dd if=/dev/zero of=/mnt/tmpfs/bigfile bs=1M count=1024
```

---

## 5. 监控与验证

### 5.1 查看 CMA 区域

```bash
# 查看 CMA 区域总览
cat /proc/meminfo | grep Cma

# 示例输出：
# CmaTotal:       4194304 kB
# CmaFree:        2097152 kB

# 查看具体 CMA 区域（需要 debugfs，需要cma_debug）
mount -t debugfs none /sys/kernel/debug
cat /sys/kernel/debug/cma/folio_cma0
```

### 5.2 查看 per-task 状态

```bash
# 查看当前进程是否启用 CMA Folio
cat /proc/self/folio_cma_enabled

# 查看子进程状态
for pid in $(pgrep myapp); do
    echo "pid $pid: $(cat /proc/$pid/folio_cma_enabled)"
done
```

---

## 6. 故障排查

**检查清单**：

1. 内核是否开启 `CONFIG_CMA_FOLIO`

   ```bash
   grep CONFIG_CMA_FOLIO /boot/config-$(uname -r)
   ```

2. 启动参数是否正确

   ```bash
   cat /proc/cmdline | grep folio_cma
   ```

3. 当前进程是否启用 per-task 开关

   ```bash
   cat /proc/self/folio_cma_enabled
   ```

4. CMA 区域是否预留成功

   ```bash
   dmesg | grep folio_cma
   cat /proc/meminfo | grep Cma
   ```

---

## 7. 参考链接

- Linux Kernel Community Documentation: [CMA](https://lwn.net/Articles/396707/)
- Linux Kernel Documentation: [Transparent Hugepage Support](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html)
- `Documentation/core-api/pin_user_pages.rst`
- `mm/cma.c`, `mm/cma_folio.c`, `mm/mempolicy.c`

---

## 快速参考卡

| 操作 | 命令 |
|------|------|
| 查看 CMA 总量/空闲量 | `cat /proc/meminfo \| grep Cma` |
| 启用当前进程 CMA Folio | `echo 1 > /proc/self/folio_cma_enabled` |
| 查看当前进程状态 | `cat /proc/self/folio_cma_enabled` |
| 启动参数预留 4G | `folio_cma=4G` |
| 按节点预留 | `folio_cma=0:2G,1:2G` |
| 查看 CMA debug 信息 | `/sys/kernel/debug/cma/folio_cma0` |

---

*本指南适用于基于 Linux 6.6 的 CMA Folio 实现。具体行为和参数可能随内核版本变化。*
