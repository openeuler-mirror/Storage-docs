# CMA Folio User Guide

## 1. CMA Basics

### 1.1 What Is CMA

Contiguous Memory Allocator (CMA) is a memory management mechanism within the Linux kernel used to allocate large blocks of contiguous physical memory.

In a Linux system, memory is typically managed by Buddy Allocator. While Buddy Allocator excels at allocating physical pages ranging from 4 KB to 4 MB, physical memory gradually becomes fragmented as system uptime increases, making it difficult to allocate large blocks of contiguous physical memory.

CMA addresses this issue by reserving a contiguous region of physical memory at kernel boot time, which is then provisioned to modules that require contiguous memory on demand.

### 1.2 Why CMA

Typical scenarios requiring contiguous physical memory include:

| Scenario | Why Contiguous Memory is Required |
|------|------------------|
| **DMA devices** | Many DMA controllers can only access contiguous physical address ranges. |
| **GPU VRAM** | GPU drivers typically require contiguous physical memory to serve as buffers. |
| **Camera/Video codecs** | V4L2 and multimedia codecs require contiguous buffers. |
| **Huge pages (THP/HugeTLB)** | Huge pages inherently require contiguous physical memory. |
| **CMA Folio** | Allocates large folios from contiguous regions to reduce TLB misses. |

### 1.3 How CMA Works

The core workflow of CMA consists of the following phases:

```text
              Boot phase                                                Runtime phase
┌──────────────────────────────────────┐               ┌────────────────────────────────────────────┐
│ The memblock allocator reserves a    │──────────────►│ The CMA region serves as MIGRATE_CMA pages │
│ contiguous region of physical memory.│               | within Buddy Allocator. These pages can be │
└──────────────────────────────────────┘               │ temporarily borrowed by the buddy system.  │
                                                       └────────────────────────────────────────────┘
                                                                              │
                                                      ┌───────────────────────▼──────────────────────────┐
                                                      │ When cma_alloc() is invoked,                     │
                                                      │ alloc_contig_range() reclaims the borrowed pages │
                                                      │ and allocates the requested contiguous memory.   │
                                                      └──────────────────────────────────────────────────┘
   ```

**Key mechanisms**:

- **Memory reservation**: During kernel boot, a contiguous block of physical memory is reserved from `memblock` via `cma_declare_contiguous()` or `cma=`.
- **MIGRATE_CMA**: The reserved memory is tagged as `MIGRATE_CMA` pages, which allows them to participate in regular buddy allocation and be reclaimed when necessary.
- **On-demand reclamation**: When a driver or subsystem invokes `cma_alloc()`, `alloc_contig_range()` isolates and migrates any active pages within that region to form a contiguous block of free memory.
- **Release back to CMA**: `cma_release()` frees the contiguous memory block back into the CMA pool.

### 1.4 Differences Between CMA and Buddy Allocator

| Item | Buddy Allocator | CMA |
|------|----------------|-----|
| Allocation granularity | 4 KB to 4 MB (limited by `MAX_ORDER`) | Several MBs to multiple GBs. |
| Contiguity | Does not guarantee physical contiguity. | Guarantees physical contiguity. |
| Fragmentation sensitivity | The longer the system runs, the harder it is to allocate large blocks | Avoids fragmentation via memory reservation |
| Memory utilization | High (allocated on demand) | High (reserved memory can be borrowed by buddy, though with migration overhead) |
| Typical users | General-purpose kernel allocations | Device drivers, THP, CMA Folio |

---

## 2. CMA Folio

### 2.1 What Is a Folio

A folio is a data structure in the Linux kernel used to represent a group of contiguous physical pages, serving as an abstraction over compound pages.

- A folio can contain multiple pages (for example, a 2 MB THP contains 32 x 64 KB pages under a 64 KB base page configuration).
- Folios streamline the lifecycle management of large pages.
- Subsystems such as the page cache, THP, and shmem all utilize folios.

The kernel introduces large folio capabilities, allowing folios to support a wide range of granularities. Large-granularity folios (at the MB or GB level) are highly dependent on contiguous memory.

### 2.2 What Is CMA Folio

CMA Folio is the integration of the CMA mechanism with large folios:

> It allocates large folios directly from the reserved CMA contiguous physical memory regions to support large page allocations for anonymous memory, shmem, and tmpfs.

**Core concepts**:

- Conventional approach: Anonymous memory and shmem use Buddy Allocator to allocate standard 4 KB or 64 KB small pages.
- CMA Folio approach: Directly allocates 2 MB, 32 MB, or 512 MB large folios from the contiguous memory regions.
- Benefits: Reduces the number of page table entries, decreases TLB misses, and boosts I/O throughput as well as memory access performance.

### 2.3 Advantages of CMA Folio

| Advantage | Description |
|------|------|
| **Larger folios** | Enables the allocation of massive PMD-order folios (for example, 512 MB under a 64 KB base page configuration). |
| **Reduced TLB misses** | Large folios cover a broader virtual address range, significantly reducing page table walks. |
| **Enhanced I/O throughput** | When the page cache utilizes large folios, data access and transfer operations become highly efficient.|
| **Lower metadata overhead** | A single folio replaces multiple individual pages, reducing the overhead associated with `struct page` management and reverse mapping (rmap).|

### 2.4 Supported Architectures

CMA Folio is primarily designed for ARM64 systems running with a 64KB base page configuration:

- Under an ARM64 64 KB page system, a PMD-order folio can scale up to 512 MB.
- The TLB efficiency of 64 KB pages is higher, making the performance gains of large folios even more pronounced.
- Currently, the Kconfig limits `CONFIG_CMA_FOLIO` to explicitly depend on the ARM64 architecture.

---

## 3. Enablement and Configuration

### 3.1 Compile-Time Configuration

Ensure that the following options are enabled in your kernel configuration:

```bash
CONFIG_CMA=y
CONFIG_CMA_FOLIO=y
CONFIG_TRANSPARENT_HUGEPAGE=y
CONFIG_NUMA=y          # If using NUMA policies
```

Dependencies for `CONFIG_CMA_FOLIO`:

```bash
config CMA_FOLIO
    bool "CMA based folio allocation"
    depends on CMA && TRANSPARENT_HUGEPAGE
    depends on ARM64
```

### 3.2 Boot Parameter Configuration

Reserve CMA Folio memory via the `folio_cma=` kernel boot parameter:

```bash
# Reserve 512 MB for each NUMA node.
folio_cma=512M

# Reserve for specific nodes (512 MB for Node 0, 1 GB for Node 1).
folio_cma=0:512M,1:1G

```

**Parameter format**:

- Flat size: `folio_cma=512M` (applied to all online nodes)
- Node-specific: `folio_cma=<nid>:<size>`
- Multiple nodes can be specified using a comma-separated list.

**Recommended configuration example**:

```bash
# Recommendation: Allocate 10% to 25% of the total system memory for CMA Folio.
folio_cma=4G transparent_hugepage=always
```

### 3.3 Runtime Control

CMA Folio uses a **per-task opt-in** mechanism and is disabled by default. It must be enabled for each process via `/proc/<pid>/folio_cma_enabled`:

```bash
# Enable for the current shell.
sudo sh -c 'echo 1 > /proc/self/folio_cma_enabled'

# Enable for a specific process (for example, PID 1234).
sudo sh -c 'echo 1 > /proc/1234/folio_cma_enabled'

# Check the status.
cat /proc/self/folio_cma_enabled
```

**Child process inheritance**:

- Child processes do not inherit the `folio_cma_enabled` setting from their parent process.
- The feature can be disabled at any time via `/proc/<pid>/folio_cma_enabled`.

---

## 4. Application Scenarios

### 4.1 Anonymous Memory Acceleration

For workloads with large anonymous memory footprints, CMA Folio ensures that anonymous memory successfully utilizes Transparent Huge Pages (THP):

```bash
# Once CMA Folio is enabled, allocate memory via mmap combined with madvise.
echo 1 > /proc/self/folio_cma_enabled
```

**Applicable scenarios**:

- Scenarios utilizing massive blocks of anonymous memory
- Applications requiring a consistently low TLB miss rate to boost performance

### 4.2 shmem/tmpfs THP

Files stored on tmpfs can also leverage CMA Folio:

```bash
# Mount tmpfs with THP enabled.
mount -t tmpfs -o size=10G,huge=always none /mnt/tmpfs

# Write to tmpfs from within a process that has folio_cma_enabled active.
echo 1 > /proc/self/folio_cma_enabled
dd if=/dev/zero of=/mnt/tmpfs/bigfile bs=1M count=1024
```

---

## 5. Monitoring and Verification

### 5.1 Viewing CMA Regions

```bash
# View the CMA region overview.
cat /proc/meminfo | grep Cma

# Example output:
# CmaTotal:       4194304 kB
# CmaFree:        2097152 kB

# View specific CMA regions (requiring debugfs and CONFIG_CMA_DEBUGFS).
mount -t debugfs none /sys/kernel/debug
cat /sys/kernel/debug/cma/folio_cma0
```

### 5.2 Viewing Per-Task Status

```bash
# Check if CMA Folio is enabled for the current process.
cat /proc/self/folio_cma_enabled

# Check the status of a specific child process.
for pid in $(pgrep myapp); do
    echo "pid $pid: $(cat /proc/$pid/folio_cma_enabled)"
done
```

---

## 6. Troubleshooting

**Checklist**:

1. Verify if `CONFIG_CMA_FOLIO` is enabled in the running kernel.

   ```bash
   grep CONFIG_CMA_FOLIO /boot/config-$(uname -r)
   ```

2. Ensure the boot parameter is correct.

   ```bash
   cat /proc/cmdline | grep folio_cma
   ```

3. Confirm that the folio_cma_enabled switch is explicitly turned on for the target process.

   ```bash
   cat /proc/self/folio_cma_enabled
   ```

4. Check whether the CMA region was successfully reserved and allocated at boot time.

   ```bash
   dmesg | grep folio_cma
   cat /proc/meminfo | grep Cma
   ```

---

## 7. References

- Linux Kernel Community Documentation: [CMA](https://lwn.net/Articles/396707/)
- Linux Kernel Documentation: [Transparent Hugepage Support](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html)
- `Documentation/core-api/pin_user_pages.rst`
- `mm/cma.c`, `mm/cma_folio.c`, `mm/mempolicy.c`

---

## Quick Reference Card

| Operation | Command |
|------|------|
| View total/free CMA memory | `cat /proc/meminfo \| grep Cma` |
| Enable CMA Folio for current process | `echo 1 > /proc/self/folio_cma_enabled` |
| Check current process status | `cat /proc/self/folio_cma_enabled` |
| Reserve 4 GB via the boot parameter | `folio_cma=4G` |
| Reserve memory by node | `folio_cma=0:2G,1:2G` |
| View CMA debug information | `/sys/kernel/debug/cma/folio_cma0` |

---

*This guide applies to the CMA Folio implementation based on Linux kernel 6.6. Specific behaviors and parameters may vary across different kernel versions.*
