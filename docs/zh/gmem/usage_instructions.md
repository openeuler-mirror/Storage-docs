# 使用说明

## 简介

GMEM通过特定的flag申请对等互访的虚拟内存，并对外提供了一些内存优化语义，通过这部分内存语义可以达到性能优化的效果。
libgmem是GMEM用户接口的抽象层，主要功能是是对上述内存语义进行封装，简化用户的使用。

## 接口说明

* 内存申请

    GMEM扩展了mmap的含义，增加了一个flag MAP_PEER_SHARED申请异构内存，使用时默认返回2MB对齐的虚拟地址。

    ```cpp
    addr = mmap(NULL , size, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS | MAP_PEER_SHARED, -1, 0);
    ```

* 内存释放
    
    通过munmap接口释放host和device的内存。

    ```cpp
    munmap(addr, size);
    ```

* 内存语义

    FreeEager：对于给定范围[addr, addr + size]的地址段，FreeEager会对范围向内对齐页面大小的完整页面进行释放（默认页面大小2M）。如果范围内不存在完整页面，将直接返回成功。

    接口成功返回0，失败返回错误码。

    ```cpp
    接口原型: int gmemFreeEager(unsigned long addr, size_t size, void *stream);
    接口用法: ret = gmemFreeEager(addr, size, stream);
    ```

    Prefetch：对于给定范围[addr, addr + size]的地址段，Prefetch会对范围向外对齐页面大小的完整页面（覆盖整个地址段）进行预取。确保指定的计算单元设备hnid在接下来对vma发起的访问不会触发page fault。

    接口成功返回0，失败返回错误码。

    ```cpp
    接口原型: int gmemPrefetch(unsigned long addr, size_t size, int hnid, void *stream);
    接口用法: ret = gmemPrefetch(addr, size, hnid, stream);
    ```

    在上述内存语义使用的时候stream为空表示同步调用，非空表示异步调用。

* 其他接口

    获取当前设备的numaid。接口成功返回设备号，失败返回错误码。

    ```cpp
    接口原型: int gmemGetNumaId(void);
    接口用法: numaid = gmemGetNumaId();
    ```

    获取内核的gmem统计信息。

    ```sh
    cat /proc/gmemstat
    ```

## 约束限制

1. 目前仅支持2M大页，所以host OS以及NPU卡内OS的透明大页需要默认开启。
2. 通过MAP_PEER_SHARED申请的异构内存目前不支持fork时继承。
