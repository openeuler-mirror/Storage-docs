# HSAK开发者指南

## 介绍

随着NVMe SSD、SCM等存储介质性能不断提升，介质层在IO栈中的时延开销不断缩减，软件栈的开销成为瓶颈，急需重构内核IO数据面，减少软件栈的开销，HSAK针对新型存储介质提供高带宽低时延的IO软件栈，相对传统IO软件栈，软件栈开销降低50%以上。
HSAK用户态IO引擎基于开源的SPDK基础上进行开发：

1. 对外提供统一的接口，屏蔽开源接口的差异。
2. 在开源基础上新增IO数据面增强特性，如DIF功能，磁盘格式化，IO批量下发，trim特性，动态增删盘等特性。
3. 提供磁盘设备管理，磁盘IO监测，维测工具等特性。

## 编译教程

1. 下载hsak源码
   
    $ git clone <https://gitee.com/openeuler/hsak.git>

2. 编译和运行依赖
   
    hsak的编译和运行依赖于spdk、dpdk、libboundscheck等组件

3. 编译
   
    $ cd hsak
   
    $ mkdir build
   
    $ cd build
   
    $ cmake ..
   
    $ make

## 注意事项

### 使用约束

- 同一台机器最多使用和管理512个NVMe设备。
- 启用HSAK执行IO相关业务时，需要确保系统有至少500M以上连续的空闲大页内存。
- 启用用户态IO组件执行相关业务时，需要确保硬盘管理组件（ublock）已经启用。
- 启用磁盘管理组件（ublock）执行相关业务时，需确保系统有足够的连续空闲内存，每次初始化ublock组件会申请20MB大页内存。
- 每次运行HSAK之前，产品需要调用setup.sh来配置大页，解绑NVMe设备内核态驱动。
- 执行libstorage_init_module成功后方可使用HSAK模块提供的其他接口；每个进程仅能执行一次libstorage_init_module调用。
- 执行libstorage_exit_module函数之后不能再使用HSAK提供的其他接口，再多线程场景特别需要注意，在所有线程结束之后再退出HSAK。
- HSAK ublock组件在一个服务器上只能启动一个服务，且最大支持64个ublock客户端并发访问，ublock服务端处理客户端请求的处理上限是20次/秒。
- HSAK ublock组件必须早于数据面IO组件和ublock客户端启动，HSAK提供的命令行工具也必须在ublock服务端启动后才能执行。
- 不要注册SIGBUS信号处理函数；spdk针对该信号有单独的处理函数；若该函数被覆盖，会导致spdk注册的SIGBUS处理函数失效，产生coredump。
