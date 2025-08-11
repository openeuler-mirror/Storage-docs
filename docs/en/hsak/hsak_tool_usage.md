# Command-Line Interface

## Command for Querying Drive Information

### Format

```shell
libstorage-list [<commands>] [<device>]
```

### Parameters

- *commands*: Only **help** is available. **libstorage-list help** is used to display the help information.

- *device*: specifies the PCI address. The format is **0000:09:00.0**. Multiple PCI addresses are allowed and separated by spaces. If no specific PCI address is set, the command line lists all enumerated device information.

### Precautions

- The fault injection function applies only to development, debugging, and test scenarios. Do not use this function on live networks. Otherwise, service and security risks may occur.

- Before running this command, ensure that the management component (Ublock) server has been started, and the user-mode I/O component (UIO) has not been started or has been correctly started.

- Drives that are not occupied by the Ublock or UIO component will be occupied during the command execution. If the Ublock or UIO component attempts to obtain the drive control permission, a storage device access conflict may occur. As a result, the command execution fails.

## Command for Switching Drivers for Drives

### Format

```shell
libstorage-shutdown reset <device> [<device2> ...]
```

### Parameters

- **reset**: switches the UIO driver to the kernel-mode driver for a specific drive.

- *device*: specifies the PCI address, for example, **0000:09:00.0**. Multiple PCI addresses are allowed and separated by spaces.

### Precautions

- The **libstorage-shutdown reset** command is used to switch a drive from the user-mode UIO driver to the kernel-mode NVMe driver.

- Before running this command, ensure that the Ublock server has been started, and the UIO component has not been started or has been correctly started.

- The **libstoage-shutdown reset** command is risky. Before switching to the NVMe driver, ensure that the user-mode instance has stopped delivering I/Os to the NVMe device, all FDs on the NVMe device have been disabled, and the instance that accesses the NVMe device has exited.

## Command for Obtaining I/O Statistics

### Format

```shell
libstorage-iostat [-t <interval>] [-i <count>] [-d <device1,device2,...>]
```

### Parameters

- -**t**: interval, in seconds. The value ranges from 1 to 3600. This parameter is of the int type. If the input parameter value exceeds the upper limit of the int type, the value is truncated to a negative or positive number.

- -**i**: number of collection times. The minimum value is **1** and the maximum value is *MAX_INT*. If this parameter is not set, information is collected at an interval by default. This parameter is of the int type. If the input parameter value exceeds the upper limit of the int type, the value is truncated to a negative or positive number.

- -**d**: name of a block device (for example, **nvme0n1**, which depends on the controller name configured in **/etc/spdk/nvme.conf.in**). You can use this parameter to collect performance data of one or more specified devices. If this parameter is not set, performance data of all detected devices is collected.

### Precautions

- The I/O statistics configuration is enabled.

- The process has delivered I/O operations to the drive whose performance information needs to be queried through the UIO component.

- If no device in the current environment is occupied by service processes to deliver I/Os, the command exits after the message "You cannot get iostat info for nvme device no deliver io" is displayed.

- When multiple queues are enabled on a drive, the I/O statistics tool summarizes the performance data of multiple queues on the drive and outputs the data in a unified manner.

- The I/O statistics tool supports data records of a maximum of 8192 drive queues.

- The I/O statistics are as follows:

    | Device      | r/s                            | w/s                             | rKB/s                               | wKB/s                                | avgrq-sz                               | avgqu-sz                   | r_await               | w_await                | await                           | svctm                                   | util%              | poll-n                     |
    | ----------- | ------------------------------ | ------------------------------- | ----------------------------------- | ------------------------------------ | -------------------------------------- | -------------------------- | --------------------- | ---------------------- | ------------------------------- | --------------------------------------- | ------------------ | -------------------------- |
    | Device name | Number of read I/Os per second | Number of write I/Os per second | Number of read I/O bytes per second | Number of write I/O bytes per second | Average size of delivered I/Os (bytes) | I/O depth of a drive queue | I/O read latency (μs) | I/O write latency (μs) | Average read/write latency (μs) | Processing latency of a single I/O (μs) | Device utilization | Number of polling timeouts |

## Commands for Drive Read/Write Operations

### Format

```shell
libstorage-rw <COMMAND> <device> [OPTIONS...]
```

### Parameters

1. **COMMAND** parameters

    - **read**: reads a specified logical block from the device to the data buffer (standard output by default).

    - **write**: writes data in a data buffer (standard input by default) to a specified logical block of the NVMe device.

    - **help**: displays the help information about the command line.

2. **device**: specifies the PCI address, for example, **0000:09:00.0**.

3. **OPTIONS** parameters

    - **--start-block, -s**: indicates the 64-bit start address of the logical block to be read or written. The default value is **0**.

    - **--block-count, -c**: indicates the number of the logical blocks to be read or written (counted from 0).

    - **--data-size, -z**: indicates the number of bytes of the data to be read or written.

    - **--namespace-id, -n**: indicates the namespace ID of the device. The default value is **1**.

    - **--data, -d**: indicates the data file used for read and write operations (The read data is saved during read operations and the written data is provided during write operations.)

    - **--limited-retry, -l**: indicates that the device controller restarts for a limited number of times to complete device read and write operations.

    - **--force-unit-access, -f**: ensures that read and write operations are completed from the nonvolatile media before the instruction is completed.

    - **--show-command, -v**: displays instruction information before sending a read/write command.

    - **--dry-run, -w**: displays only information about read and write instructions but does not perform actual read and write operations.

    - **--latency. -t**: collects statistics on the end-to-end read and write latency of the CLI.

    - **--help, -h**: displays the help information about related commands.
