# DIY谐波赤道仪（已通电验证，待实拍测试）

由于我厌倦了EQ3D赤道仪的沉重，我计划用更轻便的谐波赤道仪来替换它，然而市面上的谐波赤道仪价格太贵（与DIY的规格相比），所以这个项目就此诞生，谐波赤道仪能够在不牺牲性能的前提下，提供高精度、便携性和经济性

# 项目描述

自制谐波赤道仪的核心谐波减速器，是基于Leaderdriver拆机二手的工业谐波减速器(也适用于淘宝购买的同规格谐波减速器如“25-100”)
![image.png](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/3D-Views/1.png)
![image.jpg](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/3D-Views/3.jpg)
![image.jpg](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/3D-Views/5.JPG)
![image.jpg](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/3D-Views/4.jpg)
![image.jpg](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/3D-Views/2.jpg)

该项目旨在解决业余天文设备中的关键问题：
- 成品支架价格高昂
- 缺乏开放性和灵活性
- 复杂性或与个人设置的不兼容

此款DIY谐波赤道仪优势是：
- 🧰**经济实惠**——总造价低于3000元，加工零件价格低于1500元
- 🪶**相对便携**——不含底座重量7.5公斤
- 🏗️**易于搭建**——专为拥有基本工具的创客设计
- 💪**高性能**——精确跟踪（亚角秒级 RMS），无需配重
- ⚠️**关于当前项目状态的说明**⚠️
该项目只完成了通电和简单的goto测试未进行实拍测试

# ✨主要特点
| **特征** | **描述** |
| :----- | :----- |
| 安装类型 | GEM德式赤道仪 |
| 传动类型 | 谐波减速器+皮带传动+42步进电机 |
| 有效载重 | 最大载重30kg(最好需要配重)，无需配重载重为20-25kg |
| 减速器类型 | leaderdrive LHS-25-100-C-III |
| 控制系统 | OnStep/OnStepx |
| 连接类型 | USB-B；WIFI；蓝牙；集成DS3231SN时钟芯片 |
| 硬件功能 | RA轴制动器；RA/DEC轴光电传感器限位；DEC/RA轴光电传感器原点；<br>同时兼容step/dir，spi两种驱动模式(即兼容tmc2209，tmc5160)；12/24v输入 |
| 预计成品 | 2200-2600 RMB |
| 预计重量 | 9.5kg(包含鸠尾槽和纬度座) |

## 🔗 物料清单 (BOM)

请参阅 [BOM.md](BOM.md)

**预计总成本：约2500RMB（含纬度座）**

本项目的主体结构高度依赖定制，主要部件包含两台 25-100 规格的谐波减速器，提供了极高的传动比与扭矩，底座接口可根据个人习惯适配主流的三脚架

*注：价格为近似值，可能因电机供应商、CNC 加工批次和地点而异*

---

## 🔌 电子部分概览

控制系统完全采用定制设计，并围绕紧凑结构构建，它包括：

* **基于 ESP32-WROOM-32E16N 的主控核心**，运行强大的 [OnStep/OnStepX](https://onstep.groups.io/g/main/wiki) 系统
* **搭配 ESP07S 无线连接**，提供高效的无线/有线通讯
* **TMC系列步进电机驱动器**，实现赤经与赤纬轴精确、静音的微步进控制与指向
* **高效电源管理方案**，板载 TPS5450 降压模块和 AMS1117 LDO 稳压芯片，确保高负载下系统电压的纯净与稳定
* 集成 **零位传感器和限位开关** 接口，在运行中保障设备的安全
* 支持通过 Wi-Fi、蓝牙或 USB 路由进行远程控制，无缝对接主流天文摄影软件生态 (N.I.N.A., INDI, ASCOM 等)
* 控制板在 PCB 走线设计上充分考虑了扩展性需求，通过主板驱动器插槽旁的排针跳线 (Jumpers)，您可以轻松地在先进的 **SPI 智能控制模式**和传统的 **STEP/DIR 独立模式**之间进行物理切换(切换后需刷写固件)

* **TMC5160 SPI 模式 (推荐配置) 🚀**
    * **硬件设置**：使用跳线帽闭合 SPI 专用的针脚组，将 ESP32 的 SPI 总线直接连接至 TMC5160 驱动器
    * **系统优势**：彻底告别手动调节电位器，允许 OnStep 固件直接读取和写入寄存器，您可以在 Web 界面或 `Config.h` 中实时配置运行/保持电流、精确设置细分数（Microstepping），并在 StealthChop（极致静音）和 SpreadCycle（高动态扭矩）模式之间智能切换，最大化释放谐波减速器的性能
    * **固件配合**：在 `Config.h` 中将赤经/赤纬驱动模型定义为 `TMC5160_SPI`

* **传统 STEP/DIR 模式 (向下兼容) 🔌**
    * **硬件设置**：断开 SPI 通讯跳线
    * **适用场景**：作为一种可靠的底层回退方案。如果您需要临时替换为普通的 A4988、DRV8825，或者仅使用 TMC 驱动器的 Standalone (独立运行) 模式，此设计提供了极大的硬件容错率和维修便利性
    * **固件配合**：需要在 `Config.h` 中精确填写您通过跳线帽设定的细分值，并**必须手动使用万用表和陶瓷螺丝刀**调节驱动器上的 VREF 电位器来设定电机电流

> 💡 **硬件设计提示**：为确保 SPI 高速通讯时的数据完整性，我们在定制 PCB 上对相关通讯走线进行了严格的间距控制
**警告：在插拔驱动器或更改跳线设置前，请务必完全断开系统主电源（包括 USB 供电），以防瞬间浪涌击穿 TMC5160 或 ESP32 主控**

所有电路板原理图及 PCB Layout（严格把控了安全间距、过孔尺寸和大电流线宽）均使用开源工具设计，您可以在 [https://member.jlc.com/?spm=PCB.Homepage] 订购 PCB或对其进行重新组合以满足您的需求

## 💻 固件配置与刷写

谐波赤道仪的主控搭载了 **ESP32-WROOM-32E (16MB)**，这款大容量核心不仅能轻松运行复杂的 [OnStep / OnStepX](https://onstep.groups.io/g/main/wiki) 固件，还为内置星表、高精度对位模型 (Pointing Model) 以及功能丰富的 Smart Web UI 留出了充足的存储空间

请参阅 [Firmware_Setup.md](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/software/Firmware_Setup.md) 获取完整的 `Config.h` 参数表及详细引脚映射
---

## 🛠️ 机械部分组装指南

请参阅 [Assembly.md](Assembly.md)

下图显示了构建 Dual 25 所需的全部机械和电子零件

*(请在此处插入您在“新年首发”时拍摄的组装完成图或零配件合影)*

每个组件都经过精心挑选或设计，以确保高机械精度和易于组装。从 CNC 加工的坚固铝制主体、25-100 谐波减速器，到步进电机和控制器的 3D 打印防护外壳，每一步都经过了严格的测试与指向模型校准。
