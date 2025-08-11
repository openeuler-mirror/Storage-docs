# HSAK 接口说明

## C接口

### 宏定义和枚举

#### bdev_rw.h

##### enum libstorage_ns_lba_size

1. 原型

    ```sh
    enum libstorage_ns_lba_size
    {
    LIBSTORAGE_NVME_NS_LBA_SIZE_512 = 0x9,
    LIBSTORAGE_NVME_NS_LBA_SIZE_4K = 0xc
    };
    ```

2. 描述

磁盘sector_size（数据）大小。

##### enum libstorage_ns_md_size

1. 原型

    ```shell
    enum libstorage_ns_md_size
    {
    LIBSTORAGE_METADATA_SIZE_0 = 0,
    LIBSTORAGE_METADATA_SIZE_8 = 8,
    LIBSTORAGE_METADATA_SIZE_64 = 64
    };
    ```

2. 描述

    磁盘meta data（元数据） size大小。

3. 备注

    - ES3000 V3（单端口）支持5种扇区类型的格式化（512+0，512+8，4K+64，4K，4K+8）。

    - ES3000 V3（双端口）支持4种扇区类型的格式化（512+0，512+8，4K+64，4K）。

    - ES3000 V5 支持5种扇区类型的格式化（512+0，512+8，4K+64，4K，4K+8）。

    - Optane盘支持7种扇区类型的格式化（512+0，512+8，512+16,4K，4K+8，4K+64，4K+128）。

##### enum libstorage_ns_pi_type

1. 原型

    ```shell
    enum libstorage_ns_pi_type
    {
    LIBSTORAGE_FMT_NVM_PROTECTION_DISABLE = 0x0,
    LIBSTORAGE_FMT_NVM_PROTECTION_TYPE1 = 0x1,
    LIBSTORAGE_FMT_NVM_PROTECTION_TYPE2 = 0x2,
    LIBSTORAGE_FMT_NVM_PROTECTION_TYPE3 = 0x3,
    };
    ```

2. 描述

    磁盘支持的保护类型。

3. 备注

    ES3000仅支持保护类型0和保护类型3，Optane盘仅支持保护类型0和保护类型1。

##### enum libstorage_crc_and_prchk

1. 原型

    ```shell
    enum libstorage_crc_and_prchk
    {
    LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK = 0x0,
    LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK = 0x1,
    LIBSTORAGE_LIB_CRC_AND_DISABLE_PRCHK = 0x2,
    LIBSTORAGE_LIB_CRC_AND_ENABLE_PRCHK = 0x3,
    # define NVME_NO_REF 0x4
    LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK_NO_REF = LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK | NVME_NO_REF,
    LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK_NO_REF = LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK | NVME_NO_REF,
    };
    ```

2. 描述

    - LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK：应用层做CRC校验，HSAK不做CRC校验，关闭盘的CRC校验。

    - LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK：应用层做CRC校验，HSAK不做CRC校验，开启盘的CRC校验。

    - LIBSTORAGE_LIB_CRC_AND_DISABLE_PRCHK：应用层不做CRC校验，HSAK做CRC校验，关闭盘的CRC校验。

    - LIBSTORAGE_LIB_CRC_AND_ENABLE_PRCHK：应用层不做CRC校验，HSAK做CRC校验，开启盘的CRC校验。

    - LIBSTORAGE_APP_CRC_AND_DISABLE_PRCHK_NO_REF：应用层做CRC校验，HSAK不做CRC校验，关闭盘的CRC校验。对于PI TYPE为1的磁盘（Intel optane P4800），关闭盘的REF TAG校验。

    - LIBSTORAGE_APP_CRC_AND_ENABLE_PRCHK_NO_REF：应用层做CRC校验，HSAK不做CRC校验，开启盘的CRC校验。对于PI TYPE为1的磁盘（Intel optane P4800），关闭盘的REF TAG校验。

    - Intel optane P4800盘PI TYPE为1，默认会校验元数据区的CRC和REF TAG。

    - Intel optane P4800盘的512+8格式支持DIF，4096+64格式不支持。

    - ES3000 V3和ES3000 V5盘PI TYPE为3，默认只校验元数据区的CRC。

    - ES3000 V3的512+8格式支持DIF，4096+64格式不支持。ES3000 V5的512+8和4096+64格式均支持DIF。

    总结为如下：

    <table>
        <tr>
            <td rowspan="2"><b>端到端校验方式</b></td>
            <td rowspan="2"><b>ctrlflag</b></td>
            <td rowspan="2"><b>CRC生成者</b></td>
            <td colspan="3"><b>写流程</b></td>
            <td colspan="3"><b>读流程</b></td>
        </tr>
        <tr>
            <td><b>应用校验</b></td>
            <td><b>HSAK校验CRC</b></td>
            <td><b>盘校验CRC</b></td>
            <td><b>应用校验</b></td>
            <td><b>HSAK校验CRC</b></td>
            <td><b>盘校验CRC</b></td>
        </tr>
        <tr>
            <td rowspan="4">半程保护</td>
            <td>0</td>
            <td>控制器</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
        </tr>
        <tr>
            <td>1</td>
            <td>控制器</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>√</td>
        </tr>
        <tr>
            <td>2</td>
            <td>控制器</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
        </tr>
        <tr>
            <td>3</td>
            <td>控制器</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>X</td>
            <td>√</td>
        </tr>
        <tr>
            <td rowspan="4">全程保护</td>
            <td>0</td>
            <td>APP</td>
            <td>√</td>
            <td>X</td>
            <td>X</td>
            <td>√</td>
            <td>X</td>
            <td>X</td>
        </tr>
        <tr>
            <td>1</td>
            <td>APP</td>
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

##### enum libstorage_print_log_level

1. 原型

    ```shell
    enum libstorage_print_log_level
    {
    LIBSTORAGE_PRINT_LOG_ERROR,
    LIBSTORAGE_PRINT_LOG_WARN,
    LIBSTORAGE_PRINT_LOG_NOTICE,
    LIBSTORAGE_PRINT_LOG_INFO,
    LIBSTORAGE_PRINT_LOG_DEBUG,
    };
    ```

2. 描述

    SPDK日志打印级别：ERROR、WARN、NOTICE、INFO、DEBUG，分别对应配置文件中的0~4。

##### MAX_BDEV_NAME_LEN

1. 原型

    ```bash
    # define MAX_BDEV_NAME_LEN 24
    ```

2. 描述

    块设备名最大长度限制。

##### MAX_CTRL_NAME_LEN

1. 原型

    ```bash
    # define MAX_CTRL_NAME_LEN 16
    ```

2. 描述

    控制器名最大长度限制。

##### LBA_FORMAT_NUM

1. 原型

    ```bash
    # define LBA_FORMAT_NUM 16
    ```

2. 描述

    控制器所支持的LBA格式数目。

##### LIBSTORAGE_MAX_DSM_RANGE_DESC_COUNT

1. 原型

    ```bash
    # define LIBSTORAGE_MAX_DSM_RANGE_DESC_COUNT 256
    ```

2. 描述

    数据集管理命令中16字节集的最大数目。

#### ublock.h

##### UBLOCK_NVME_UEVENT_SUBSYSTEM_UIO

1. 原型

    ```bash
    # define UBLOCK_NVME_UEVENT_SUBSYSTEM_UIO 1
    ```

2. 描述

    用于定义uevent事件所对应的子系统是内核uio，在业务收到uevent事件时，通过该宏定义判断是否为需要处理的内核uio事件。

    数据结构struct ublock_uevent中成员int subsystem的值取值为UBLOCK_NVME_UEVENT_SUBSYSTEM_UIO，当前仅此一个可选值。

##### UBLOCK_TRADDR_MAX_LEN

1. 原型

    ```bash
    # define UBLOCK_TRADDR_MAX_LEN 256
    ```

2. 描述

    以"域:总线:设备.功能"（%04x:%02x:%02x.%x）格式表示的PCI地址字符串的最大长度，其实实际长度远小于256字节。

##### UBLOCK_PCI_ADDR_MAX_LEN

1. 原型

    ```bash
    # define UBLOCK_PCI_ADDR_MAX_LEN 256
    ```

2. 描述

    PCI地址字符串最大长度，实际长度远小于256字节；此处PCI地址格式可能的形式为：

    - 全地址：%x:%x:%x.%x 或 %x.%x.%x.%x。

    - 功能值为0：%x:%x:%x。

    - 域值为0：%x:%x.%x 或 %x.%x.%x。

    - 域和功能值为0：%x:%x 或 %x.%x。

##### UBLOCK_SMART_INFO_LEN

1. 原型

    ```bash
    # define UBLOCK_SMART_INFO_LEN 512
    ```

2. 描述

    获取NVMe盘SMART信息结构体的大小，为512字节。

##### enum ublock_rpc_server_status

1. 原型

    ```bash
    enum ublock_rpc_server_status {
    // start rpc server or not
    UBLOCK_RPC_SERVER_DISABLE = 0,
    UBLOCK_RPC_SERVER_ENABLE = 1,
    };
    ```

2. 描述

    用于表示HSAK内部RPC服务状态，启用或关闭。

##### enum ublock_nvme_uevent_action

1. 原型

    ```bash
    enum ublock_nvme_uevent_action {
    UBLOCK_NVME_UEVENT_ADD = 0,
    UBLOCK_NVME_UEVENT_REMOVE = 1,
    UBLOCK_NVME_UEVENT_INVALID,
    };
    ```

2. 描述

    用于表示uevent热插拔事件是插入硬盘还是移除硬盘。

##### enum ublock_subsystem_type

1. 原型

    ```bash
    enum ublock_subsystem_type {
    SUBSYSTEM_UIO = 0,
    SUBSYSTEM_NVME = 1,
    SUBSYSTEM_TOP
    };
    ```

2. 描述

    指定回调函数类型，用于区分产品注册回调函数时是针对于uio驱动还是针对于内核nvme驱动。

### 数据结构

#### bdev_rw.h

##### struct libstorage_namespace_info

1. 原型

    ```conf
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

2. 描述

    该数据结构中包含硬盘namespace相关信息。

3. 结构体成员

    | **成员**                     | 描述                                           |
    |------------------------------|------------------------------------------------|
    | char name[MAX_BDEV_NAME_LEN] | Namespace名字                                  |
    | uint64_t size                | 该namespace所分配的硬盘大小，字节为单位        |
    | uint64_t sectors             | 扇区数                                         |
    | uint32_t sector_size         | 每扇区大小，字节为单位                         |
    | uint32_t md_size             | Metadata大小，字节为单位                       |
    | uint32_t max_io_xfer_size    | 最大允许的单次IO操作数据大小，字节为单位       |
    | uint16_t id                  | Namespace ID                                   |
    | uint8_t pi_type              | 数据保护类型，取值自enum libstorage_ns_pi_type |
    | uint8_t is_active :1         | Namespace是否激活                              |
    | uint8_t ext_lba :1           | Namespace是否支持扩展LBA                       |
    | uint8_t dsm :1               | Namespace是否支持数据集管理                    |
    | uint8_t pad :3               | 保留字段                                       |
    | uint64_t reserved            | 保留字段                                       |

##### struct libstorage_nvme_ctrlr_info

1. 原型

    ```conf
    struct libstorage_nvme_ctrlr_info
    {
    char name[MAX_CTRL_NAME_LEN];
    char address[24];
    struct
    {
    uint32_t domain;
    uint8_t bus;
    uint8_t dev;
    uint8_t func;
    } pci_addr;
    uint64_t totalcap; /* Total NVM Capacity in bytes */
    uint64_t unusecap; /* Unallocated NVM Capacity in bytes */
    int8_t sn[20]; /* Serial number */
    uint8_t fr[8]; /* Firmware revision */
    uint32_t max_num_ns; /* Number of namespaces */
    uint32_t version;
    uint16_t num_io_queues; /* num of io queues */
    uint16_t io_queue_size; /* io queue size */
    uint16_t ctrlid; /* Controller id */
    uint16_t pad1;
    struct
    {
    struct
    {
    /** metadata size */
    uint32_t ms : 16;
    /** lba data size */
    uint32_t lbads : 8;
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
    uint32_t ns_manage : 1; /* Supports the Namespace Management and Namespace Attachment commands */
    uint32_t directives : 1; /* Controller support Directives or not */
    uint32_t streams : 1; /* Controller support Streams Directives or not */
    uint32_t dsm : 1; /* Controller support Dataset Management or not */
    uint32_t reserved : 11;
    } cap_info;
    };
    ```

2. 描述

    该数据结构中包含硬盘控制器相关信息。

3. 结构体成员

    | **成员** | **描述** |
    |----------|----------|
    | char name[MAX_CTRL_NAME_LEN] | 控制器名字                        |
    | char address[24]                | PCI地址，字符串形式               |
    | struct<br>{<br>uint32_t domain;<br>uint8_t bus;<br>uint8_t dev;<br>uint8_t func;<br>} pci_addr                            | PCI地址，分段形式                 |
    | uint64_t totalcap                | 控制器的总容量大小（字节为单位）Optane盘基于NVMe 1.0协议，不支持该字段 |
    | uint64_t unusecap                | 控制器未使用的容量大小（字节为单位）Optane盘基于NVMe 1.0协议，不支持该字段 |
    | int8_t sn[20];                 | 硬盘序列号。不带'0'的ASCII字符串 |
    | uint8_t fr[8];                 | 硬盘firmware版本号。不带'0'的ASCII字符串 |
    | uint32_t max_num_ns            | 最大允许的namespace数             |
    | uint32_t version                 | 控制器支持的NVMe标准协议版本号    |
    | uint16_t num_io_queues         | 硬盘支持的IO队列数量              |
    | uint16_t io_queue_size         | IO队列最大深度                    |
    | uint16_t ctrlid                  | 控制器ID                          |
    | uint16_t pad1                    | 保留字段                          |

    struct cap_info子结构体成员：

    | **成员**                          | **描述**                          |
    |-----------------------------------|------------------------------------|
    | struct<br>{<br>uint32_t ms : 16;<br>uint32_t lbads : 8;<br>uint32_t reserved : 8;<br>}lbaf[LBA_FORMAT_NUM] | ms：元数据大小，最小为8字节<br>lbads：指示LBA大小为2^lbads，lbads不小于9 |
    | uint8_t nlbaf                    | 控制器所支持的LBA格式数           |
    | uint8_t pad2[3]                | 保留字段                          |
    | uint32_t cur_format : 4         | 控制器当前的LBA格式               |
    | uint32_t cur_extended : 1       | 控制器当前是否支持扩展型LBA       |
    | uint32_t cur_pi : 3             | 控制器当前的保护类型              |
    | uint32_t cur_pil : 1            | 控制器当前的PI（保护信息）位于元数据的first eight bytes或者last eight bytes |
    | uint32_t cur_can_share : 1     | namespace是否支持多路径传输       |
    | uint32_t mc_extented : 1        | 元数据是否作为数据缓冲区的一部分进行传输 |
    | uint32_t mc_pointer : 1         | 元数据是否与数据缓冲区分离        |
    | uint32_t pi_type1 : 1           | 控制器是否支持保护类型一          |
    | uint32_t pi_type2 : 1           | 控制器是否支持保护类型二          |
    | uint32_t pi_type3 : 1           | 控制器是否支持保护类型三          |
    | uint32_t md_start : 1           | 控制器是否支持PI（保护信息）位于元数据的first eight bytes |
    | uint32_t md_end : 1             | 控制器是否支持PI（保护信息）位于元数据的last eight bytes |
    | uint32_t ns_manage : 1          | 控制器是否支持namespace管理       |
    | uint32_t directives : 1          | 是否支持Directives命令集          |
    | uint32_t streams : 1             | 是否支持Streams Directives        |
    | uint32_t dsm : 1                 | 是否支持Dataset Management命令    |
    | uint32_t reserved : 11           | 保留字段                          |

##### struct libstorage_dsm_range_desc

1. 原型

    ```bash
    struct libstorage_dsm_range_desc
    {
    /* RESERVED */
    uint32_t reserved;

    /* NUMBER OF LOGICAL BLOCKS */
    uint32_t block_count;

    /* UNMAP LOGICAL BLOCK ADDRESS */uint64_t lba;};
    ```

2. 描述

    数据管理命令集中单个16字节集的定义。

3. 结构体成员

    | **成员**             | **描述**     |
    |----------------------|--------------|
    | uint32_t reserved    | 保留字段      |
    | uint32_t block_count | 单位LBA的数量 |
    | uint64_t lba         | 起始LBA       |

##### struct libstorage_ctrl_streams_param

1. 原型

    ```bash
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

2. 描述

   NVMe盘支持的Streams属性值。

3. 结构体成员

    | **成员**          | **描述**                                 |
    |---------------|--------------------------------------|
    | uint16_t msl  | 硬盘支持的最大Streams资源数          |
    | uint16_t nssa | 每个NVM子系统可使用的Streams资源数   |
    | uint16_t nsso | 每个NVM子系统已经使用的Streams资源数 |
    | uint16_t pad  | 保留字段                             |

##### struct libstorage_bdev_streams_param

1. 原型

    ```bash
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

2. 描述

    Namespace的Streams属性值。

3. 结构体成员

    |**成员**          |        **描述**    |
    |-------------------------|---------------------------------|
    |uint32_t sws             |性能最优的写粒度，单位：sectors|
    |uint16_t sgs             |Streams分配的写粒度，单位：sws|
    |uint16_t nsa             |Namespace可使用的私有Streams资源数|
    |uint16_t nso             |Namespace已使用的私有Streams资源数|
    |uint16_t reserved[3]   |保留字段|

##### struct libstorage_mgr_info

1. 原型

    ```bash
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

2. 描述

    磁盘管理信息（与管理面使用的磁盘信息一致）。

3. 结构体成员

    |**成员**             |                   **描述**|
    |-------------------------|------------------------------------|
    |char pci[24]                          |磁盘PCI地址字符串|
    |char ctrlName[MAX_CTRL_NAME_LEN]   |磁盘控制器名字符串|
    |uint64_t sector_size                  |磁盘扇区大小|
    |uint64_t cap_size                     |磁盘容量，单位：字节|
    |uint16_t device_id                    |磁盘设备ID|
    |uint16_t subsystem_device_id         |磁盘子系统设备ID|
    |uint16­_t vendor_id                   |磁盘厂商ID|
    |uint16_t subsystem_vendor_id         |磁盘子系统厂商ID|
    |uint16_t controller_id                |磁盘控制器ID|
    |int8_t serial_number[20]            |磁盘序列号|
    |int8_t model_number[40]             |设备型号|
    |uint8_t firmware_revision[8]        |固件版本号|

##### struct __attribute__((packed)) libstorage_smart_info

1. 原型

    ```bash
    /* same with struct spdk_nvme_health_information_page in nvme_spec.h */
    struct __attribute__((packed)) libstorage_smart_info {
    /* details of uint8_t critical_warning

    union spdk_nvme_critical_warning_state {

    uint8_t raw;
    *

    struct {

    uint8_t available_spare : 1;

    uint8_t temperature : 1;

    uint8_t device_reliability : 1;

    uint8_t read_only : 1;

    uint8_t volatile_memory_backup : 1;

    uint8_t reserved : 3;

    } bits;

    };
    */
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

2. 描述

    该数据结构定义了硬盘SMART INFO信息内容。

3. 结构体成员

    | **成员**                          | **描述（具体可以参考NVMe协议）**  |
    |-----------------------------------|------------------------------------|
    | uint8_t critical_warning        | 该域表示控制器状态的重要的告警，bit位设置为1表示有效，可以设置<br>多个bit位有效。重要的告警信息通过异步事件返回给主机端。<br>Bit0：设置为1时表示冗余空间小于设定的阈值<br>Bit1：设置为1时表示温度超过或低于一个重要的阈值<br>Bit2：设置为1时表示由于重要的media错误或者internal error，器件的可靠性已经降低。<br>Bit3：设置为1时，该介质已经被置为只读模式。<br>Bit4：设置为1时，表示控制器的易失性器件fail，该域仅在控制器内部存在易失性器件时有效。<br>Bit 5~7：保留 |
    | uint16_t temperature             | 表示整个器件的温度，单位为Kelvin。 |
    | uint8_t available_spare         | 表示可用冗余空间的百分比（0到100%）。       |
    | uint8_t available_spare_threshold                          | 可用冗余空间的阈值，低于该阈值时上报异步事件。 |
    | uint8_t percentage_used         | 该值表示用户实际使用和厂家设定的器件寿命的百分比，100表示已经达<br>到厂家预期的寿命，但可能不会失效，可以继续使用。该值允许大于100<br>，高于254的值都会被置为255。 |
    | uint8_t reserved[26]           | 保留                              |
    | uint64_t data_units_read[2]  | 该值表示主机端从控制器中读走的512字节数目，其中1表示读走100<br>0个512字节，该值不包括metadata。当LBA大小不为512<br>B时，控制器将其转换成512B进行计算。16进制表示。 |
    | uint64_t data_units_written[2]   | 该值表示主机端写入控制器中的512字节数目，其中1表示写入1000<br>个512字节，该值不包括metadata。当LBA大小不为512B<br>时，控制器将其转换成512B进行计算。16进制表示。 |
    | uint64_t host_read_commands[2]   | 表示下发到控制器的读命令的个数。  |
    | uint64_t host_write_commands[2]  | 表示下发到控制器的写命令的个数    |
    | uint64_t controller_busy_time[2] | 表示控制器处理I/O命令的busy时间，从命令下发SQ到完成命令返回到CQ的整个过程都为busy。该值以分钟为单位。 |
    | uint64_t power_cycles[2]      | 上下电次数。                      |
    | uint64_t power_on_hours[2]   | power-on时间小时数。              |
    | uint64_t unsafe_shutdowns[2]  | 异常关机次数，掉电时仍未接收到CC.SHN时该值加1。 |
    | uint64_t media_errors[2]      | 表示控制器检测到不可恢复的数据完整性错误的次数,<br>其中包括不可纠的ECC错误，CRC错误，LBA tag不匹配。 |
    | uint64_t num_error_info_log_entries[2] | 该域表示控制器生命周期内的错误信息日志的entry数目。 |
    | uint32_t warning_temp_time     | 温度超过warning告警值的累积时间，单位分钟。 |
    | uint32_t critical_temp_time    | 温度超过critical告警值的累积时间，单位分钟。 |
    | uint16_t temp_sensor[8]       | 温度传感器1~8的温度值，单位Kelvin。 |
    | uint8_t reserved2[296]         | 保留                              |

##### libstorage_dpdk_contig_mem

1. 原型

    ```bash
    struct libstorage_dpdk_contig_mem {
    uint64_t virtAddr;
    uint64_t memLen;
    uint64_t allocLen;
    };
    ```

2. 描述

    DPDK内存初始化之后，通知业务层初始化完成的回调函数参数中描述一段连续虚拟内存的信息。

    当前HSAK预留了800M内存，其他内存通过该结构体中的allocLen返回给业务层，用于业务层申请内存自行管理。

    HSAK需要预留的总内存是800M左右，每一个内存段上预留的内存是根据环境的NUMA节点数来计算的。在NUMA节点过多时，每个内存段上预留的内存过小，会导致HSAK初始化失败。因此HSAK只支持最多4个NUMA节点的环境。

3. 结构体成员

    |  **成员**      |       **描述** |
    |--------------------|----------------------|
      |uint64_t virtAddr   |虚拟内存起始地址。|
      |uint64_t memLen     |虚拟内存长度，单位：字节。|
      |uint64_t allocLen   |该内存段中可用的内存长度，单位：字节。|

##### struct libstorage_dpdk_init_notify_arg

1. 原型

    ```bash
    struct libstorage_dpdk_init_notify_arg {
    uint64_t baseAddr;
    uint16_t memsegCount;
    struct libstorage_dpdk_contig_mem *memseg;
    };
    ```

2. 描述

    用于DPDK内存初始化之后，通知业务层初始化完成的回调函数参数，表示所有虚拟内存段信息。

3. 结构体成员

    |  **成员**              |     **描述**|
    |------------------------|-----------------------|
      |uint64_t baseAddr                              |虚拟内存起始地址。|
      |uint16_t memsegCount                           |有效的'memseg'数组成员个数，即连续的虚拟内存段的段数。|
      |struct libstorage_dpdk_contig_mem *memseg   |指向内存段数组的指针，每个数组元素都是一段连续的虚拟内存，两两元素之间是不连续的。|

##### struct libstorage_dpdk_init_notify

1. 原型

    ```bash
    struct libstorage_dpdk_init_notify {
    const char *name;
    void (*notifyFunc)(const struct libstorage_dpdk_init_notify_arg *arg);
    TAILQ_ENTRY(libstorage_dpdk_init_notify) tailq;
    };
    ```

2. 描述

    用于DPDK内存初始化之后，通知业务层回调函数注册的结构体。

3. 结构体成员

    |  **成员**                     |   **描述**|
    |-------------------------------|--------------------------|
      |const char *name                                                             |注册的回调函数的业务层模块名字。|
      |void (*notifyFunc)(const struct libstorage_dpdk_init_notify_arg*arg)   |DPDK内存初始化之后，通知业务层初始化完成的回调函数参数。|
      |TAILQ_ENTRY(libstorage_dpdk_init_notify) tailq                            |存放回调函数注册的链表。|

#### ublock.h

##### struct ublock_bdev_info

1. 原型

    ```bash
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

2. 描述

    该数据结构中包含硬盘设备信息。

3. 结构体成员

    |  **成员**        |   **描述**|
    |------------------|------------|
      |uint64_t sector_size            |硬盘扇区大小，比如512字节 |
      |uint64_t cap_size               |硬盘总容量，字节为单位    |
      |uint16_t device_id              |设备id号                  |
      |uint16_t subsystem_device_id   |子系统的设备id号           |
      |uint16_t vendor_id              |设备厂商主id号            |
      |uint16_t subsystem_vendor_id   |设备厂商子id号             |
      |uint16_t controller_id          |设备控制器id号            |
      |int8_t serial_number[20]      |设备序列号                  |
      |int8_t model_number[40]       |设备型号                    |
      |int8_t firmware_revision[8]   |固件版本号                  |

##### struct ublock_bdev

1. 原型

    ```bash
    struct ublock_bdev {
    char pci[UBLOCK_PCI_ADDR_MAX_LEN];
    struct ublock_bdev_info info;
    struct spdk_nvme_ctrlr *ctrlr;
    TAILQ_ENTRY(ublock_bdev) link;
    };
    ```

2. 描述

    该数据结构中包含指定PCI地址的硬盘信息，而结构本身为队列的一个节点。

3. 结构体成员

    |**成员**                           |       **描述**                                                                                   |
    |-----------------------------------|----------------------------------------------------------------------------------------------------|
    |char pci[UBLOCK_PCI_ADDR_MAX_LEN]  | PCI地址                                                                                           |
    |struct ublock_bdev_info info       |     硬盘设备信息                                                                                  |
    |struct spdk_nvme_ctrlr *ctrlr      |    设备控制器数据结构，该结构体内成员不对外开放，外部业务可通过SPDK开源接口获取相应成员数据。    |
    |TAILQ_ENTRY(ublock_bdev) link      |     队列前后指针结构体                                                                            |

##### struct ublock_bdev_mgr

1. 原型

    ```bash
    struct ublock_bdev_mgr {
    TAILQ_HEAD(, ublock_bdev) bdevs;
    };
    ```

2. 描述

    该数据结构内定义了一个ublock_bdev队列的头结构。

3. 结构体成员

    |**成员**                         |    **描述**     |
    |---------------------------------|------------------|
    |TAILQ_HEAD(, ublock_bdev) bdevs; |  队列头结构体    |

##### struct __attribute__((packed)) ublock_SMART_info

1. 原型

    ```bash
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

2. 描述

    该数据结构定义了硬盘SMART INFO信息内容。

3. 结构体成员

    | **成员**                          | **描述（具体可以参考NVMe协议）**  |
    |-----------------------------------|-----------------------------------|
    | uint8_t critical_warning        | 该域表示控制器状态的重要的告警，bit位设置为1表示有效，可以设置<br>多个bit位有效。重要的告警信息通过异步事件返回给主机端。<br>Bit0：设置为1时表示冗余空间小于设定的阈值<br>Bit1：设置为1时表示温度超过或低于一个重要的阈值<br>Bit2：设置为1时表示由于重要的media错误或者internal error，器件的可靠性已经降低。<br>Bit3：设置为1时，该介质已经被置为只读模式。<br>Bit4：设置为1时，表示控制器的易失性器件fail，该域仅在控制器内部存在易失性器件时有效。<br>Bit 5~7：保留 |
    | uint16_t temperature             | 表示整个器件的温度，单位为Kelvin。 |
    | uint8_t available_spare         | 表示可用冗余空间的百分比（0到100%）。       |
    | uint8_t available_spare_threshold | 可用冗余空间的阈值，低于该阈值时上报异步事件。 |
    | uint8_t percentage_used         | 该值表示用户实际使用和厂家设定的器件寿命的百分比，100表示已经达<br>到厂家预期的寿命，但可能不会失效，可以继续使用。该值允许大于100<br>，高于254的值都会被置为255。 |
    | uint8_t reserved[26]           | 保留                              |
    | uint64_t data_units_read[2]  | 该值表示主机端从控制器中读走的512字节数目，其中1表示读走100<br>0个512字节，该值不包括metadata。当LBA大小不为512B<br>时，控制器将其转换成512B进行计算。16进制表示。 |
    | uint64_t data_units_written[2]   | 该值表示主机端写入控制器中的512字节数目，其中1表示写入1000<br>个512字节，该值不包括metadata。当LBA大小不为512B<br>时，控制器将其转换成512B进行计算。16进制表示。 |
    | uint64_t host_read_commands[2]   | 表示下发到控制器的读命令的个数。  |
    | uint64_t host_write_commands[2]  | 表示下发到控制器的写命令的个数    |
    | uint64_t controller_busy_time[2] | 表示控制器处理I/O命令的busy时间，从命令下发SQ到完成命令返回到CQ的整个过程都为busy。该值以分钟为单位。|
    | uint64_t power_cycles[2]      | 上下电次数。                      |
    | uint64_t power_on_hours[2]   | power-on时间小时数。              |
    | uint64_t unsafe_shutdowns[2]  | 异常关机次数，掉电时仍未接收到CC.SHN时该值加1。 |
    | uint64_t media_errors[2]      | 表示控制器检测到不可恢复的数据完整性错误的次数，其中包括不可纠的E<br>CC错误，CRC错误，LBA<br>tag不匹配。 |
    | uint64_t num_error_info_log_entries[2] | 该域表示控制器生命周期内的错误信息日志的entry数目。 |
    | uint32_t warning_temp_time     | 温度超过warning告警值的累积时间，单位分钟。 |
    | uint32_t critical_temp_time    | 温度超过critical告警值的累积时间，单位分钟。 |
    | uint16_t temp_sensor[8]       | 温度传感器1~8的温度值，单位Kelvin。 |
    | uint8_t reserved2[296]         | 保留                              |

##### struct ublock_nvme_error_info

1. 原型

    ```bash
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

2. 描述

    该数据结构中包含设备控制器中单条错误信息具体内容，不同控制器可支持的错误条数可能不同。

3. 结构体成员

    |**成员**                |    **描述（具体可以参考NVMe协议）**                                                                                                  |
    |------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
    |uint64_t error_count    |  Error序号，累增。                                                                                                                     |
    |uint16_t sqid           |   此字段指示与错误信息关联的命令的提交队列标识符。如果错误无法关联特定命令，则该字段应设置为FFFFh。                                    |
    |uint16_t cid            |   此字段指示与错误信息关联的命令标识符。如果错误无法关联特定命令，则该字段应设置为FFFFh。                                              |
    |uint16_t status         |   此字段指示已完成命令的"状态字段"。                                                                                                   |
    |uint16_t error_location |  此字段指示与错误信息关联的命令参数。                                                                                                  |
    |uint64_t lba            |   该字段表示遇到错误情况的第一个LBA。                                                                                                  |
    |uint32_t nsid           |   该字段表示遇到错误情况的namespace。                                                                                                  |
    |uint8_t vendor_specific |  如果有其他供应商特定的错误信息可用，则此字段提供与该页面关联的日志页面标识符。 值00h表示没有可用的附加信息。有效值的范围为80h至FFh。  |
    |uint8_t reserved[35]    | 保留                                                                                                                                   |

##### struct ublock_uevent

1. 原型

    ```bash
    struct ublock_uevent {
    enum ublock_nvme_uevent_action action;
    int subsystem;
    char traddr[UBLOCK_TRADDR_MAX_LEN + 1];
    };
    ```

2. 描述

    该数据结构中包含用于表示uevent事件的相关参数。

3. 结构体成员

  | **成员**                               |       **描述**                                                                                                         |
  |----------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
  | enum ublock_nvme_uevent_action action  |    通过枚举，表示uevent事件类型为插入硬盘，还是移除硬盘。                                                               |
  | int subsystem                          |       表示uevent事件的子系统类型，当前仅支持UBLOCK_NVME_UEVENT_SUBSYSTEM_UIO，如果应用程序收到其他值，则可不处理。      |
  | char traddr[UBLOCK_TRADDR_MAX_LEN + 1] |  以"域:总线:设备.功能"（%04x:%02x:%02x.%x）格式表示的PCI地址字符串。                                                    |

##### struct ublock_hook

1. 原型

    ```bash
    struct ublock_hook
    {
    ublock_callback_func ublock_callback;
    void *user_data;
    };
    ```

2. 描述

    该数据结构用于注册回调函数。

3. 结构体成员

    |  **成员**                             |     **描述**                                                              |
    |---------------------------------------|---------------------------------------------------------------------------|
    |  ublock_callback_func ublock_callback |  表示回调时执行的函数，类型为bool func(void *info, void*user_data).      |
    |  void *user_data                      |   传给回调函数的用户参数                                                  |

##### struct ublock_ctrl_iostat_info

1. 原型

    ```bash
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

2. 描述

    该数据结构用于获取控制器的IO统计信息。

3. 结构体成员

    |  **成员**                   |    **描述**                                 |
    |-----------------------------|---------------------------------------------|
    |  uint64_t num_read_ops      | 获取的该控制器的读IO个数（累加值）           |
    |  uint64_t num_write_ops     | 获取的该控制器的写IO个数（累加值）           |
    |  uint64_t read_latency_ms   | 获取的该控制器的读时延（累加值，ms）         |
    |  uint64_t write_latency_ms  | 获取的该控制器的写时延（累加值，ms）         |
    |  uint64_t io_outstanding    |  获取的该控制器的队列深度                    |
    |  uint64_t num_poll_timeout  | 获取的该控制器的轮询超时次数（累加值）       |
    |  uint64_t io_ticks_ms       | 获取的该控制器的IO处理时延（累加值，ms）     |

### API

#### bdev_rw.h

##### libstorage_get_nvme_ctrlr_info

1. 接口原型

    uint32_t libstorage_get_nvme_ctrlr_info(struct libstorage_nvme_ctrlr_info** ppCtrlrInfo);

2. 接口描述

    获取所有控制器信息。

3. 参数

    | **参数成员**                      | **描述**                          |
    |-----------------------------------|-----------------------------------|
    | struct libstorage_nvme_ctrlr_info** ppCtrlrInfo| 出参，返回所有获取到的控制器信息。<br>说明:<br>使用后务必通过free接口释放内存。 |

4. 返回值

    |  **返回值** |  **描述**                                           |
    |-------------|----------------------------------------------|
    |  0          |  控制器信息获取失败，或未获取到任何控制器信息        |
    |  大于0      |  获取到的控制器个数                                  |

##### libstorage_get_mgr_info_by_esn

1. 接口原型

    ```bash
    int32_t libstorage_get_mgr_info_by_esn(const char *esn, struct libstorage_mgr_info *mgr_info);
    ```

2. 接口描述

    数据面获取设备序列号（ESN）对应的NVMe磁盘的管理信息。

3. 参数

    | **参数成员**                      | **描述**                          |
    |---------------------------|----------------------------------------------|
    | const char *esn                  | 被查询设备的ESN号<br>说明:<br>ESN号是最大有效长度为20的字符串（不包括字符串结束符），但该长<br>度根据不同硬件厂商可能存在差异，如不足20字符，需要在字符串末尾加<br>空格补齐。 |
    | struct libstorage_mgr_info *mgr_info      | 出参，返回所有获取到的NVMe磁盘管理信息。 |

4. 返回值

    |  **返回值** |  **描述**                               |
    |-------------|------------------------------------------|
    |  0          |  查询ESN对应的NVMe磁盘管理信息成功。     |
    |  -1         |  查询ESN对应的NVMe磁盘管理信息失败。     |
    |  -2         |  未获取到任何匹配ESN的NVMe磁盘。         |

##### libstorage_get_mgr_smart_by_esn

1. 接口原型

    ```bash
    int32_t libstorage_get_mgr_smart_by_esn(const char *esn, uint32_t nsid, struct libstorage_smart_info *mgr_smart_info);
    ```

2. 接口描述

    数据面获取设备序列号（ESN）对应的NVMe磁盘的SMART信息。

3. 参数

    | **参数成员**                      | **描述**                          |
    |-------------------------------|------------------------------------------|
    | const char *esn                  | 被查询设备的ESN号<br>说明:<br>ESN号是最大有效长度为20的字符串（不包括字符串结束符），但该长<br>度根据不同硬件厂商可能存在差异，如不足20字符，需要在字符串末尾加<br>空格补齐。 |
    | uint32_t nsid                    | 指定的namespace                   |
    | struct libstorage_mgr_info *mgr_info      | 出参，返回所有获取到的NVMe磁盘SMART信息。 |

4. 返回值

    |  **返回值** |  **描述**                             |
    |-------------|---------------------------------------|
    |  0          |  查询ESN对应的NVMe磁盘SMART信息成功。  |
    |  -1         |  查询ESN对应的NVMe磁盘SMART信息失败。  |
    |  -2         |  未获取到任何匹配ESN的NVMe磁盘。       |

##### libstorage_get_bdev_ns_info

1. 接口原型

    ```bash
    uint32_t libstorage_get_bdev_ns_info(const char* bdevName, struct libstorage_namespace_info** ppNsInfo);
    ```

2. 接口描述

   根据设备名称，获取namespace信息。

3. 参数

    | **参数成员**                      | **描述**                          |
    |-------------------------------|---------------------------------------|
    | const char* bdevName             | 设备名称                          |
    | struct libstorage_namespace_info** ppNsInfo | 出参，返回namespace信息。<br>说明<br>使用后务必通过free接口释放内存。 |

4. 返回值

    |  **返回值**|   **描述**   |
    |------------|---------------|
    |  0         |   获取失败    |
    |  1         |   获取成功    |

##### libstorage_get_ctrl_ns_info

1. 接口原型

    ```bash
    uint32_t libstorage_get_ctrl_ns_info(const char* ctrlName, struct libstorage_namespace_info** ppNsInfo);
    ```

2. 接口描述

   根据控制器名称，获取所有namespace信息。

3. 参数

    | **参数成员**                      | **描述**                          |
    |-------------------------------|---------------------------------------|
    | const char* ctrlName             | 控制器名称                        |
    | struct libstorage_namespace_info** ppNsInfo| 出参，返回所有namespace信息。<br>说明<br>使用后务必通过free接口释放内存。 |

4. 返回值

    |  **返回值**|   **描述**                               |
    |------------|-------------------------------------------|
    |  0         |   获取失败，或未获取到任何namespace信息   |
    |  大于0     |   获取到的namespace个数                   |

##### libstorage_create_namespace

1. 接口原型

    ```bash
    int32_t libstorage_create_namespace(const char* ctrlName, uint64_t ns_size, char** outputName);
    ```

2. 接口描述

    在指定控制器上创建namespace（前提是控制器具有namespace管理能力）。

    Optane盘基于NVMe 1.0协议，不支持namespace管理，因此不支持该接口的使用。

    ES3000 V3和V5默认只支持一个namespace。在控制器上默认会存在一个namespace，如果要创建新的namespace，需要将原有namespace删除。

3. 参数

    | **参数成员**          | **描述**                                      |
    |--------------------------|-------------------------------------------|
    | const char* ctrlName | 控制器名称                                    |
    | uint64_t ns_size    | 要创建的namespace大小（以sertor_size为单位） |
    | char** outputName   | 出参：创建的namespace名称<br>说明<br>使用后务必通过free接口释放内存。 |

4. 返回值

    |  **返回值** |  **描述**                        |
    |-------------|-----------------------------------|
    |  小于等于0  |  创建namespace失败                |
    |  大于0      |  所创建的namespace编号（从1开始） |

##### libstorage_delete_namespace

1. 接口原型

    ```bash
    int32_t libstorage_delete_namespace(const char* ctrlName, uint32_t ns_id);
    ```

2. 接口描述

   在指定控制器上删除namespace。Optane盘基于NVMe 1.0协议，不支持namespace管理，因此不支持该接口的使用。

3. 参数

    |  **参数成员**        |   **描述**         |
    |-----------------------|-------------------|
    |  const char* ctrlName |  控制器名字        |
    |  uint32_t ns_id       | Namespace ID       |

4. 返回值

    | **返回值** | **描述**                                            |
    |-------------|-------------------------------------------|
    | 0          | 删除成功                                            |
    | 非0        | 删除失败<br>说明<br>删除namespace前要求先停止IO相关动作，否则删除失败。   |

##### libstorage_delete_all_namespace

1. 接口原型

    ```bash
    int32_t libstorage_delete_all_namespace(const char* ctrlName);
    ```

2. 接口描述

    删除指定控制器上所有namespace。Optane盘基于NVMe 1.0协议，不支持namespace管理，因此不支持该接口的使用。

3. 参数

    |  **参数成员**         |   **描述**     |
    |------------------------|----------------|
    |  const char* ctrlName  |控制器名称      |

4. 返回值

    | **返回值** | **描述**                                            |
    |-------------|-------------------------------------------|
    | 0          | 删除成功                                            |
    | 非0        | 删除失败<br>说明<br>删除namespace前要求先停止IO相关动作，否则删除失败。   |

##### libstorage_nvme_create_ctrlr

1. 接口原型

    ```bash
    int32_t libstorage_nvme_create_ctrlr(const char *pci_addr, const char *ctrlr_name);
    ```

2. 接口描述

    根据PCI地址创建NVMe控制器。

3. 参数

    |  **参数成员**     |    **描述**      |
    |--------------------|-------------------|
    |  char *pci_addr    |PCI地址           |
    |  char *ctrlr_name  |控制器名称        |

4. 返回值

    |  **返回值** |  **描述**  |
    |-------------|--------------|
    |  小于0      |  创建失败   |
    |  0          |  创建成功   |

##### libstorage_nvme_delete_ctrlr

1. 接口原型

    ```bash
    int32_t libstorage_nvme_delete_ctrlr(const char *ctrlr_name);
    ```

2. 接口描述

    根据控制器名称销毁NVMe控制器。

3. 参数

    |  **参数成员**          |    **描述**     |
    |-------------------------|-----------------|
    |  const char *ctrlr_name |  控制器名称     |

   确保已下发的io已经全部返回后方可调用本接口。

4. 返回值

    |  **返回值** |  **描述**   |
    |-------------|--------------|
    |  小于0      |  销毁失败    |
    |  0          |  销毁成功    |

##### libstorage_nvme_reload_ctrlr

1. 接口原型

    ```bash
    int32_t libstorage_nvme_reload_ctrlr(const char *cfgfile);
    ```

2. 接口描述

   根据配置文件增删NVMe控制器。

3. 参数

    |  **参数成员**       |   **描述**         |
    |----------------------|-------------------|
    |  const char *cfgfile |  配置文件路径      |

    使用本接口删盘时，需要确保已下发的io已经全部返回。

4. 返回值

    |  **返回值** |  **描述**                                          |
    |-------------|-----------------------------------------------------|
    |  小于0      |  根据配置文件增删盘失败（可能部分控制器增删成功）   |
    |  0          |  根据配置文件增删盘成功                             |

> 使用限制

- 目前最多支持在配置文件中配置36个控制器。

- 重加载接口会尽可能创建多的控制器，某个控制器创建失败，不会影响其他控制器的创建。

- 无法保证并发场景下最终的盘初始化情况与最后调用传入的配置文件相符。

- 对正在下发io的盘通过reload删除时，会导致io失败。

- 修改配置文件中pci地址对应的控制器名称(e.g.nvme0)，调用此接口后无法生效。

- reload仅针对于增删盘的场景有效，配置文件中的其他配置项修改无法重载。

##### libstorage_low_level_format_nvm

1. 接口原型

    ```bash
    int8_t libstorage_low_level_format_nvm(const char* ctrlName, uint8_t lbaf,
    enum libstorage_ns_pi_type piType,
    bool pil_start, bool ms_extented, uint8_t ses);
    ```

2. 接口描述

   低级格式化NVMe盘。

3. 参数

    |  **参数成员**                      |     **描述**                                                               |
    |-------------------------------------|----------------------------------------------------------------------------|
    |  const char* ctrlName               |  控制器名称                                                                |
    |  uint8_t lbaf                       |  所要使用的LBA格式                                                         |
    |  enum libstorage_ns_pi_type piType  |所要使用的保护类型                                                          |
    |  bool pil_start                     |  pi信息位于元数据的first eight bytes（1） or last eight bytes （0）        |
    |  bool ms_extented                   |  是否要格式化成扩展型                                                      |
    |  uint8_t ses                        |  格式化时是否进行安全擦除（当前仅支持设置为0：no-secure earse）            |

4. 返回值

    |  **返回值** |  **描述**                  |
    |-------------|-----------------------------|
    |  小于0      |  格式化失败                 |
    |  大于等于0  |  当前格式化成功的LBA格式    |

> 使用限制

- 该低级格式化接口会清除磁盘namespace的数据和元数据，请谨慎使用。

- ES3000盘在格式化时耗时数秒，Intel Optane盘在格式化时需耗时数分钟，在使用该接口时需要等待其执行完成。若强行杀掉格式化进程，会导致格式化失败。

- 在格式化执行之前，需要停止数据面的IO操作。如果当前磁盘正在处理IO请求，格式化操作会概率性出现失败，并且在格式化成功的情况下会存在硬盘丢弃正在处理的IO的可能，所以在格式化前，请保证数据面的IO操作已停止。

- 格式化过程中会reset控制器，导致之前已经初始化的磁盘资源不可用。因此格式化完成之后，需要重启数据面IO进程。

- ES3000 V3支持保护类型0和3，支持PI start和PI end，仅支持mc extended。ES3000 V3的512+8格式支持DIF，4096+64格式不支持。

- ES3000 V5支持保护类型0和3，支持PI start和PI end，支持mc extended和mc pointer。ES3000 V5的512+8和4096+64格式均支持DIF。

- Optane盘支持保护类型0和1，仅支持PI end，仅支持mc extended。Optane的512+8格式支持DIF，4096+64格式不支持。

|  **磁盘类型**       | **LBA格式**     | **磁盘类型**  |  **LBA格式**      |
|----------------------|-----------------|---------------|-------------------|
|  Intel Optane P4800  | lbaf0:512+0<br>lbaf1:512+8<br>lbaf2:512+16<br>lbaf3:4096+0<br>lbaf4:4096+8<br>lbaf5:4096+64<br>lbaf6:4096+128 | ES3000 V3、V5 |  lbaf0:512+0<br>lbaf1:512+8<br>lbaf2:4096+64<br>lbaf3:4096+0<br>lbaf4:4096+8  |

##### LIBSTORAGE_CALLBACK_FUNC

1. 接口原型

    ```bash
    typedef void (*LIBSTORAGE_CALLBACK_FUNC)(int32_t cb_status, int32_t sct_code, void* cb_arg);
    ```

2. 接口描述

    注册的HSAK io完成回调函数。

3. 参数

    | **参数成员**                      | **描述**                          |
    |------------------------------------|-----------------------------|
    | int32_t cb_status               | io 状态码，0为成功，负值为系统错误码，正值为硬盘错误码（不同错误码的<br>含义见[附录](#附录)）           |
    | int32_t sct_code                | io 状态码类型（0：[GENERIC](#generic)；<br>1：[COMMAND_SPECIFIC](#command_specific)；<br>2：[MEDIA_DATA_INTERGRITY_ERROR](#media_data_intergrity_error)<br>7：VENDOR_SPECIFIC）                |
    | void* cb_arg                    | 回调函数的入参                    |

4. 返回值

   无。

##### libstorage_deallocate_block

1. 接口原型

    ```bash
    int32_t libstorage_deallocate_block(int32_t fd, struct libstorage_dsm_range_desc *range, uint16_t range_count, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. 接口描述

   告知NVMe盘可释放的块。

3. 参数

    | **参数成员**                      | **描述**                          |
    |------------------------------------|-----------------------------|
    | int32_t fd                       | 已打开的硬盘文件描述符            |
    | struct libstorage_dsm_range_desc *range | NVMe盘可释放的块描述列表<br>说明<br>该参数需要使用libstorage_mem_reserve分配<br>大页内存，分配内存时需要4K对齐，即align设置为4096。<br>盘的TRIM的范围根据不同的盘进行约束，超过盘侧的最大TRIM范围<br>可能触发数据异常。 |
    | uint16_t range_count            | 数组range的成员数                 |
    | LIBSTORAGE_CALLBACK_FUNC cb     | 回调函数                          |
    | void* cb_arg                    | 回调函数参数                      |

4. 返回值

    |  **返回值** |  **描述**     |
    |-------------|----------------|
    |  小于0      |  请求下发失败  |
    |  0          |  请求下发成功  |

##### libstorage_async_write

1. 接口原型

    ```bash
    int32_t libstorage_async_write(int32_t fd, void *buf, size_t nbytes, off64_t offset, void *md_buf, size_t md_len, enum libstorage_crc_and_prchk dif_flag, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. 接口描述

   HSAK下发异步IO写请求的接口（写缓冲区为连续buffer）。

3. 参数

    | **参数成员**                      | **描述**                          |
    |------------------------------------|-----------------------------|
    | int32_t fd                       | 块设备的文件描述符                |
    | void *buf                        | IO写数据的缓冲区（四字节对齐，不能跨4K页面边界）<br>说明<br>注：扩展型LBA要包含元数据内存大小。 |
    | size_t nbytes                    | 单次写IO大小（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | off64_t offset                   | LBA的写偏移（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | void *md_buf                    | 元数据的缓冲区（仅适用于分离型LBA，扩展型LBA设置为NULL即可） |
    | size_t md_len                   | 元数据的缓冲区长度（仅适用于分离型LBA，扩展型LBA设置为0即可） |
    | enum libstorage_crc_and_prchk dif_flag | 是否计算DIF、是否开启盘的校验     |
    | LIBSTORAGE_CALLBACK_FUNC cb     | 注册的回调函数                    |
    | void* cb_arg                    | 回调函数的参数                    |

4. 返回值

    |  **返回值**|  **描述**         |
    |------------|--------------------|
    |  0         |  IO写请求提交成功  |
    |  非0       |  IO写请求提交失败  |

##### libstorage_async_read

1. 接口原型

    ```bash
    int32_t libstorage_async_read(int32_t fd, void *buf, size_t nbytes, off64_t offset, void *md_buf, size_t md_len, enum libstorage_crc_and_prchk dif_flag, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. 接口描述

   HSAK下发异步IO读请求的接口（读缓冲区为连续buffer）。

3. 参数

    | **参数成员**                      | **描述**                          |
    |------------------------------------|----------------------------------|
    | int32_t fd                       | 块设备的文件描述符                |
    | void *buf                        | IO读数据的缓冲区（四字节对齐，不能跨4K页面边界）<br>说明<br>扩展型LBA要包含元数据内存大小。 |
    | size_t nbytes                    | 单次读IO大小（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | off64_t offset                   | LBA的读偏移（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | void *md_buf                    | 元数据的缓冲区（仅适用于分离型LBA，扩展型LBA设置为NULL即可） |
    | size_t md_len                   | 元数据的缓冲区长度（仅适用于分离型LBA，扩展型LBA设置为0即可） |
    | enum libstorage_crc_and_prchk dif_flag | 是否计算DIF、是否开启盘的校验     |
    | LIBSTORAGE_CALLBACK_FUNC cb     | 注册的回调函数                    |
    | void* cb_arg                    | 回调函数的参数                    |

4. 返回值

    |  **返回值** |  **描述**          |
    |-------------|------------------|
    |  0          |  IO读请求提交成功   |
    |  非0        |  IO读请求提交失败   |

##### libstorage_async_writev

1. 接口原型

    ```bash
    int32_t libstorage_async_writev(int32_t fd, struct iovec *iov, int iovcnt, size_t nbytes, off64_t offset, void *md_buf, size_t md_len, enum libstorage_crc_and_prchk dif_flag, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. 接口描述

   HSAK下发异步IO写请求的接口（写缓冲区为离散buffer）。

3. 参数

    | **参数成员**                      | **描述**                          |
    |------------------------------------|----------------------------------|
    | int32_t fd                       | 块设备的文件描述符                |
    | struct iovec *iov                | IO写数据的缓冲区<br>说明<br>扩展型LBA要包含元数据大小。<br>地址要求四字节对齐，长度不超过4GB。 |
    | int iovcnt                        | IO写数据的缓冲区个数              |
    | size_t nbytes                    | 单次写IO大小（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | off64_t offset                   | LBA的写偏移（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | void *md_buf                    | 元数据的缓冲区（仅适用于分离型LBA，扩展型LBA设置为NULL即可） |
    | size_t md_len                   | 元数据的缓冲区长度（仅适用于分离型LBA，扩展型LBA设置为0即可） |
    | enum libstorage_crc_and_prchk dif_flag | 是否计算DIF、是否开启盘的校验     |
    | LIBSTORAGE_CALLBACK_FUNC cb     | 注册的回调函数                    |
    | void* cb_arg                    | 回调函数的参数                    |

4. 返回值

    |  **返回值**  | **描述**         |
    |--------------|-------------------|
    |  0           | IO写请求提交成功  |
    |  非0         | IO写请求提交失败  |

##### libstorage_async_readv

1. 接口原型

    ```bash
    int32_t libstorage_async_readv(int32_t fd, struct iovec *iov, int iovcnt, size_t nbytes, off64_t offset, void *md_buf, size_t md_len, enum libstorage_crc_and_prchk dif_flag, LIBSTORAGE_CALLBACK_FUNC cb, void* cb_arg);
    ```

2. 接口描述

   HSAK下发异步IO读请求的接口（读缓冲区为离散buffer）。

3. 参数

    | **参数成员**                      | **描述**                          |
    |------------------------------------|----------------------------------|
    | int32_t fd                       | 块设备的文件描述符                |
    | struct iovec *iov                | IO读数据的缓冲区<br>说明<br>扩展型LBA要包含元数据大小。<br>地址要求四字节对齐，长度不超过4GB。 |
    | int iovcnt                        | IO读数据的缓冲区个数              |
    | size_t nbytes                    | 单次读IO大小（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | off64_t offset                   | LBA的读偏移（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | void *md_buf                    | 元数据的缓冲区（仅适用于分离型LBA，扩展型LBA设置为NULL即可） |
    | size_t md_len                   | 元数据的缓冲区长度（仅适用于分离型LBA，扩展型LBA设置为0即可） |
    | enum libstorage_crc_and_prchk dif_flag | 是否计算DIF、是否开启盘的校验     |
    | LIBSTORAGE_CALLBACK_FUNC cb     | 注册的回调函数                    |
    | void* cb_arg                    | 回调函数的参数                    |

4. 返回值

    |  **返回值** |  **描述**           |
    |-------------|----------------------|
    |  0          |  IO读请求提交成功    |
    |  非0        |  IO读请求提交失败    |

##### libstorage_sync_write

1. 接口原型

    ```bash
    int32_t libstorage_sync_write(int fd, const void *buf, size_t nbytes, off_t offset);
    ```

2. 接口描述

   HSAK下发同步IO写请求的接口（写缓冲区为连续buffer）。

3. 参数

    | **参数成员**            | **描述**           |
    |------------------------------------|----------------------------------|
    | int32_t fd     | 块设备的文件描述符                               |
    | void *buf      | IO写数据的缓冲区（四字节对齐，不能跨4K页面边界）<br>说明<br>扩展型LBA要包含元数据内存大小。 |
    | size_t nbytes  | 单次写IO大小（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | off64_t offset | LBA的写偏移（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |

4. 返回值

    |  **返回值** |  **描述**             |
    |-------------|-----------------------|
    |  0          |  IO写请求提交成功      |
    |  非0        |  IO写请求提交失败      |

##### libstorage_sync_read

1. 接口原型

    ```bash
    int32_t libstorage_sync_read(int fd, const void *buf, size_t nbytes, off_t offset);
    ```

2. 接口描述

   HSAK下发同步IO读请求的接口（读缓冲区为连续buffer）。

3. 参数

    | **参数成员**    | **描述**       |
    |------------------------------------|----------------------------------|
    | int32_t fd     | 块设备的文件描述符                               |
    | void *buf      | IO读数据的缓冲区（四字节对齐，不能跨4K页面边界）<br>说明<br>扩展型LBA要包含元数据内存大小。 |
    | size_t nbytes  | 单次读IO大小（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |
    | off64_t offset | LBA的读偏移（单位：字节。sector_size的整数倍）<br>说明<br>仅包含数据大小，扩展型LBA也不含元数据大小。 |

4. 返回值

    |  **返回值** |  **描述**            |
    |-------------|-----------------------|
    |  0          |  IO读请求提交成功     |
    |  非0        |  IO读请求提交失败     |

##### libstorage_open

1. 接口原型

    ```bash
    int32_t libstorage_open(const char* devfullname);
    ```

2. 接口描述

   打开块设备。

3. 参数

    |  **参数成员**           |   **描述**                      |
    |--------------------------|---------------------------------|
    |  const char* devfullname |  块设备名称（格式为nvme0n1）    |

4. 返回值

    |  **返回值** |  **描述**                                                        |
    |-------------|-------------------------------------------------------------------|
    |  -1         |  打开失败（如设备名不对，或打开的fd数目>NVME盘的可使用通道数目）  |
    |  大于0      |  块设备的文件描述符                                               |

    开启nvme.conf.in中的MultiQ开关以后，同一个线程多次打开同一个设备，会返回不同的fd；否则仍返回同一个fd。该特性只针对NVME设备。

##### libstorage_close

1. 接口原型

    ```bash
    int32_t libstorage_close(int32_t fd);
    ```

2. 接口描述

   关闭块设备。

3. 参数

    |  **参数成员** |  **描述**              |
    |--------------|---------------------------|
    |  int32_t fd   |已打开的块设备的文件描述符    |

4. 返回值

    |  **返回值**|   **描述**                    |
    |------------|--------------------------------|
    |  -1        |   无效文件描述符               |
    |  -16       |   文件描述符正忙，需要重试     |
    |  0         |   关闭成功                     |

##### libstorage_mem_reserve

1. 接口原型

    ```bash
    void* libstorage_mem_reserve(size_t size, size_t align);
    ```

2. 接口描述

   从DPDK预留的大页内存中分配内存空间。

3. 参数

    |  **参数成员**|   **描述**                     |
    |---------------|-------------------------------|
    |  size_t size  |  需要分配的内存的大小          |
    |  size_t align |  所分配的内存空间按照align对齐 |

4. 返回值

    |  **返回值** |  **描述**                |
    |-------------|---------------------------|
    |  NULL       |  分配失败                 |
    |  非NULL     |  所分配内存空间的地址     |

##### libstorage_mem_free

1. 接口原型

    ```bash
    void libstorage_mem_free(void* ptr);
    ```

2. 接口描述

    释放ptr指向的内存空间。

3. 参数

    |  **参数成员** |  **描述**                |
    |---------------|--------------------------|
    |  void* ptr    |所要释放的内存空间的地址   |

4. 返回值

   无。

##### libstorage_alloc_io_buf

1. 接口原型

    ```bash
    void* libstorage_alloc_io_buf(size_t nbytes);
    ```

2. 接口描述

   从SPDK的buf_small_pool或者buf_large_pool中分配内存。

3. 参数

    |  **参数成员** |   **描述**                  |
    |----------------|-----------------------------|
    |  size_t nbytes |  所需要分配的缓冲区大小     |

4. 返回值

    |  **返回值** |  **描述**               |
    |-------------|--------------------------|
    |  非NULL     |  所分配的缓冲区的首地址  |

##### libstorage_free_io_buf

1. 接口原型

    ```bash
    int32_t libstorage_free_io_buf(void *buf, size_t nbytes);
    ```

2. 接口描述

   释放所分配的内存到SPDK的buf_small_pool或者buf_large_pool中。

3. 参数

    |  **参数成员** |   **描述**                   |
    |----------------|------------------------------|
    |  void *buf     |  所要释放的缓冲区的首地址    |
    |  size_t nbytes |  所要释放的缓冲区的大小      |

4. 返回值

    |  **返回值** |  **描述**   |
    |-------------|--------------|
    |  -1         |  释放失败    |
    |  0          |  释放成功    |

##### libstorage_init_module

1. 接口原型

    ```bash
    int32_t libstorage_init_module(const char* cfgfile);
    ```

2. 接口描述

   HSAK模块初始化接口。

3. 参数

    |  **参数成员**       |   **描述**           |
    |----------------------|---------------------|
    |  const char* cfgfile |  HSAK 配置文件名称  |

4. 返回值

    |  **返回值** |  **描述**    |
    |-------------|---------------|
    |  非0        |  初始化失败   |
    |  0          |  初始化成功   |

##### libstorage_exit_module

1. 接口原型

    ```bash
    int32_t libstorage_exit_module(void);
    ```

2. 接口描述

   HSAK模块退出接口。

3. 参数

   无。

4. 返回值

    |  **返回值** |  **描述**     |
    |-------------|---------------|
    |  非0        |  退出清理失败  |
    |  0          |  退出清理成功  |

##### LIBSTORAGE_REGISTER_DPDK_INIT_NOTIFY

1. 接口原型

    ```bash
    LIBSTORAGE_REGISTER_DPDK_INIT_NOTIFY(_name, _notify)
    ```

2. 接口描述

   业务层注册函数，用于注册DPDK初始化完成时的回调函数。

3. 参数

    |  **参数成员** | **描述**                                                                                          |
    |----------------|---------------------------------------------------------------------------------------------------|
    |  _name         |业务层模块名称。                                                                                   |
    |  _notify       |业务层注册的回调函数原型：void (*notifyFunc)(const struct libstorage_dpdk_init_notify_arg*arg);   |

4. 返回值

   无。

#### ublock.h

##### init_ublock

1. 接口原型

    ```bash
    int init_ublock(const char *name, enum ublock_rpc_server_status flg);
    ```

2. 接口描述

    初始化Ublock功能模块，本接口必须在其他所有Ublock接口之前被调用。如果flag被置为UBLOCK_RPC_SERVER_ENABLE，即ublock作为rpc server，则同一个进程只能初始化一次。

    在ublock作为rpc server启动时，会同时启动一个server的monitor线程。monitor线程监控到rpc server线程出现异常（如卡死时），会主动调用exit触发进程退出。

    此时依赖于产品的脚本再次拉起相关进程。

3. 参数

    | **参数成员**                      | **描述**                          |
    |----------------------------------|---------------------------------|
    | const char *name                 | 模块名字，缺省值为"ublock"，建议该参数可以传NULL。 |
    | enum ublock_rpc_server_status<br>flg | 是否启用RPC的标记值：UBLOCK_RPC_SERVER_<br>DISABLE或UBLOCK_RPC_SERVER_ENAB<br>LE；<br>在不启用RPC情况下，如果硬盘被业务进程占用，那么Ublock模块<br>将无法获取该硬盘信息。 |

4. 返回值

   | **返回值**                        | **描述**                          |
    |----------------------------------|---------------------------------|
    | 0                                 | 初始化成功。                      |
    | -1                                | 初始化失败，可能原因：Ublock模块已经被初始化。 |
    | 进程exit                          | Ublock认为在两种情况下属于无法修复异常，直接调用exit接口<br>退出进程：<br>-   需要创建RPC服务，但RPC服务现场创建失败。<br>-   创建热插拔监控线程，但失败。 |

##### ublock_init

1. 接口原型

    ```bash
    # define ublock_init(name) init_ublock(name, UBLOCK_RPC_SERVER_ENABLE)
    ```

2. 接口描述

   本身是对init_ublock接口的宏定义，可理解为将Ublock初始化为需要RPC服务。

3. 参数

    |  **参数成员** |  **描述**                                         |
    |---------------|----------------------------------------------------|
    |  name         | 模块名字，缺省值为"ublock"，建议该参数可以传NULL。 |

4. 返回值

    | **返回值**                        | **描述**                          |
    |---------------|----------------------------------------------------|
    | 0                                 | 初始化成功。                      |
    | -1                                | 初始化失败，可能原因：Ublock rpc<br>server模块已经被初始化。 |
    | 进程exit                          | Ublock认为在两种情况下属于无法修复异常，直接调用exit接口<br>退出进程：<br>-   需要创建RPC服务，但RPC服务现场创建失败。<br>-   创建热插拔监控线程，但失败。 |

##### ublock_init_norpc

1. 接口原型

    ```bash
    # define ublock_init_norpc(name) init_ublock(name, UBLOCK_RPC_SERVER_DISABLE)
    ```

2. 接口描述

     本身是对init_ublock接口的宏定义，可理解为将Ublock初始化为无RPC服务。

3. 参数

    |  **参数成员** |  **描述**                                           |
    |---------------|------------------------------------------------------|
    |  name         | 模块名字，缺省值为"ublock"，建议该参数可以传NULL。   |

4. 返回值

    | **返回值**                        | **描述**                          |
    |---------------------------------|-----------------------------|
    | 0                                 | 初始化成功。                      |
    | -1                                | 初始化失败，可能原因：Ublock<br>client模块已经被初始化。 |
    | 进程exit                          | Ublock认为在两种情况下属于无法修复异常，直接调用exit接口<br>退出进程：<br>-   需要创建RPC服务，但RPC服务现场创建失败。<br>-   创建热插拔监控线程，但失败。 |

##### ublock_fini

1. 接口原型

    ```cpp
    void ublock_fini(void);
    ```

2. 接口描述

   销毁Ublock功能模块，本接口将销毁Ublock模块以及内部创建的资源，本接口同Ublock初始化接口需要配对使用。

3. 参数

    无。

4. 返回值

   无。

##### ublock_get_bdevs

1. 接口原型

    ```bash
    int ublock_get_bdevs(struct ublock_bdev_mgr* bdev_list);
    ```

2. 接口描述

    业务进程通过调用本接口获取设备列表（环境上所有的NVME设备，包括内核态驱动和用户态驱动），获取的NVMe设备列表中只有PCI地址，不包含具体设备信息，需要获取具体设备信息，请调用接口ublock_get_bdev。

3. 参数

    |  **参数成员**                       |     **描述**                                        |
    |-------------------------------------|------------------------------------------------------|
    |  struct ublock_bdev_mgr* bdev_list  |出参，返回设备队列，bdev_list指针需要在外部分配。     |

4. 返回值

    |  **返回值** |  **描述**            |
    |-------------|-----------------------|
    |  0          |  获取设备队列成功。   |
    |  -2         |  环境中没有NVMe设备。 |
    |  其余值     |  获取设备队列失败。   |

##### ublock_free_bdevs

1. 接口原型

    ```bash
    void ublock_free_bdevs(struct ublock_bdev_mgr* bdev_list);
    ```

2. 接口描述

    业务进程通过调用本接口释放设备列表。

3. 参数

    |  **参数成员**                       |     **描述**                                                 |
    |-------------------------------------|--------------------------------------------------------------|
    |  struct ublock_bdev_mgr* bdev_list  |设备队列头指针，设备队列清空后，bdev_list指针本身不会被释放。 |

4. 返回值

   无。

##### ublock_get_bdev

1. 接口原型

    ```bash
    int ublock_get_bdev(const char *pci, struct ublock_bdev *bdev);
    ```

2. 接口描述

    业务进程通过调用本接口获取具体某个设备的信息，设备信息中：NVMe设备的序列号、型号、fw版本号信息以字符数组形式保存，不是字符串形式（不同硬盘控制器返回形式不同，不保证数组结尾必定含有"0"）。

    本接口调用后，对应设备会被Ublock占用，请务必在完成相应业务操作后立即调用ublock_free_bdev释放资源。

3. 参数

    |  **参数成员**             |    **描述**                                     |
    |---------------------------|--------------------------------------------------|
    |  const char *pci          |  需要获取信息的设备PCI地址                       |
    |  struct ublock_bdev *bdev | 出参，返回设备信息，bdev指针需要在外部分配。     |

4. 返回值

    |  **返回值**  | **描述**                                                    |
    |-------------|------------------------------------------------------------|
    |  0           | 获取设备信息成功。                                           |
    |  -1          | 获取设备信息失败，如参数错误等。                             |
    |  -11(EAGAIN) | 获取设备信息失败，如rpc查询失败，需要重试（建议sleep 3s）。  |

##### ublock_get_bdev_by_esn

1. 接口原型

    ```bash
    int ublock_get_bdev_by_esn(const char *esn, struct ublock_bdev *bdev);
    ```

2. 接口描述

   业务进程通过调用本接口，根据给定的ESN号获取对应设备的信息，设备信息中：NVMe设备的序列号、型号、fw版本号信息以字符数组形式保存，不是字符串形式（不同硬盘控制器返回形式不同，不保证数组结尾必定含有"0"）。

   本接口调用后，对应设备会被Ublock占用，请务必在完成相应业务操作后立即调用ublock_free_bdev释放资源。

3. 参数

    | **参数成员**                      | **描述**                          |
    |---------------------------|--------------------------------------------------|
    | const char *esn                  | 需要获取信息的设备ESN号。<br>说明<br>ESN号是最大有效长度为20的字符串（不包括字符串结束符），但该长<br>度根据不同硬件厂商可能存在差异，如不足20字符，需要在字符串末尾加<br>空格补齐。 |
    | struct ublock_bdev *bdev        | 出参，返回设备信息，bdev指针需要在外部分配。 |

4. 返回值

    |  **返回值** |  **描述**                                                   |
    |-------------|--------------------------------------------------------------|
    |  0          |  获取设备信息成功。                                          |
    |  -1         |  获取设备信息失败，如参数错误等                              |
    |  -11(EAGAIN)|  获取设备信息失败，如rpc查询失败，需要重试（建议sleep 3s）。 |

##### ublock_free_bdev

1. 接口原型

    ```cpp
    void ublock_free_bdev(struct ublock_bdev *bdev);
    ```

2. 接口描述

   业务进程通过调用本接口释放设备资源。

3. 参数

    |  **参数成员**             |   **描述**                                                  |
    |----------------------------|-------------------------------------------------------------|
    |  struct ublock_bdev *bdev  | 设备信息指针，该指针内数据清空后，bdev指针本身不会被释放。  |

4. 返回值

   无。

##### TAILQ_FOREACH_SAFE

1. 接口原型

    ```bash
    # define TAILQ_FOREACH_SAFE(var, head, field, tvar) 
    for ((var) = TAILQ_FIRST((head)); 
    (var) && ((tvar) = TAILQ_NEXT((var), field), 1); 
    (var) = (tvar))
    ```

2. 接口描述

   提供安全访问队列每个成员的宏定义。

3. 参数

    |  **参数成员** |  **描述**                                                                                         |
    |---------------|----------------------------------------------------------------------------------------------------|
    |  var          | 当前操作的队列节点成员                                                                             |
    |  head         | 队列头指针，一般情况下是指通过TAILQ_HEAD(xx, xx) obj定义的obj的地址                                |
    |  field        | 队列节点中用于保存队列前后指针的结构体名字，一般情况下是指通过TAILQ_ENTRY（xx）name定义的名字name  |
    |  tvar         | 下一个队列节点成员                                                                                 |

4. 返回值

   无。

##### ublock_get_SMART_info

1. 接口原型

    ```bash
    int ublock_get_SMART_info(const char *pci, uint32_t nsid, struct ublock_SMART_info *smart_info);
    ```

2. 接口描述

   业务进程通过调用本接口获取指定设备的SMART信息。

3. 参数

    |  **参数成员**                         |      **描述**               |
    |---------------------------------------|----------------------------|
    |  const char *pci                      |    设备PCI地址              |
    |  uint32_t nsid                        |    指定的namespace          |
    |  struct ublock_SMART_info *smart_info | 出参，返回设备SMART信息     |

4. 返回值

    |  **返回值** |  **描述**                                                    |
    |-------------|---------------------------------------------------------------|
    |  0          |  获取SMART信息成功。                                          |
    |  -1         |  获取SMART信息失败，如参数错误等。                            |
    |  -11(EAGAIN)|  获取SMART信息失败，如rpc查询失败，需要重试（建议sleep 3s）。 |

##### ublock_get_SMART_info_by_esn

1. 接口原型

    ```bash
    int ublock_get_SMART_info_by_esn(const char *esn, uint32_t nsid, struct ublock_SMART_info *smart_info);
    ```

2. 接口描述

    业务进程通过调用本接口获取ESN号对应设备的SMART信息。

3. 参数

    | **参数成员**                      | **描述**                          |
    |--------------------------|-----------------------------------------------|
    | const char *esn                  | 设备ESN号<br>说明<br>ESN号是最大有效长度为20的字符串（不包括字符串结束符），但该长<br>度根据不同硬件厂商可能存在差异，如不足20字符，需要在字符串末尾加<br>空格补齐。 |
    | uint32_t nsid                    | 指定的namespace                   |
    | struct ublock_SMART_info<br>*smart_info | 出参，返回设备SMART信息           |

4. 返回值

    |  **返回值**  |  **描述**                                                    |
    |-------------|--------------------------------------------------------------|
    |  0           |  获取SMART信息成功。                                          |
    |  -1          |  获取SMART信息失败，如参数错误等。                            |
    |  -11(EAGAIN) |  获取SMART信息失败，如rpc查询失败，需要重试（建议sleep 3s）。 |

##### ublock_get_error_log_info

1. 接口原型

    ```bash
    int ublock_get_error_log_info(const char *pci, uint32_t err_entries, struct ublock_nvme_error_info *errlog_info);
    ```

2. 接口描述

   业务进程通过调用本接口获取指定设备的Error log信息。

3. 参数

    |  **参数成员**                               |       **描述**                                                                                                                                    |
    |---------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
    |  const char *pci                            |     设备PCI地址                                                                                                                                    |
    |  uint32_t err_entries                       |    指定希望获取的Error Log条数，最多256条                                                                                                          |
    |  struct ublock_nvme_error_info *errlog_info | 出参，返回设备Error log信息，errlog_info指针需要调用者申请空间，且确保申请的空间大于或等于err_entries * sizeof (struct ublock_nvme_error_info)     |

4. 返回值

    |  **返回值**                         |  **描述**                                                     |
    |-------------------------------------|--------------------------------------------------------------|
    |  获取到的Error log条数，大于或等于0 |  获取Error log成功。                                           |
    |  -1                                 |  获取Error log失败，如参数错误等。                             |
    |  -11(EAGAIN)                        |  获取Error log失败，如rpc查询失败，需要重试（建议sleep 3s）。  |

##### ublock_get_log_page

1. 接口原型

    ```bash
    int ublock_get_log_page(const char *pci, uint8_t log_page, uint32_t nsid, void *payload, uint32_t payload_size);
    ```

2. 接口描述

   业务进程通过调用本接口获取指定设备，指定log page的信息。

3. 参数

    |  **参数成员**         |    **描述**                                                                                                              |
    |------------------------|-------------------------------------------------------------------------------------------------------------------------|
    |  const char *pci       |   设备PCI地址                                                                                                           |
    |  uint8_t log_page      |  指定希望获取的log page ID，比如0xC0， 0xCA代表ES3000 V5盘自定义的SMART信息                                             |
    |  uint32_t nsid         |   指定namespace ID，各个log page对按namespace获取支持情况不一致，如果不支持按namespace获取，调用者需要显示传0xFFFFFFFF  |
    |  void *payload         |   出参，存储log page信息，由调用者负责申请内存                                                                          |
    |  uint32_t payload_size |  申请的payload大小，不大于4096 Bytes                                                                                    |

4. 返回值

    |  **返回值** | **描述**                          |
    |-------------|------------------------------------|
    |  0          | 获取log page成功                   |
    |  -1         | 获取Error log失败，如参数错误等    |

##### ublock_info_get_pci_addr

1. 接口原型

    ```bash
    char *ublock_info_get_pci_addr(const void *info);
    ```

2. 接口描述

    业务进程的回调函数中，通过调用本接口获取热插拔设备的PCI地址。

    info占用的内存以及返回的PCI地址占用得内存不需要业务进程进行释放。

3. 参数

    |  **参数成员**     |   **描述**                                     |
    |-------------------|---------------------------------------------|
    |  const void *info | 热插拔监控线程传递给回调函数的热插拔事件信息    |

4. 返回值

    |  **返回值** |  **描述**         |
    |-------------|--------------------|
    |  NULL       |  获取失败          |
    |  非NULL     |  获取的PCI地址     |

##### ublock_info_get_action

1. 接口原型

    ```bash
    enum ublock_nvme_uevent_action ublock_info_get_action(const void *info);
    ```

2. 接口描述

    业务进程的回调函数中，通过调用本接口获取热插拔事件的类型。

    info占用的内存不需要业务进程进行释放。

3. 参数

    |  **参数成员**     |   **描述**                                    |
    |-------------------|------------------------------------------------|
    |  const void *info | 热插拔监控线程传递给回调函数的热插拔事件信息   |

4. 返回值

    |  **返回值**    |  **描述**                                                                   |
    |----------------|------------------------------------------------------------------------------|
    |  热插拔事件类型|  触发回调函数的事件类型，详见结构体enum ublock_nvme_uevent_action的定义。  |

##### ublock_get_ctrl_iostat

1. 接口原型

    ```bash
    int ublock_get_ctrl_iostat(const char* pci, struct ublock_ctrl_iostat_info *ctrl_iostat);
    ```

2. 接口描述

    业务进程通过调用本接口获取控制器的IO统计信息。

3. 参数

    |  **参数成员**                                 |      **描述**                                           |
    |-----------------------------------------------|----------------------------------------------------------|
    |  const char* pci                              |    需要获取IO统计信息的控制器的PCI地址。                 |
    |  struct ublock_ctrl_iostat_info *ctrl_iostat  |出参，返回IO统计信息，ctrl_iostat指针需要在外部分配。     |

4. 返回值

    |  **返回值** |  **描述**                                      |
    |-------------|-------------------------------------------------|
    |  0          |  获取IO统计信息成功。                           |
    |  -1         |  获取IO统计信息失败（无效参数、RPC error）。    |
    |  -2         |  获取IO统计信息失败（NVMe盘没有被IO进程接管）。 |
    |  -3         |  获取IO统计信息失败（IO统计开关未打开）。       |

##### ublock_nvme_admin_passthru

1. 接口原型

    ```bash
    int32_t ublock_nvme_admin_passthru(const char *pci, void *cmd, void *buf, size_t nbytes);
    ```

2. 接口描述

    业务进程通过调用该接口透传nvme admin命令给nvme设备。当前仅支持获取identify字段的nvme admin命令。

3. 参数

    |  **参数成员**   |   **描述**                                                                                         |
    |------------------|----------------------------------------------------------------------------------------------------|
    |  const char *pci |  nvme admin命令目的控制器的PCI地址。                                                               |
    |  void *cmd       |  nvme admin命令结构体指针，结构体大小为64字节，内容参考nvme spec。当前仅支持获取identify字段命令。 |
    |  void *buf       |  保存nvme admin命令返回内容，其空间由用户分配，大小为nbytes。                                      |
    |  size_t nbytes   |  用户buf的大小。identify字段为4096字节，获取identify命令的nbytes为4096。                           |

4. 返回值

    |  **返回值**|  **描述**           |
    |------------|--------------------|
    |  0         |  用户命令执行成功。  |
    |  -1        |  用户命令执行失败。  |

## 附录

### GENERIC

通用类型错误码参考

|sc |value|
|---------------------------------------------|---------------|
|  NVME_SC_SUCCESS                            |      0x00     |
|  NVME_SC_INVALID_OPCODE                     |     0x01      |
|  NVME_SC_INVALID_FIELD                      |     0x02      |
|  NVME_SC_COMMAND_ID_CONFLICT                |    0x03       |
|  NVME_SC_DATA_TRANSFER_ERROR                |    0x04       |
|  NVME_SC_ABORTED_POWER_LOSS                 |    0x05       |
|  NVME_SC_INTERNAL_DEVICE_ERROR              |    0x06       |
|  NVME_SC_ABORTED_BY_REQUEST                 |    0x07       |
|  NVME_SC_ABORTED_SQ_DELETION                |    0x08       |
|  NVME_SC_ABORTED_FAILED_FUSED               |    0x09       |
|  NVME_SC_ABORTED_MISSING_FUSED              |    0x0a       |
|  NVME_SC_INVALID_NAMESPACE_OR_FORMAT        |   0x0b        |
|  NVME_SC_COMMAND_SEQUENCE_ERROR             |    0x0c       |
|  NVME_SC_INVALID_SGL_SEG_DESCRIPTOR         |   0x0d        |
|  NVME_SC_INVALID_NUM_SGL_DESCIRPTORS        |   0x0e        |
|  NVME_SC_DATA_SGL_LENGTH_INVALID            |   0x0f        |
|  NVME_SC_METADATA_SGL_LENGTH_INVALID        |   0x10        |
|  NVME_SC_SGL_DESCRIPTOR_TYPE_INVALID        |   0x11        |
|  NVME_SC_INVALID_CONTROLLER_MEM_BUF         |   0x12        |
|  NVME_SC_INVALID_PRP_OFFSET                 |    0x13       |
|  NVME_SC_ATOMIC_WRITE_UNIT_EXCEEDED         |   0x14        |
|  NVME_SC_OPERATION_DENIED                   |     0x15      |
|  NVME_SC_INVALID_SGL_OFFSET                 |    0x16       |
|  NVME_SC_INVALID_SGL_SUBTYPE                |    0x17       |
|  NVME_SC_HOSTID_INCONSISTENT_FORMAT         |    0x18       |
|  NVME_SC_KEEP_ALIVE_EXPIRED                 |    0x19       |
|  NVME_SC_KEEP_ALIVE_INVALID                 |    0x1a       |
|  NVME_SC_ABORTED_PREEMPT                    |     0x1b      |
|  NVME_SC_SANITIZE_FAILED                    |     0x1c      |
|  NVME_SC_SANITIZE_IN_PROGRESS               |    0x1d       |
|  NVME_SC_SGL_DATA_BLOCK_GRANULARITY_INVALID |  0x1e         |
|  NVME_SC_COMMAND_INVALID_IN_CMB             |   0x1f        |
|  NVME_SC_LBA_OUT_OF_RANGE                   |   0x80        |
|  NVME_SC_CAPACITY_EXCEEDED                  |     0x81      |
|  NVME_SC_NAMESPACE_NOT_READY                |    0x82       |
|  NVME_SC_RESERVATION_CONFLICT               |     0x83      |
|  NVME_SC_FORMAT_IN_PROGRESS                 |    0x84       |

### COMMAND_SPECIFIC

特定命令错误码参考

|sc |value|
|---------------------------------------------|---------------|
|  NVME_SC_COMPLETION_QUEUE_INVALID           |    0x00       |
|  NVME_SC_INVALID_QUEUE_IDENTIFIER           |    0x01       |
|  NVME_SC_MAXIMUM_QUEUE_SIZE_EXCEEDED        |   0x02        |
|  NVME_SC_ABORT_COMMAND_LIMIT_EXCEEDED       |   0x03        |
|  NVME_SC_ASYNC_EVENT_REQUEST_LIMIT_EXCEEDED |  0x05         |
|  NVME_SC_INVALID_FIRMWARE_SLOT              |    0x06       |
|  NVME_SC_INVALID_FIRMWARE_IMAGE             |    0x07       |
|  NVME_SC_INVALID_INTERRUPT_VECTOR           |    0x08       |
|  NVME_SC_INVALID_LOG_PAGE                   |    0x09       |
|  NVME_SC_INVALID_FORMAT                     |     0x0a      |
|  NVME_SC_FIRMWARE_REQ_CONVENTIONAL_RESET    |   0x0b        |
|  NVME_SC_INVALID_QUEUE_DELETION             |    0x0c       |
|  NVME_SC_FEATURE_ID_NOT_SAVEABLE            |   0x0d        |
|  NVME_SC_FEATURE_NOT_CHANGEABLE             |    0x0e       |
|  NVME_SC_FEATURE_NOT_NAMESPACE_SPECIFIC     |   0x0f        |
|  NVME_SC_FIRMWARE_REQ_NVM_RESET             |   0x10        |
|  NVME_SC_FIRMWARE_REQ_RESET                 |    0x11       |
|  NVME_SC_FIRMWARE_REQ_MAX_TIME_VIOLATION    |  0x12         |
|  NVME_SC_FIRMWARE_ACTIVATION_PROHIBITED     |    0x13       |
|  NVME_SC_OVERLAPPING_RANGE                  |     0x14      |
|  NVME_SC_NAMESPACE_INSUFFICIENT_CAPACITY    |    0x15       |
|  NVME_SC_NAMESPACE_ID_UNAVAILABLE           |    0x16       |
|  NVME_SC_NAMESPACE_ALREADY_ATTACHED         |    0x18       |
|  NVME_SC_NAMESPACE_IS_PRIVATE               |    0x19       |
|  NVME_SC_NAMESPACE_NOT_ATTACHED             |    0x1a       |
|  NVME_SC_THINPROVISIONING_NOT_SUPPORTED     |    0x1b       |
|  NVME_SC_CONTROLLER_LIST_INVALID            |    0x1c       |
|  NVME_SC_DEVICE_SELF_TEST_IN_PROGRESS       |  0x1d         |
|  NVME_SC_BOOT_PARTITION_WRITE_PROHIBITED    |   0x1e        |
|  NVME_SC_INVALID_CTRLR_ID                   |    0x1f       |
|  NVME_SC_INVALID_SECONDARY_CTRLR_STATE      |   0x20        |
|  NVME_SC_INVALID_NUM_CTRLR_RESOURCES        |   0x21        |
|  NVME_SC_INVALID_RESOURCE_ID                |    0x22       |
|  NVME_SC_CONFLICTING_ATTRIBUTES             |     0x80      |
|  NVME_SC_INVALID_PROTECTION_INFO            |    0x81       |
|  NVME_SC_ATTEMPTED_WRITE_TO_RO_PAGE         |  0x82         |

### MEDIA_DATA_INTERGRITY_ERROR

介质异常错误码参考

|sc |value|
|-----------------------------------------|---------------|
|  NVME_SC_WRITE_FAULTS                   |    0x80      |
|  NVME_SC_UNRECOVERED_READ_ERROR         |   0x81       |
|  NVME_SC_GUARD_CHECK_ERROR              |   0x82       |
|  NVME_SC_APPLICATION_TAG_CHECK_ERROR    |  0x83        |
|  NVME_SC_REFERENCE_TAG_CHECK_ERROR      |  0x84        |
|  NVME_SC_COMPARE_FAILURE                |    0x85      |
|  NVME_SC_ACCESS_DENIED                  |    0x86      |
|  NVME_SC_DEALLOCATED_OR_UNWRITTEN_BLOCK |  0x87        |
