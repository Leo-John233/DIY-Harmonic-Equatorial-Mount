<div align="center">
  <a href="./README.md"><img src="https://img.shields.io/badge/简体中文-4285F4.svg?style=flat&logo=googletranslate&logoColor=white" alt="简体中文"></a>
  <a href="./README_EN.md"><img src="https://img.shields.io/badge/English-4285F4.svg?style=flat&logo=googletranslate&logoColor=white" alt="English"></a>
</div>

## 📄 License / 开源协议

本项目采用组合开源协议，对软件和硬件部分分别进行授权：

* **Software / 软件部分** (如固件代码): 采用 [GNU General Public License v3.0 (GPL-3.0)](LICENSE-GPL) 授权，详细信息请参阅 `LICENSE-GPL` 文件
* **Hardware / 硬件部分** (如 PCB 设计、原理图、机械 3D 模型): 采用 [CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S v2)](LICENSE-CERN) 授权，详细信息请参阅 `LICENSE-CERN` 文件

若参考了该项目希望能够指明参考来源，注明原作者，保持开源精神，欢迎大家在此基础上进行改进和创新，但请勿将本项目的设计用于商业用途，除非您获得了明确许可

## 🛑致各位试图拿本开源项目“创业”的商业奇才们
本着开源精神，我非常欢迎大家复刻、交流和自己动手做着玩，但如果您是抱着“低价造个赤道仪拿去卖”的伟大商业宏图，并且打算把原作者当免费技术顾问的，请仔细阅读以下几点：

**🚫本项目不提供“保姆级开厂培训”**
开源提供的是设计本身，不是您商业化流水线上的免费技术支持，如果您连最基本的“水平轴承在哪”、“圆弧槽是干啥用的”都看不明白，连 STP 文件导入 SolidWorks 都不知道怎么处理，建议先去补习一下基础的机械常识，我不负责为您解答“怎么装”、“怎么动”、“怎么改”，更不负责给您的工厂流水线工人做技术培训，请不要把原作者当成您的“免费技术顾问”，如果您不懂机械设计，请先学会自己看图纸，学会自己分析问题，学会自己解决问题

**✂️魔改出了 Bug，请自己背锅**
把模型等比缩放、乱改减速机孔位，改完之后跑来跟我抱怨“俯仰调节机构装不上”、“没法适配”、“太难理解”？拜托，自己魔改导致的问题，请不要跑来怪原作者，我都开源了，您还指望我给您的“定制缩水版”做免费 Debug 吗？

**💰嫌贵别用，用了别嫌**
一边刷着小红书觉得这套开源设计“好高级”，一边又跑来跟我吐槽用这套方案“成本涨了200块”、“太麻烦了”，觉得贵、觉得麻烦，您可以不用，既然决定要抄这套作业去赚钱，就别在原作者面前哭穷了，毕竟我也不想看到您辛辛苦苦开了个工厂，结果因为技术问题被客户投诉了还跑来找我背锅

**⚖️友情警告**
想拿这套类似结构随便改个外观就去大规模量产售卖？自己做着玩大家其乐融融，但如果要量产商用，请自己评估好侵权风险，别怪我没提醒过您，出了事请自己承担，别拉上我，开源不等于“免费商用”，如果您想拿去商用，请自己搞定技术壁垒，做好侵权风险评估，做好售后服务准备，做好质量控制准备，做好客户投诉处理准备，做好工厂管理准备，做好资金链管理准备，做好一切商业化运营的准备，不要到时候出了问题还跑来怪原作者

大家都很忙，祝您的“平价赤道仪”大卖，Peace & Love

👉 **[奇文共赏：点击此处阅读“白嫖式创业奇才”的逆天聊天实录](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Chat%20history/readme.md)**

---
## DIY谐波赤道仪（已完成实拍与导星测试）

由于我厌倦了 EQ3D 赤道仪的沉重，我计划用更轻便的谐波赤道仪来替换它。然而市面上的谐波赤道仪价格较高，尤其是与 DIY 可实现的规格相比，因此这个项目就此诞生。该谐波赤道仪旨在不牺牲性能的前提下，兼顾高精度、便携性和经济性

该项目旨在解决业余天文设备中的关键问题：
- 成品支架价格高昂
- 缺乏开放性和灵活性
- 复杂性或与个人设置的不兼容

此款DIY谐波赤道仪优势是：
- 🧰**经济实惠**——总造价低于3000元，加工零件价格低于1500元
- 🪶**相对便携**——不含底座重量7.5公斤
- 🏗️**易于搭建**——专为拥有基本工具的创客设计
- 💪**高性能**——已完成PHD2导星拍摄测试
- ⚠️**关于当前项目状态的说明**⚠️
该项目已完成通电、基础 GoTo、导星和约13kg实际成像负载下的实拍验证，稳定导星时段的 PHD2 总 RMS 通常约为 0.4″ 至 0.6″，不同装配、负载和环境下的结果可能有所差异

## ✨主要特点
| **特征** | **描述** |
| :----- | :----- |
| 安装类型 | GEM德式赤道仪 |
| 传动类型 | 谐波减速器+皮带传动+42步进电机 |
| 有效载重 | 设计最大载重 30 kg（建议配重），无配重设计载重 20-25 kg，当前已完成约 13 kg 实拍验证 |
| 减速器类型 | leaderdrive LHS-25-100-C-III |
| 控制系统 | OnStep/OnStepx |
| 连接类型 | USB-B；WIFI；蓝牙；集成DS3231SN时钟芯片 |
| 硬件功能 | RA轴制动器；RA/DEC轴光电传感器限位；DEC/RA轴光电传感器原点；<br>同时兼容step/dir，spi两种驱动模式(即兼容tmc2209，tmc5160)；12/24v输入 |
| 预计成本 | 2500-2900 RMB |
| 预计重量 | 9.5 kg（包含鸠尾槽和纬度座） |
| 已测导星表现 | 稳定时段 PHD2 总 RMS 通常约为 0.4″ 至 0.6″ |

---

## 最终成效

自制谐波赤道仪的核心部件为 Leaderdrive 拆机二手工业谐波减速器，也适用于同规格的 25-100 谐波减速器
![image.png](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/1.png)
![image.jpg](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/3.jpg)
![image.jpg](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/5.JPG)
![image.jpg](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/4.jpg)
![image.jpg](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/2.jpg)

---

## 实拍与导星测试

本次拍摄目标为 **Herschel 36**

![Herschel 36](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/Herschel-36.jpg)

| **实拍项目** | **配置** |
| :----- | :----- |
| 主摄影器材 | 铁皮桶全改小黑，搭配 MPCC 彗差校正镜 |
| 主相机 | Nikon D7000 |
| 导星相机 | ZWO ASI662MC |
| 实际成像负载 | 约 13 kg |
| 测试环境 | 微风 |

## 当前验证状态

- [√] 主控板通电和基础通信
- [√] OnStep ASCOM 连接
- [√] 基础 GoTo 和恒星时跟踪
- [√] PHD2 校准和闭环导星
- [√] Dither 和 Settling
- [√] 约 52 分 45 秒连续导星记录
- [√] 约 13 kg 实际成像负载下的 Herschel 36 实拍
- [√] 不同载重和重心位置下的多夜重复测试
- [√] 无配重与带配重状态下的系统对比
- [ ] 关闭导星修正和 Dither 后的长时间本征周期误差测试
- [ ] 不同焦距、曝光时间和环境条件下的长期实拍验证

## PHD2 导星测试

### 测试配置

| 项目 | 配置 |
| :--- | :--- |
| 测试软件 | PHD2 Guiding 2.6.13 |
| 赤道仪连接 | OnStep ASCOM |
| 导星相机 | ZWO ASI662MC |
| 导星焦距 | 200 mm |
| 导星相机像元尺寸 | 2.9 μm |
| 导星图像比例 | 约 2.99″/px |
| 导星曝光 | 0.2 s |
| 实际成像负载 | 约 13 kg |
| 测试环境 | 微风 |
| 完整记录时长 | 52 分 45 秒 |

### 结果汇总

| 测试片段 | RA RMS | Dec RMS | 总 RMS | 说明 |
| :--- | ---: | ---: | ---: | :--- |
| 稳定导星片段 | 0.43″ | 0.50″ | 0.66″ | PHD2 实时统计截图 |
| 52 分 45 秒完整记录 | 0.45″ | 0.82″ | 0.94″ | 包含 Dither、Settling 和少量异常峰值 |

实际使用过程中，稳定导星阶段的总 RMS 通常约为 0.4″ 至 0.6″。稳定片段截图显示总 RMS 为 0.66″，完整记录由于包含 Dither、Settling 和少量异常峰值，总 RMS 为 0.94″

### 结果分析

- **稳定阶段两轴误差较为均衡**：RA RMS 为 0.43″，Dec RMS 为 0.50″，两轴差值较小，未表现出明显的单轴持续振荡。总 RMS 为 0.66″，按 2.99″/px 的导星图像比例换算约为 0.22 px，其中 RA 约为 0.14 px，Dec 约为 0.17 px
- **完整记录的 RMS 主要由 Dec 瞬态波动拉高**：完整记录中的 RA RMS 为 0.45″，与稳定片段接近，说明 RA 长时间表现基本保持一致。Dec RMS 上升至 0.82″，表明完整记录中的 Dither、Settling、微风或少量异常峰值主要影响了 Dec 统计结果，不代表稳定跟踪阶段持续存在 0.82″ 的 Dec 误差
- **约 13 kg 实际负载下维持稳定导星**：在铁皮桶全改小黑、Nikon D7000、MPCC 和导星系统组成的约 13 kg 成像负载下，稳定阶段未观察到明显发散或持续性机械振荡，说明当前结构刚性、传动控制和导星响应能够满足实际成像需求
- **0.2 s 导星曝光体现了较快的修正响应**：较短曝光有助于快速修正误差，但也更容易受到视宁度和瞬时扰动影响。因此，0.4″ 至 0.6″ 应视为本次设备、参数和环境组合下的实测表现，不应直接作为所有场景下的固定性能指标
- **频域结果不能直接作为本征周期误差**：频域分析中选中的低频分量周期约为 2206.6 s，显示振幅约为 0.1″。由于数据来自导星记录，并包含低频漂移、Dither 和 Settling，该结果只能用于观察频率成分，不能直接认定为谐波减速器的本征周期误差

<details>
<summary>展开查看 PHD2 测试截图与频域分析</summary>

![PHD2 stable guiding](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/2026-07-03%20002555.png)

![PHD2 complete log](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/2026-07-03%20133401.png)

![PHD2 drift-corrected analysis](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/2026-07-03%20153632.png)

![PHD2 frequency analysis](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/2026-07-03%20153353.png)

移除 RA 导星修正后的曲线仍包含低频漂移、Dither 和 Settling 的影响，不能直接将整段曲线的峰峰值视为谐波减速器的本征周期误差

频域分析中选中的低频分量周期约为 2206.6 s，软件显示振幅约为 0.1″。该结果仅用于观察频率成分，不能替代关闭导星修正和 Dither 后进行的长时间无导星周期误差测试

</details>

---

## 🔗 物料清单 (BOM)

请参阅 [BOM.md](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/tree/main/BOM)

**预计总成本：约2500RMB（含纬度座）**

本项目的主体结构高度依赖定制，主要部件包含两台 25-100 规格的谐波减速器，提供了极高的传动比与扭矩，底座接口可根据个人习惯适配主流的三脚架

*注：价格为近似值，可能因电机供应商、CNC 加工批次和地点而异*

---

## 🔌电子部分概览

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

所有电路板原理图及 PCB Layout（严格把控了安全间距、过孔尺寸和大电流线宽）均使用开源工具设计，您可以在 [JLC PCB](https://member.jlc.com/?spm=PCB.Homepage) 订购 PCB或对其进行重新组合以满足您的需求

### PCB 制作与装配说明

![PCB front](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/pcb/PCB-1.jpg)

![PCB reverse-side TMC5160 installation](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/pcb/PCB-2.jpg)

![PCB reverse-side TMC2209 installation](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/pcb/PCB-3.jpg)

PCB 制作和装配建议按以下顺序进行：

1. **提交 PCB 文件**
   * 将仓库中对应版本的 Gerber 压缩包直接上传至 PCB 厂商
   * 下单前检查板框尺寸、安装孔、槽位和连接器开口，避免生产平台自动缩放或修改外形
   * 如果自行修改板厚、铜厚、表面处理或元件封装，需要重新确认外壳间隙、连接器高度和大电流走线能力

2. **核对元件和方向**
   * 焊接前根据原理图、BOM、PCB 丝印和封装方向逐项核对元件
   * 重点检查二极管、LED、电解电容、稳压芯片、ESP32、ESP-07S 和其他有方向要求的器件
   * CR1220 电池按 PCB 丝印方向安装，照片所示安装方式为正极朝外
   * ESP32 天线区域附近不要布置金属挡板、屏蔽片或紧贴外壳，以免影响无线连接

3. **建议焊接顺序**
   * 先焊接电阻、电容、二极管和小型 IC 等低矮贴片器件
   * 再焊接电源模块、排针、驱动器插座、按键和 LED 接口
   * 最后安装 XH2.54 插座、USB、网络接口、DC 插座、XT60 和其他较高连接器
   * TMC 驱动器、ESP 模块和纽扣电池应在完成短路检查和基础供电测试后再安装

4. **驱动器安装**
   * 驱动器安装在 PCB 背面，插入前必须核对 PCB 丝印与驱动器上的 `GND`、`VM`、`EN`、`STEP` 和 `DIR` 引脚
   * 不同厂商的 TMC2209 或 TMC5160 模块可能存在引脚、焊盘或散热结构差异，不能只根据模块外观判断方向
   * 使用 TMC5160 SPI 模式时闭合对应 SPI 跳线，并在固件中选择 `TMC5160_SPI`
   * 使用 TMC2209 或其他 STEP/DIR 驱动器时断开 SPI 跳线，并按照实际硬件设置填写细分参数和驱动电流
   * 驱动器插反或带电插拔可能立即损坏驱动器和主控板

5. **首次通电检查**
   * 通电前使用万用表检查主电源正负极之间是否存在明显短路
   * 首次测试建议使用带限流功能的实验电源，并从较低限流值开始
   * 首次通电时不要连接电机、传感器和驱动器，先确认电源部分没有异常发热，并检查主要电压轨是否正常
   * 确认主控、USB 通信和无线连接正常后，再逐个安装驱动器并连接 RA、Dec 电机
   * 初次电机测试应使用较低电流和较低速度，确认转向、原点和限位逻辑正确后再提高参数

6. **接口连接**
   * `LIMIT-RA`、`LIMIT-DEC`、`HOME-RA` 和 `HOME-DEC` 分别连接对应轴的限位和原点传感器
   * `RA-XH2.54`、`DEC-XH2.54` 和 `BRAKE` 接口应根据原理图确认引脚顺序后连接
   * 状态灯、连接灯和电源灯接口需要注意 LED 极性
   * 不建议同时连接多个主电源入口，除非已根据原理图确认其连接关系和使用方式

> 💡**警告：任何焊接、驱动器插拔、跳线调整、传感器接线或电机接线操作都必须在主电源和 USB 供电完全断开后进行，首次上电前应再次检查电源极性、焊锡短路、驱动器方向和连接器引脚顺序**

## 💻固件配置与刷写

谐波赤道仪的主控搭载了 **ESP32-WROOM-32E (16MB)**，这款大容量核心不仅能轻松运行复杂的 [OnStep / OnStepX](https://onstep.groups.io/g/main/wiki) 固件，还为内置星表、高精度对位模型 (Pointing Model) 以及功能丰富的 Smart Web UI 留出了充足的存储空间

请参阅 [Firmware_Setup.md](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/software/Firmware_Setup.md) 获取完整的 `Config.h` 参数表及详细引脚映射

---

## 🛠️机械部分组装指南

请参阅 [Assembly.md](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/hardware/Assembly.md)

每个组件都经过精心挑选或设计，以确保高机械精度和易于组装，从 CNC 加工的坚固铝制主体、25-100 谐波减速器，到步进电机和控制器的 3D 打印防护外壳，每一步都经过了严格的测试与指向模型校准，以实现亚角秒级的跟踪精度
