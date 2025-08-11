# Installation and Deployment

This section describes how to install and deploy GMEM.

## Software and Hardware Requirements

* Kunpeng 920 CPU
* Ascend 910 processor
* openEuler 23.09

## Environment Requirements

* The root permission is required for using and configuring GMEM.
* GMEM can be enabled or disabled only at the system level.
* The administrator must ensure that the GMEM configuration is safe and available.

## Installing GMEM

* Prepare files.

  [CANN Community Version History - Ascend Community (hiascend.com)](https://www.hiascend.com/en/software/cann/community-history)

  [Firmware and Driver - Ascend Community (hiascend.com)](https://www.hiascend.com/en/hardware/firmware-drivers/community?product=2&model=19&cann=6.0.1.alpha001&driver=1.0.18.alpha)

  | Source                                                        | Software Package                                                      |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | openEuler 23.09                                              | kernel-6.4.0-xxx.aarch64.rpm<br>kernel-devel-6.4.0-xxx.aarch64.rpm<br>libgmem-xxx.aarch64.rpm <br>libgmem-devel-xxx.aarch64.rpm |
  | Ascend community                                                    | CANN package:<br>Ascend-cann-toolkit-xxx-linux.aarch64.rpm<br>NPU firmware and driver:<br>Ascend-hdk-910-npu-driver-xxx.aarch64.rpm<br>Ascend-hdk-910-npu-firmware-xxx.noarch.rpm |
  | Contact the maintainers of the GMEM community.<br>[@yang_yanchao](https://gitee.com/yang_yanchao) email: <yangyanchao6@huawei.com><br>[@LemmyHuang](https://gitee.com/LemmyHuang) email: <huangliming5@huawei.com> | gmem-example-xxx.aarch64.rpm<br>mindspore-xxx-linux_aarch64.whl |

* Install the kernel.

  Ensure that GMEM compilation options are enabled (enabled by default) for the openEuler kernel.

  ```sh
  [root@localhost ~]# cat /boot/config-`uname -r` | grep CONFIG_GMEM
  CONFIG_GMEM=y
  CONFIG_GMEM_DEV=m

  [root@localhost ~]#  cat /boot/config-`uname -r` | grep CONFIG_REMOTE_PAGER
  CONFIG_REMOTE_PAGER=m
  CONFIG_REMOTE_PAGER_MASTER=m
  ```

  Add **gmem=on** to the boot options.

  ```sh
  [root@localhost gmem]# cat /proc/cmdline
  BOOT_IMAGE=/vmlinuz-xxx root=/dev/mapper/openeuler-root ... gmem=on
  ```

  Configure **transparent_hugepage**.

  ```sh
  echo always > /sys/kernel/mm/transparent_hugepage/enabled
  ```

* Install the user-mode dynamic library libgmem.

  ```sh
  yum install libgmem libgmem-devel
  ```

* Install the CANN framework.

  Install the matching CANN, including the toolkit, driver, and firmware. After the installation is complete, restart the system.

  ```sh
  rpm -ivh Ascend-cann-toolkit-xxx-linux.aarch64.rpm
  # Use the tool provided by libgmem to install the NPU driver.
  sh /usr/local/gmem/install_npu_driver.sh Ascend-hdk-910-npu-driver-xxx.aarch64.rpm
  rpm -ivh Ascend-hdk-910-npu-firmware-xxx.noarch.rpm
  ```

  Run the environment configuration script in the **Ascend** directory to configure environment variables.

  ```sh
  source /usr/local/Ascend/ascend-toolkit/set_env.sh
  ```

  Check whether the NPU is working properly.

  ```sh
  [root@localhost ~]# npu-smi info
  +-------------------------------------------------------------------------------------------+
  | npu-smi 22.0.4.1                 Version: 22.0.4.1                                        |
  +----------------------+---------------+----------------------------------------------------+
  | NPU   Name           | Health        | Power(W)    Temp(C)           Hugepages-Usage(page)|
  | Chip                 | Bus-Id        | AICore(%)   Memory-Usage(MB)  HBM-Usage(MB)        |
  +======================+===============+====================================================+
  | 0     910B           | OK            | 79.4        82                0    / 0             |
  | 0                    | 0000:81:00.0  | 0           1979 / 15039      0    / 32768         |
  +======================+===============+====================================================+
  ```

* Install the gmem-example software package.

  gmem-example updates the host driver, NPU driver, and NPU kernel. After the installation is complete, restart the system for the driver to take effect.

  ```sh
  rpm -ivh gmem-example-xxx.aarch64.rpm
  ```

* Install MindSpore.

  Obtain the correct MindSpore version and install it. After the installation, run the following command to check whether MindSpore functions are normal:

  ```sh
  python -c "import mindspore;mindspore.run_check()"
  MindSpore version:  x.x.x
  The result of multiplication calculation is correct, MindSpore has been installed on platform [Ascend] successfully!
  ```

## Performing Training or Inference

After installation is complete, you can execute MindSpore-based training or inference directly without any adaptation.
