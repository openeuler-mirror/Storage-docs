# 安装与部署

本章介绍如何安装和部署GMEM。

## 软硬件要求

* 鲲鹏920处理器
* 昇腾910芯片
* 操作系统：openEuler 23.09

## 环境准备

* 使用和配置GMEM需要使用root权限。
* GMEM的开关只能在系统层面开启或关闭。
* 请管理员确保GMEM的配置安全、可用。

## 安装GMEM

* 文件准备

  [CANN社区版历史版本-昇腾社区 (hiascend.com)](https://www.hiascend.com/software/cann/community-history) 

  [固件与驱动-昇腾社区 (hiascend.com)](https://www.hiascend.com/zh/hardware/firmware-drivers/community?product=2&model=19&cann=6.0.1.alpha001&driver=1.0.18.alpha) 

  | 来源                                                         | 软件包                                                       |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | openEuler 23.09                                              | kernel-6.4.0-xxx.aarch64.rpm<br/>kernel-devel-6.4.0-xxx.aarch64.rpm<br/>libgmem-xxx.aarch64.rpm <br/>libgmem-devel-xxx.aarch64.rpm |
  | 昇腾社区                                                     | # CANN软件包<br/>Ascend-cann-toolkit-xxx-linux.aarch64.rpm<br/># NPU固件与驱动<br/>Ascend-hdk-910-npu-driver-xxx.aarch64.rpm<br/>Ascend-hdk-910-npu-firmware-xxx.noarch.rpm |
  | 联系GMEM社区维护人员<br/>[@yang_yanchao](https://gitee.com/yang_yanchao) email: <yangyanchao6@huawei.com><br/>[@LemmyHuang](https://gitee.com/LemmyHuang) email: <huangliming5@huawei.com> | gmem-example-xxx.aarch64.rpm<br/>mindspore-xxx-linux_aarch64.whl |

* 安装内核

  使用的openEuler内核版本，确认GMEM相关编译选项已打开（当前默认已经打开）。

  ```sh
  [root@localhost ~]# cat /boot/config-`uname -r` | grep CONFIG_GMEM
  CONFIG_GMEM=y
  CONFIG_GMEM_DEV=m
  
  [root@localhost ~]#  cat /boot/config-`uname -r` | grep CONFIG_REMOTE_PAGER
  CONFIG_REMOTE_PAGER=m
  CONFIG_REMOTE_PAGER_MASTER=m
  ```

  在启动项中添加`gmem=on` 。

  ```sh
  [root@localhost gmem]# cat /proc/cmdline
  BOOT_IMAGE=/vmlinuz-xxx root=/dev/mapper/openeuler-root ... gmem=on
  ```

  修改`transparent_hugepage` 。

  ```sh
  echo always > /sys/kernel/mm/transparent_hugepage/enabled 
  ```

* 安装用户态动态库 libgmem。

  ```sh
  yum install libgmem libgmem-devel 
  ```

* 安装CANN框架。

  安装版本配套的CANN，包括toolkit，driver以及firmware，根据指引完成安装后重启系统。

  ```sh
  rpm -ivh Ascend-cann-toolkit-xxx-linux.aarch64.rpm
  # 使用libgmem提供的工具安装npu-driver
  sh /usr/local/gmem/install_npu_driver.sh Ascend-hdk-910-npu-driver-xxx.aarch64.rpm
  rpm -ivh Ascend-hdk-910-npu-firmware-xxx.noarch.rpm
  ```

  通过Ascend目录下的环境配置脚本配置好环境变量。

  ```sh
  source /usr/local/Ascend/ascend-toolkit/set_env.sh
  ```

  查看NPU设备是否正常。

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

* 安装gmem-example软件包。

  gmem-example会更新host驱动、NPU侧驱动及NPU侧内核。安装完成后重启系统使驱动生效。

  ```sh
  rpm -ivh gmem-example-xxx.aarch64.rpm
  ```

* 安装mindspore。

  获取正确的mindspore版本并安装，安装后可通过执行以下命令验证mindspore功能是否正常。

  ```sh
  python -c "import mindspore;mindspore.run_check()"
  MindSpore version:  x.x.x
  The result of multiplication calculation is correct, MindSpore has been installed on platform [Ascend] successfully!
  ```

## 执行训练或推理任务

基于mindspore的训练或推理任务，在完成以上安装流程后，可直接执行，不需要做任何适配。
