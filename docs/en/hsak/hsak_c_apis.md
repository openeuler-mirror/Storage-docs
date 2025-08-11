# C APIs

## Macro Definition and Enumeration

### bdev_rw.h

#### enum libstorage_ns_lba_size

1. Prototype

    ```c
    enum libstorage_ns_lba_size
    {
    LIBSTORAGE_NVME_NS_LBA_SIZE_512 = 0x9,
    LIBSTORAGE_NVME_NS_LBA_SIZE_4K = 0xc
    };
    ```

2. Description

    Sector (data) size of a drive.

#### enum libstorage_ns_md_size

1. Prototype

    ```c
    enum libstorage_ns_md_size
    {
    LIBSTORAGE_METADATA_SIZE_0 = 0,
    LIBSTORAGE_METADATA_SIZE_8 = 8,
    LIBSTORAGE_METADATA_SIZE_64 = 64
    };
    ```

2. Description

    Metadata size of a drive.

3. Remarks

    - ES3000 V3 (single-port) supports formatting of five sector types (512+0, 512+8, 4K+64, 4K, and 4K+8).

    - ES3000 V3 (dual-port) supports formatting of four sector types (512+0, 512+8, 4K+64, and 4K).

    - ES3000 V5 supports formatting of five sector types (512+0, 512+8, 4K+64, 4K, and 4K+8).

    - Optane drives support formatting of seven sector types (512+0, 512+8, 512+16,4K, 4K+8, 4K+64, and 4K+128).

#### enum libstorage_ns_pi_type

1. Prototype

    ```c
    enum libstorage_ns_pi_type
    {
    LIBSTORAGE_FMT_NVM_PROTECTION_DISABLE = 0x0,
    LIBSTORAGE_FMT_NVM_PROTECTION_TYPE1 = 0x1,
    LIBSTORAGE_FMT_NVM_PROTECTION_TYPE2 = 0x2,
    LIBSTORAGE_FMT_NVM_PROTECTION_TYPE3 = 0x3,
    };
    ```

2. Description

    Protection type supported by drives.

3. Remarks

    ES3000 supports only protection types 0 and 3. Optane drives support only protection types 0 and 1.

#### enum libstorage_crc_and_prchk

1. Prototype

    ```c
    enum libstorage_crc_and_prchk
    {
    LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK = 0x0,
    LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK = 0x1,
    LIBSTORAGE_LIB_CRC_AND_DISABLE_PRCHK = 0x2,
    LIBSTORAGE_LIB_CRC_AND_ENABLE_PRCHK = 0x3,
    #define NVME_NO_REF 0x4
    LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK_NO_REF = LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK | NVME_NO_REF,
    LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK_NO_REF = LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK | NVME_NO_REF,
    };
    ```

2. Description

    - **LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK**: Cyclic redundancy check (CRC) is performed for the application layer, but not for HSAK. CRC is disabled for drives.

    - **LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK**: CRC is performed for the application layer, but not for HSAK. CRC is enabled for drives.

    - **LIBSTORAGE_LIB_CRC_AND_DISABLE_PRCHK**: CRC is performed for HSAK, but not for the application layer. CRC is disabled for drives.

    - **LIBSTORAGE_LIB_CRC_AND_ENABLE_PRCHK**: CRC is performed for HSAK, but not for the application layer. CRC is enabled for drives.

    - **LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK_NO_REF**: CRC is performed for the application layer, but not for HSAK. CRC is disabled for drives. REF tag verification is disabled for drives whose PI TYPE is 1 (Intel Optane P4800).

    - **LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK_NO_REF**: CRC is performed for the application layer, but not for HSAK. CRC is enabled for drives. REF tag verification is disabled for drives whose PI TYPE is 1 (Intel Optane P4800).

    - If PI TYPE of an Intel Optane P4800 drive is 1, the CRC and REF tag of the metadata area are verified by default.

    - Intel Optane P4800 drives support DIF in 512+8 format but does not support DIF in 4096+64 format.

    - For ES3000 V3 and ES3000 V5, PI TYPE of the drives is 3. By default, only the CRC of the metadata area is performed.

    - ES3000 V3 supports DIF in 512+8 format but does not support DIF in 4096+64 format. ES3000 V5 supports DIF in both 512+8 and 4096+64 formats.

    The summary is as follows:

    <table>
        <tr>
            <td rowspan="2"><b>E2E Verification Mode</b></td>
            <td rowspan="2"><b>Ctrl Flag</b></td>
            <td rowspan="2"><b>CRC Generator </b></td>
            <td colspan="3"><b>Write Process</b></td>
            <td colspan="3"><b>Read Process</b></td>
        </tr>
        <tr>
            <td><b>Application Verification</b></td>
            <td><b>CRC for HSAK</b></td>
            <td><b>CRC for Drives</b></td>
            <td><b>Application Verification</b></td>
            <td><b>CRC for HSAK</b></td>
            <td><b>CRC for Drives</b></td>
        </tr>
        <tr>
            <td rowspan="4">Halfway protection</td>
            <td>0</td>
            <td>Controller</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
        </tr>
        <tr>
            <td>1</td>
            <td>Controller</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>√</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Controller</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Controller</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>√</td>
        </tr>
        <tr>
            <td rowspan="4">Full protection</td>
            <td>0</td>
            <td>App</td>
            <td>√</td>
            <td>X</td>
            <td>X</td>
            <td>√</td>
            <td>X</td>
            <td>X</td>
        </tr>
        <tr>
            <td>1</td>
            <td>App</td>
            <td>√</td>
            <td>X</td>
            <td>√</td>
            <td>√</td>
            <td>X</td>
            <td>√</td>
        </tr>
        <tr>
            <td>2</td>
            <td>HSAK</td>
            <td>X</td>
            <td>√</td>
            <td>X</td>
            <td>X</td>
            <td>√</td>
            <td>X</td>
        </tr>
        <tr>
            <td>3</td>
            <td>HSAK</td>
            <td>X</td>
            <td>√</td>
            <td>√</td>
            <td>X</td>
            <td>√</td>
            <td>√</td>
        </tr>
    </table>

#### enum libstorage_print_log_level

1. Prototype

    ```c
    enum libstorage_print_log_level
    {
    LIBSTORAGE_PRINT_LOG_ERROR,
    LIBSTORAGE_PRINT_LOG_WARN,
    LIBSTORAGE_PRINT_LOG_NOTICE,
    LIBSTORAGE_PRINT_LOG_INFO,
    LIBSTORAGE_PRINT_LOG_DEBUG,
    };
    ```

2. Description

    Storage Performance Development Kit (SPDK) log print levels: ERROR, WARN, NOTICE, INFO, and DEBUG, corresponding to 0 to 4 in the configuration file.

#### MAX_BDEV_NAME_LEN

1. Prototype

    ```c
    #define MAX_BDEV_NAME_LEN 24
    ```

2. Description

    Maximum length of a block device name.

#### MAX_CTRL_NAME_LEN

1. Prototype

    ```c
    #define MAX_CTRL_NAME_LEN 16
    ```

2. Description

    Maximum length of a controller.

#### LBA_FORMAT_NUM

1. Prototype

    ```c
    #define LBA_FORMAT_NUM 16
    ```

2. Description

    Number of LBA formats supported by a controller.

#### LIBSTORAGE_MAX_DSM_RANGE_DESC_COUNT

1. Prototype

    ```c
    #define LIBSTORAGE_MAX_DSM_RANGE_DESC_COUNT 256
    ```

2. Description

    Maximum number of 16-byte sets in the dataset management command.

### ublock.h

#### UBLOCK_NVME_UEVENT_SUBSYSTEM_UIO

1. Prototype

    ```c
    #define UBLOCK_NVME_UEVENT_SUBSYSTEM_UIO 1
    ```

2. Description

    This macro is used to define that the subsystem corresponding to the uevent event is the userspace I/O subsystem (UIO) provided by the kernel. When the service receives the uevent event, this macro is used to determine whether the event is a UIO event that needs to be processed.

    The value of the int subsystem member in struct ublock_uevent is **UBLOCK_NVME_UEVENT_SUBSYSTEM_UIO**. Currently, only this value is available.

#### UBLOCK_TRADDR_MAX_LEN

1. Prototype

    ```c
    #define UBLOCK_TRADDR_MAX_LEN 256
    ```

2. Description

    The *Domain:Bus:Device.Function* (**%04x:%02x:%02x.%x**) format indicates the maximum length of the PCI address character string. The actual length is far less than 256 bytes.

#### UBLOCK_PCI_ADDR_MAX_LEN

1. Prototype

    ```c
    #define UBLOCK_PCI_ADDR_MAX_LEN 256
    ```

2. Description

    Maximum length of the PCI address character string. The actual length is far less than 256 bytes. The possible formats of the PCI address are as follows:

    - Full address: **%x:%x:%x.%x** or **%x.%x.%x.%x**

    - When the **Function** value is **0**: **%x:%x:%x**

    - When the **Domain** value is **0**: **%x:%x.%x** or **%x.%x.%x**

    - When the **Domain** and **Function** values are **0**: **%x:%x** or **%x.%x**

#### UBLOCK_SMART_INFO_LEN

1. Prototype

    ```c
    #define UBLOCK_SMART_INFO_LEN 512
    ```

2. Description

    Size of the structure for the S.M.A.R.T. information of an NVMe drive, which is 512 bytes.

#### enum ublock_rpc_server_status

1. Prototype

    ```c
    enum ublock_rpc_server_status {
    // start rpc server or not
    UBLOCK_RPC_SERVER_DISABLE = 0,
    UBLOCK_RPC_SERVER_ENABLE = 1,
    };
    ```

2. Description

    Status of the RPC service in HSAK. The status can be enabled or disabled.

#### enum ublock_nvme_uevent_action

1. Prototype

    ```c
    enum ublock_nvme_uevent_action {
    UBLOCK_NVME_UEVENT_ADD = 0,
    UBLOCK_NVME_UEVENT_REMOVE = 1,
    UBLOCK_NVME_UEVENT_INVALID,
    };
    ```

2. Description

    Indicates whether the uevent hot swap event is to insert or remove a drive.

#### enum ublock_subsystem_type

1. Prototype

    ```c
    enum ublock_subsystem_type {
    SUBSYSTEM_UIO = 0,
    SUBSYSTEM_NVME = 1,
    SUBSYSTEM_TOP
    };
    ```

2. Description

    Type of the callback function, which is used to determine whether the callback function is registered for the UIO driver or kernel NVMe driver.

## Data Structure

### bdev_rw.h

#### struct libstorage_namespace_info

1. Prototype

    ```c
    struct libstorage_namespace_info
    {
    char name[MAX_BDEV_NAME_LEN];
    uint64_t size; /** namespace size in bytes */
    uint64_t sectors; /** number of sectors */
    uint32_t sector_size; /** sector size in bytes */
    uint32_t md_size; /** metadata size in bytes */
    uint32_t max_io_xfer_size; /** maximum i/o size in bytes */
    uint16_t id; /** namespace id */
    uint8_t pi_type; /** end-to-end data protection information type */
    uint8_t is_active :1; /** namespace is active or not */
    uint8_t ext_lba :1; /** namespace support extending LBA size or not */
    uint8_t dsm :1; /** namespace supports Dataset Management or not */
    uint8_t pad :3;
    uint64_t reserved;
    };
    ```

2. Description

    This data structure contains the namespace information of a drive.

3. Struct members

    | Member                       | Description                                                  |
    | ---------------------------- | ------------------------------------------------------------ |
    | char name\[MAX_BDEV_NAME_LEN] | Name of the namespace.                                       |
    | uint64_t size                | Size of the drive space allocated to the namespace, in bytes. |
    | uint64_t sectors             | Number of sectors.                                           |
    | uint32_t sector_size         | Size of each sector, in bytes.                               |
    | uint32_t md_size             | Metadata size, in bytes.                                     |
    | uint32_t max_io_xfer_size    | Maximum size of data in a single I/O operation, in bytes.    |
    | uint16_t id                  | Namespace ID.                                                |
    | uint8_t pi_type              | Data protection type. The value is obtained from enum libstorage_ns_pi_type. |
    | uint8_t is_active :1         | Namespace active or not.                                     |
    | uint8_t ext_lba :1           | Whether the namespace supports logical block addressing (LBA) in extended mode. |
    | uint8_t dsm :1               | Whether the namespace supports dataset management.           |
    | uint8_t pad :3               | Reserved parameter.                                          |
    | uint64_t reserved            | Reserved parameter.                                          |

#### struct libstorage_nvme_ctrlr_info

1. Prototype

    ```c
    struct libstorage_nvme_ctrlr_info {
        char name[MAX_CTRL_NAME_LEN];
        char address[24];
        struct {
            uint32_t domain;
            uint8_t bus;
            uint8_t dev;
            uint8_t func;
        } pci_addr;
        uint64_t totalcap; /* Total NVM Capacity in bytes */
        uint64_t unusecap; /* Unallocated NVM Capacity in bytes */
        int8_t sn[20];     /* Serial number */
        uint8_t fr[8];     /* Firmware revision */
        uint32_t max_num_ns; /* Number of namespaces */
        uint32_t version;
        uint16_t num_io_queues; /* num of io queues */
        uint16_t io_queue_size; /* io queue size */
        uint16_t ctrlid;        /* Controller id */
        uint16_t pad1;
        struct {
            struct {
                uint32_t ms : 16;    /* metadata size */
                uint32_t lbads : 8;  /* lba data size */
                uint32_t reserved : 8;
            } lbaf[LBA_FORMAT_NUM];
            uint8_t nlbaf;
            uint8_t pad2[3];
            uint32_t cur_format : 4;
            uint32_t cur_extended : 1;
            uint32_t cur_pi : 3;
            uint32_t cur_pil : 1;
            uint32_t cur_can_share : 1;
            uint32_t mc_extented : 1;
            uint32_t mc_pointer : 1;
            uint32_t pi_type1 : 1;
            uint32_t pi_type2 : 1;
            uint32_t pi_type3 : 1;
            uint32_t md_start : 1;
            uint32_t md_end : 1;
            uint32_t ns_manage : 1;  /* Supports the Namespace Management and Namespace Attachment commands */
            uint32_t directives : 1; /* Controller support Directives or not */
            uint32_t streams : 1;    /* Controller support Streams Directives or not */
            uint32_t dsm : 1;        /* Controller support Dataset Management or not */
            uint32_t reserved : 11;
        } cap_info;
    };
    ```

2. Description

    This data structure contains the controller information of a drive.

3. Struct members

    | Member                                                       | Description                                                  |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | char name\[MAX_CTRL_NAME_LEN]                                 | Controller name.                                             |
    | char address\[24]                                             | PCI address, which is a character string.                    |
    | struct<br>{<br>uint32_t domain;<br>uint8_t bus;<br>uint8_t dev;<br>uint8_t func;<br>} pci_addr | PCI address, in segments.                                    |
    | uint64_t totalcap                                            | Total capacity of the controller, in bytes. Optane drives are based on the NVMe 1.0 protocol and do not support this parameter. |
    | uint64_t unusecap                                            | Free capacity of the controller, in bytes. Optane drives are based on the NVMe 1.0 protocol and do not support this parameter. |
    | int8_t sn\[20];                                               | Serial number of a drive, which is an ASCII character string without **0**. |
    | uint8_t fr\[8];                                               | Drive firmware version, which is an ASCII character string without **0**. |
    | uint32_t max_num_ns                                          | Maximum number of namespaces.                                |
    | uint32_t version                                             | NVMe protocol version supported by the controller.           |
    | uint16_t num_io_queues                                       | Number of I/O queues supported by a drive.                   |
    | uint16_t io_queue_size                                       | Maximum length of an I/O queue.                              |
    | uint16_t ctrlid                                              | Controller ID.                                               |
    | uint16_t pad1                                                | Reserved parameter.                                          |

    Members of the struct cap_info substructure:

    | Member                                                       | Description                                                  |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | struct<br>{<br>uint32_t ms : 16;<br>uint32_t lbads : 8;<br>uint32_t reserved : 8;<br>}lbaf\[LBA_FORMAT_NUM] | **ms**: metadata size. The minimum value is 8 bytes.<br>**lbads**: The LBA size is 2^lbads, and the value of **lbads** is greater than or equal to 9. |
    | uint8_t nlbaf                                                | Number of LBA formats supported by the controller.           |
    | uint8_t pad2\[3]                                              | Reserved parameter.                                          |
    | uint32_t cur_format : 4                                      | Current LBA format of the controller.                        |
    | uint32_t cur_extended : 1                                    | Whether the controller supports LBA in extended mode.        |
    | uint32_t cur_pi : 3                                          | Current protection type of the controller.                   |
    | uint32_t cur_pil : 1                                         | The current protection information (PI) of the controller is located in the first or last eight bytes of the metadata. |
    | uint32_t cur_can_share : 1                                   | Whether the namespace supports multi-path transmission.      |
    | uint32_t mc_extented : 1                                     | Whether metadata is transmitted as part of the data buffer.  |
    | uint32_t mc_pointer : 1                                      | Whether metadata is separated from the data buffer.          |
    | uint32_t pi_type1 : 1                                        | Whether the controller supports protection type 1.           |
    | uint32_t pi_type2 : 1                                        | Whether the controller supports protection type 2.           |
    | uint32_t pi_type3 : 1                                        | Whether the controller supports protection type 3.           |
    | uint32_t md_start : 1                                        | Whether the controller supports protection information in the first eight bytes of metadata. |
    | uint32_t md_end : 1                                          | Whether the controller supports protection information in the last eight bytes of metadata. |
    | uint32_t ns_manage : 1                                       | Whether the controller supports namespace management.        |
    | uint32_t directives : 1                                      | Whether the Directives command set is supported.             |
    | uint32_t streams : 1                                         | Whether Streams Directives is supported.                     |
    | uint32_t dsm : 1                                             | Whether Dataset Management commands are supported.           |
    | uint32_t reserved : 11                                       | Reserved parameter.                                          |

#### struct libstorage_dsm_range_desc

1. Prototype

    ```c
    struct libstorage_dsm_range_desc
    {
    /* RESERVED */
    uint32_t reserved;

    /* NUMBER OF LOGICAL BLOCKS */
    uint32_t block_count;

    /* UNMAP LOGICAL BLOCK ADDRESS */uint64_t lba;};
    ```

2. Description

    Definition of a single 16-byte set in the data management command set.

3. Struct members

    | Member               | Description              |
    | -------------------- | ------------------------ |
    | uint32_t reserved    | Reserved parameter.      |
    | uint32_t block_count | Number of LBAs per unit. |
    | uint64_t lba         | Start LBA.               |

#### struct libstorage_ctrl_streams_param

1. Prototype

    ```c
    struct libstorage_ctrl_streams_param
    {
    /* MAX Streams Limit */
    uint16_t msl;

    /* NVM Subsystem Streams Available */
    uint16_t nssa;

    /* NVM Subsystem Streams Open */uint16_t nsso;

    uint16_t pad;
    };
    ```

2. Description

    Streams attribute value supported by NVMe drives.

3. Struct members

    | Member        | Description                                                  |
    | ------------- | ------------------------------------------------------------ |
    | uint16_t msl  | Maximum number of Streams resources supported by a drive.    |
    | uint16_t nssa | Number of Streams resources that can be used by each NVM subsystem. |
    | uint16_t nsso | Number of Streams resources used by each NVM subsystem.      |
    | uint16_t pad  | Reserved parameter.                                          |

#### struct libstorage_bdev_streams_param

1. Prototype

    ```c
    struct libstorage_bdev_streams_param
    {
    /* Stream Write Size */
    uint32_t sws;

    /* Stream Granularity Size */
    uint16_t sgs;

    /* Namespace Streams Allocated */
    uint16_t nsa;

    /* Namespace Streams Open */
    uint16_t nso;

    uint16_t reserved[3];
    };
    ```

2. Description

    Streams attribute value of the namespace.

3. Struct members

    | Member               | Description                                                  |
    | -------------------- | ------------------------------------------------------------ |
    | uint32_t sws         | Write granularity with the optimal performance, in sectors.  |
    | uint16_t sgs         | Write granularity allocated to Streams, in sws.              |
    | uint16_t nsa         | Number of private Streams resources that can be used by a namespace. |
    | uint16_t nso         | Number of private Streams resources used by a namespace.     |
    | uint16_t reserved\[3] | Reserved parameter.                                          |

#### struct libstorage_mgr_info

1. Prototype

    ```c
    struct libstorage_mgr_info
    {
    char pci[24];
    char ctrlName[MAX_CTRL_NAME_LEN];
    uint64_t sector_size;
    uint64_t cap_size;
    uint16_t device_id;
    uint16_t subsystem_device_id;
    uint16_t vendor_id;
    uint16_t subsystem_vendor_id;
    uint16_t controller_id;
    int8_t serial_number[20];
    int8_t model_number[40];
    uint8_t firmware_revision[8];
    };
    ```

2. Description

    Drive management information (consistent with the drive information used by the management plane).

3. Struct members

    | Member                           | Description                                    |
    | -------------------------------- | ---------------------------------------------- |
    | char pci\[24]                     | Character string of the drive PCI address.     |
    | char ctrlName\[MAX_CTRL_NAME_LEN] | Character string of the drive controller name. |
    | uint64_t sector_size             | Drive sector size.                             |
    | uint64_t cap_size                | Drive capacity, in bytes.                      |
    | uint16_t device_id               | Drive device ID.                               |
    | uint16_t subsystem_device_id     | Drive subsystem device ID.                     |
    | uint16\*t vendor\*id              | Drive vendor ID.                               |
    | uint16_t subsystem_vendor_id     | Drive subsystem vendor ID.                     |
    | uint16_t controller_id           | Drive controller ID.                           |
    | int8_t serial_number\[20]         | Drive serial number.                           |
    | int8_t model_number\[40]          | Device model.                                  |
    | uint8_t firmware_revision\[8]     | Firmware version.                              |

#### struct **attribute**((packed)) libstorage_smart_info

1. Prototype

    ```c
    /* same with struct spdk_nvme_health_information_page in nvme_spec.h */
    struct __attribute__((packed)) libstorage_smart_info {
        /* details of uint8_t critical_warning
        *
        * union spdk_nvme_critical_warning_state {
        *     uint8_t raw;
        *     struct {
        *         uint8_t available_spare : 1;
        *         uint8_t temperature : 1;
        *         uint8_t device_reliability : 1;
        *         uint8_t read_only : 1;
        *         uint8_t volatile_memory_backup : 1;
        *         uint8_t reserved : 3;
        *     } bits;
        * };
        */
        uint8_t critical_warning;
        uint16_t temperature;
        uint8_t available_spare;
        uint8_t available_spare_threshold;
        uint8_t percentage_used;
        uint8_t reserved[26];

        /*
        * Note that the following are 128-bit values, but are
        * defined as an array of 2 64-bit values.
        */
        /* Data Units Read is always in 512-byte units. */
        uint64_t data_units_read[2];
        /* Data Units Written is always in 512-byte units. */
        uint64_t data_units_written[2];
        /* For NVM command set, this includes Compare commands. */
        uint64_t host_read_commands[2];
        uint64_t host_write_commands[2];
        /* Controller Busy Time is reported in minutes. */
        uint64_t controller_busy_time[2];
        uint64_t power_cycles[2];
        uint64_t power_on_hours[2];
        uint64_t unsafe_shutdowns[2];
        uint64_t media_errors[2];
        uint64_t num_error_info_log_entries[2];

        /* Controller temperature related. */
        uint32_t warning_temp_time;
        uint32_t critical_temp_time;
        uint16_t temp_sensor[8];
        uint8_t reserved2[296];
    };

    ```

2. Description

    This data structure defines the S.M.A.R.T. information of a drive.

3. Struct members

    | Member                                 | **Description (For details, see the NVMe protocol.)**        |
    | -------------------------------------- | ------------------------------------------------------------ |
    | uint8_t critical_warning               | Critical alarm of the controller status. If a bit is set to 1, the bit is valid. You can set multiple bits to be valid. Critical alarms are returned to the host through asynchronous events.<br>Bit 0: When this bit is set to 1, the redundant space is less than the specified threshold.<br>Bit 1: When this bit is set to 1, the temperature is higher or lower than a major threshold.<br>Bit 2: When this bit is set to 1, component reliability is reduced due to major media errors or internal errors.<br>Bit 3: When this bit is set to 1, the medium has been set to the read-only mode.<br>Bit 4: When this bit is set to 1, the volatile component of the controller fails. This parameter is valid only when the volatile component exists in the controller.<br>Bits 5-7: reserved. |
    | uint16_t temperature                   | Temperature of a component. The unit is Kelvin.              |
    | uint8_t available_spare                | Percentage of the available redundant space (0 to 100%).     |
    | uint8_t available_spare_threshold      | Threshold of the available redundant space. An asynchronous event is reported when the available redundant space is lower than the threshold. |
    | uint8_t percentage_used                | Percentage of the actual service life of a component to the service life of the component expected by the manufacturer. The value **100** indicates that the actual service life of the component has reached to the expected service life, but the component can still be used. The value can be greater than 100, but any value greater than 254 will be set to 255. |
    | uint8_t reserved\[26]                   | Reserved.                                                    |
    | uint64_t data_units_read\[2]            | Number of 512 bytes read by the host from the controller. The value **1** indicates that 1000 x 512 bytes are read, which exclude metadata. If the LBA size is not 512 bytes, the controller converts it into 512 bytes for calculation. The value is expressed in hexadecimal notation. |
    | uint64_t data_units_written\[2]         | Number of 512 bytes written by the host to the controller. The value **1** indicates that 1000 x 512 bytes are written, which exclude metadata. If the LBA size is not 512 bytes, the controller converts it into 512 bytes for calculation. The value is expressed in hexadecimal notation. |
    | uint64_t host_read_commands\[2]         | Number of read commands delivered to the controller.         |
    | uint64_t host_write_commands\[2];       | Number of write commands delivered to the controller.        |
    | uint64_t controller_busy_time\[2]       | Busy time for the controller to process I/O commands. The process from the time the commands are delivered to the time the results are returned to the CQ is busy. The time is expressed in minutes. |
    | uint64_t power_cycles\[2]               | Number of machine on/off cycles.                             |
    | uint64_t power_on_hours\[2]             | Power-on duration, in hours.                                 |
    | uint64_t unsafe_shutdowns\[2]           | Number of abnormal power-off times. The value is incremented by 1 when CC.SHN is not received during power-off. |
    | uint64_t media_errors\[2]               | Number of times that the controller detects unrecoverable data integrity errors, including uncorrectable ECC errors, CRC errors, and LBA tag mismatch. |
    | uint64_t num_error_info_log_entries\[2] | Number of entries in the error information log within the controller lifecycle. |
    | uint32_t warning_temp_time             | Accumulated time when the temperature exceeds the warning alarm threshold, in minutes. |
    | uint32_t critical_temp_time            | Accumulated time when the temperature exceeds the critical alarm threshold, in minutes. |
    | uint16_t temp_sensor\[8]                | Temperature of temperature sensors 1-8. The unit is Kelvin.  |
    | uint8_t reserved2\[296]                 | Reserved.                                                    |

#### libstorage_dpdk_contig_mem

1. Prototype

    ```c
    struct libstorage_dpdk_contig_mem {
    uint64_t virtAddr;
    uint64_t memLen;
    uint64_t allocLen;
    };
    ```

2. Description

    Description about a contiguous virtual memory segment in the parameters of the callback function that notifies the service layer of initialization completion after the DPDK memory is initialized.

    Currently, 800 MB memory is reserved for HSAK. Other memory is returned to the service layer through **allocLen** in this struct for the service layer to allocate memory for self-management.

    The total memory to be reserved for HSAK is about 800 MB. The memory reserved for each memory segment is calculated based on the number of NUMA nodes in the environment. When there are too many NUMA nodes, the memory reserved on each memory segment is too small. As a result, HSAK initialization fails. Therefore, HSAK supports only the environment with a maximum of four NUMA nodes.

3. Struct members

    | Member            | Description                                              |
    | ----------------- | -------------------------------------------------------- |
    | uint64_t virtAddr | Start address of the virtual memory.                     |
    | uint64_t memLen   | Length of the virtual memory, in bytes.                  |
    | uint64_t allocLen | Available memory length in the memory segment, in bytes. |

#### struct libstorage_dpdk_init_notify_arg

1. Prototype

    ```c
    struct libstorage_dpdk_init_notify_arg {
    uint64_t baseAddr;
    uint16_t memsegCount;
    struct libstorage_dpdk_contig_mem *memseg;
    };
    ```

2. Description

    Callback function parameter used to notify the service layer of initialization completion after DPDK memory initialization, indicating information about all virtual memory segments.

3. Struct members

    | Member                                    | Description                                                  |
    | ----------------------------------------- | ------------------------------------------------------------ |
    | uint64_t baseAddr                         | Start address of the virtual memory.                         |
    | uint16_t memsegCount                      | Number of valid **memseg** array members, that is, the number of contiguous virtual memory segments. |
    | struct libstorage_dpdk_contig_mem *memseg | Pointer to the memory segment array. Each array element is a contiguous virtual memory segment, and every two elements are discontiguous. |

#### struct libstorage_dpdk_init_notify

1. Prototype

    ```c
    struct libstorage_dpdk_init_notify {
    const char *name;
    void (*notifyFunc)(const struct libstorage_dpdk_init_notify_arg *arg);
    TAILQ_ENTRY(libstorage_dpdk_init_notify) tailq;
    };
    ```

2. Description

    Struct used to notify the service layer of the callback function registration after the DPDK memory is initialized.

3. Struct members

    | Member                                                       | Description                                                  |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | const char *name                                             | Name of the service-layer module of the registered callback function. |
    | void (*notifyFunc)(const struct libstorage_dpdk_init_notify_arg*arg) | Callback function parameter used to notify the service layer of initialization completion after the DPDK memory is initialized. |
    | TAILQ_ENTRY(libstorage_dpdk_init_notify) tailq               | Linked list that stores registered callback functions.       |

### ublock.h

#### struct ublock_bdev_info

1. Prototype

    ```c
    struct ublock_bdev_info {
    uint64_t sector_size;
    uint64_t cap_size; // cap_size
    uint16_t device_id;
    uint16_t subsystem_device_id; // subsystem device id of nvme control
    uint16_t vendor_id;
    uint16_t subsystem_vendor_id;
    uint16_t controller_id;
    int8_t serial_number[20];
    int8_t model_number[40];
    int8_t firmware_revision[8];
    };
    ```

2. Description

    This data structure contains the device information of a drive.

3. Struct members

    | Member                       | Description                                     |
    | ---------------------------- | ----------------------------------------------- |
    | uint64_t sector_size         | Sector size of a drive, for example, 512 bytes. |
    | uint64_t cap_size            | Total drive capacity, in bytes.                 |
    | uint16_t device_id           | Device ID.                                      |
    | uint16_t subsystem_device_id | Device ID of a subsystem.                       |
    | uint16_t vendor_id           | Main ID of the device vendor.                   |
    | uint16_t subsystem_vendor_id | Sub-ID of the device vendor.                    |
    | uint16_t controller_id       | ID of the device controller.                    |
    | int8_t serial_number\[20]     | Device serial number.                           |
    | int8_t model_number\[40]      | Device model.                                   |
    | int8_t firmware_revision\[8]  | Firmware version.                               |

#### struct ublock_bdev

1. Prototype

    ```c
    struct ublock_bdev {
    char pci[UBLOCK_PCI_ADDR_MAX_LEN];
    struct ublock_bdev_info info;
    struct spdk_nvme_ctrlr *ctrlr;
    TAILQ_ENTRY(ublock_bdev) link;
    };
    ```

2. Description

    The data structure contains the drive information of the specified PCI address, and the structure itself is a node of the queue.

3. Struct members

    | Member                            | Description                                                  |
    | --------------------------------- | ------------------------------------------------------------ |
    | char pci\[UBLOCK_PCI_ADDR_MAX_LEN] | PCI address.                                                 |
    | struct ublock_bdev_info info      | Drive information.                                           |
    | struct spdk_nvme_ctrlr *ctrlr     | Data structure of the device controller. The members in this structure are not open to external systems. External services can obtain the corresponding member data through the SPDK open source interface. |
    | TAILQ_ENTRY(ublock_bdev) link     | Structure of the pointers before and after a queue.          |

#### struct ublock_bdev_mgr

1. Prototype

    ```c
    struct ublock_bdev_mgr {
    TAILQ_HEAD(, ublock_bdev) bdevs;
    };
    ```

2. Description

    This data structure defines the header structure of a ublock_bdev queue.

3. Struct members

    | Member                           | Description             |
    | -------------------------------- | ----------------------- |
    | TAILQ_HEAD(, ublock_bdev) bdevs; | Queue header structure. |

#### struct **attribute**((packed)) ublock_SMART_info

1. Prototype

    ```c
    struct __attribute__((packed)) ublock_SMART_info {
    uint8_t critical_warning;
    uint16_t temperature;
    uint8_t available_spare;
    uint8_t available_spare_threshold;
    uint8_t percentage_used;
    uint8_t reserved[26];
    /*

    Note that the following are 128-bit values, but are

    defined as an array of 2 64-bit values.
    */
    /* Data Units Read is always in 512-byte units. */
    uint64_t data_units_read[2];
    /* Data Units Written is always in 512-byte units. */
    uint64_t data_units_written[2];
    /* For NVM command set, this includes Compare commands. */
    uint64_t host_read_commands[2];
    uint64_t host_write_commands[2];
    /* Controller Busy Time is reported in minutes. */
    uint64_t controller_busy_time[2];
    uint64_t power_cycles[2];
    uint64_t power_on_hours[2];
    uint64_t unsafe_shutdowns[2];
    uint64_t media_errors[2];
    uint64_t num_error_info_log_entries[2];
    /* Controller temperature related. */
    uint32_t warning_temp_time;
    uint32_t critical_temp_time;
    uint16_t temp_sensor[8];
    uint8_t reserved2[296];
    };
    ```

2. Description

    This data structure defines the S.M.A.R.T. information of a drive.

3. Struct members

    | Member                                 | Description (For details, see the NVMe protocol.)            |
    | -------------------------------------- | ------------------------------------------------------------ |
    | uint8_t critical_warning               | Critical alarm of the controller status. If a bit is set to 1, the bit is valid. You can set multiple bits to be valid. Critical alarms are returned to the host through asynchronous events.<br>Bit 0: When this bit is set to 1, the redundant space is less than the specified threshold.<br>Bit 1: When this bit is set to 1, the temperature is higher or lower than a major threshold.<br>Bit 2: When this bit is set to 1, component reliability is reduced due to major media errors or internal errors.<br>Bit 3: When this bit is set to 1, the medium has been set to the read-only mode.<br>Bit 4: When this bit is set to 1, the volatile component of the controller fails. This parameter is valid only when the volatile component exists in the controller.<br>Bits 5-7: reserved. |
    | uint16_t temperature                   | Temperature of a component. The unit is Kelvin.              |
    | uint8_t available_spare                | Percentage of the available redundant space (0 to 100%).     |
    | uint8_t available_spare_threshold      | Threshold of the available redundant space. An asynchronous event is reported when the available redundant space is lower than the threshold. |
    | uint8_t percentage_used                | Percentage of the actual service life of a component to the service life of the component expected by the manufacturer. The value **100** indicates that the actual service life of the component has reached to the expected service life, but the component can still be used. The value can be greater than 100, but any value greater than 254 will be set to 255. |
    | uint8_t reserved\[26]                   | Reserved.                                                    |
    | uint64_t data_units_read\[2]            | Number of 512 bytes read by the host from the controller. The value **1** indicates that 1000 x 512 bytes are read, which exclude metadata. If the LBA size is not 512 bytes, the controller converts it into 512 bytes for calculation. The value is expressed in hexadecimal notation. |
    | uint64_t data_units_written\[2]         | Number of 512 bytes written by the host to the controller. The value **1** indicates that 1000 x 512 bytes are written, which exclude metadata. If the LBA size is not 512 bytes, the controller converts it into 512 bytes for calculation. The value is expressed in hexadecimal notation. |
    | uint64_t host_read_commands\[2]         | Number of read commands delivered to the controller.         |
    | uint64_t host_write_commands\[2];       | Number of write commands delivered to the controller.        |
    | uint64_t controller_busy_time\[2]       | Busy time for the controller to process I/O commands. The process from the time the commands are delivered to the time the results are returned to the CQ is busy. The value is expressed in minutes. |
    | uint64_t power_cycles\[2]               | Number of machine on/off cycles.                             |
    | uint64_t power_on_hours\[2]             | Power-on duration, in hours.                                 |
    | uint64_t unsafe_shutdowns\[2]           | Number of abnormal power-off times. The value is incremented by 1 when CC.SHN is not received during power-off. |
    | uint64_t media_errors\[2]               | Number of unrecoverable data integrity errors detected by the controller, including uncorrectable ECC errors, CRC errors, and LBA tag mismatch. |
    | uint64_t num_error_info_log_entries\[2] | Number of entries in the error information log within the controller lifecycle. |
    | uint32_t warning_temp_time             | Accumulated time when the temperature exceeds the warning alarm threshold, in minutes. |
    | uint32_t critical_temp_time            | Accumulated time when the temperature exceeds the critical alarm threshold, in minutes. |
    | uint16_t temp_sensor\[8]                | Temperature of temperature sensors 1-8. The unit is Kelvin.  |
    | uint8_t reserved2\[296]                 | Reserved.                                                    |

#### struct ublock_nvme_error_info

1. Prototype

    ```c
    struct ublock_nvme_error_info {
    uint64_t error_count;
    uint16_t sqid;
    uint16_t cid;
    uint16_t status;
    uint16_t error_location;
    uint64_t lba;
    uint32_t nsid;
    uint8_t vendor_specific;
    uint8_t reserved[35];
    };
    ```

2. Description

    This data structure contains the content of a single error message in the device controller. The number of errors supported by different controllers may vary.

3. Struct members

    | Member                  | Description (For details, see the NVMe protocol.)            |
    | ----------------------- | ------------------------------------------------------------ |
    | uint64_t error_count    | Error sequence number, which increases in ascending order.   |
    | uint16_t sqid           | Submission queue identifier for the command associated with an error message. If an error cannot be associated with a specific command, this parameter should be set to **FFFFh**. |
    | uint16_t cid            | Command identifier associated with an error message. If an error cannot be associated with a specific command, this parameter should be set to **FFFFh**. |
    | uint16_t status         | Status of a completed command.                               |
    | uint16_t error_location | Command parameter associated with an error message.          |
    | uint64_t lba            | First LBA when an error occurs.                              |
    | uint32_t nsid           | Namespace where an error occurs.                             |
    | uint8_t vendor_specific | Log page identifier associated with the page if other vendor-specific error messages are available. The value **00h** indicates that no additional information is available. The valid value ranges from 80h to FFh. |
    | uint8_t reserved\[35]    | Reserved.                                                    |

#### struct ublock_uevent

1. Prototype

    ```c
    struct ublock_uevent {
    enum ublock_nvme_uevent_action action;
    int subsystem;
    char traddr[UBLOCK_TRADDR_MAX_LEN + 1];
    };
    ```

2. Description

    This data structure contains parameters related to the uevent event.

3. Struct members

    | Member                                 | Description                                                  |
    | -------------------------------------- | ------------------------------------------------------------ |
    | enum ublock_nvme_uevent_action action  | Whether the uevent event type is drive insertion or removal through enumeration. |
    | int subsystem                          | Subsystem type of the uevent event. Currently, only **UBLOCK_NVME_UEVENT_SUBSYSTEM_UIO** is supported. If the application receives other values, no processing is required. |
    | char traddr\[UBLOCK_TRADDR_MAX_LEN + 1] | PCI address character string in the *Domain:Bus:Device.Function* (**%04x:%02x:%02x.%x**) format. |

#### struct ublock_hook

1. Prototype

    ```c
    struct ublock_hook
    {
    ublock_callback_func ublock_callback;
    void *user_data;
    };
    ```

2. Description

    This data structure is used to register callback functions.

3. Struct members

    | Member                               | Description                                                  |
    | ------------------------------------ | ------------------------------------------------------------ |
    | ublock_callback_func ublock_callback | Function executed during callback. The type is bool func(void *info, void*user_data). |
    | void *user_data                      | User parameter transferred to the callback function.         |

#### struct ublock_ctrl_iostat_info

1. Prototype

    ```c
    struct ublock_ctrl_iostat_info
    {
    uint64_t num_read_ops;
    uint64_t num_write_ops;
    uint64_t read_latency_ms;
    uint64_t write_latency_ms;
    uint64_t io_outstanding;
    uint64_t num_poll_timeout;
    uint64_t io_ticks_ms;
    };
    ```

2. Description

    This data structure is used to obtain the I/O statistics of a controller.

3. Struct members

    | Member                    | Description                                                  |
    | ------------------------- | ------------------------------------------------------------ |
    | uint64_t num_read_ops     | Accumulated number of read I/Os of the controller.           |
    | uint64_t num_write_ops    | Accumulated number of write I/Os of the controller.          |
    | uint64_t read_latency_ms  | Accumulated read latency of the controller, in ms.           |
    | uint64_t write_latency_ms | Accumulated write latency of the controller, in ms.          |
    | uint64_t io_outstanding   | Queue depth of the controller.                               |
    | uint64_t num_poll_timeout | Accumulated number of polling timeouts of the controller.    |
    | uint64_t io_ticks_ms      | Accumulated I/O processing latency of the controller, in ms. |

## API

### bdev_rw.h

#### libstorage_get_nvme_ctrlr_info

1. Prototype

    uint32_t libstorage_get_nvme_ctrlr_info(struct libstorage_nvme_ctrlr_info** ppCtrlrInfo);

2. Description

    Obtains information about all controllers.

3. Parameters

    | Parameter                                       | Description                                                  |
    | ----------------------------------------------- | ------------------------------------------------------------ |
    | struct libstorage_nvme_ctrlr_info** ppCtrlrInfo | Output parameter, which returns all obtained controller information.<br>Note:<br>Free the memory using the free API in a timely manner. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | Failed to obtain controller information or no controller information is obtained. |
    | > 0          | Number of obtained controllers.                              |

#### libstorage_get_mgr_info_by_esn

1. Prototype

    ```c
    int32_t libstorage_get_mgr_info_by_esn(const char *esn, struct libstorage_mgr_info *mgr_info);
    ```

2. Description

    Obtains the management information about the NVMe drive corresponding to the ESN.

3. Parameters

    | Parameter                            | Description                                                  |
    | ------------------------------------ | ------------------------------------------------------------ |
    | const char *esn                      | ESN of the target device.<br>Note:<br>An ESN is a string of a maximum of 20 characters (excluding the end character of the string), but the length may vary according to hardware vendors. For example, if the length is less than 20 characters, spaces are padded at the end of the character string.<br> |
    | struct libstorage_mgr_info *mgr_info | Output parameter, which returns all obtained NVMe drive management information. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | Succeeded in querying the NVMe drive management information corresponding to an ESN. |
    | -1           | Failed to query the NVMe drive management information corresponding to an ESN. |
    | -2           | No NVMe drive matching an ESN is obtained.                   |

#### libstorage_get_mgr_smart_by_esn

1. Prototype

    ```c
    int32_t libstorage_get_mgr_smart_by_esn(const char *esn, uint32_t nsid, struct libstorage_smart_info *mgr_smart_info);
    ```

2. Description

    Obtains the S.M.A.R.T. information of the NVMe drive corresponding to an ESN.

3. Parameters

    | Parameter                            | Description                                                  |
    | ------------------------------------ | ------------------------------------------------------------ |
    | const char *esn                      | ESN of the target device.<br>Note:<br>An ESN is a string of a maximum of 20 characters (excluding the end character of the string), but the length may vary according to hardware vendors. For example, if the length is less than 20 characters, spaces are padded at the end of the character string.<br> |
    | uint32_t nsid                        | Specified namespace.                                         |
    | struct libstorage_mgr_info *mgr_info | Output parameter, which returns all obtained S.M.A.R.T. information of NVMe drives. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | Succeeded in querying the S.M.A.R.T. information of the NVMe drive corresponding to an ESN. |
    | -1           | Failed to query the S.M.A.R.T. information of the NVMe drive corresponding to an ESN. |
    | -2           | No NVMe drive matching an ESN is obtained.                   |

#### libstorage_get_bdev_ns_info

1. Prototype

    ```c
    uint32_t libstorage_get_bdev_ns_info(const char* bdevName, struct libstorage_namespace_info** ppNsInfo);
    ```

2. Description

    Obtains namespace information based on the device name.

3. Parameters

    | Parameter                                   | Description                                                  |
    | ------------------------------------------- | ------------------------------------------------------------ |
    | const char* bdevName                        | Device name.                                                 |
    | struct libstorage_namespace_info** ppNsInfo | Output parameter, which returns namespace information.<br>Note:<br>Free the memory using the free API in a timely manner. |

4. Return value

    | Return Value | Description                  |
    | ------------ | ---------------------------- |
    | 0            | The operation failed.        |
    | 1            | The operation is successful. |

#### libstorage_get_ctrl_ns_info

1. Prototype

    ```c
    uint32_t libstorage_get_ctrl_ns_info(const char* ctrlName, struct libstorage_namespace_info** ppNsInfo);
    ```

2. Description

    Obtains information about all namespaces based on the controller name.

3. Parameters

    | Parameter                                   | Description                                                  |
    | ------------------------------------------- | ------------------------------------------------------------ |
    | const char* ctrlName                        | Controller name.                                             |
    | struct libstorage_namespace_info** ppNsInfo | Output parameter, which returns information about all namespaces.<br>Note:<br>Free the memory using the free API in a timely manner. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | Failed to obtain the namespace information or no namespace information is obtained. |
    | > 0          | Number of namespaces obtained.                               |

#### libstorage_create_namespace

1. Prototype

    ```c
    int32_t libstorage_create_namespace(const char* ctrlName, uint64_t ns_size, char** outputName);
    ```

2. Description

    Creates a namespace on a specified controller (the prerequisite is that the controller supports namespace management).

    Optane drives are based on the NVMe 1.0 protocol and do not support namespace management. Therefore, this API is not supported.

    ES3000 V3 and V5 support only one namespace by default. By default, a namespace exists on the controller. To create a namespace, delete the original namespace.

3. Parameters

    | Parameter            | Description                                                  |
    | -------------------- | ------------------------------------------------------------ |
    | const char* ctrlName | Controller name.                                             |
    | uint64_t ns_size     | Size of the namespace to be created (unit: sector_size).     |
    | char** outputName    | Output parameter, which indicates the name of the created namespace.<br>Note:<br>Free the memory using the free API in a timely manner. |

4. Return value

    | Return Value | Description                                    |
    | ------------ | ---------------------------------------------- |
    | ≤ 0          | Failed to create the namespace.                |
    | > 0          | ID of the created namespace (starting from 1). |

#### libstorage_delete_namespace

1. Prototype

    ```c
    int32_t libstorage_delete_namespace(const char* ctrlName, uint32_t ns_id);
    ```

2. Description

    Deletes a namespace from a specified controller. Optane drives are based on the NVMe 1.0 protocol and do not support namespace management. Therefore, this API is not supported.

3. Parameters

    | Parameter            | Description      |
    | -------------------- | ---------------- |
    | const char* ctrlName | Controller name. |
    | uint32_t ns_id       | Namespace ID     |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | Deletion succeeded.                                          |
    | Other values | Deletion failed.<br>Note:<br>Before deleting a namespace, stop I/O operations. Otherwise, the namespace fails to be deleted. |

#### libstorage_delete_all_namespace

1. Prototype

    ```c
    int32_t libstorage_delete_all_namespace(const char* ctrlName);
    ```

2. Description

    Deletes all namespaces from a specified controller. Optane drives are based on the NVMe 1.0 protocol and do not support namespace management. Therefore, this API is not supported.

3. Parameters

    | Parameter            | Description      |
    | -------------------- | ---------------- |
    | const char* ctrlName | Controller name. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | Deletion succeeded.                                          |
    | Other values | Deletion failed.<br>Note:<br>Before deleting a namespace, stop I/O operations. Otherwise, the namespace fails to be deleted. |

#### libstorage_nvme_create_ctrlr

1. Prototype

    ```c
    int32_t libstorage_nvme_create_ctrlr(const char *pci_addr, const char *ctrlr_name);
    ```

2. Description

    Creates an NVMe controller based on the PCI address.

3. Parameters

    | Parameter        | Description      |
    | ---------------- | ---------------- |
    | char *pci_addr   | PCI address.     |
    | char *ctrlr_name | Controller name. |

4. Return value

    | Return Value | Description         |
    | ------------ | ------------------- |
    | < 0          | Creation failed.    |
    | 0            | Creation succeeded. |

#### libstorage_nvme_delete_ctrlr

1. Prototype

    ```c
    int32_t libstorage_nvme_delete_ctrlr(const char *ctrlr_name);
    ```

2. Description

    Destroys an NVMe controller based on the controller name.

3. Parameters

    | Parameter              | Description      |
    | ---------------------- | ---------------- |
    | const char *ctrlr_name | Controller name. |

    This API can be called only after all delivered I/Os are returned.

4. Return value

    | Return Value | Description            |
    | ------------ | ---------------------- |
    | < 0          | Destruction failed.    |
    | 0            | Destruction succeeded. |

#### libstorage_nvme_reload_ctrlr

1. Prototype

    ```c
    int32_t libstorage_nvme_reload_ctrlr(const char *cfgfile);
    ```

2. Description

    Adds or deletes an NVMe controller based on the configuration file.

3. Parameters

    | Parameter           | Description                     |
    | ------------------- | ------------------------------- |
    | const char *cfgfile | Path of the configuration file. |

    Before using this API to delete a drive, ensure that all delivered I/Os have been returned.

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | < 0          | Failed to add or delete drives based on the configuration file. (Drives may be successfully added or deleted for some controllers.) |
    | 0            | Drives are successfully added or deleted based on the configuration file. |

    > Constraints

    - Currently, a maximum of 36 controllers can be configured in the configuration file.

    - The reload API creates as many controllers as possible. If a controller fails to be created, the creation of other controllers is not affected.

    - In concurrency scenarios, the final drive initialization status may be inconsistent with the input configuration file.

    - If you delete a drive that is delivering I/Os by reloading the drive, I/Os fail.

    - After the controller name (for example, **nvme0**) corresponding to the PCI address in the configuration file is modified, the modification does not take effect after this interface is called.

    - The reload function is valid only when drives are added or deleted. Other configuration items in the configuration file cannot be reloaded.

#### libstorage_low_level_format_nvm

1. Prototype

    ```c
    int8_t libstorage_low_level_format_nvm(const char* ctrlName, uint8_t lbaf,
    enum libstorage_ns_pi_type piType,
    bool pil_start, bool ms_extented, uint8_t ses);
    ```

2. Description

    Low-level formats NVMe drives.

3. Parameters

    | Parameter                         | Description                                                  |
    | --------------------------------- | ------------------------------------------------------------ |
    | const char* ctrlName              | Controller name.                                             |
    | uint8_t lbaf                      | LBA format to be used.                                       |
    | enum libstorage_ns_pi_type piType | Protection type to be used.                                  |
    | bool pil_start                    | The protection information is stored in first eight bytes (1) or last eight bytes (0) of the metadata. |
    | bool ms_extented                  | Whether to format to the extended type.                      |
    | uint8_t ses                       | Whether to perform secure erase during formatting. Currently, only the value **0** (no-secure erase) is supported. |

4. Return value

    | Return Value | Description                                       |
    | ------------ | ------------------------------------------------- |
    | < 0          | Formatting failed.                                |
    | ≥ 0          | LBA format generated after successful formatting. |

    > Constraints

    - This low-level formatting API will clear the data and metadata of the drive namespace. Exercise caution when using this API.

    - It takes several seconds to format an ES3000 drive and several minutes to format an Intel Optane drive. Before using this API, wait until the formatting is complete. If the formatting process is forcibly stopped, the formatting fails.

    - Before formatting, stop the I/O operations on the data plane. If the drive is processing I/O requests, the formatting may fail occasionally. If the formatting is successful, the drive may discard the I/O requests that are being processed. Therefore, before formatting the drive, ensure that the I/O operations on the data plane are stopped.

    - During the formatting, the controller is reset. As a result, the initialized drive resources are unavailable. Therefore, after the formatting is complete, restart the I/O process on the data plane.

    - ES3000 V3 supports protection types 0 and 3, PI start and PI end, and mc extended. ES3000 V3 supports DIF in 512+8 format but does not support DIF in 4096+64 format.

    - ES3000 V5 supports protection types 0 and 3, PI start and PI end, mc extended, and mc pointer. ES3000 V5 supports DIF in both 512+8 and 4096+64 formats.

    - Optane drives support protection types 0 and 1, PI end, and mc extended. Optane drives support DIF in 512+8 format but does not support DIF in 4096+64 format.

    | **Drive Type**     | **LBA Format**                                               | **Drive Type** | **LBA Format**                                               |
    | ------------------ | ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
    | Intel Optane P4800 | lbaf0:512+0<br>lbaf1:512+8<br>lbaf2:512+16<br>lbaf3:4096+0<br>lbaf4:4096+8<br>lbaf5:4096+64<br>lbaf6:4096+128 | ES3000 V3, V5  | lbaf0:512+0<br>lbaf1:512+8<br>lbaf2:4096+64<br>lbaf3:4096+0<br>lbaf4:4096+8 |

#### LIBSTORAGE_CALLBACK_FUNC

1. Prototype

    ```c
    typedef void (*LIBSTORAGE_CALLBACK_FUNC)(int32_t cb_status, int32_t sct_code, void* cb_arg);
    ```

2. Description

    Registered HSAK I/O completion callback function.

3. Parameters

    | Parameter         | Description                                                  |
    | ----------------- | ------------------------------------------------------------ |
    | int32_t cb_status | I/O status code. The value **0** indicates success, a negative value indicates system error code, and a positive value indicates drive error code (for different error codes,<br>see [Appendixes](#Appendixes)). |
    | int32_t sct_code  | I/O status code type: <br/>0: [GENERIC](#generic)<br/>1: [COMMAND_SPECIFIC](#command_specific)<br>2: [MEDIA_DATA_INTERGRITY_ERROR](#media_data_intergrity_error)<br>7: VENDOR_SPECIFIC |
    | void* cb_arg      | Input parameter of the callback function.                    |

4. Return value

    None.

#### libstorage_deallocate_block

1. Prototype

    ```c
    int32_t libstorage_deallocate_block(int32_t fd, struct libstorage_dsm_range_desc *range, uint16_t range_count, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. Description

    Notifies NVMe drives of the blocks that can be released.

3. Parameters

    | Parameter                               | Description                                                  |
    | --------------------------------------- | ------------------------------------------------------------ |
    | int32_t fd                              | Open drive file descriptor.                                  |
    | struct libstorage_dsm_range_desc *range | Description of blocks that can be released on NVMe drives.<br>Note:<br>This parameter requires **libstorage_mem_reserve** to allocate huge page memory. 4 KB alignment is required during memory allocation, that is, align is set to 4096.<br>The TRIM range of drives is restricted based on different drives. Exceeding the maximum TRIM range on the drives may cause data exceptions. |
    | uint16_t range_count                    | Number of members in the array range.                        |
    | LIBSTORAGE_CALLBACK_FUNC cb             | Callback function.                                           |
    | void* cb_arg                            | Callback function parameter.                                 |

4. Return value

    | Return Value | Description                     |
    | ------------ | ------------------------------- |
    | < 0          | Failed to deliver the request.  |
    | 0            | Request submitted successfully. |

#### libstorage_async_write

1. Prototype

    ```c
    int32_t libstorage_async_write(int32_t fd, void *buf, size_t nbytes, off64_t offset, void *md_buf, size_t md_len, enum libstorage_crc_and_prchk dif_flag, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. Description

    Delivers asynchronous I/O write requests (the write buffer is a contiguous buffer).

3. Parameters

    | Parameter                              | Description                                                  |
    | -------------------------------------- | ------------------------------------------------------------ |
    | int32_t fd                             | File descriptor of the block device.                         |
    | void *buf                              | Buffer for I/O write data (four-byte aligned and cannot cross the 4 KB page boundary).<br>Note:<br>The LBA in extended mode must contain the metadata memory size. |
    | size_t nbytes                          | Size of a single write I/O, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | off64_t offset                         | Write offset of the LBA, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | void *md_buf                           | Metadata buffer. (Applicable only to LBAs in separated mode. Set this parameter to **NULL** for LBAs in extended mode.) |
    | size_t md_len                          | Buffer length of metadata. (Applicable only to LBAs in separated mode. Set this parameter to **0** for LBAs in extended mode.) |
    | enum libstorage_crc_and_prchk dif_flag | Whether to calculate DIF and whether to enable drive verification. |
    | LIBSTORAGE_CALLBACK_FUNC cb            | Registered callback function.                                |
    | void* cb_arg                           | Parameters of the callback function.                         |

4. Return value

    | Return Value | Description                                    |
    | ------------ | ---------------------------------------------- |
    | 0            | I/O write requests are submitted successfully. |
    | Other values | Failed to submit I/O write requests.           |

#### libstorage_async_read

1. Prototype

    ```c
    int32_t libstorage_async_read(int32_t fd, void *buf, size_t nbytes, off64_t offset, void *md_buf, size_t md_len, enum libstorage_crc_and_prchk dif_flag, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. Description

    Delivers asynchronous I/O read requests (the read buffer is a contiguous buffer).

3. Parameters

    | Parameter                              | Description                                                  |
    | -------------------------------------- | ------------------------------------------------------------ |
    | int32_t fd                             | File descriptor of the block device.                         |
    | void *buf                              | Buffer for I/O read data (four-byte aligned and cannot cross the 4 KB page boundary).<br>Note:<br>LBAs in extended mode must contain the metadata memory size. |
    | size_t nbytes                          | Size of a single read I/O, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | off64_t offset                         | Read offset of the LBA, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. The LBA in extended mode does not include the metadata size. |
    | void *md_buf                           | Metadata buffer. (Applicable only to LBAs in separated mode. Set this parameter to **NULL** for LBAs in extended mode.). |
    | size_t md_len                          | Buffer length of metadata. (Applicable only to LBAs in separated mode. Set this parameter to **0** for LBAs in extended mode.). |
    | enum libstorage_crc_and_prchk dif_flag | Whether to calculate DIF and whether to enable drive verification. |
    | LIBSTORAGE_CALLBACK_FUNC cb            | Registered callback function.                                |
    | void* cb_arg                           | Parameters of the callback function.                         |

4. Return value

    | Return Value | Description                                   |
    | ------------ | --------------------------------------------- |
    | 0            | I/O read requests are submitted successfully. |
    | Other values | Failed to submit I/O read requests.           |

#### libstorage_async_writev

1. Prototype

    ```c
    int32_t libstorage_async_writev(int32_t fd, struct iovec *iov, int iovcnt, size_t nbytes, off64_t offset, void *md_buf, size_t md_len, enum libstorage_crc_and_prchk dif_flag, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. Description

    Delivers asynchronous I/O write requests (the write buffer is a discrete buffer).

3. Parameters

    | Parameter                              | Description                                                  |
    | -------------------------------------- | ------------------------------------------------------------ |
    | int32_t fd                             | File descriptor of the block device.                         |
    | struct iovec *iov                      | Buffer for I/O write data.<br>Note:<br>LBAs in extended mode must contain the metadata size.<br>The address must be 4-byte-aligned and the length cannot exceed 4 GB. |
    | int iovcnt                             | Number of buffers for I/O write data.                        |
    | size_t nbytes                          | Size of a single write I/O, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | off64_t offset                         | Write offset of the LBA, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | void *md_buf                           | Metadata buffer. (Applicable only to LBAs in separated mode. Set this parameter to **NULL** for LBAs in extended mode.) |
    | size_t md_len                          | Length of the metadata buffer. (Applicable only to LBAs in separated mode. Set this parameter to **0** for LBAs in extended mode.) |
    | enum libstorage_crc_and_prchk dif_flag | Whether to calculate DIF and whether to enable drive verification. |
    | LIBSTORAGE_CALLBACK_FUNC cb            | Registered callback function.                                |
    | void* cb_arg                           | Parameters of the callback function.                         |

4. Return value

    | Return Value | Description                                    |
    | ------------ | ---------------------------------------------- |
    | 0            | I/O write requests are submitted successfully. |
    | Other values | Failed to submit I/O write requests.           |

#### libstorage_async_readv

1. Prototype

    ```c
    int32_t libstorage_async_readv(int32_t fd, struct iovec *iov, int iovcnt, size_t nbytes, off64_t offset, void *md_buf, size_t md_len, enum libstorage_crc_and_prchk dif_flag, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. Description

    Delivers asynchronous I/O read requests (the read buffer is a discrete buffer).

3. Parameters

    | Parameter                              | Description                                                  |
    | -------------------------------------- | ------------------------------------------------------------ |
    | int32_t fd                             | File descriptor of the block device.                         |
    | struct iovec *iov                      | Buffer for I/O read data.<br>Note:<br>LBAs in extended mode must contain the metadata size.<br>The address must be 4-byte-aligned and the length cannot exceed 4 GB. |
    | int iovcnt                             | Number of buffers for I/O read data.                         |
    | size_t nbytes                          | Size of a single read I/O, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | off64_t offset                         | Read offset of the LBA, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | void *md_buf                           | Metadata buffer. (Applicable only to LBAs in separated mode. Set this parameter to **NULL** for LBAs in extended mode.) |
    | size_t md_len                          | Length of the metadata buffer. (Applicable only to LBAs in separated mode. Set this parameter to **0** for LBAs in extended mode.) |
    | enum libstorage_crc_and_prchk dif_flag | Whether to calculate DIF and whether to enable drive verification. |
    | LIBSTORAGE_CALLBACK_FUNC cb            | Registered callback function.                                |
    | void* cb_arg                           | Parameters of the callback function.                         |

4. Return value

    | Return Value | Description                                   |
    | ------------ | --------------------------------------------- |
    | 0            | I/O read requests are submitted successfully. |
    | Other values | Failed to submit I/O read requests.           |

#### libstorage_sync_write

1. Prototype

    ```c
    int32_t libstorage_sync_write(int fd, const void *buf, size_t nbytes, off_t offset);
    ```

2. Description

    Delivers synchronous I/O write requests (the write buffer is a contiguous buffer).

3. Parameters

    | Parameter      | Description                                                  |
    | -------------- | ------------------------------------------------------------ |
    | int32_t fd     | File descriptor of the block device.                         |
    | void *buf      | Buffer for I/O write data (four-byte aligned and cannot cross the 4 KB page boundary).<br>Note:<br>LBAs in extended mode must contain the metadata memory size. |
    | size_t nbytes  | Size of a single write I/O, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | off64_t offset | Write offset of the LBA, in bytes. (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |

4. Return value

    | Return Value | Description                                    |
    | ------------ | ---------------------------------------------- |
    | 0            | I/O write requests are submitted successfully. |
    | Other values | Failed to submit I/O write requests.           |

#### libstorage_sync_read

1. Prototype

    ```c
    int32_t libstorage_sync_read(int fd, const void *buf, size_t nbytes, off_t offset);
    ```

2. Description

    Delivers synchronous I/O read requests (the read buffer is a contiguous buffer).

3. Parameters

    | Parameter      | Description                                                  |
    | -------------- | ------------------------------------------------------------ |
    | int32_t fd     | File descriptor of the block device.                         |
    | void *buf      | Buffer for I/O read data (four-byte aligned and cannot cross the 4 KB page boundary).<br>Note:<br>LBAs in extended mode must contain the metadata memory size. |
    | size_t nbytes  | Size of a single read I/O, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |
    | off64_t offset | Read offset of the LBA, in bytes (an integer multiple of **sector_size**).<br>Note:<br>Only the data size is included. LBAs in extended mode do not include the metadata size. |

4. Return value

    | Return Value | Description                                   |
    | ------------ | --------------------------------------------- |
    | 0            | I/O read requests are submitted successfully. |
    | Other values | Failed to submit I/O read requests.           |

#### libstorage_open

1. Prototype

    ```c
    int32_t libstorage_open(const char* devfullname);
    ```

2. Description

    Opens a block device.

3. Parameters

    | Parameter               | Description                              |
    | ----------------------- | ---------------------------------------- |
    | const char* devfullname | Block device name (format: **nvme0n1**). |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | -1           | Opening failed. For example, the device name is incorrect, or the number of opened FDs is greater than the number of available channels of the NVMe drive. |
    | > 0          | File descriptor of the block device.                         |

    After the MultiQ function in **nvme.conf.in** is enabled, different FDs are returned if a thread opens the same device for multiple times. Otherwise, the same FD is returned. This attribute applies only to the NVMe device.

#### libstorage_close

1. Prototype

    ```c
    int32_t libstorage_close(int32_t fd);
    ```

2. Description

    Closes a block device.

3. Parameters

    | Parameter  | Description                                |
    | ---------- | ------------------------------------------ |
    | int32_t fd | File descriptor of an opened block device. |

4. Return value

    | Return Value | Description                                     |
    | ------------ | ----------------------------------------------- |
    | -1           | Invalid file descriptor.                        |
    | -16          | The file descriptor is busy. Retry is required. |
    | 0            | Close succeeded.                                |

#### libstorage_mem_reserve

1. Prototype

    ```c
    void* libstorage_mem_reserve(size_t size, size_t align);
    ```

2. Description

    Allocates memory space from the huge page memory reserved by the DPDK.

3. Parameters

    | Parameter    | Description                         |
    | ------------ | ----------------------------------- |
    | size_t size  | Size of the memory to be allocated. |
    | size_t align | Aligns allocated memory space.      |

4. Return value

    | Return Value | Description                            |
    | ------------ | -------------------------------------- |
    | NULL         | Allocation failed.                     |
    | Other values | Address of the allocated memory space. |

#### libstorage_mem_free

1. Prototype

    ```c
    void libstorage_mem_free(void* ptr);
    ```

2. Description

    Frees the memory space pointed to by **ptr**.

3. Parameters

    | Parameter | Description                              |
    | --------- | ---------------------------------------- |
    | void* ptr | Address of the memory space to be freed. |

4. Return value

    None.

#### libstorage_alloc_io_buf

1. Prototype

    ```c
    void* libstorage_alloc_io_buf(size_t nbytes);
    ```

2. Description

    Allocates memory from buf_small_pool or buf_large_pool of the SPDK.

3. Parameters

    | Parameter     | Description                         |
    | ------------- | ----------------------------------- |
    | size_t nbytes | Size of the buffer to be allocated. |

4. Return value

    | Return Value | Description                            |
    | ------------ | -------------------------------------- |
    | Other values | Start address of the allocated buffer. |

#### libstorage_free_io_buf

1. Prototype

    ```c
    int32_t libstorage_free_io_buf(void *buf, size_t nbytes);
    ```

2. Description

    Frees the allocated memory to buf_small_pool or buf_large_pool of the SPDK.

3. Parameters

    | Parameter     | Description                              |
    | ------------- | ---------------------------------------- |
    | void *buf     | Start address of the buffer to be freed. |
    | size_t nbytes | Size of the buffer to be freed.          |

4. Return value

    | Return Value | Description        |
    | ------------ | ------------------ |
    | -1           | Freeing failed.    |
    | 0            | Freeing succeeded. |

#### libstorage_init_module

1. Prototype

    ```c
    int32_t libstorage_init_module(const char* cfgfile);
    ```

2. Description

    Initializes the HSAK module.

3. Parameters

    | Parameter           | Description                          |
    | ------------------- | ------------------------------------ |
    | const char* cfgfile | Name of the HSAK configuration file. |

4. Return value

    | Return Value | Description               |
    | ------------ | ------------------------- |
    | Other values | Initialization failed.    |
    | 0            | Initialization succeeded. |

#### libstorage_exit_module

1. Prototype

    ```c
    int32_t libstorage_exit_module(void);
    ```

2. Description

    Exits the HSAK module.

3. Parameters

    None.

4. Return value

    | Return Value | Description                       |
    | ------------ | --------------------------------- |
    | Other values | Failed to exit the cleanup.       |
    | 0            | Succeeded in exiting the cleanup. |

#### LIBSTORAGE_REGISTER_DPDK_INIT_NOTIFY

1. Prototype

    ```c
    LIBSTORAGE_REGISTER_DPDK_INIT_NOTIFY(_name, _notify)
    ```

2. Description

    Service layer registration function, which is used to register the callback function when the DPDK initialization is complete.

3. Parameters

    | Parameter | Description                                                  |
    | --------- | ------------------------------------------------------------ |
    | _name     | Name of a module at the service layer.                       |
    | _notify   | Prototype of the callback function registered at the service layer: **void (*notifyFunc)(const struct libstorage_dpdk_init_notify_arg *arg);** |

4. Return value

    None

### ublock.h

#### init_ublock

1. Prototype

    ```c
    int init_ublock(const char *name, enum ublock_rpc_server_status flg);
    ```

2. Description

    Initializes the Ublock module. This API must be called before other Ublock APIs. If the flag is set to **UBLOCK_RPC_SERVER_ENABLE**, that is, Ublock functions as the RPC server, the same process can be initialized only once.

    When Ublock is started as the RPC server, the monitor thread of a server is started at the same time. When the monitor thread detects that the RPC server thread is abnormal (for example, thread suspended), the monitor thread calls the exit function to trigger the process to exit.

    In this case, the product script is used to start the process again.

3. Parameters

    | Parameter                            | Description                                                  |
    | ------------------------------------ | ------------------------------------------------------------ |
    | const char *name                     | Module name. The default value is **ublock**. You are advised to set this parameter to **NULL**. |
    | enum ublock_rpc_server_status<br>flg | Whether to enable RPC. The value can be **UBLOCK_RPC_SERVER_DISABLE** or **UBLOCK_RPC_SERVER_ENABLE**.<br> If RPC is disabled and the drive is occupied by service processes, the Ublock module cannot obtain the drive information. |

4. Return value

    | Return Value  | Description                                                  |
    | ------------- | ------------------------------------------------------------ |
    | 0             | Initialization succeeded.                                    |
    | -1            | Initialization failed. Possible cause: The Ublock module has been initialized. |
    | Process exits | Ublock considers that the following exceptions cannot be rectified and directly calls the exit API to exit the process:<br>- The RPC service needs to be created, but it fails to be created onsite.<br>- Failed to create a hot swap monitoring thread. |

#### ublock_init

1. Prototype

    ```c
    #define ublock_init(name) init_ublock(name, UBLOCK_RPC_SERVER_ENABLE)
    ```

2. Description

    It is the macro definition of the init_ublock API. It can be regarded as initializing Ublock into the required RPC service.

3. Parameters

    | Parameter | Description                                                  |
    | --------- | ------------------------------------------------------------ |
    | name      | Module name. The default value is **ublock**. You are advised to set this parameter to **NULL**. |

4. Return value

    | Return Value  | Description                                                  |
    | ------------- | ------------------------------------------------------------ |
    | 0             | Initialization succeeded.                                    |
    | -1            | Initialization failed. Possible cause: The Ublock RPC server module has been initialized. |
    | Process exits | Ublock considers that the following exceptions cannot be rectified and directly calls the exit API to exit the process:<br>- The RPC service needs to be created, but it fails to be created onsite.<br>- Failed to create a hot swap monitoring thread. |

#### ublock_init_norpc

1. Prototype

    ```c
    #define ublock_init_norpc(name) init_ublock(name, UBLOCK_RPC_SERVER_DISABLE)
    ```

2. Description

    It is the macro definition of the init_ublock API and can be considered as initializing Ublock into a non-RPC service.

3. Parameters

    | Parameter | Description                                                  |
    | --------- | ------------------------------------------------------------ |
    | name      | Module name. The default value is **ublock**. You are advised to set this parameter to **NULL**. |

4. Return value

    | Return Value  | Description                                                  |
    | ------------- | ------------------------------------------------------------ |
    | 0             | Initialization succeeded.                                    |
    | -1            | Initialization failed. Possible cause: The Ublock client module has been initialized. |
    | Process exits | Ublock considers that the following exceptions cannot be rectified and directly calls the exit API to exit the process:<br>- The RPC service needs to be created, but it fails to be created onsite.<br>- Failed to create a hot swap monitoring thread. |

#### ublock_fini

1. Prototype

    ```c
    void ublock_fini(void);
    ```

2. Description

    Destroys the Ublock module and internally created resources. This API must be used together with the Ublock initialization API.

3. Parameters

    None.

4. Return value

    None.

#### ublock_get_bdevs

1. Prototype

    ```c
    int ublock_get_bdevs(struct ublock_bdev_mgr* bdev_list);
    ```

2. Description

    Obtains the device list (all NVMe devices in the environment, including kernel-mode and user-mode drivers). The obtained NVMe device list contains only PCI addresses and does not contain specific device information. To obtain specific device information, call ublock_get_bdev.

3. Parameters

    | Parameter                         | Description                                                  |
    | --------------------------------- | ------------------------------------------------------------ |
    | struct ublock_bdev_mgr* bdev_list | Output parameter, which returns the device queue. The **bdev_list** pointer must be allocated externally. |

4. Return value

    | Return Value | Description                                |
    | ------------ | ------------------------------------------ |
    | 0            | The device queue is obtained successfully. |
    | -2           | No NVMe device exists in the environment.  |
    | Other values | Failed to obtain the device list.          |

#### ublock_free_bdevs

1. Prototype

    ```c
    void ublock_free_bdevs(struct ublock_bdev_mgr* bdev_list);
    ```

2. Description

    Releases a device list.

3. Parameters

    | Parameter                         | Description                                                  |
    | --------------------------------- | ------------------------------------------------------------ |
    | struct ublock_bdev_mgr* bdev_list | Head pointer of the device queue. After the device queue is cleared, the **bdev_list** pointer is not released. |

4. Return value

    None.

#### ublock_get_bdev

1. Prototype

    ```c
    int ublock_get_bdev(const char *pci, struct ublock_bdev *bdev);
    ```

2. Description

    Obtains information about a specific device. In the device information, the serial number, model, and firmware version of the NVMe device are saved as character arrays instead of character strings. (The return format varies depending on the drive controller, and the arrays may not end with 0.)

    After this API is called, the corresponding device is occupied by Ublock. Therefore, call ublock_free_bdev to free resources immediately after the required service operation is complete.

3. Parameters

    | Parameter                | Description                                                  |
    | ------------------------ | ------------------------------------------------------------ |
    | const char *pci          | PCI address of the device whose information needs to be obtained. |
    | struct ublock_bdev *bdev | Output parameter, which returns the device information. The **bdev** pointer must be allocated externally. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | The device information is obtained successfully.             |
    | -1           | Failed to obtain device information due to incorrect parameters. |
    | -11(EAGAIN)  | Failed to obtain device information due to the RPC query failure. A retry is required (3s sleep is recommended). |

#### ublock_get_bdev_by_esn

1. Prototype

    ```c
    int ublock_get_bdev_by_esn(const char *esn, struct ublock_bdev *bdev);
    ```

2. Description

    Obtains information about the device corresponding to an ESN. In the device information, the serial number, model, and firmware version of the NVMe device are saved as character arrays instead of character strings. (The return format varies depending on the drive controller, and the arrays may not end with 0.)

    After this API is called, the corresponding device is occupied by Ublock. Therefore, call ublock_free_bdev to free resources immediately after the required service operation is complete.

3. Parameters

    | Parameter                | Description                                                  |
    | ------------------------ | ------------------------------------------------------------ |
    | const char *esn          | ESN of the device whose information is to be obtained.<br>Note:<br>An ESN is a string of a maximum of 20 characters (excluding the end character of the string), but the length may vary according to hardware vendors. For example, if the length is less than 20 characters, spaces are padded at the end of the character string. |
    | struct ublock_bdev *bdev | Output parameter, which returns the device information. The **bdev** pointer must be allocated externally. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | The device information is obtained successfully.             |
    | -1           | Failed to obtain device information due to incorrect parameters. |
    | -11(EAGAIN)  | Failed to obtain device information due to the RPC query failure. A retry is required (3s sleep is recommended). |

#### ublock_free_bdev

1. Prototype

    ```c
    void ublock_free_bdev(struct ublock_bdev *bdev);
    ```

2. Description

    Frees device resources.

3. Parameters

    | Parameter                | Description                                                  |
    | ------------------------ | ------------------------------------------------------------ |
    | struct ublock_bdev *bdev | Pointer to the device information. After the data in the pointer is cleared, the **bdev** pointer is not freed. |

4. Return value

    None.

#### TAILQ_FOREACH_SAFE

1. Prototype

    ```c
    #define TAILQ_FOREACH_SAFE(var, head, field, tvar) 
    for ((var) = TAILQ_FIRST((head)); 
    (var) && ((tvar) = TAILQ_NEXT((var), field), 1); 
    (var) = (tvar))
    ```

2. Description

    Provides a macro definition for each member of the secure access queue.

3. Parameters

    | Parameter | Description                                                  |
    | --------- | ------------------------------------------------------------ |
    | var       | Queue node member on which you are performing operations.    |
    | head      | Queue head pointer. Generally, it refers to the object address defined by **TAILQ_HEAD(xx, xx) obj**. |
    | field     | Name of the struct used to store the pointers before and after the queue in the queue node. Generally, it is the name defined by **TAILQ_ENTRY (xx) name**. |
    | tvar      | Next queue node member.                                      |

4. Return value

    None.

#### ublock_get_SMART_info

1. Prototype

    ```c
    int ublock_get_SMART_info(const char *pci, uint32_t nsid, struct ublock_SMART_info *smart_info);
    ```

2. Description

    Obtains the S.M.A.R.T. information of a specified device.

3. Parameters

    | Parameter                            | Description                                                  |
    | ------------------------------------ | ------------------------------------------------------------ |
    | const char *pci                      | Device PCI address.                                          |
    | uint32_t nsid                        | Specified namespace.                                         |
    | struct ublock_SMART_info *smart_info | Output parameter, which returns the S.M.A.R.T. information of the device. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | The S.M.A.R.T. information is obtained successfully.         |
    | -1           | Failed to obtain S.M.A.R.T. information due to incorrect parameters. |
    | -11(EAGAIN)  | Failed to obtain S.M.A.R.T. information due to the RPC query failure. A retry is required (3s sleep is recommended). |

#### ublock_get_SMART_info_by_esn

1. Prototype

    ```c
    int ublock_get_SMART_info_by_esn(const char *esn, uint32_t nsid, struct ublock_SMART_info *smart_info);
    ```

2. Description

    Obtains the S.M.A.R.T. information of the device corresponding to an ESN.

3. Parameters

    | Parameter                               | Description                                                  |
    | --------------------------------------- | ------------------------------------------------------------ |
    | const char *esn                         | Device ESN.<br>Note:<br>An ESN is a string of a maximum of 20 characters (excluding the end character of the string), but the length may vary according to hardware vendors. For example, if the length is less than 20 characters, spaces are padded at the end of the character string. |
    | uint32_t nsid                           | Specified namespace.                                         |
    | struct ublock_SMART_info<br>*smart_info | Output parameter, which returns the S.M.A.R.T. information of the device. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | The S.M.A.R.T. information is obtained successfully.         |
    | -1           | Failed to obtain SMART information due to incorrect parameters. |
    | -11(EAGAIN)  | Failed to obtain S.M.A.R.T. information due to the RPC query failure. A retry is required (3s sleep is recommended). |

#### ublock_get_error_log_info

1. Prototype

    ```c
    int ublock_get_error_log_info(const char *pci, uint32_t err_entries, struct ublock_nvme_error_info *errlog_info);
    ```

2. Description

    Obtains the error log information of a specified device.

3. Parameters

    | Parameter                                  | Description                                                  |
    | ------------------------------------------ | ------------------------------------------------------------ |
    | const char *pci                            | Device PCI address.                                          |
    | uint32_t err_entries                       | Number of error logs to be obtained. A maximum of 256 error logs can be obtained. |
    | struct ublock_nvme_error_info *errlog_info | Output parameter, which returns the error log information of the device. For the **errlog_info** pointer, the caller needs to apply for space and ensure that the obtained space is greater than or equal to err_entries x size of (struct ublock_nvme_error_info). |

4. Return value

    | Return Value                                                 | Description                                                  |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | Number of obtained error logs. The value is greater than or equal to 0. | Error logs are obtained successfully.                        |
    | -1                                                           | Failed to obtain error logs due to incorrect parameters.     |
    | -11(EAGAIN)                                                  | Failed to obtain error logs due to the RPC query failure. A retry is required (3s sleep is recommended). |

#### ublock_get_log_page

1. Prototype

    ```c
    int ublock_get_log_page(const char *pci, uint8_t log_page, uint32_t nsid, void *payload, uint32_t payload_size);
    ```

2. Description

    Obtains information about a specified device and log page.

3. Parameters

    | Parameter             | Description                                                  |
    | --------------------- | ------------------------------------------------------------ |
    | const char *pci       | Device PCI address.                                          |
    | uint8_t log_page      | ID of the log page to be obtained. For example, **0xC0** and **0xCA** indicate the customized S.M.A.R.T. information of ES3000 V5 drives. |
    | uint32_t nsid         | Namespace ID. Some log pages support obtaining by namespace while some do not. If obtaining by namespace is not supported, the caller must transfer **0XFFFFFFFF**. |
    | void *payload         | Output parameter, which stores log page information. The caller is responsible for allocating memory. |
    | uint32_t payload_size | Size of the applied payload, which cannot be greater than 4096 bytes. |

4. Return value

    | Return Value | Description                                          |
    | ------------ | ---------------------------------------------------- |
    | 0            | The log page is obtained successfully.               |
    | -1           | Failed to obtain error logs due to parameter errors. |

#### ublock_info_get_pci_addr

1. Prototype

    ```c
    char *ublock_info_get_pci_addr(const void *info);
    ```

2. Description

    Obtains the PCI address of the hot swap device.

    The memory occupied by info and the memory occupied by the returned PCI address do not need to be freed by the service process.

3. Parameters

    | Parameter        | Description                                                  |
    | ---------------- | ------------------------------------------------------------ |
    | const void *info | Hot swap event information transferred by the hot swap monitoring thread to the callback function. |

4. Return value

    | Return Value | Description                       |
    | ------------ | --------------------------------- |
    | NULL         | Failed to obtain the information. |
    | Other values | Obtained PCI address.             |

#### ublock_info_get_action

1. Prototype

    ```c
    enum ublock_nvme_uevent_action ublock_info_get_action(const void *info);
    ```

2. Description

    Obtains the type of the hot swap event.

    The memory occupied by info does not need to be freed by service process.

3. Parameters

    | Parameter        | Description                                                  |
    | ---------------- | ------------------------------------------------------------ |
    | const void *info | Hot swap event information transferred by the hot swap monitoring thread to the callback function. |

4. Return value

    | Return Value               | Description                                                  |
    | -------------------------- | ------------------------------------------------------------ |
    | Type of the hot swap event | Type of the event that triggers the callback function. For details, see the definition in **5.1.2.6 enum ublock_nvme_uevent_action**. |

#### ublock_get_ctrl_iostat

1. Prototype

    ```c
    int ublock_get_ctrl_iostat(const char* pci, struct ublock_ctrl_iostat_info *ctrl_iostat);
    ```

2. Description

    Obtains the I/O statistics of a controller.

3. Parameters

    | Parameter                                   | Description                                                  |
    | ------------------------------------------- | ------------------------------------------------------------ |
    | const char* pci                             | PCI address of the controller whose I/O statistics are to be obtained. |
    | struct ublock_ctrl_iostat_info *ctrl_iostat | Output parameter, which returns I/O statistics. The **ctrl_iostat** pointer must be allocated externally. |

4. Return value

    | Return Value | Description                                                  |
    | ------------ | ------------------------------------------------------------ |
    | 0            | Succeeded in obtaining I/O statistics.                       |
    | -1           | Failed to obtain I/O statistics due to invalid parameters or RPC errors. |
    | -2           | Failed to obtain I/O statistics because the NVMe drive is not taken over by the I/O process. |
    | -3           | Failed to obtain I/O statistics because the I/O statistics function is disabled. |

#### ublock_nvme_admin_passthru

1. Prototype

    ```c
    int32_t ublock_nvme_admin_passthru(const char *pci, void *cmd, void *buf, size_t nbytes);
    ```

2. Description

    Transparently transmits the **nvme admin** command to the NVMe device. Currently, only the **nvme admin** command for obtaining the identify parameter is supported.

3. Parameters

    | Parameter       | Description                                                  |
    | --------------- | ------------------------------------------------------------ |
    | const char *pci | PCI address of the destination controller of the **nvme admin** command. |
    | void *cmd       | Pointer to the **nvme admin** command struct. The struct size is 64 bytes. For details, see the NVMe specifications. Currently, only the command for obtaining the identify parameter is supported. |
    | void *buf       | Saves the output of the **nvme admin** command. The space is allocated by users and the size is expressed in nbytes. |
    | size_t nbytes   | Size of the user buffer. The buffer for the identify parameter is 4096 bytes, and that for the command to obtain the identify parameter is 4096 nbytes. |

4. Return value

    | Return Value | Description                                |
    | ------------ | ------------------------------------------ |
    | 0            | The user command is executed successfully. |
    | -1           | Failed to execute the user command.        |

# Appendixes

## GENERIC

Generic Error Code Reference

| sc                                         | value |
| ------------------------------------------ | ----- |
| NVME_SC_SUCCESS                            | 0x00  |
| NVME_SC_INVALID_OPCODE                     | 0x01  |
| NVME_SC_INVALID_FIELD                      | 0x02  |
| NVME_SC_COMMAND_ID_CONFLICT                | 0x03  |
| NVME_SC_DATA_TRANSFER_ERROR                | 0x04  |
| NVME_SC_ABORTED_POWER_LOSS                 | 0x05  |
| NVME_SC_INTERNAL_DEVICE_ERROR              | 0x06  |
| NVME_SC_ABORTED_BY_REQUEST                 | 0x07  |
| NVME_SC_ABORTED_SQ_DELETION                | 0x08  |
| NVME_SC_ABORTED_FAILED_FUSED               | 0x09  |
| NVME_SC_ABORTED_MISSING_FUSED              | 0x0a  |
| NVME_SC_INVALID_NAMESPACE_OR_FORMAT        | 0x0b  |
| NVME_SC_COMMAND_SEQUENCE_ERROR             | 0x0c  |
| NVME_SC_INVALID_SGL_SEG_DESCRIPTOR         | 0x0d  |
| NVME_SC_INVALID_NUM_SGL_DESCIRPTORS        | 0x0e  |
| NVME_SC_DATA_SGL_LENGTH_INVALID            | 0x0f  |
| NVME_SC_METADATA_SGL_LENGTH_INVALID        | 0x10  |
| NVME_SC_SGL_DESCRIPTOR_TYPE_INVALID        | 0x11  |
| NVME_SC_INVALID_CONTROLLER_MEM_BUF         | 0x12  |
| NVME_SC_INVALID_PRP_OFFSET                 | 0x13  |
| NVME_SC_ATOMIC_WRITE_UNIT_EXCEEDED         | 0x14  |
| NVME_SC_OPERATION_DENIED                   | 0x15  |
| NVME_SC_INVALID_SGL_OFFSET                 | 0x16  |
| NVME_SC_INVALID_SGL_SUBTYPE                | 0x17  |
| NVME_SC_HOSTID_INCONSISTENT_FORMAT         | 0x18  |
| NVME_SC_KEEP_ALIVE_EXPIRED                 | 0x19  |
| NVME_SC_KEEP_ALIVE_INVALID                 | 0x1a  |
| NVME_SC_ABORTED_PREEMPT                    | 0x1b  |
| NVME_SC_SANITIZE_FAILED                    | 0x1c  |
| NVME_SC_SANITIZE_IN_PROGRESS               | 0x1d  |
| NVME_SC_SGL_DATA_BLOCK_GRANULARITY_INVALID | 0x1e  |
| NVME_SC_COMMAND_INVALID_IN_CMB             | 0x1f  |
| NVME_SC_LBA_OUT_OF_RANGE                   | 0x80  |
| NVME_SC_CAPACITY_EXCEEDED                  | 0x81  |
| NVME_SC_NAMESPACE_NOT_READY                | 0x82  |
| NVME_SC_RESERVATION_CONFLICT               | 0x83  |
| NVME_SC_FORMAT_IN_PROGRESS                 | 0x84  |

## COMMAND_SPECIFIC

Error Code Reference for Specific Commands

| sc                                         | value |
| ------------------------------------------ | ----- |
| NVME_SC_COMPLETION_QUEUE_INVALID           | 0x00  |
| NVME_SC_INVALID_QUEUE_IDENTIFIER           | 0x01  |
| NVME_SC_MAXIMUM_QUEUE_SIZE_EXCEEDED        | 0x02  |
| NVME_SC_ABORT_COMMAND_LIMIT_EXCEEDED       | 0x03  |
| NVME_SC_ASYNC_EVENT_REQUEST_LIMIT_EXCEEDED | 0x05  |
| NVME_SC_INVALID_FIRMWARE_SLOT              | 0x06  |
| NVME_SC_INVALID_FIRMWARE_IMAGE             | 0x07  |
| NVME_SC_INVALID_INTERRUPT_VECTOR           | 0x08  |
| NVME_SC_INVALID_LOG_PAGE                   | 0x09  |
| NVME_SC_INVALID_FORMAT                     | 0x0a  |
| NVME_SC_FIRMWARE_REQ_CONVENTIONAL_RESET    | 0x0b  |
| NVME_SC_INVALID_QUEUE_DELETION             | 0x0c  |
| NVME_SC_FEATURE_ID_NOT_SAVEABLE            | 0x0d  |
| NVME_SC_FEATURE_NOT_CHANGEABLE             | 0x0e  |
| NVME_SC_FEATURE_NOT_NAMESPACE_SPECIFIC     | 0x0f  |
| NVME_SC_FIRMWARE_REQ_NVM_RESET             | 0x10  |
| NVME_SC_FIRMWARE_REQ_RESET                 | 0x11  |
| NVME_SC_FIRMWARE_REQ_MAX_TIME_VIOLATION    | 0x12  |
| NVME_SC_FIRMWARE_ACTIVATION_PROHIBITED     | 0x13  |
| NVME_SC_OVERLAPPING_RANGE                  | 0x14  |
| NVME_SC_NAMESPACE_INSUFFICIENT_CAPACITY    | 0x15  |
| NVME_SC_NAMESPACE_ID_UNAVAILABLE           | 0x16  |
| NVME_SC_NAMESPACE_ALREADY_ATTACHED         | 0x18  |
| NVME_SC_NAMESPACE_IS_PRIVATE               | 0x19  |
| NVME_SC_NAMESPACE_NOT_ATTACHED             | 0x1a  |
| NVME_SC_THINPROVISIONING_NOT_SUPPORTED     | 0x1b  |
| NVME_SC_CONTROLLER_LIST_INVALID            | 0x1c  |
| NVME_SC_DEVICE_SELF_TEST_IN_PROGRESS       | 0x1d  |
| NVME_SC_BOOT_PARTITION_WRITE_PROHIBITED    | 0x1e  |
| NVME_SC_INVALID_CTRLR_ID                   | 0x1f  |
| NVME_SC_INVALID_SECONDARY_CTRLR_STATE      | 0x20  |
| NVME_SC_INVALID_NUM_CTRLR_RESOURCES        | 0x21  |
| NVME_SC_INVALID_RESOURCE_ID                | 0x22  |
| NVME_SC_CONFLICTING_ATTRIBUTES             | 0x80  |
| NVME_SC_INVALID_PROTECTION_INFO            | 0x81  |
| NVME_SC_ATTEMPTED_WRITE_TO_RO_PAGE         | 0x82  |

## MEDIA_DATA_INTERGRITY_ERROR

Error Code Reference for Medium Exceptions

| sc                                     | value |
| -------------------------------------- | ----- |
| NVME_SC_WRITE_FAULTS                   | 0x80  |
| NVME_SC_UNRECOVERED_READ_ERROR         | 0x81  |
| NVME_SC_GUARD_CHECK_ERROR              | 0x82  |
| NVME_SC_APPLICATION_TAG_CHECK_ERROR    | 0x83  |
| NVME_SC_REFERENCE_TAG_CHECK_ERROR      | 0x84  |
| NVME_SC_COMPARE_FAILURE                | 0x85  |
| NVME_SC_ACCESS_DENIED                  | 0x86  |
| NVME_SC_DEALLOCATED_OR_UNWRITTEN_BLOCK | 0x87  |
