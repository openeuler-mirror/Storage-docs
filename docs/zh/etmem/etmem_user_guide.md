# etmem用户指南

## 介绍

随着CPU算力的发展，尤其是ARM核成本的降低，内存成本和内存容量成为约束业务成本和性能的核心痛点，因此如何节省内存成本，如何扩大内存容量成为存储迫切要解决的问题。

etmem内存分级扩展技术，通过DRAM+内存压缩/高性能存储新介质形成多级内存存储，对内存数据进行分级，将分级后的内存冷数据从内存介质迁移到高性能存储介质中，达到内存容量扩展的目的，从而实现内存成本下降。

etmem软件包运行的工具主要分为etmem客户端和etmemd服务端。etmemd服务端工具，运行后常驻，其中实现了目的进程的内存冷热识别及淘汰等功能。etmem客户端工具，调用时运行一次，根据命令参数的不同，控制etmemd服务端响应不同的操作。

## 编译教程

1. 下载etmem源码

    ```bash
    git clone https://gitee.com/openeuler/etmem.git
    ```

2. 编译和运行依赖

    etmem的编译和运行依赖于libboundscheck组件

    安装命令：

    ```bash
    yum install libboundscheck
    ```

    通过rpm包进行确认是否安装：

    ```bash
    rpm -qi libboundscheck
    ```

3. 编译

    ```bash
    cd etmem

    mkdir build

    cd build

    cmake ..

    make
    ```

## 注意事项

### 运行依赖

etmem作为内存扩展工具，需要依赖于内核态的特性支持，为了可以识别内存访问情况和支持主动将内存写入swap分区来达到内存垂直扩展的需求，etmem在运行时需要插入`etmem_scan`和`etmem_swap`模块：

```bash
modprobe etmem_scan
modprobe etmem_swap
```

### 权限限制

运行etmem进程需要root权限，root用户具有系统最高权限，在使用root用户进行操作时，请严格按照操作指导进行操作，避免其他操作造成系统管理及安全风险。

### 使用约束

- etmem的客户端和服务端需要在同一个服务器上部署，不支持跨服务器通信的场景。
- etmem仅支持扫描进程名小于或等于15个字符长度的目标进程。在使用进程名时，支持的进程名有效字符为：“字母”， “数字”，特殊字符“./%-_”以及上述三种的组合，其余组合认为是非法字符。
- 在使用AEP介质进行内存扩展的时候，依赖于系统可以正确识别AEP设备并将AEP设备初始化为`numa node`。并且配置文件中的`vm_flags`字段只能配置为`ht`。
- 引擎私有命令仅针对对应引擎和引擎下的任务有效，比如cslide所支持的`showhostpages`和`showtaskpages`。
- 第三方策略实现代码中，`eng_mgt_func`接口中的`fd`不能写入`0xff`和`0xfe`字。
- 支持在一个工程内添加多个不同的第三方策略动态库，以配置文件中的`eng_name`来区分。
- 禁止并发扫描同一个进程。
- 未加载`etmem_scan`和`etmem_swap` ko时，禁止使用`/proc/xxx/idle_pages`和`/proc/xxx/swap_pages`文件。
- etmem对应配置文件，其权限要求为属主为root用户，且权限为600或400，配置文件大小不超过10M。
- etmem在进行第三方策略注入时，第三方策略的`so`权限要求为属主为root用户，且权限为500或700。

## 使用说明

### etmem配置文件

在运行etmem进程之前，需要管理员预先规划哪些进程需要做内存扩展，将进程信息配置到etmem配置文件中，并配置内存扫描的周期、扫描次数、内存冷热阈值等信息。

配置文件的示例文件在源码包中，放置在`/etc/etmem`文件路径下，按照功能划分为3个示例文件，

```text
/etc/etmem/cslide_conf.yaml
/etc/etmem/slide_conf.yaml
/etc/etmem/thirdparty_conf.yaml
```

示例内容分别为：

```sh
#slide引擎示例
#slide_conf.yaml
[project]
name=test
loop=1
interval=1
sleep=1
sysmem_threshold=50
swapcache_high_vmark=10
swapcache_low_vmark=6

[engine]
name=slide
project=test

[task]
project=test
engine=slide
name=background_slide
type=name
value=mysql
T=1
max_threads=1
swap_threshold=10g
swap_flag=yes

#cslide引擎示例
#cslide_conf.yaml
[engine]
name=cslide
project=test
node_pair=2,0;3,1
hot_threshold=1
node_mig_quota=1024
node_hot_reserve=1024

[task]
project=test
engine=cslide
name=background_cslide
type=pid
name=23456
vm_flags=ht
anon_only=no
ign_host=no

#thirdparty引擎示例
#thirdparty_conf.yaml
[engine]
name=thirdparty
project=test
eng_name=my_engine
libname=/usr/lib/etmem_fetch/my_engine.so
ops_name=my_engine_ops
engine_private_key=engine_private_value

[task]
project=test
engine=my_engine
name=background_third
type=pid
value=12345
task_private_key=task_private_value
```

配置文件各字段说明：

| 配置项       | 配置项含义               | 是否必须 | 是否有参数 | 参数范围       | 示例说明                                                            |
|-----------|---------------------|------|-------|------------|-----------------------------------------------------------------|
| [project] | project公用配置段起始标识    | 否    | 否     | NA         | project参数的开头标识，表示下面的参数直到另外的[xxx]或文件结尾为止的范围内均为project section的参数 |
| name      | project的名字          | 是    | 是     | 64个字以内的字符串 | 用来标识project，engine和task在配置时需要指定要挂载到的project                     |
| loop      | 内存扫描的循环次数           | 是    | 是     | 1~120      | loop=3 //扫描3次                                                   |
| interval  | 每次内存扫描的时间间隔         | 是    | 是     | 1~1200     | interval=5 //每次扫描之间间隔5s                                         |
| sleep     | 每个内存扫描+操作的大周期之间时间间隔 | 是    | 是     | 1~1200     | sleep=10 //每次大周期之间间隔10s                                         |
| sysmem_threshold| slide engine的配置项，系统内存换出阈值 | 否    | 是     | 0~100     | sysmem_threshold=50 //系统内存剩余量小于50%时，etmem才会触发内存换出|
| swapcache_high_wmark| slide engine的配置项，swacache可以占用系统内存的比例，高水线 | 否    | 是     | 1~100     | swapcache_high_wmark=5 //swapcache内存占用量可以为系统内存的5%，超过该比例，etmem会触发swapcache回收<br/> 注： swapcache_high_wmark需要大于swapcache_low_wmark|
| swapcache_low_wmark| slide engine的配置项，swacache可以占用系统内存的比例，低水线 | 否    | 是     | [1~swapcache_high_wmark)     | swapcache_low_wmark=3 //触发swapcache回收后，系统会将swapcache内存占用量回收到低于3%|
| [engine]      | engine公用配置段起始标识                           | 否                  | 否     | NA                                               | engine参数的开头标识，表示下面的参数直到另外的[xxx]或文件结尾为止的范围内均为engine section的参数 |
| project       | 声明所在的project                              | 是                  | 是     | 64个字以内的字符串                                       | 已经存在名字为test的project，则可以写为project=test                        |
| engine        | 声明所在的engine                               | 是                  | 是     | slide/cslide/thirdparty                          | 声明使用的是slide或cslide或third party策略                              |
| node_pair     | cslide engine的配置项，声明系统中AEP和DRAM的node pair | engine为cslide时必须配置 | 是     | 成对配置AEP和DRAM的node号，AEP和DRAM之间用逗号隔开，没对pair之间用分号隔开 | node_pair=2,0;3,1                                            |
| hot_threshold | cslide engine的配置项，声明内存冷热水线的阈值             | engine为cslide时必须配置 | 是     | 大于等于0，小于等于INT_MAX的整数                                          | hot_threshold=3 //访问次数小于3的内存会被识别为冷内存                         |
|node_mig_quota|cslide engine的配置项，流控，声明每次DRAM和AEP互相迁移时单向最大流量|engine为cslide时必须配置|是|大于等于0，小于等于INT_MAX的整数|node_mig_quota=1024 //单位为MB，AEP到DRAM或DRAM到AEP搬迁一次最大1024M|
|node_hot_reserve|cslide engine的配置项，声明DRAM中热内存的预留空间大小|engine为cslide时必须配置|是|大于等于0，小于等于INT_MAX的整数|node_hot_reserve=1024 //单位为MB，当所有虚拟机热内存大于此配置值时，热内存也会迁移到AEP中|
|eng_name|thirdparty engine的配置项，声明engine自己的名字，供task挂载|engine为thirdparty时必须配置|是|64个字以内的字符串|eng_name=my_engine //对此第三方策略engine挂载task时，task中写明engine=my_engine|
|libname|thirdparty engine的配置项，声明第三方策略的动态库的地址，绝对地址|engine为thirdparty时必须配置|是|256个字以内的字符串|libname=/user/lib/etmem_fetch/code_test/my_engine.so|
|ops_name|thirdparty engine的配置项，声明第三方策略的动态库中操作符号的名字|engine为thirdparty时必须配置|是|256个字以内的字符串|ops_name=my_engine_ops //第三方策略实现接口的结构体的名字|
|engine_private_key|thirdparty engine的配置项，预留给第三方策略自己解析私有参数的配置项，选配|否|否|根据第三方策略私有参数自行限制|根据第三方策略私有engine参数自行配置|
| [task]  | task公用配置段起始标识 | 否 | 否 | NA          | task参数的开头标识，表示下面的参数直到另外的[xxx]或文件结尾为止的范围内均为task section的参数 |
| project | 声明所挂的project  | 是 | 是 | 64个字以内的字符串  | 已经存在名字为test的project，则可以写为project=test                     |
| engine  | 声明所挂的engine   | 是 | 是 | 64个字以内的字符串  | 所要挂载的engine的名字                                            |
| name    | task的名字       | 是 | 是 | 64个字以内的字符串  | name=background1 //声明task的名字是backgound1                   |
| type    | 目标进程识别的方式     | 是 | 是 | pid/name    | pid代表通过进程号识别，name代表通过进程名称识别                               |
| value   | 目标进程识别的具体字段   | 是 | 是 | 实际的进程号/进程名称 | 与type字段配合使用，指定目标进程的进程号或进程名称，由使用者保证配置的正确及唯一性               |
| T                | engine为slide的task配置项，声明内存冷热水线的阈值                               | engine为slide时必须配置 | 是 | 0~loop * 3           | T=3 //访问次数小于3的内存会被识别为冷内存                                        |
| max_threads      | engine为slide的task配置项，etmemd内部线程池最大线程数，每个线程处理一个进程/子进程的内存扫描+操作任务 | 否                 | 是 | 1~2 * core数 + 1，缺省值为1 | 对外部无表象，控制etmemd服务端内部处理线程个数，当目标进程有多个子进程时，配置越大，并发执行的个数也多，但占用资源也越多 |
| vm_flags         | engine为cslide的task配置项，通过指定flag扫描的vma，不配置此项时扫描则不会区分             | 否                 | 是 | 256长度以内的字符串，不同flag以空格隔开           | vm_flags=ht //扫描flags为ht（大页）的vma内存                              |
| anon_only        | engine为cslide的task配置项，标识是否只扫描匿名页                               | 否                 | 是 | yes/no               | anon_only=no //配置为yes时只扫描匿名页，配置为no时非匿名页也会扫描                     |
| ign_host         | engine为cslide的task配置项，标识是否忽略host上的页表扫描信息                       | 否                 | 是 | yes/no               | ign_host=no //yes为忽略，no为不忽略                                     |
| task_private_key | engine为thirdparty的task配置项，预留给第三方策略的task解析私有参数的配置项，选配           | 否                 | 否 | 根据第三方策略私有参数自行限制      | 根据第三方策略私有task参数自行配置                                             |
| swap_threshold |slide engine的配置项，进程内存换出阈值           | 否                 | 是 | 进程可用内存绝对值      | swap_threshold=10g //进程占用内存在低于10g时不会触发换出。<br/>当前版本下，仅支持g/G作为内存绝对值单位。与sysmem_threshold配合使用，仅系统内存低于阈值时，进行白名单中进程阈值判断 |
| swap_flag|slide engine的配置项，进程指定内存换出           | 否                 | 是 | yes/no      | swap_flag=yes//使能进程指定内存换出 |

### etmemd服务端启动

在使用etmem提供的服务时，首先根据需要修改相应的配置文件，然后运行etmemd服务端，常驻在系统中来操作目标进程的内存。除了支持在命令行中通过二进制来启动etmemd的进程外，还可以通过配置`service`文件来使etmemd服务端通过`systemctl`方式拉起，此场景需要通过`mode-systemctl`参数来指定支持

#### 使用方法

可以通过下列示例命令启动etmemd的服务端：

```bash
etmemd -l 0 -s etmemd_socket
```

或者

```bash
etmemd --log-level 0 --socket etmemd_socket
```

其中`-l`的`0`和`-s`的`etmemd_socket`是用户自己输入的参数，参数具体含义参考以下列表。

#### 命令行参数说明

| 参数            | 参数含义                           | 是否必须 | 是否有参数 | 参数范围              | 示例说明                                                     |
| --------------- | ---------------------------------- | -------- | ---------- | --------------------- | ------------------------------------------------------------ |
| -l或\-\-log-level | etmemd日志级别                     | 否       | 是         | 0~3                   | 0：debug级别 <br/>  1：info级别 <br/>  2：warning级别 <br/>  3：error级别  <br/> 只有大于等于配置的级别才会打印到/var/log/message文件中 |
| -s或\-\-socket    | etmemd监听的名称，用于与客户端交互 | 是       | 是         | 107个字符之内的字符串 | 指定服务端监听的名称                                         |
| -m或\-\-mode-systemctl| 指定通过systemctl方式来拉起stmemd服务| 否| 否| NA| service文件中需要指定-m参数|
| -h或\-\-help      | 帮助信息                           | 否       | 否         | NA                    | 执行时带有此参数会打印后退出                                 |

### 通过etmem客户端添加或者删除工程/引擎/任务

#### 场景描述

1）管理员创建etmem的project/engine/task（一个工程可包含多个etmem engine，一个engine可以包含多个任务）

2）管理员删除已有的etmem project/engine/task（删除工程前，会自动先停止该工程中的所有任务）

#### 使用方法

在etmemd服务端正常运行后，通过etmem客户端，通过第二个参数指定为obj，来进行创建或删除动作，对project/engine/task则是通过配置文件中配置的内容来进行识别和区分。

- 添加对象：

    ```bash
    etmem obj add -f /etc/etmem/slide_conf.yaml -s etmemd_socket
    ```

    或

    ```bash
    etmem obj add --file /etc/etmem/slide_conf.yaml --socket etmemd_socket
    ```

- 删除对象：

    ```bash
    etmem obj del -f /etc/etmem/slide_conf.yaml -s etmemd_socket
    ```

    或

    ```bash
    etmem obj del --file /etc/etmem/slide_conf.yaml --socket etmemd_socket
    ```

#### 命令行参数说明

| 参数         | 参数含义                                                     | 是否必须 | 是否有参数 | 示例说明                                                 |
| ------------ | ------------------------------------------------------------ | -------- | ---------- | -------------------------------------------------------- |
| -f或\-\-file   | 指定对象的配置文件                                        | 是      | 是         | 需要指定路径名称                                         |
| -s或\-\-socket | 与etmemd服务端通信的socket名称，需要与etmemd启动时指定的保持一致 | 是      | 是         | 必须配置，在有多个etmemd时，由管理员选择与哪个etmemd通信 |

### 通过etmem客户端查询/启动/停止工程

#### 场景描述

在已经通过`etmem obj add`添加工程之后，在还未调用`etmem obj del`删除工程之前，可以对etmem的工程进行启动和停止。

1）管理员启动已添加的工程

2）管理员停止已启动的工程

在管理员调用`obj del`删除工程时，如果工程已经启动，则会自动停止。

#### 使用方法

对于已经添加成功的工程，可以通过`etmem project`的命令来控制工程的启动和停止，命令示例如下：

- 查询工程

    ```bash
    etmem project show -n test -s etmemd_socket
    ```

    或

    ```bash
    etmem project show --name test --socket etmemd_socket
    ```

- 启动工程

    ```bash
    etmem project start -n test -s etmemd_socket
    ```

    或

    ```bash
    etmem project start --name test --socket etmemd_socket
    ```

- 停止工程

    ```bash
    etmem project stop -n test -s etmemd_socket
    ```

    或

    ```bash
    etmem project stop --name test --socket etmemd_socket
    ```

- 打印帮助

    ```bash
    etmem project help
    ```

#### 命令行参数说明

| 参数         | 参数含义                                                     | 是否必须 | 是否有参数 | 示例说明                                                 |
| ------------ | ------------------------------------------------------------ | -------- | ---------- | -------------------------------------------------------- |
| -n或\-\-name   | 指定project名称                                              | 是      | 是         | project名称，与配置文件一一对应                          |
| -s或\-\-socket | 与etmemd服务端通信的socket名称，需要与etmemd启动时指定的保持一致 | 是     | 是         | 必须配置，在有多个etmemd时，由管理员选择与哪个etmemd通信 |

### 通过etmem客户端，支持内存阈值换出以及指定内存换出

当前支持的策略中，只有slide策略支持私有的功能特性

- 进程或系统内存阈值换出

为了获得业务的极致性能，需要考虑etmem内存扩展进行内存换出的时机；当系统可用内存足够，系统内存压力不大时，不进行内存交换；当进程占用内存不高时，不进行内存交换；提供系统内存换出阈值控制以及进程内存换出阈值控制。

- 进程指定内存换出

在存储环境下，具有IO时延敏感型业务进程，上述进程内存不希望进行换出，因此提供一种机制，由业务指定可换出内存

针对进程或系统内存阈值换出，进程指定内存换出功能，可以在配置文件中添加`sysmem_threshold`，`swap_threshold`，`swap_flag`参数，示例如下，具体含义请参考etmem配置文件说明章节。

```sh
#slide_conf.yaml
[project]
name=test
loop=1
interval=1
sleep=1
sysmem_threshold=50

[engine]
name=slide
project=test

[task]
project=test
engine=slide
name=background_slide
type=name
value=mysql
T=1
max_threads=1
swap_threshold=10g
swap_flag=yes
```

#### 系统内存阈值换出

配置文件中`sysmem_threshold`用于指示系统内存阈值换出功能，`sysmem_threshold`取值范围为0-100，如果配置文件中设定了`sysmem_threshold`，那么只有系统内存剩余量低于该比例时，etmem才会触发内存换出流程

示例使用方法如下：

1. 参考示例编写配置文件，配置文件中填写`sysmem_threshold`参数，例如`sysmem_threshold=20`
2. 启动服务端，并通过服务端添加，启动工程。

    ```bash
    etmemd -l 0 -s monitor_app &
    etmem obj add -f etmem_config -s monitor_app
    etmem project start -n test -s monitor_app
    etmem project show -s monitor_app
    ```

3. 观察内存换出结果，只有系统可用内存低于20%时，etmem才会触发内存换出

#### 进程内存阈值换出

配置文件中`swap_threshold`用于指示进程内存阈值换出功能，`swap_threshold`为进程内存占用量绝对值(格式为"数字+单位g/G")，如果配置文件中设定了`swap_threshold`，那么该进程内存占用量在小于该设定的可用内存量时，etmem不会针对该进程触发换出流程

示例使用方法如下：

1. 参考示例编写配置文件，配置文件中填写`swap_threshold`参数，例如`swap_threshold=5g`
2. 启动服务端，并通过服务端添加，启动工程。

    ```bash
    etmemd -l 0 -s monitor_app &
    etmem obj add -f etmem_config -s monitor_app
    etmem project start -n test -s monitor_app
    etmem project show -s monitor_app
    ```

3. 观察内存换出结果，只有进程占用内存绝对值高于5G时，etmem才会触发内存换出

#### 进程指定内存换出

配置文件中`swap_flag`用于指示进程指定内存换出功能，`swap_flag`取值仅有两个：`yes/no`，如果配置文件中设定了`swap_flag`为no或者未配置，那么etmem换出功能无变化，如果`swap_flag`设定为yes，那么etmem仅仅换出进程指定的内存。

示例使用方法如下：

1. 参考示例编写配置文件，配置文件中填写`swap_flag`参数，例如`swap_flag=yes`
2. 业务进程对需要进行换出的内存打标记

    ```bash
    madvise(addr_start, addr_len, MADV_SWAPFLAG)
    ```

3. 启动服务端，并通过服务端添加，启动工程。

    ```bash
    etmemd -l 0 -s monitor_app &
    etmem obj add -f etmem_config -s monitor_app
    etmem project start -n test -s monitor_app
    etmem project show -s monitor_app
    ```

4. 观察内存换出结果，只有进程打标记的部分内存会被换出，其余内存保留在DRAM中，不会被换出

针对进程指定页面换出的场景中，在原扫描接口`idle_pages`中添加`ioctl`命令字的形式，来确认不带有特定标记的vma不进行扫描与换出操作

扫描管理接口

- 函数原型

    ```c
    ioctl(fd, cmd, void *arg);
    ```

- 输入参数

    ```text
    1. fd:文件描述符，通过open调用在/proc/pid/idle_pages下打开文件获得

    2.cmd:控制扫描行为，当前支持如下cmd：
    VMA_SCAN_ADD_FLAGS:新增vma指定内存换出标记，仅扫描带有特定标记的VMA
    VMA_SCAN_REMOVE_FLAGS:删除新增的VMA指定内存换出标记

    3.args:int指针参数，传递具体标记掩码，当前仅支持如下参数：
    VMA_SCAN_FLAG：在etmem_scan.ko扫描模块开始扫描前，会调用接口walk_page_test接口判断vma地址是否符合扫描要求，此标记置位时，会仅扫描带有特定换出标记的vma地址段，而忽略其他vma地址
    ```

- 返回值

    ```text
    1.成功，返回0
    2.失败返回非0
    ```

- 注意事项

    ```text
    所有不支持的标记都会被忽略，但是不会返回错误
    ```

### 通过etmem客户端，支持swapcache内存回收指令

用户态etmem发起内存淘汰回收操作，通过`write procfs`接口与内核态的内存回收模块交互，内存回收模块解析用户态下发的虚拟地址，获取地址对应的page页面，并调用内核原生接口将该page对应内存进行换出回收，在内存换出的过程中，swapcache会占用部分系统内存，为进一步节约内存，添加swapcache内存回收功能.

针对swapcache内存回收功能，可以在配置文件中添加`swapcache_high_wmark`，`swapcache_low_wmark`参数。

- `swapcache_high_wmark`: swapcache可以占用系统内存的高水位线
- `swapcache_low_wmark`：swapcache可以占用系统内存的低水位线

在etmem进行一轮内存换出后，会进行swapcache占用系统内存比例的检查，当占用比例超过高水位线后，会通过`swap_pages`下发`ioctl`命令，触发swapcache内存回收，并回收到低水位线停止

配置参数示例如下，具体请参考etmem配置文件相关章节：

```sh
#slide_conf.yaml
[project]
name=test
loop=1
interval=1
sleep=1
swapcache_high_vmark=5
swapcache_low_vmark=3

[engine]
name=slide
project=test

[task]
project=test
engine=slide
name=background_slide
type=name
value=mysql
T=1
max_threads=1
```

针对swap换出场景中，需要通过swapcache内存回收进一步节约内存，在原内存换出接口`swap_pages`中通过添加`ioctl`接口的方式，来提供swapcache水线的设定以及swapcache内存占用量回收的启动与关闭

- 函数原型

    ```c
    ioctl(fd, cmd, void *arg);
    ```

- 输入参数

    ```text
    1. fd:文件描述符，通过open调用在/proc/pid/idle_pages下打开文件获得

    2.cmd:控制扫描行为，当前支持如下cmd：
    RECLAIM_SWAPCACHE_ON:启动swapcache内存换出
    RECLAIM_SWAPCACHE_OFF:关闭swapcache内存换出
    SET_SWAPCACHE_WMARK:设定swapcache内存水线

    3.args:int指针参数，传递具体标记掩码，当前仅支持如下参数：
    参数用来传递swapcache水线具体值
    ```

- 返回值

    ```text
    1.成功，返回0
    2.失败返回非0
    ```

- 注意事项

    ```text
    所有不支持的标记都会被忽略，但是不会返回错误
    ```

### 通过etmem客户端，执行引擎私有命令或功能

当前支持的策略中，只有cslide策略支持私有的命令

- `showtaskpages`
- `showhostpages`

针对使用此策略引擎的engine和engine所有的task，可以通过这两个命令分别查看task相关的页面访问情况和虚拟机的host上系统大页的使用情况。

示例命令如下：

```bash
etmem engine showtaskpages <-t task_name> -n proj_name -e cslide -s etmemd_socket
etmem engine showhostpages -n proj_name -e cslide -s etmemd_socket
```

**注意** ：`showtaskpages`和`showhostpages`仅支持引擎使用cslide的场景

#### 命令行参数说明

| 参数 | 参数含义 | 是否必须 | 是否有参数 | 实例说明 |
|----|------|------|-------|------|
|-n或\-\-proj_name| 指定project的名字| 是| 是| 指定已经存在，所需要执行的project的名字|
|-s或\-\-socket| 与etmemd服务端通信的socket名称，需要与etmemd启动时指定的保持一致| 是| 是| 必须配置，在有多个etmemd时，由管理员选择与哪个etmemd通信|
|-e或\-\-engine| 指定执行的引擎的名字| 是| 是| 指定已经存在的，所需要执行的引擎的名字|
|-t或\-\-task_name| 指定执行的任务的名字| 否| 是| 指定已经存在的，所需要执行的任务的名字|

### 支持kernel swap功能开启与关闭

针对swap换出到磁盘场景，当etmem用于内存扩展时，用户可以选择是否同时开启内核swap功能。用户可以关闭内核原生swap机制，以免原生swap机制换出不应被换出的内存，导致用户态进程出现问题。

通过提供sys接口实现上述控制，在`/sys/kernel/mm/swap`目录下创建`kobj`对象，对象名为`kernel_swap_enable`，缺省值为`true`，用于控制kernel swap的启动与关闭

具体示例如下：

```sh
#开启kernel swap
echo true > /sys/kernel/mm/swap/kernel_swap_enbale
或者
echo 1 > /sys/kernel/mm/swap/kernel_swap_enbale

#关闭kernel swap
echo false > /sys/kernel/mm/swap/kernel_swap_enbale
或者
echo 0 > /sys/kernel/mm/swap/kernel_swap_enbale

```

### etmem支持随系统自启动

#### 场景描述

etmemd支持由用户配置`systemd`配置文件后，以`fork`模式作为`systemd`服务被拉起运行

#### 使用方法

编写`service`配置文件，来启动etmemd，必须使用-m参数来指定此模式，例如

```bash
etmemd -l 0 -s etmemd_socket -m
```

#### 命令行参数说明

| 参数             | 参数含义       | 是否必须 | 是否有参数 | 参数范围 | 实例说明      |
|----------------|------------|------|-------|------|-----------|
| -l或\-\-log-level | etmemd日志级别 | 否    | 是     | 0~3  | 0：debug级别；1：info级别；2：warning级别；3：error级别；只有大于等于配置的级别才会打印到/var/log/message文件中|
| -s或\-\-socket |etmemd监听的名称，用于与客户端交互 | 是 | 是| 107个字符之内的字符串| 指定服务端监听的名称|
|-m或\-\-mode-systemctl | etmemd作为service被拉起时，命令中需要指定此参数来支持 | 否 | 否 | NA | NA |
| -h或\-\-help | 帮助信息 | 否  |否 |NA |执行时带有此参数会打印后退出|

### etmem支持第三方内存扩展策略

#### 场景描述

etmem支持用户注册第三方内存扩展策略，同时提供扫描模块动态库，运行时通过第三方策略淘汰算法淘汰内存。

用户使用etmem所提供的扫描模块动态库并实现对接etmem所需要的结构体中的接口

#### 使用方法

用户使用自己实现的第三方扩展淘汰策略，主要需要按下面步骤进行实现和操作：

1. 按需调用扫描模块提供的扫描接口，

2. 按照etmem头文件中提供的函数模板来实现各个接口，最终封装成结构体

3. 编译出第三方扩展淘汰策略的动态库

4. 在配置文件中按要求声明类型为thirdparty的engine

5. 将动态库的名称和接口结构体的名称按要求填入配置文件中task对应的字段

其他操作步骤与使用etmem的其他engine类似

接口结构体模板

```c
struct engine_ops {

/* 针对引擎私有参数的解析，如果有，需要实现，否则置NULL */

int (*fill_eng_params)(GKeyFile *config, struct engine *eng);

/* 针对引擎私有参数的清理，如果有，需要实现，否则置NULL */

void (*clear_eng_params)(struct engine *eng);

/* 针对任务私有参数的解析，如果有，需要实现，否则置NULL */

int (*fill_task_params)(GKeyFile *config, struct task *task);

/* 针对任务私有参数的清理，如果有，需要实现，否则置NULL */

void (*clear_task_params)(struct task *tk);

/* 启动任务的接口 */

int (*start_task)(struct engine *eng, struct task *tk);

/* 停止任务的接口 */

void (*stop_task)(struct engine *eng, struct task *tk);

/* 填充pid相关私有参数 */

int (*alloc_pid_params)(struct engine *eng, struct task_pid **tk_pid);

/* 销毁pid相关私有参数 */

void (*free_pid_params)(struct engine *eng, struct task_pid **tk_pid);

/* 第三方策略自身所需要的私有命令支持，如果没有，置为NULL */

int (*eng_mgt_func)(struct engine *eng, struct task *tk, char *cmd, int fd);

};
```

扫描模块对外接口说明

| 接口名称         |接口描述|
| ------------ | --------------------- |
| etmemd_scan_init | scan模块初始化|
| etmemd_scan_exit | scan模块析构|
| etmemd_get_vmas | 获取需要扫描的vma|
| etmemd_free_vmas | 释放etmemd_get_vmas扫描到的vma|
| etmemd_get_page_refs | 扫描vmas中的页面|
| etmemd_free_page_refs | 释放etmemd_get_page_refs获取到的页访问信息链表|

针对扫描虚拟机的场景中，在原扫描接口`idle_pages`中添加`ioctl`接口的方式，来提供区分扫描`ept`的粒度和是否忽略host上页访问标记的机制

针对进程指定页面换出的场景中，在原扫描接口`idle_pages`中添加`ioctl`命令字的形式，来确认不带有特定标记的vma不进行扫描和换出操作

扫描管理接口:

- 函数原型

    ```c
    ioctl(fd, cmd, void *arg);
    ```

- 输入参数

    ```text
    1. fd:文件描述符，通过open调用在/proc/pid/idle_pages下打开文件获得

    2.cmd:控制扫描行为，当前支持如下cmd：
    IDLE_SCAN_ADD_FLAG：新增一个扫描标记
    IDLE_SCAM_REMOVE_FLAGS：删除一个扫描标记
    VMA_SCAN_ADD_FLAGS:新增vma指定内存换出标记，仅扫描带有特定标记的VMA
    VMA_SCAN_REMOVE_FLAGS:删除新增的VMA指定内存换出标记

    3.args:int指针参数，传递具体标记掩码，当前仅支持如下参数：
    SCAN_AS_HUGE:扫描ept页表时，按照2M大页粒度扫描页是否被访问过。此标记未置位时，按照ept页表自身粒度扫描
    SCAN_IGN_HUGE：扫描虚拟机时，忽略host侧页表上的访问标记。此标记未置位时，不会忽略host侧页表上的访问标记。
    VMA_SCAN_FLAG：在etmem_scan.ko扫描模块开始扫描前，会调用接口walk_page_test接口判断vma地址是否符合扫描要求，此标记置位时，会仅扫描带有特定换出标记的vma地址段，而忽略其他vma地址
    ```

- 返回值

    ```text
    1. 成功，返回0
    2. 失败返回非0
    ```

- 注意事项

    ```text
    所有不支持的标记都会被忽略，但是不会返回错误
    ```

配置文件示例如下所示，具体含义请参考配置文件说明章节：

```text
#thirdparty
[engine]
name=thirdparty
project=test
eng_name=my_engine
libname=/user/lib/etmem_fetch/code_test/my_engine.so
ops_name=my_engine_ops
engine_private_key=engine_private_value
[task]
project=test
engine=my_engine
name=background1
type=pid
value=1798245
task_private_key=task_private_value
```

 **注意** ：

用户需使用etmem所提供的扫描模块动态库并实现对接etmem所需要的结构体中的接口

`eng_mgt_func`接口中的`fd`不能写入`0xff`和`0xfe`字

支持在一个工程内添加多个不同的第三方策略动态库，以配置文件中的`eng_name`来区分

### etmem客户端和服务端帮助说明

通过下列命令可以打印etmem服务端帮助说明

```bash
etmemd -h
```

或

```bash
etmemd --help
```

通过下列命令可以打印etmem客户端帮助说明

```bash
etmem help
```

通过下列命令可以打印etmem客户端操作工程/引擎/任务相关帮助说明

```bash
etmem obj help
```

通过下列命令可以打印etmem客户端对项目相关帮助说明

```bash
etmem project help
```

## 参与贡献

1. Fork本仓库
2. 新建个人分支
3. 提交代码
4. 新建Pull Request
