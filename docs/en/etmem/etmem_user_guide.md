# etmem User Guide

## Introduction

The development of CPU computing power, particularly lower costs of ARM cores, makes memory cost and capacity become the core frustration that restricts business costs and performance. Therefore, the most pressing issue is how to save memory cost and how to expand memory capacity.

etmem is a tiered memory expansion technology that uses DRAM+memory compression/high-performance storage media to form tiered memory storage. Memory data is tiered, and cold data is migrated from memory media to high-performance storage media to release memory space and reduce memory costs.

The tools provided by the etmem software package include the etmem client and the etmemd server. etmemd runs continuously after being launched and implements functions such as recognition and elimination of cold and hot memory of target processes. etmem runs once when called and controls etmemd to respond with different operations based on different command parameters.

## Compilation

1. Download the etmem source code.

    ```shell
    git clone https://gitee.com/openeuler/etmem.git
    ```

2. Install the compilation and running dependency. The compilation and running of etmem depend on the libboundscheck component.

    Install the dependency:

    ```bash
    yum install libboundscheck
    ```

    Use the `rpm` command to check if the package is installed:

    ```bash
    rpm -qi libboundscheck
    ```

3. Build source code.

    ```bash
    cd etmem
    mkdir build
    cd build
    cmake ..
    make
    ```

## Precautions

### Dependencies

As a memory expansion tool, etmem needs to rely on kernel features. To identify memory access conditions and support the active writing of memory into the swap partition to achieve the requirement of vertical memory expansion, etmem needs to insert the **etmem_scan** and **etmem_swap** modules at runtime:

```bash
modprobe etmem_scan
modprobe etmem_swap
```

### Restrictions

The etmem process requires root privileges. The root user has the highest system privileges. When using the root user to perform operations, strictly follow the operation instructions to avoid system management and security risks.

### Constraints

- The client and server of etmem must be deployed on the same server. Cross-server communication is not supported.
- etmem can scan target processes whose process name is less than or equal to 15 characters. Supported characters in process names are letters, numbers, periods (.), slashes (/), hyphens (-), and underscores (\_).
- When AEP media is used for memory expansion, it relies on the system being able to correctly recognize the AEP device and initialize the device as a NUMA node. Additionally, the **vm_flags** field in the configuration file can only be configured as **ht**.
- The private commands of the engine are only valid for the corresponding engine and tasks under the engine, such as `showhostpages` and `showtaskpages` supported by cslide.
- In a third-party policy implementations, **fd** in the `eng_mgt_func` interface cannot be written with the **0xff** and **0xfe** characters.
- Multiple different third-party policy dynamic libraries, distinguished by **eng_name** in the configuration file, can be added within a project.
- Concurrent scanning of the same process is prohibited.
- Using the **/proc/xxx/idle_pages** and **/proc/xxx/swap_pages** files is prohibited when **etmem_scan** and **etmem_swap** modules are not loaded.
- The etmem configuration file requires the owner to be the root user, with permissions of 600 or 400. The size of the configuration file cannot exceed 10 MB.
- When etmem injects a third-party policy, the **so** of the third-party policy requires the owner to be the root user, with permissions of 500 or 700.

## Instructions

### etmem Configuration Files

Before running the etmem process, the administrator needs to decide the memory of which processes needs to be expanded, configure the process information in the etmem configuration files, and configure information such as the memory scanning cycle, scanning times, and  memory hot and cold thresholds.

The configuration file examples are included in the source package and stored in the **/etc/etmem** directory. There are three example files:

```text
/etc/etmem/cslide_conf.yaml
/etc/etmem/slide_conf.yaml
/etc/etmem/thirdparty_conf.yaml
```

Contents of the files are as follows:

```sh
#slide engine example
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

#cslide engine example
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

#Third-party engine example
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

Fields in the configuration files are described as follows:

| Item                 | Description                                                                                                                                                                        | Mandatory                             | Contains Parameters | Parameter Range                                                                                                      | Example                                                                                                                                                                                                                                                                                                           |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| \[project\]          | Beginning identifier of the project public configuration section                                                                                                                   | No                                    | No                  | N/A                                                                                                                  | Beginning identifier of the project parameters, indicating that the parameters below are within the range of the project section until another \[xxx\] or the end of the file                                                                                                                                     |
| name                 | Name of the project                                                                                                                                                                | Yes                                   | Yes                 | String of up to 64 characters                                                                                        | Specifies that the project, engine and task need to be mounted to the specified project during configuration.                                                                                                                                                                                                     |
| loop                 | Number of loops for memory scan                                                                                                                                                    | Yes                                   | Yes                 | 1~120                                                                                                                | loop=3 // Memory is scanned 3 times.                                                                                                                                                                                                                                                                              |
| interval             | Time interval for each memory scan                                                                                                                                                 | Yes                                   | Yes                 | 1~1200                                                                                                               | interval=5 // The interval is 5s.                                                                                                                                                                                                                                                                                 |
| sleep                | Time interval for each memory scan+operation                                                                                                                                       | Yes                                   | Yes                 | 1~1200                                                                                                               | sleep=10 //The interval is 10s                                                                                                                                                                                                                                                                                    |
| sysmem_threshold     | Memory swapping threshold. This is a slide engine configuration item.                                                                                                              | No                                    | Yes                 | 0~100                                                                                                                | sysmem_threshold=50 // When available memory is less than 50%, etmem swaps out memory.                                                                                                                                                                                                                            |
| swapcache_high_wmark | High watermark of swapcache. This is a slide engine configuration item.                                                                                                            | No                                    | Yes                 | 1~100                                                                                                                | swapcache_high_wmark=5 // swapcache can be up to 5% of the system memory. If this ratio is reached, etmem triggers swapcache recycling.<br> Note: swapcache_high_wmark must be greater than swapcache_low_wmark.                                                                                                 |
| swapcache_low_wmark  | Low watermark of swapcache. This is a slide engine configuration item.                                                                                                             | No                                    | Yes                 | \[1~swapcache_high_wmark\)                                                                                           | swapcache_low_wmark=3 //When swapcache recycling is triggered, the system recycles the swapcache memory occupancy to less than 3%.                                                                                                                                                                                |
| \[engine\]           | Beginning identifier of the engine public configuration section                                                                                                                    | No                                    | No                  | N/A                                                                                                                  | Beginning identifier of the engine parameters, indicating that the parameters below are within the range of the engine section until another \[xxx\] or the end of the file                                                                                                                                       |
| project              | project to which the engine belongs                                                                                                                                                | Yes                                   | Yes                 | String of up to 64 characters                                                                                        | If a project named test exists, the item can be **project=test**.                                                                                                                                                                                                                                                 |
| engine               | engine to which the engine belongs                                                                                                                                                 | Yes                                   | Yes                 | slide/cslide/thirdparty                                                                                              | Specifies the policy to use (**slide**, **cslide**, or **thirdparty**)                                                                                                                                                                                                                                            |
| node_pair            | Node pair of AEP and DRAM. This is a cslide engine configuration item.                                                                                                             | Yes when **engine** is **cslide**     | Yes                 | Pair the node numbers of AEP and DRAM. Separate AEP and DRAM using a comma, and separate each pair using semicolons. | node_pair=2,0;3,1                                                                                                                                                                                                                                                                                                 |
| hot_threshold        | Threshold of hot memory watermark. This is a cslide engine configuration item.                                                                                                     | Yes when **engine** is **cslide**     | Yes                 | An integer greater than or equal to 0 and less than or equal to INT_MAX                                              | hot_threshold=3 // Memory with less than 3 accesses will be recognized as cold memory.                                                                                                                                                                                                                            |
| node_mig_quota       | Maximum one-way flow when DRAM and AEP migrate mutually. This is a cslide engine configuration item.                                                                               | Yes when **engine** is **cslide**     | Yes                 | An integer greater than or equal to 0 and less than or equal to INT_MAX                                              | node_mig_quota=1024 // The unit is MB. A maximum of 1024 MB can be migrated from AEP to DRAM or from DRAM to AEP each time.                                                                                                                                                                                       |
| node_hot_reserve     | Size of the reserved space for hot memory in DRAM. This is a cslide engine configuration item.                                                                                     | Yes when **engine** is **cslide**     | Yes                 | An integer greater than or equal to 0 and less than or equal to INT_MAX                                              | node_hot_reserve=1024 //The unit is MB. When the hot memory of all VMs is greater than this configuration value, the hot memory will also be migrated to AEP.                                                                                                                                                     |
| eng_name             | Name of the engine for mounting by task. This is a third-party engine configuration item.                                                                                          | Yes when **engine** is **thirdparty** | Yes                 | String of up to 64 characters                                                                                        | eng_name=my_engine // When mounting a task to the third-party policy engine, specify **engine=my_engine** in the task.                                                                                                                                                                                            |
| libname              | Absolute path to the dynamic library of the third-party policy. This is a third-party engine configuration item.                                                                   | Yes when **engine** is **thirdparty** | Yes                 | String of up to 256 characters                                                                                       | libname=/user/lib/etmem_fetch/code_test/my_engine.so                                                                                                                                                                                                                                                              |
| ops_name             | Name of the operator in the dynamic library of the third-party policy. This is a third-party engine configuration item.                                                            | Yes when **engine** is **thirdparty** | Yes                 | String of up to 256 characters                                                                                       | ops_name=my_engine_ops // Name of the struct for the third-party policy implementation interface                                                                                                                                                                                                                  |
| engine_private_key   | Reserved item for third-party policies to parse private parameters by themselves. This is a third-party engine configuration item.                                                 | No                                    | No                  | Restrict according to the third-party policy's private parameters.                                                   | Configure the private engine parameters according to the third-party policy.                                                                                                                                                                                                                                      |
| \[task\]             | Beginning identifier of the task public configuration section                                                                                                                      | No                                    | No                  | N/A                                                                                                                  | Beginning identifier of the task parameters, indicating that the parameters below are within the range of the project section until another \[xxx\] or the end of the file                                                                                                                                        |
| project              | project to which the task belongs                                                                                                                                                  | Yes                                   | Yes                 | String of up to 64 characters                                                                                        | If a project named test exists, the item can be **project=test**.                                                                                                                                                                                                                                                 |
| engine               | engine to which the task belongs                                                                                                                                                   | Yes                                   | Yes                 | String of up to 64 characters                                                                                        | Name of the engine to which the task belongs                                                                                                                                                                                                                                                                      |
| name                 | Name of the task                                                                                                                                                                   | Yes                                   | Yes                 | String of up to 64 characters                                                                                        | name=background1 // The name of the task is background1.                                                                                                                                                                                                                                                          |
| type                 | How the target process is identified                                                                                                                                               | Yes                                   | Yes                 | pid/name                                                                                                             | **pid** specifies to identify by PID. **name** specifies to identify by name.                                                                                                                                                                                                                                     |
| value                | Value to be identified for the target process                                                                                                                                      | Yes                                   | Yes                 | Actual PID/name                                                                                                      | Used with **type** to specify the PID or name of the target process. Ensure the configuration is correct and unique.                                                                                                                                                                                              |
| T                    | Threshold of hot memory watermark. This is a slide engine configuration item.                                                                                                      | Yes when **engine** is **slide**      | Yes                 | 0~loop * 3                                                                                                           | T=3 // Memory with less than 3 accesses will be recognized as cold memory.                                                                                                                                                                                                                                        |
| max_threads          | Maximum number of threads in the etmem internal thread pool, with each thread handling a process/subprocess memory scan+operation task. This is a slide engine configuration item. | No                                    | Yes                 | 1~2 * number of cores + 1, the default value is 1.                                                                   | Controls the number of internal processing threads for the etmemd server without external representation. When the target process has multiple child processes, the larger the item value, the more concurrent executions, but the more resources consumed.                                                       |
| vm_flags             | Flag of the VMA to be scanned. This is a cslide engine configuration item.                                                                                                         | No                                    | Yes                 | String of up to 256 characters, with different flags separated by spaces.                                            | vm_flags=ht // Scans memory of the VMA whose flag is ht.                                                                                                                                                                                                                                                          |
| anon_only            | Scans anonymous pages only. This is a cslide engine configuration item.                                                                                                            | No                                    | Yes                 | yes/no                                                                                                               | anon_only=no                                                                                                                                                                                                                                                                                                      |
| ign_host             | Ignores page table scan information on the host. This is a cslide engine configuration item.                                                                                       | No                                    | Yes                 | yes/no                                                                                                               | ign_host=no                                                                                                                                                                                                                                                                                                       |
| task_private_key     | Reserved for a task of a third-party policy to parse private parameters. This is a third-party engine configuration item.                                                          | No                                    | No                  | Restrict according to the third-party policy's private parameters.                                                   | Configure the private task parameters according to the third-party policy.                                                                                                                                                                                                                                        |
| swap_threshold       | Process memory swapping threshold. This is a slide engine configuration item.                                                                                                      | No                                    | Yes                 | Absolute value of memory available to the process                                                                    | swap_threshold=10g // Memory swapping will not be triggered when the process memory is less than 10 GB.<br> Currently, the unit can only be **g** or **G**. This item is used with **sysmem_threshold**. When system memory is lower than **sysmem_threshold**, memory of processes in the allowlist is checked. |
| swap_flag            | Enables process memory swapping. This is a slide engine configuration item.                                                                                                        | No                                    | Yes                 | yes/no                                                                                                               | swap_flag=yes                                                                                                                                                                                                                                                                                                     |

### Starting etmemd

Modify related configuration files before using etmem services. After being started, etmemd stays in the system to operate the memory of the target processes.To start etmemd, you can either run the `etmemd` command or configure a service file for `systemctl` to start etmemd. The latter requires the `mode-systemctl` option.

#### How to Use

Run the following command to start etmemd:

```bash
etmemd -l 0 -s etmemd_socket
```

or

```bash
etmemd --log-level 0 --socket etmemd_socket
```

The `0` parameter of option `-l` and the `etmemd_socket` parameter of option `-s` are user-defined parameters and are described as follows.

#### Command Parameters

| Option            | Description                           | Mandatory |  Contains Parameters | Parameter Range       | Example                                                            |
| --------------- | ---------------------------------- | -------- | ---------- | --------------------- | ------------------------------------------------------------ |
| -l or \-\-log-level | etmemd log level                      | No       | Yes         | 0~3                   | 0: debug level <br>  1: info level <br>  2: warning level <br>  3: error level  <br> Logs whose levels are higher than the specified value are printed to **/var/log/message**. |
| -s or \-\-socket    | Socket listened by etmemd to interact with the client | Yes       | Yes         | String of up to 107 characters | Socket listened by etmemd                                         |
| -m or \-\-mode-systemctl| Starts the etmemd service through systemctl |   No| No| N/A|    The `-m` option needs to be specified in the service file.|
| -h or \-\-help      | Prints help information                           | No       | No         | N/A                    | This option prints help information and exit.                                 |

### Adding and Deleting Projects, Engines, and Tasks Using the etmem Client

#### Scenario

1. The administrator adds a project, engine, or task to etmem (a project can contain multiple etmem engines, an engine can contain multiple tasks).

2. The administrator deletes an existing etmem project, engine, or task (all tasks in a project is stopped before the project is deleted).

#### Usage

When etmemd is running normally, run `etmem` with the `obj` option to perform addition and deletion. etmem automatically identifies projects, engines, or tasks according to the content of the configuration file.

- Add an object.

    ```bash
    etmem obj add -f /etc/etmem/slide_conf.yaml -s etmemd_socket
    ```

    or

    ```bash
    etmem obj add --file /etc/etmem/slide_conf.yaml --socket etmemd_socket
    ```

- Delete an object.

    ```bash
    etmem obj del -f /etc/etmem/slide_conf.yaml -s etmemd_socket
    ```

    or

    ```bash
    etmem obj del --file /etc/etmem/slide_conf.yaml --socket etmemd_socket
    ```

#### Command Parameters

| Option           | Description                                                                                                    | Mandatory | Contains Parameters | Parameter Range                                                                                       | Example |
| ---------------- | -------------------------------------------------------------------------------------------------------------- | --------- | ------------------- | ----------------------------------------------------------------------------------------------------- | ------- |
| -f or \-\-file   | Specifies the configuration file of the object.                                                                | Yes       | Yes                 | Specify the path.                                                                                     |         |
| -s or \-\-socket | Socket used for communication with etmemd, which must be the same as the one specified when etmemd is started. | Yes       | Yes                 | The administrator can use this option to specify an etmemd server when multiple etmemd servers exist. |         |

### Querying, Starting, and Stopping Projects Using the etmem Client

#### Scenario

A project is added by using `etmem obj add` and is not deleted by using `etmem obj del`. In this case,  the project can be started and stopped.

1. The administrator starts an added project.

2. The administrator stops a started project.

A started project will be stopped if the administrator run `obj del` to delete the project.

#### Usage

Added projects can be started and stopped by using `etmem project` commands.

- Query a project.

    ```bash
    etmem project show -n test -s etmemd_socket
    ```

    or

    ```bash
    etmem project show --name test --socket etmemd_socket
    ```

- Start a project.

    ```bash
    etmem project start -n test -s etmemd_socket
    ```

    or

    ```bash
    etmem project start --name test --socket etmemd_socket
    ```

- Stop a project.

    ```bash
    etmem project stop -n test -s etmemd_socket
    ```

    or

    ```bash
    etmem project stop --name test --socket etmemd_socket
    ```

- Print help information.

    ```bash
    etmem project help
    ```

#### Command Parameters

| Option           | Description                                                                                                    | Mandatory | Contains Parameters | Parameter Range                                                                                       | Example |
| ---------------- | -------------------------------------------------------------------------------------------------------------- | --------- | ------------------- | ----------------------------------------------------------------------------------------------------- | ------- |
| -n or \-\-name   | Name of the project                                                                                            | Yes       | Yes                 | The project name corresponds to the configuration file.                                               |         |
| -s or \-\-socket | Socket used for communication with etmemd, which must be the same as the one specified when etmemd is started. | Yes       | Yes                 | The administrator can use this option to specify an etmemd server when multiple etmemd servers exist. |         |

### Specifying System Memory Swapping Threshold and Process Memory Swapping Using the etmem Client

Only slide policies support private features.

- Process or system memory swapping threshold

It is necessary to consider the timing of etmem memory swapping for optimal performance. Memory swapping is not performed when the system has enough available memory or a process occupies a low amount of memory. Memory swapping threshold can be specified for the system and processes.

- Process memory swapping

The memory of I/O latency-sensitive service processes should not be swapped in the storage scenario. In this case, you can disable memory swapping for certain services.

Process and system memory swapping thresholds and process memory swapping are controlled by the **sysmem_threshold**, **swap_threshold**, and **swap_flag** parameters in the configuration file. For details, see [etmem Configuration Files](#etmem-configuration-files).

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

#### System Memory Swapping Threshold

The **sysmem_threshold** parameter is used to set system memory swapping threshold. The value range for **sysmem_threshold** is 0 to 100. If **sysmem_threshold** is set in the configuration file, etmem will swap memory when system memory is lower than **sysmem_threshold**.

For example:

1. Compose the configuration according to the example. Set **sysmem_threshold** to **20**.
2. Start the server, add a project to the server, and start the project.

    ```bash
    etmemd -l 0 -s monitor_app &
    etmem obj add -f etmem_config -s monitor_app
    etmem project start -n test -s monitor_app
    etmem project show -s monitor_app
    ```

3. Observe the memory swapping results. etmem swaps memory only when the system available memory is less than 20%.

#### Process Memory Swapping Threshold

The **swap_threshold** parameter is used to set process memory swapping threshold. **swap_threshold** is the absolute memory usage of a process in the format of \<number\>**g/G**. If **swap_threshold** is set in the configuration file, etmem will not swap memory of the process when the process memory usage is lower then **swap_threshold**.

For example:

1. Compose the configuration according to the example. Set **swap_threshold** to **5g**.
2. Start the server, add a project to the server, and start the project.

    ```bash
    etmemd -l 0 -s monitor_app &
    etmem obj add -f etmem_config -s monitor_app
    etmem project start -n test -s monitor_app
    etmem project show -s monitor_app
    ```

3. Observe the memory swapping results. etmem swaps memory only when the process memory usage reaches 5 GB.

#### Process Memory Swapping

The **swap_flag** parameter is used to enable the process memory swapping feature. The value of **swap_flag** can be **yes** or **no**. If **swap_flag** is **no** or not configured, etmem swaps memory normally. If **swap_flag** is **yes**, etmem swaps memory of the specified processes only.

For example:

1. Compose the configuration according to the example. Set **swap_flag** to **yes**.
2. Flag the memory to be swapped for the service process.

    ```bash
    madvise(addr_start, addr_len, MADV_SWAPFLAG)
    ```

3. Start the server, add a project to the server, and start the project.

    ```bash
    etmemd -l 0 -s monitor_app &
    etmem obj add -f etmem_config -s monitor_app
    etmem project start -n test -s monitor_app
    etmem project show -s monitor_app
    ```

4. Observe the memory swapping results. Only the flagged memory is swapped. Other memory is retained in the DRAM.

In the process memory page swapping scenario, `ioctl` is added to the original scan interface file **idle_pages** to ensure that VMAs that are not flagged do not participate in memory scanning and swapping.

Scan management interface:

- Function prototype

    ```c
    ioctl(fd, cmd, void *arg);
    ```

- Input parameters
    1. fd: file descriptor, which is obtained by opening a file under /proc/pid/idle_pages using the open system call
    2. cmd: controls the scan actions. The following values are supported:
    VMA_SCAN_ADD_FLAGS: adds VMA memory swapping flags to scan only flagged VMAs
    VMA_SCAN_REMOVE_FLAGS: removes added VMA memory swapping flags
    3. args: integer pointer parameter used to pass a specific mask. The following value is supported:
    VMA_SCAN_FLAG: Before the etmem_scan.ko module starts scanning, the walk_page_test interface is called to determine whether the VMA address meets the scanning requirements. If this flag is set, only the VMA addresses that contain specific swap flags are scanned.

- Return values
    1. 0 if the command succeeds
    2. Other values if the command fails

- Precautions
    Unsupported flags are ignored and do not return errors.

### Specifying swapcache Memory Recycling Instructions Using the etmem Client

The user-mode etmem initiates a memory elimination and recycling operation and interacts with the kernel-mode memory recycling module through the **write procfs** interface. The memory recycling module parses the virtual address sent from the user space, obtains the page corresponding to the address, and calls the native kernel interface to swap and recycle the memory corresponding to the page. During memory swapping, swapcache will use some system memory. To further save memory, the swapcache memory recycling feature is added.

Add **swapcache_high_wmark** and **swapcache_low_wmark** parameters to use the swapcache memory recycling feature.

- **swapcache_high_wmark**: High system memory water of swapcache
- **swapcache_low_wmark**: Low system memory water of swapcache

After etmem swaps memory, it checks the swapcache memory occupancy. When the occupancy exceeds the high watermark, an `ioctl` instruction will be issued through **swap_pages** to trigger the swapcache memory recycling and stop when swapcache memory occupancy reaches the low watermark.

An example configuration file is as follows. For details, see [etmem Configuration Files](#etmem-configuration-files).

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

During memory swapping, swapcache memory needs to be recycled to further save memory. An `ioctl` interface is added to the original memory swap interface to configure swapcache watermarks and swapcache memory recycling.

- Function prototype

    ```c
    ioctl(fd, cmd, void *arg);
    ```

- Input parameters
    1. fd: file descriptor, which is obtained by opening a file under /proc/pid/idle_pages using the open system call
    2. cmd: controls the scan actions. The following values are supported:
    RECLAIM_SWAPCACHE_ON: enables swapcache memory swapping
    RECLAIM_SWAPCACHE_OFF: disables swapcache memory swapping
    SET_SWAPCACHE_WMARK: configures swapcache memory watermarks
    3. args: integer pointer parameter used to pass a specific mask. The following value is supported:
    Parameters that pass the values of swapcache watermarks

- Return values
    1. 0 if the command succeeds
    2. Other values if the command fails

- Precautions
    Unsupported flags are ignored and do not return errors.

### Executing Private Commands and Functions Using the etmem Client

Only the cslide policy support private commands.

- `showtaskpages`
- `showhostpages`

For engines and tasks of engines that use the cslide policy, you can run the commands above to query the page access of tasks and the usage of system huge pages on the host of VMs.

For example:

```bash
etmem engine showtaskpages <-t task_name> -n proj_name -e cslide -s etmemd_socket

etmem engine showhostpages -n proj_name -e cslide -s etmemd_socket
```

**Note**: `showtaskpages` and `showhostpages` are supported by the cslide policy only.

#### Command Parameters

| Option              | Description                                                                                                    | Mandatory | Contains Parameters | Parameter Range                                                                                       | Example |
| ------------------- | -------------------------------------------------------------------------------------------------------------- | --------- | ------------------- | ----------------------------------------------------------------------------------------------------- | ------- |
| -n or \-\-proj_name | Name of the project                                                                                            | Yes       | Yes                 | Name of an existing project to run                                                                    |         |
| -s or \-\-socket    | Socket used for communication with etmemd, which must be the same as the one specified when etmemd is started. | Yes       | Yes                 | The administrator can use this option to specify an etmemd server when multiple etmemd servers exist. |         |
| -e or \-\-engine    | Name of the engine to run                                                                                      | Yes       | Yes                 | Name of an existing engine to run                                                                     |         |
| -t or \-\-task_name | Name of the task to run                                                                                        | No        | Yes                 | Name of an existing task to run                                                                       |         |

### Enabling and Disabling Kernel Swap

When etmem swaps memory to the drive to expand memory, you can choose to enable the kernel swap feature. You can disable the native kernel swap mechanism to void swapping memory undesirably, resulting in problems with user-mode processes.

A sys interface is provided to implement such control. A **kobj** named **kernel_swap_enable** is created in **/sys/kernel/mm/swap** to enable and disable kerne swap. The default value of **kernel_swap_enable** is **true**.

For example: 

```sh
# Enable kernel swap
echo true > /sys/kernel/mm/swap/kernel_swap_enable
or
echo 1 > /sys/kernel/mm/swap/kernel_swap_enable

# Disable kernel swap
echo false > /sys/kernel/mm/swap/kernel_swap_enable
or
echo 0 > /sys/kernel/mm/swap/kernel_swap_enable

```

### Starting etmem Upon System Startup

#### Scenario

You can configure the systemd configuration file to run etmemd as a forking service of systemd.

#### Usage

Compose a service configuration file to start etmemd with the `-m` option. For example:

```bash
etmemd -l 0 -s etmemd_socket -m
```

#### Command Parameters

| Option            | Description                           | Mandatory |  Contains Parameters | Parameter Range       | Example                                                            |
| --------------- | ---------------------------------- | -------- | ---------- | --------------------- | ------------------------------------------------------------ |
| -l or \-\-log-level | etmemd log level                      | No       | Yes         | 0~3                   | 0: debug level <br>  1: info level <br>  2: warning level <br>  3: error level  <br> Logs whose levels are higher than the specified value are printed to **/var/log/message**. |
| -s or \-\-socket    | Socket listened by etmemd to interact with the client | Yes       | Yes         | String of up to 107 characters | Socket listened by etmemd                                         |
| -m or \-\-mode-systemctl| Starts the etmemd service through systemctl |   No| No| N/A|    The `-m` option needs to be specified in the service file.|
| -h or \-\-help      | Prints help information                           | No       | No         | N/A                    | This option prints help information and exit.                                 |

### Supporting Third-party Memory Expansion Policies With etmem

#### Scenario

etmem provides third-party memory expansion policy registration and module scanning dynamic library and can eliminate memory according to third-party policies.

You can use the module scanning dynamic library to implement the interface of the struct required for connecting to etmem.

#### Usage

To use a third-party memory expansion elimination policy, perform the following steps:

1. Invoke the scanning interface of the module as required.

2. Implement the interfaces using the function template provided by the etmem header file and encapsulate them into a struct.

3. Build a dynamic library of the third-party memory expansion elimination policy.

4. Specify the **thirdparty** engine in the configuration file.

5. Enter the names of the library and the interface struct to the corresponding **task** fields in the configuration file.

Other steps are similar to those of using other engines.

Interface struct template:

```c
struct engine_ops {

/* Parsing private parameters of the engine. Implement the interface if required, otherwise, set it to NULL. */

int (*fill_eng_params)(GKeyFile *config, struct engine *eng);

/* Clearing private parameters of the engine. Implement the interface if required, otherwise, set it to NULL. */

void (*clear_eng_params)(struct engine *eng);

/* Parsing private parameters of the task. Implement the interface if required, otherwise, set it to NULL. */

int (*fill_task_params)(GKeyFile *config, struct task *task);

/* Clearing private parameters of the task. Implement the interface if required, otherwise, set it to NULL. */

void (*clear_task_params)(struct task *tk);

/* Task starting interface */

int (*start_task)(struct engine *eng, struct task *tk);

/* Task stopping interface */

void (*stop_task)(struct engine *eng, struct task *tk);

/* Allocate PID-related private parameters */

int (*alloc_pid_params)(struct engine *eng, struct task_pid **tk_pid);

/* Destroy PID-related private parameters */

void (*free_pid_params)(struct engine *eng, struct task_pid **tk_pid);

/* Support for private commands required by the third-party policy. If this interface is not required, set it to NULL */

int (*eng_mgt_func)(struct engine *eng, struct task *tk, char *cmd, int fd);

};
```

External interfaces of the scanning module:

| Interface         |Description|
| ------------ | --------------------- |
| etmemd_scan_init | Initializes the scanning module|
| etmemd_scan_exit | Exits the scanning module|
| etmemd_get_vmas | Gets the VMAs to be scanned|
| etmemd_free_vmas | Releases VMAs scanned by `etmemd_get_vmas`|
| etmemd_get_page_refs | Scans pages in VMAs|
| etmemd_free_page_refs | Release the page access information list obtained by `etmemd_get_page_refs` |

In the VM scanning scenario, `ioctl` is added to the original scan interface file **idle_pages** to distinguish the EPT scanning granularity and specify whether to ignore page access flags on the hosts.

In the process memory page swapping scenario, `ioctl` is added to the original scan interface file **idle_pages** to ensure that VMAs that are not flagged do not participate in memory scanning and swapping.

Scan management interface:

- Function prototype

    ```c
    ioctl(fd, cmd, void *arg);
    ```

- Input parameters
    1. fd: file descriptor, which is obtained by opening a file under /proc/pid/idle_pages using the open system call
    2. cmd: controls the scan actions. The following values are supported:
    IDLE_SCAN_ADD_FLAG: adds a scanning flag
    IDLE_SCAM_REMOVE_FLAGS: removes a scanning flag
    VMA_SCAN_ADD_FLAGS: adds VMA memory swapping flags to scan only flagged VMAs
    VMA_SCAN_REMOVE_FLAGS: removes added VMA memory swapping flags
    3. args: integer pointer parameter used to pass a specific mask. The following value is supported:
    SCAN_AS_HUGE: scans the pages according to the 2 MB granularity to see if the pages have been accessed when scanning the EPT page table. If this parameter is not set, the granularity will be the granularity of the EPT page table itself.
    SCAN_IGN_HUGE: ignores page access flags on the hosts when scanning VMs.
    VMA_SCAN_FLAG: Before the etmem_scan.ko module starts scanning, the walk_page_test interface is called to determine whether the VMA address meets the scanning requirements. If this flag is set, only the VMA addresses that contain specific swap flags are scanned.

- Return values
    1. 0 if the command succeeds
    2. Other values if the command fails

- Precautions
    Unsupported flags are ignored and do not return errors.

An example configuration file is as follows. For details, see [etmem Configuration Files](#etmem-configuration-files).

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

 **Note**:

You need to use the module scanning dynamic library to implement the interface of the struct required for connecting to etmem.

**fd** in the `eng_mgt_func` interface cannot be written with the **0xff** and **0xfe** characters.

Multiple different third-party policy dynamic libraries, distinguished by **eng_name** in the configuration file, can be added within a project.

### Help Information of the etmem Client and Server

Run the following command to print help information of the etmem server:

```bash
etmemd -h
```

or:

```bash
etmemd --help
```

Run the following command to print help information of the etmem client:

```bash
etmem help
```

Run the following command to print help information of project, engine, and task operations:

```bash
etmem obj help
```

Run the following command to print help information of projects:

```bash
etmem project help
```

## How to Contribute

1. Fork this repository.
2. Create a branch.
3. Commit your code.
4. Create a pull request (PR).
