# Usage Instructions

## Introduction

GMEM applies for virtual memory that is peer-to-peer accessible through a specific flag and provides some memory optimization semantics externally. Performance can be optimized through the memory semantics.
libgmem is the abstraction layer of the GMEM user API. It encapsulates the preceding memory semantics to simplify user operations.

## Available APIs

* Memory application

  GMEM extends `mmap` semantics and adds a MAP_PEER_SHARED flag to apply for heterogeneous memory. When the flag is used, 2 MB-aligned virtual addresses are returned by default.

  ```c
  addr = mmap(NULL , size, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS | MAP_PEER_SHARED, -1, 0);
  ```

* Memory release

  `munmap` is used to release memory of hosts and devices.

  ```c
  munmap(addr, size);
  ```

* Memory semantics

  `FreeEager`: For an address segment within the specified range \[**addr**, **addr** + **size**], `FreeEager` releases a complete page that aligns the page size inwards (the default page size is 2 MB). If no complete page exists in the range, a success message is returned.

  If the API is invoked successfully, 0 is returned. Otherwise, an error code is returned.

  ```c
  Prototype: `int gmemFreeEager(unsigned long addr, size_t size, void *stream);`
  Usage: `ret = gmemFreeEager(addr, size, stream);`
  ```

  `Prefetch`: For an address segment with the specified range \[**addr**, **addr** + **size**], `Prefetch` prefetches a complete page (covering the entire address segment) whose range is outward aligned with the page size. This ensures that the subsequent access to the virtual memory area initiated by the specified computing unit device **hnid** does not trigger a page fault.

  If the API is invoked successfully, 0 is returned. Otherwise, an error code is returned.

  ```c
  Prototype: `int gmemPrefetch(unsigned long addr, size_t size, int hnid, void *stream);`
  Usage: `ret = gmemPrefetch(addr, size, hnid, stream);`
  ```

  If the value of **stream** is empty, the invocation is synchronous. Otherwise, the invocation is asynchronous.

* Other APIs

  Obtain the NUMA ID of the current device. If the API is invoked successfully, the NUMA ID is returned. Otherwise, an error code is returned.

  ```c
  Prototype: `int gmemGetNumaId (void);`
  Usage: `numaid = gmemGetNumaId ();`
  ```

   Obtain the GMEM statistics of the kernel.

  ```sh
  cat /proc/gmemstat
  ```

## Constraints

1. GMEM supports only 2 MB huge pages. Therefore, transparent huge pages of the host OS and NPU OS must be enabled to use GMEM.
2. The heterogeneous memory obtained using `MAP_PEER_SHARED` cannot be inherited during forking.
