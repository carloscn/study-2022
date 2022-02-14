# 2022 SP Promotion

时间2月中旬-6月中旬，学习scope需要包含以下内容：

* ARM64
* Linux kernel
* Compiler linker

设备以NXP imx8设备为主（[ARMv8设备](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-8-processors/i-mx-8-family-arm-cortex-a53-cortex-a72-virtualization-vision-3d-graphics-4k-video:i.MX8))

![](_media/%E5%AD%A6%E4%B9%A0%E8%AE%A1%E5%88%92/image-20220209112215688.png)

穿插学习法，ARM64和Linux内容有很大的相关性，穿插着学习。

## ARM64

![](_media/%E5%AD%A6%E4%B9%A0%E8%AE%A1%E5%88%92/arm64.png)

* 总共40.5h，计划每天2h的视频理论学习时间50h（包含笔记时间），需要25天时间

### Linux Kernel

* 奔跑吧Linux内核 
* Linux内核视频

### Compiler Linker

* 静态：编译和链接
* 静态：目标文件里都有什么
* 静态链接
* PE/COFF-windows格式
* 动态：动态装载与进程
* 动态链接
* Linux共享库的组织
* windows的动态链接
* 内存
* 运行库
* 系统调用API
* 运行库的实现

## 计划表单

| Time Line    | ARM64            | Linux Kernel   | Linker                 | 实验环境           |
| ------------ | ---------------- | -------------- | ---------------------- | ------------------ |
| 9-Feb        | 课程介绍         |                |                        |                    |
| 10-Feb       | ARM处理器介绍    |                |                        |                    |
| 11/12/13-Feb | QEMU环境搭建     |                |                        | RASPI实验环境+QEMU |
| 14-Feb       | ARM64汇编        |                | 编译与链接             |                    |
|              | GUN AS汇编器介绍 |                | 目标文件有什么         |                    |
|              | LD链接器介绍     |                | 静态链接               |                    |
|              | 内嵌汇编         | 内核简介       | 可执行文件的装载和进程 |                    |
|              | ARM64异常处理    | 内核出发       | Linux共性库的组织      |                    |
|              | GIC中断控制      | 中断和异常     |                        |                    |
|              | 内存管理         | 内存寻址       | 内存                   |                    |
|              | MMU实验          | 内存管理       |                        |                    |
|              | MMU芯片手册      |                |                        |                    |
|              | cache            |                |                        |                    |
|              | TLB基础          |                |                        |                    |
|              | 内存屏障         |                |                        |                    |
|              | 一致性和内存屏障 |                |                        |                    |
|              | 原子操作         | 内核同步       |                        |                    |
|              | 浮点数指令       | 进程           |                        |                    |
|              | NEO适量          | 进程调度       |                        |                    |
|              | 可扩展矢量计算   | 进程通信       |                        |                    |
|              | GICv3            | 进程地址空间   |                        |                    |
|              | SMMUv3           |                |                        |                    |
|              | SMMUv3           |                |                        |                    |
|              | AXI5-Lite        |                |                        |                    |
|              | ACE-Lite         | 定时测量       |                        |                    |
|              |                  | 系统调用       | 动态链接               |                    |
|              |                  | 信号           | 运行库                 |                    |
|              |                  | 虚拟文件系统   | 系统调用和API          |                    |
|              |                  | IO体系设备驱动 | 运行库的实现           |                    |
|              |                  | 块设备驱动     |                        |                    |
|              |                  | 页高速缓存     |                        |                    |
|              |                  | 访问文件       |                        |                    |

## 参考资料

1. ARM手册： https://developer.arm.com/documentation/ddi0487/latest

2. 工具： https://github.com/NaoTu/DesktopNaotu/releases/tag/v3.2.3