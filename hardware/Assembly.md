# Dual 25 谐波赤道仪组装说明书

结合 `pictures/Assembly process` 目录下的实拍记录，分步讲解从机械结构到电控系统的完整拼装过程

## 🛠️ 准备工作与物料清单 (BOM)

在开始组装之前，请确保您已备齐以下核心零部件与工具(具体所需五金参考bom表)：

* **传动核心**：25-100型谐波减速器（1:100减速比）
* **惰轮**：2GT带宽10mm 80齿惰轮和2GT带宽10mm高19mm 20齿惰轮
* **电机与保护**：步进电机、掉电刹车（电磁阻尼/抱闸）模块
* **主控与驱动**：ESP32 + ESP07S 组合、TMC5160 步进电机驱动模块
* **电路底板**：定制设计的主控 PCB（支持 12V/24V 电源平面）
* **线材与辅料**：**电源开关引线**、各型号内六角螺丝、润滑脂、螺丝胶
*注：惰轮部分需要您有一定机加工能力对成品惰轮按照图纸中的规格进行改造，若无法改造，请自行修改模型以适配市面上的标准零件*
---

## ⚙️ 阶段一：机械结构组装

### 1. 谐波减速器的安装 (RA/DEC 轴)
在此步骤中，我们将安装赤道仪的核心部件，考虑到径向慢速旋转载荷的受力要求，安装时请务必保证法兰接触面的平整贴合

1.  将减速器法兰面与铝板螺孔精准对齐，尽可能的让铝板中心圆通孔与减速器中空轴同轴心，使用内六角螺丝并按对角线顺序均匀拧紧，确保减速器外壳受力均匀，避免因局部过紧导致的变形或过松引起的振动
2.  其中两个螺孔需要拧入更长的螺丝，且不需要拧上防松螺母以便后续电机座的安装
3.  其中有4个螺孔不需要拧入螺丝，以便后续安装3d打印的减速器保护罩
4.  将除去步骤2中的剩余的核心螺丝组件多余的部分拧入防松螺母，确保减速器与铝板之间的连接牢固且不会因振动而松动

> 📷 *参考图片：*
> ![安装谐波减速器](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(1).JPG)
> ![安装谐波减速器](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(2).JPG)
> ![安装谐波减速器](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(3).JPG)

### 2. 传动部分的组合
1.  将加工过的80齿轮2GT惰轮安装在减速器的输入轴上，确保惰轮与输入轴同轴心，使用内六角螺丝并按对角线顺序均匀拧紧，确保连接牢固且不会因振动而松动
2.  将20齿轮2GT惰轮安装在步进电机的输出轴上  
3.  将步进电机通过铜柱安装至电机座上
4.  **掉电刹车安装**：在用于RA轴的双出轴步进电机的尾端，替换原有电机的螺丝并安装电磁抱闸模块，能有效防止设备意外断电时由于重锤失衡导致的望远镜“磕头”受损
5.  提前将2GT带宽10mm同步带套在电机惰轮上，便于后续安装

> 📷 *参考图片：*
> ![RA轴电机与刹车模块安装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(15).PNG)
> ![RA轴电机与刹车模块安装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(16).PNG)
> ![DEC轴电机安装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(17).PNG)

### 3. RA轴纬度座法兰的组装
1.  将RA轴法兰盘与连接轴A和连接轴B通过内六角螺丝连接
2.  在步骤1的过程中，同时组装推杆和推块结构，确保推杆能够在推块内顺畅滑动，并且连接轴A和连接轴B能够通过推杆实现相对运动
3.  将步骤1和2中组装好的整体结构通过内六角螺丝安装到谐波减速器的法兰盘上，在安装过程同时安装3d打印好减速器保护罩

> 📷 *参考图片：*
> ![纬度座的组装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(7).JPG)
> ![纬度座的组装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(8).JPG)
> ![纬度座的组装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(10).JPG)

### 4. 电机，同步带与减速器的连接以及传感器，法兰盘的安装
1.  通过电机座上预留的孔位安装至步骤1中铝板上预留的更长螺丝上，绷紧同步带，并在螺丝上拧上防松螺母，确保同步带在传动过程中不会发生打滑或过度松弛
2.  并适当调整电机输出轴上的惰轮高度，使其与减速器输入轴上的惰轮处于同一水平面，确保同步带的张力均匀分布，避免因不平行导致的过度磨损或传动效率降低
3.  在完成上述步骤后，手动旋转大惰轮，检查同步带的运行是否平稳，确保没有异常的摩擦声或振动，这将有助于确认传动系统的正确安装和调整
4.  在完成安装后，建议在同步带上喷涂适量的矽质润滑剂，以减少摩擦和异响并延长同步带的使用寿命
5.  安装RA/DEC轴的光电传感器限位开关和原点开关，确保它们的位置能够准确检测到轴的极限位置和原点位置，以实现安全保护和精准定位
6.  将3d打印好的减速器保护罩安装在减速器外部，使用预留的螺孔和内六角螺丝固定，确保保护罩能够有效防止灰尘和杂物进入减速器，同时不干扰其正常运行
7.  安装RA/DEC轴的法兰盘，确保其与轴同轴心，并使用内六角螺丝均匀拧紧，确保法兰盘在运行过程中不会发生偏移或松动

> 📷 *参考图片：*
> ![RA轴电机与刹车模块，传感器，法兰盘的安装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(5).JPG)
> ![DEC轴电机，同步带，传感器的安装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(9).JPG)
> ![DEC轴法兰盘的安装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(6).JPG)

### 5. 外壳的组装
1.  将组装好RA轴和DEC轴的机械结构通过铝块连接，利用大理石水平面确保所有铝板都能够垂直正交安装并且不会发生干涉
2.  并在螺丝连接处涂抹适量的螺丝胶，确保在设备运行过程中机械连接的稳定性和可靠性

> 📷 *参考图片：*
> ![外壳的组装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(11).JPG)

---

## ⚡ 阶段二：电控组装与走线

### 1. PCB 底板与驱动安装
1.  将制作好的 PCB 板通过铜柱固定在电控仓内部，安装时请仔细检查背面的 12V/24V 大电流电源平面，确保其与金属外壳之间有足够的绝缘距离。
2.  依次将 TMC5160 驱动模块和 ESP32 主控板插接至对应排母，注意核对引脚方向，切勿插反。

> 📷 *参考图片：*
> ![PCB及驱动安装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(18).JPG)
> ![PCB及驱动安装](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(19).JPG)

### 2. 规范接线与理线
为保证设备在野外环境与大电流驱动下的绝对稳定，本项目的接线有着严格的要求：
1.  主电源线路必须使用标准的**电源开关引线**进行连接至 PCB。**严禁使用任何随意的“飞线”**作为电源通路的替代方案。
2.  将 RA 轴和 DEC 轴步进电机的控制线接入 TMC5160 对应的端子。
3.  连接掉电保护系统的控制线，并整理所有线束，确保赤道仪在全方位极限旋转时线缆不会发生缠绕或过度拉扯。

> 📷 *参考图片：*
> ![规范的电源引线与内部走线](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(12).JPG)

---

## 💻 阶段三：固件配置与初步调试

Dual 25 的硬件逻辑专为标准的 OnStep 环境设计。

1.  **固件选择**：系统运行的是**普通的 OnStep 固件**（⚠️请注意：本项目**不适用** OnStepX，切勿刷错固件版本）。
2.  **通讯确认**：确保 ESP32 的引脚映射与您修改好的 OnStep 固件相匹配，同时确认 ESP07S 的网络通讯连接正常。
3.  **上电自检**：接入电源，测试电磁抱闸是否能正常吸合/释放，并通过上位机监测 TMC5160 的运行状态及细分设置。

> 📷 *参考图片：*
> ![电控系统点亮与联机测试](https://github.com/LeoJohn233/DIY-Harmonic-Equatorial-Mount/blob/main/pictures/Assembly%20process/DSC%20(13).JPG)

---

## 🤝 开源协议与贡献
如果您在组装过程中发现了可优化的机械公差或更好的布线方案，欢迎提交 Issue 或 Pull Request。本项目采用 `GPLv3 + CERN-OHL-S` 双开源协议，感谢您对开源天文硬件社区的贡献！