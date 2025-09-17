# Development with HSAK

## **nvme.conf.in** Configuration File

By default, the HSAK configuration file is located in **/etc/spdk/nvme.conf.in**. You can modify the configuration file based on service requirements. The content of the configuration file is as follows:

- \[Global\]

1. **ReactorMask**: cores used for I/O polling. The value is a hexadecimal number and cannot be set to core 0. The bits from the least significant one to the most significant one indicate different CPU cores. For example, 0x1 indicates core 0, and 0x6 indicates cores 1 and 2. This parameter supports a maximum of 34 characters, including the hexadecimal flag **0x**. Each hexadecimal character can be F at most, indicating four cores. Therefore, a maximum of 128 (32 x 4) cores are supported.
2. **LogLevel**: HSAK log print level (**0**: error; **1**: warning; **2**: notice; **3**: info; **4**: debug).
3. **MemSize**: memory occupied by HSAK (The minimum value is 500 MB.)
4. **MultiQ**: whether to enable multi-queue on the same block device.
5. **E2eDif**: DIF type (**1**: half-way protection; **2**: full protection). Drives from different vendors may have different DIF support capabilities. For details, see the documents provided by hardware vendors.
6. **IoStat**: whether to enable the I/O statistics function. The options are **Yes** and **No**.
7. **RpcServer**: whether to start the RPC listening thread. The options are **Yes** and **No**.
8. **NvmeCUSE**: whether to enable the CUSE function. The options are **Yes** and **No**. After the function is enabled, the NVMe character device is generated in the **/dev/spdk** directory.

- \[Nvme\]

1. **TransportID**: PCI address and name of the NVMe controller. The format is **TransportID "trtype:PCIe traddr:0000:09:00.0" nvme0**.
2. **RetryCount**: number of retries upon an I/O failure. The value **0** indicates no retry. The maximum value is **255**.
3. **TimeoutUsec**: I/O timeout interval. If this parameter is set to **0** or left blank, no timeout interval is set. The unit is μs.
4. **ActionOnTimeout**: I/O timeout behavior (**None**: prints information only; **Reset**: resets the controller; **abort**: aborts the command). The default value is **None**.

- \[Reactor\]

1. **BatchSize**: number of I/Os that can be submitted in batches. The default value is **8**, and the maximum value is **32**.

## Header File Reference

HSAK provides two external header files. Include the two files when using HSAK for development.

1. **bdev_rw.h**: defines the macros, enumerations, data structures, and APIs of the user-mode I/O operations on the data plane.
2. **ublock.h**: defines macros, enumerations, data structures, and APIs for functions such as device management and information obtaining on the management plane.

## Service Running

After software development and compilation, you must run the **setup.sh** script to rebind the NVMe drive driver to the user mode before running the software. The script is located in **/opt/spdk** by default.
Run the following commands to change the drive driver's binding mode from kernel to user and reserve 1024 x 2 MB huge pages:

```shell
[root@localhost ~]# cd /opt/spdk
[root@localhost spdk]# ./setup.sh
0000:3f:00.0 (8086 2701): nvme -> uio_pci_generic
0000:40:00.0 (8086 2701): nvme -> uio_pci_generic
```

Run the following commands to restore the drive driver's mode from user to kernel and free the reserved huge pages:

```shell
[root@localhost ~]# cd /opt/spdk
[root@localhost spdk]# ./setup.sh reset
0000:3f:00.0 (8086 2701): uio_pci_generic -> nvme
0000:40:00.0 (8086 2701): uio_pci_generic -> nvme
```

## User-Mode I/O Read and Write Scenarios

Call HSAK APIs in the following sequence to read and write service data through the user-mode I/O channel:

1. Initialize the HSAK UIO module.
    Call **libstorage_init_module** to initialize the HSAK user-mode I/O channel.

2. Open a drive block device.
    Call **libstorage_open** to open a specified block device. If multiple block devices need to be opened, call this API repeatedly.

3. Allocate I/O memory.
    Call **libstorage_alloc_io_buf** or **libstorage_mem_reserve** to allocate memory. **libstorage_alloc_io_buf** can allocate a maximum of 65 KB I/Os, and **libstorage_mem_reserve** can allocate unlimited memory unless there is no available space.

4. Perform read and write operations on a drive.
    You can call the following APIs to perform read and write operations based on service requirements:

    - libstorage_async_read
    - libstorage_async_readv
    - libstorage_async_write
    - libstorage_async_writev
    - libstorage_sync_read
    - libstorage_sync_write

5. Free I/O memory.
    Call **libstorage_free_io_buf** or **libstorage_mem_free** to free memory, which must correspond to the API used to allocate memory.

6. Close a drive block device.
    Call **libstorage_close** to close a specified block device. If multiple block devices are opened, call this API repeatedly to close them.

    | API                     | Description                                                                         |
    | ----------------------- | ----------------------------------------------------------------------------------- |
    | libstorage_init_module  | Initializes the HSAK module.                                                        |
    | libstorage_open         | Opens a block device.                                                               |
    | libstorage_alloc_io_buf | Allocates memory from buf_small_pool or buf_large_pool of SPDK.                     |
    | libstorage_mem_reserve  | Allocates memory space from the huge page memory reserved by DPDK.                  |
    | libstorage_async_read   | Delivers asynchronous I/O read requests (the read buffer is a contiguous buffer).   |
    | libstorage_async_readv  | Delivers asynchronous I/O read requests (the read buffer is a discrete buffer).     |
    | libstorage_async_write  | Delivers asynchronous I/O write requests (the write buffer is a contiguous buffer). |
    | libstorage_async_wrtiev | Delivers asynchronous I/O write requests (the write buffer is a discrete buffer).   |
    | libstorage_sync_read    | Delivers synchronous I/O read requests (the read buffer is a contiguous buffer).    |
    | libstorage_sync_write   | Delivers synchronous I/O write requests (the write buffer is a contiguous buffer).  |
    | libstorage_free_io_buf  | Frees the allocated memory to buf_small_pool or buf_large_pool of SPDK.             |
    | libstorage_mem_free     | Frees the memory space that libstorage_mem_reserve allocates.                       |
    | libstorage_close        | Closes a block device.                                                              |
    | libstorage_exit_module  | Exits the HSAK module.                                                              |

## Drive Management Scenarios

HSAK contains a group of C APIs, which can be used to format drives and create and delete namespaces.

1. Call the C API to initialize the HSAK UIO component. If the HSAK UIO component has been initialized, skip this operation.

    libstorage_init_module

2. Call corresponding APIs to perform drive operations based on service requirements. The following APIs can be called separately:

    - libstorage_create_namespace

    - libstorage_delete_namespace

    - libstorage_delete_all_namespace

    - libstorage_nvme_create_ctrlr

    - libstorage_nvme_delete_ctrlr

    - libstorage_nvme_reload_ctrlr

    - libstorage_low_level_format_nvm

    - libstorage_deallocate_block

3. If you exit the program, destroy the HSAK UIO. If other services are using the HSAK UIO, you do not need to exit the program and destroy the HSAK UIO.

    libstorage_exit_module

    | API                             | Description                                                                                                            |
    | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
    | libstorage_create_namespace     | Creates a namespace on a specified controller (the prerequisite is that the controller supports namespace management). |
    | libstorage_delete_namespace     | Deletes a namespace from a specified controller.                                                                       |
    | libstorage_delete_all_namespace | Deletes all namespaces from a specified controller.                                                                    |
    | libstorage_nvme_create_ctrlr    | Creates an NVMe controller based on the PCI address.                                                                   |
    | libstorage_nvme_delete_ctrlr    | Destroys an NVMe controller based on the controller name.                                                              |
    | libstorage_nvme_reload_ctrlr    | Automatically creates or destroys the NVMe controller based on the input configuration file.                           |
    | libstorage_low_level_format_nvm | Low-level formats an NVMe drive.                                                                                       |
    | libstorage_deallocate_block     | Notifies NVMe drives of blocks that can be freed for garbage collection.                                               |

## Data-Plane Drive Information Query

The I/O data plane of HSAK provides a group of C APIs for querying drive information. Upper-layer services can process service logic based on the queried information.

1. Call the C API to initialize the HSAK UIO component. If the HSAK UIO component has been initialized, skip this operation.

    libstorage_init_module

2. Call corresponding APIs to query information based on service requirements. The following APIs can be called separately:

    - libstorage_get_nvme_ctrlr_info

    - libstorage_get_mgr_info_by_esn

    - libstorage_get_mgr_smart_by_esn

    - libstorage_get_bdev_ns_info

    - libstorage_get_ctrl_ns_info

3. If you exit the program, destroy the HSAK UIO. If other services are using the HSAK UIO, you do not need to exit the program and destroy the HSAK UIO.

    libstorage_exit_module

    | API                             | Description                                                              |
    | ------------------------------- | ------------------------------------------------------------------------ |
    | libstorage_get_nvme_ctrlr_info  | Obtains information about all controllers.                               |
    | libstorage_get_mgr_info_by_esn  | Obtains the management information of the drive corresponding to an ESN. |
    | libstorage_get_mgr_smart_by_esn | Obtains the S.M.A.R.T. information of the drive corresponding to an ESN. |
    | libstorage_get_bdev_ns_info     | Obtains namespace information based on the device name.                  |
    | libstorage_get_ctrl_ns_info     | Obtains information about all namespaces based on the controller name.   |

## Management-Plane Drive Information Query

The management plane component Ublock of HSAK provides a group of C APIs for querying drive information on the management plane.

1. Call the C API to initialize the HSAK Ublock server.

2. Call the HSAK UIO component initialization API in another process based on service requirements.

3. If multiple processes are required to query drive information, initialize the Ublock client.

4. Call the APIs listed in the following table on the Ublock server process or client process to query information.

5. After obtaining the block device list, call the APIs listed in the following table to free resources.

6. If you exit the program, destroy the HSAK Ublock module (the destruction method on the server is the same as that on the client).

    | API                          | Description                                                  |
    | ---------------------------- | ------------------------------------------------------------ |
    | init_ublock                  | Initializes the Ublock function module. This API must be called before the other Ublock APIs. A process can be initialized only once because the init_ublock API initializes DPDK. The initial memory allocated by DPDK is bound to the process PID. One PID can be bound to only one memory. In addition, DPDK does not provide an API for freeing the memory. The memory can be freed only by exiting the process. |
    | ublock_init                  | It is the macro definition of the init_ublock API. It can be considered as initializing Ublock to an RPC service. |
    | ublock_init_norpc            | It is the macro definition of the init_ublock API. It can be considered as initializing Ublock to a non-RPC service. |
    | ublock_get_bdevs             | Obtains the device list. The obtained device list contains only PCI addresses and does not contain specific device information. To obtain specific device information, call the ublock_get_bdev API. |
    | ublock_get_bdev              | Obtains information about a specific device, including the device serial number, model, and firmware version. The information is stored in character arrays instead of character strings. |
    | ublock_get_bdev_by_esn       | Obtains the device information based on the specified ESN, including the serial number, model, and firmware version. |
    | ublock_get_SMART_info        | Obtains the S.M.A.R.T. information of a specified device.    |
    | ublock_get_SMART_info_by_esn | Obtains the S.M.A.R.T. information of the device corresponding to an ESN. |
    | ublock_get_error_log_info    | Obtains the error log information of a device.               |
    | ublock_get_log_page          | Obtains information about a specified log page of a specified device. |
    | ublock_free_bdevs            | Frees the device list.                                       |
    | ublock_free_bdev             | Frees device resources.                                      |
    | ublock_fini                  | Destroys the Ublock module. This API destroys the Ublock module and internally created resources. This API must be used together with the Ublock initialization API. |

## Log Management

HSAK logs are exported to **/var/log/messages** through syslog by default and managed by the rsyslog service of the OS. If a custom log directory is required, use rsyslog to configure the log directory.

1. Modify the **/etc/rsyslog.conf** configuration file.

2. Restart the rsyslog service:

    ```shell
    if ($programname == 'LibStorage') then {
        action(type="omfile" fileCreateMode="0600" file="/var/log/HSAK/run.log")
        stop
    }
    ```

3. Start the HSAK process. The log information is redirected to the target directory.

    ```shell
    sysemctl restart rsyslog
    ```

4. If redirected logs need to be dumped, manually configure log dump in the **/etc/logrotate.d/syslog** file.
