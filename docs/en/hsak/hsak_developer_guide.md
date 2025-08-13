# HSAK Developer Guide

## Overview

As the performance of storage media such as NVMe SSDs and SCMs is continuously improved, the latency overhead of the media layer in the I/O stack continuously reduces, and the overhead of the software stack becomes a bottleneck. Therefore, the kernel I/O data plane needs to be reconstructed to reduce the overhead of the software stack. HSAK provides a high-bandwidth and low-latency I/O software stack for new storage media, which reduces the overhead by more than 50% compared with the traditional I/O software stack.
The HSAK user-mode I/O engine is developed based on the open-source SPDK.

1. A unified interface is provided for external systems to shield the differences between open-source interfaces.
2. Enhanced I/O data plane features are added, such as DIF, drive formatting, batch I/O delivery, trim, and dynamic drive addition and deletion.
3. Features such as drive device management, drive I/O monitoring, and maintenance and test tools are provided.

## Compilation Tutorial

1. Download the HSAK source code.

    ```shell
    git clone <https://gitee.com/openeuler/hsak.git>
    ```

2. Install the compilation and running dependencies.

    The compilation and running of HSAK depend on components such as Storage Performance Development Kit (SPDK), Data Plane Development Kit (DPDK), and libboundscheck.

3. Start the compilation.

    ```shell
    cd hsak
    mkdir build
    cd build
    cmake ..
    make
    ```

## Precautions

### Constraints

- A maximum of 512 NVMe devices can be used and managed on the same machine.
- When HSAK is enabled to execute I/O-related services, ensure that the system has at least 500 MB continuous idle huge page memory.
- Before enabling the user-mode I/O component to execute services, ensure that the drive management component (Ublock) has been enabled.
- When the drive management component (Ublock) is enabled to execute services, ensure that the system has sufficient continuous idle memory. Each time the Ublock component is initialized, 20 MB huge page memory is allocated.
- Before HSAK is run, **setup.sh** is called to configure huge page memory and unbind the kernel-mode driver of the NVMe device.
- Other interfaces provided by the HSAK module can be used only after libstorage_init_module is successfully executed. Each process can call libstorage_init_module only once.
- After the libstorage_exit_module function is executed, other interfaces provided by HSAK cannot be used. In multi-thread scenarios, exit HSAK after all threads end.
- Only one service can be started for the HSAK Ublock component on a server and supports concurrent access of a maximum of 64 Ublock clients. The Ublock server can process a maximum of 20 client requests per second.
- The HSAK Ublock component must be started earlier than the data plane I/O component and Ublock clients. The command line tool provided by HSAK can be executed only after the Ublock server is started.
- Do not register the function for processing the SIGBUS signal. SPDK has an independent processing function for the signal. If the processing function is overwritten, the registered signal processing function becomes invalid and a core dump occurs.
