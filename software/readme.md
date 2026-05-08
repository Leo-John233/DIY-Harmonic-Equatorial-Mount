# OnStep 优化补丁日志 (For N.I.N.A. & APP)

## 📌 背景与痛点

在使用 OnStep 驱动谐波赤道仪时，由于谐波减速器不具备物理离合、静摩擦力小且扭矩极大，如果在开机未回零（未建立绝对坐标）的情况下误触指令，或设备两端不平衡，极易导致设备撞台、甚至断电溜车砸毁设备,同时，在拦截这些危险指令时，如果处理不当，会引发 **LX200 通讯协议底层冲突**，导致以下严重问题：

1. **N.I.N.A. 串口崩溃**：强行返回自定义中文字符串会导致 ASCOM 驱动解析失败 (`Parsing Error`)。
2. **蓝牙极高延迟 (Log Flooding)**：使用 `return` 强行掐断通讯或频繁触发 `CE_PARKED` 错误，会导致底层日志系统疯狂刷屏，撑爆 ESP32 的蓝牙缓存，致使 APP 按键指令严重卡顿。
3. **幽灵跟踪**：GOTO 被拦截后，天文软件的保护机制或预处理逻辑会强行唤醒恒星跟踪。


---

## 🎯 功能特性

本补丁对 `Command.ino`、`OnStep.ino`、`guide.io` 及相关初始化逻辑进行了深度优化，实现了完美的“开机防撞锁”、“防溜车机制”与“脱困逻辑”：

* **开机防溜车抱闸**：通电瞬间自动使能电机并进入静止状态，利用电机的保持转矩（Holding Torque）充当电子抱闸，防止严重不平衡时的重力滑落。
* **物理级一键回零**：复用 ST4 导星接口，外接实体按键长按 3 秒即可触发寻找零位（Find Home），并带有硬件级防抖与电机脉冲屏蔽锁。
* **GOTO 拦截伪装**：开机未回零前拦截 GOTO，并向 N.I.N.A. 返回标准 `3` (Standby) 状态，优雅触发弹窗报错，强制关闭误唤醒的跟踪。
* **Tracking 拦截与防堵塞**：未回零前拒绝所有开启跟踪指令，回复标准 `0` (失败)，彻底消除 APP 死等造成的蓝牙高延迟。
* **方向键静默拦截**：未回零或触发限位时，静默锁死对应方向，伪装成无错误 (`CE_NONE`)，实现零延迟、无日志溢出的丝滑手控。
* **Unpark 拦截**：未回零前遵循 LX200 协议拒绝解除停放，防止意外启动。
* **智能限位脱困与单次急停**：撞击限位后仅触发单次急停报警（防刷屏），并仅锁死撞击方向，允许向反方向安全逃离。

---

## 🛠️ 代码修改指南

### 1. 开机自动使能电机防止“溜车”滑落 (修改初始化逻辑)

**定位**: 搜索 OnStep 初始化或开机默认跟踪状态配置的代码段（通常在 `setup()` 尾部或处理 `TrackingNone` 初始化的逻辑中）。
**修改逻辑**: 针对谐波减速器断电易滑落的物理特性，在开机瞬间强制物理激活步进电机驱动器。

```cpp
    // --- [修改开始] 开机防滑落抱死 ---
    // 1. 将状态设置为“TrackingNone”
    // 作用：系统进入工作状态（Active），但不发送步进脉冲，电机保持绝对静止
    trackingState = TrackingNone;
    
    // 2. 物理激活驱动器 
    // 作用：拉低 EN 引脚，让电机通电产生保持转矩（Holding Torque），充当电子抱闸
    enableStepperDrivers();
    // --- [修改结束] ---
```

### 2. 复用 ST4 接口实现物理“一键回零” (修改 `ST4` 逻辑)

**定位**: 搜索 `void ST4()` 函数内部，定位到 `// standard hand control` 下方区域。
**修改逻辑**: 巧妙利用闲置的 ST4 导星接口，外接物理按键（同时短接 East 和 West 引脚至 GND）。加入 `homingLockout` 防抖锁。

```cpp
  // standard hand control
  const long Shed_ms=4000;
  const long AltMode_ms=2000;

  // =================================================================
  // [新增] 长按 3 秒立即触发回零，附带按键屏蔽锁，保护电机脉冲
  // =================================================================
  static bool homingLockout = false; // 屏蔽锁状态标志

  // 1. 如果锁是开启的，说明已经触发了回原点，此时检测是否松手
  if (homingLockout) {
    // 只要有任意一个按键还按着，就会直接 return 退出，彻底屏蔽干扰！
    if (!st4e.isDown() && !st4w.isDown() && !st4n.isDown() && !st4s.isDown()) {
      homingLockout = false; 
    }
    return; // 拦截点，保护回原点过程不被打断
  }

  // 2. 长按检测逻辑
  if ((trackingState != TrackingMoveTo) && (!waitingHome)) {
    // 检测是否同时按下了东键(E)和西键(W)
    if (st4e.isDown() && st4w.isDown()) {
      
      // 如果按下的时间达到了 3000 毫秒（3秒）
      if ((st4e.timeDown() > 3000) && (st4w.timeDown() > 3000)) {
        homingLockout = true; // 第一步：立刻上锁，忽略接下来的所有按键状态
        soundBeep();          // 蜂鸣器滴一声，提示用户
        
        // 停止一切当前的追踪和导星动作
        stopGuideAxis1();
        stopGuideAxis2();
        stopSlewingAndTracking(SS_ALL_FAST);
        
        // 第二步：立刻触发回原点，不需要等松手
        goHome(true); 
        return; 
      }
      
      // 如果按下了，但是还没满 3 秒：
      // 需要阻止原本的 AltMode 或常规导星逻辑触发，叫停轴 1 并立刻返回等待计时
      stopGuideAxis1(); 
      return; 
    }
  }
  // =================================================================
```

### 3. 拦截 GOTO 并强制关闭跟踪 (修改 `Command.ino`)

**定位**: 搜索 `if (command[1] == 'S' && parameter[0] == 0)` (处理 `:MS#` 指令处)
**修改逻辑**: 伪装成待机状态 (`3`)，优雅弹窗，关闭跟踪，且不产生报错日志。

```cpp
      // :MS#       Goto the Target Object
      if (command[1] == 'S' && parameter[0] == 0)  {
        
        // --- [新增] 拦截 GOTO，伪装成 Standby (待机) 状态 ---
        if (!systemHasHomed) {
            reply[0] = '3'; // 返回 3，代表 controller in standby
            reply[1] = 0;
            boolReply = false;
            supress_frame = true;
            
            // 骗过 OnStep 底层日志系统，防止蓝牙卡顿
            commandError = CE_NONE; 

            // 顺手强制关闭可能被 APP 提前唤醒的跟踪功能
            trackingState = TrackingNone; 

        } else {
            // ... [保留原本的 GOTO 逻辑] ...
            newTargetRA=origTargetRA; newTargetDec=origTargetDec;
#if TELESCOPE_COORDINATES == TOPOCENTRIC
            topocentricToObservedPlace(&newTargetRA,&newTargetDec);
#endif
            CommandErrors e=goToEqu(newTargetRA,newTargetDec);
            if (e >= CE_GOTO_ERR_BELOW_HORIZON && e <= CE_GOTO_ERR_UNSPECIFIED) reply[0]=(char)(e-CE_GOTO_ERR_BELOW_HORIZON)+'1';
            if (e == CE_NONE) reply[0]='0';
            reply[1]=0;
            boolReply=false;
            supress_frame=true;
            commandError=e;
        }
      } else
```

### 4. 规范拦截开启跟踪消除闲置高延迟 (修改 `Command.ino`)

**定位**: 搜索 `if (command[0] == 'T'` 下方的 `if (command[1] == 'e') {` (即 `:Te#` 指令)
**修改逻辑**: 强制规范回复 `0`，防止 APP 死等导致蓝牙崩溃。

```cpp
        // :Te#       Tracking enable
        if (command[1] == 'e') {
          
          // --- [新增] 必须标准回复 0，否则 APP 死等导致蓝牙崩溃！ ---
          if (!systemHasHomed) {
              reply[0] = '0'; // 明确告诉 APP 被拒绝了
              reply[1] = 0; 
              boolReply = false; 
              supress_frame = true; 
              commandError = CE_NONE; // 保持静默，不刷日志
              return; 
          }
          // --------------------------------------------------------

          if (isParked()) commandError=CE_PARKED; else
          // ... [保留原有 trackingState 逻辑] ...
```

### 5. 拦截手动方向键实现零延迟静默锁 (修改 `Command.ino`)

**定位**: 搜索 `:Me#` 或 `Move Telescope East or West`
**修改逻辑**: 剥离 `return`，使用 `CE_NONE` 欺骗日志系统，实现静默锁死。

```cpp
      // :Me# :Mw#  Move Telescope East or West at current guide rate
      if ((command[1] == 'e' || command[1] == 'w') && parameter[0] == 0) {
        
        // --- [修改] 没回零，或者按了东且东面锁死，或者按了西且西面锁死 ---
        if (!systemHasHomed || (command[1] == 'e' && Axis1_LimitLock == 1) || (command[1] == 'w' && Axis1_LimitLock == -1)) {
            boolReply = false;       // 告诉底层：不用返回字符
            commandError = CE_NONE;  // 伪装成无错误！
        } else {
            commandError=startGuideAxis1(command[1],currentGuideRate,GUIDE_TIME_LIMIT*1000,false);
            boolReply=false;
        }
      } else
      
      // :Mn# :Ms#  Move Telescope North or South at current guide rate
      if ((command[1] == 'n' || command[1] == 's') && parameter[0] == 0) {
        
        // --- [修改] 没回零，或者按了北且北面锁死，或者按了南且南面锁死 ---
        if (!systemHasHomed || (command[1] == 'n' && Axis2_LimitLock == 1) || (command[1] == 's' && Axis2_LimitLock == -1)) {
            boolReply = false;       
            commandError = CE_NONE;  
        } else {
            commandError=startGuideAxis2(command[1],currentGuideRate,GUIDE_TIME_LIMIT*1000,false);
            boolReply=false;
        }
      } else
```


### 6. 优化智能脱困与单次急停防刷屏 (修改 `OnStep.ino`)

**定位**: 搜索 `loop2()` 中限位判断代码。
**修改逻辑**: 移除易导致意外锁死的坐标推断逻辑，增加单次报错锁防刷屏。

```cpp
        // =========================================================
        // 恢复纯粹的撞击方向锁死逻辑
        // =========================================================
        if (Axis1_LimitLock == 0 && currentMotionDir1 != 0) {
            Axis1_LimitLock = currentMotionDir1;
        }
        if (Axis2_LimitLock == 0 && currentMotionDir2 != 0) {
            Axis2_LimitLock = currentMotionDir2;
        }

        // =========================================================
        // 执行急停 (带有单次触发锁)
        // =========================================================
        if (!isEscaping) {
            // --- [核心修改] 加一把单次触发锁，防止刷屏撑爆蓝牙 ---
            if (generalError != ERR_LIMIT_SENSE) {
                generalError = ERR_LIMIT_SENSE;
                stopGuideAxis1(); 
                stopGuideAxis2();
                stopSlewingAndTracking(SS_LIMIT_HARD);
            }
        }
```
---

## 📌 优化：谐波大角度寻零补偿与全自动对齐

在完美解决了开机防溜车和防撞锁死之后，针对 DIY 谐波赤道仪的**物理零位标定**与**高减速比机械保护**，原生 OnStep 固件仍存在以下严重痛点：

1. **原生补偿范围极度受限**：OnStep 原生的 `Home Offset` 采用“角秒”为单位，且底层宏定义（`HOME_OFFSET_RANGE_AXIS1`）将其死锁在几千角秒（通常不到 1°）以内,DIY 谐波赤道仪的光电传感器安装误差往往超过 1°，导致原生补偿直接失效报错
2. **操作割裂且不连贯**：寻零完成（碰到传感器）后，赤道仪仅仅是停在触发点并重置内部坐标，必须依靠用户手动在星图软件中再次点击“GOTO 零位”才能回到绝对基准，极不优雅。
3. **高减速比下的机械冲击**：谐波减速器（100:1）叠加同步带（1:4）产生极高的总减速比（400:1）,原生默认的 GOTO 加速度和限位急停距离过短，极易在起步和触发限位时对柔轮产生毁灭性的反冲剪切力

---

## 🎯 进阶功能特性

本补丁对 `Constants.h`、`AxisTile.cpp`、`Home.ino` 及 `Config.h` 进行了修改，实现了：

* **UI 直观解锁**：在网页端提供以“度 (Degrees)”为单位的无限位直观偏移量输入，自动存储至 EEPROM。
* **寻零闭环全自动到位**：寻零结束瞬间完成坐标静默篡改，并**自动无缝唤醒 GOTO 引擎**，一步倒车/前进至绝对物理零位。

---

### 7. 分配寻零偏移量 EEPROM 存储地址 (修改 `Constants.h`)

**定位**: 搜索 `// rotator base address` 所在区域。
**修改逻辑**: 利用 `GSB` 尾部绝对安全的空闲内存块，为双轴偏移量分配浮点数（Float）存储地址，实现永久记忆。

```cpp
// offsets for the rotator
#define EE_rotSpos                      0  // 4
#define EE_rotBacklashPos               4  // 2
#define EE_rotBacklash                  6  // 2

// =========================================================
// --- [核心修改] 新增：寻零偏移量 EEPROM 地址 ---
// =========================================================
// 利用 GSB 尾部的绝对安全空闲空间，避开所有系统内置参数
#define EE_homeOffsetAxis1         GSB+182 // 占用 4 bytes (float)
#define EE_homeOffsetAxis2         GSB+186 // 占用 4 bytes (float)

// ---------------------------------------------------------------------------------------------------------------------------------
// Unique identifier for the current initialization format for NV, do not change
#define NV_INIT_KEY 915307551
```


### 8. 解锁网页端 UI 直观输入 (修改 AxisTile.cpp)

**定位**: 搜索 sendAxisParams(&a, 1);（针对 Axis 1）和 sendAxisParams(&a, 2);（针对 Axis 2）。
**修改逻辑**: 绕过原生的角秒级偏移量输入框，增加基于“度”的新输入控件，并利用 F() 宏和 L_HOME_OFFSET 实现多语言适配及内存优化。
```cpp
sprintf_P(temp, html_configAxisMax, (int)a.max, 1, 0, 360, "&deg;,");
            data.concat(temp);
            www.sendContentAndClear(data);

            // =========================================================
            // --- [核心修改] 新增：轴 1 寻零偏移量输入框 (支持多语言翻译) ---
            // =========================================================
            data.concat(F("<br/><label>" L_HOME_OFFSET " (Deg):</label>&nbsp;<input type='number' name='ho1' step='0.01' style='width:5em;' placeholder='0.0'>"));
            www.sendContentAndClear(data);
            
            sendAxisParams(&a, 1);

            data.concat(F("<br /><button type='submit'>" L_UPLOAD "</button> "));
```
*(注：Axis 2 处执行完全相同的修改，仅将 name='ho1' 改为 name='ho2'。后端数据提取逻辑由 Command.ino 承接)*


### 9. 重写底层寻零闭环与自动 GOTO 补偿 (修改 Home.ino)

**定位**: 搜索 if (findHomeMode == FH_DONE && guideDirAxis1 == 0 && guideDirAxis2 == 0) 的寻零结束事件。
**修改逻辑**: 拦截坐标初始化事件，挂起系统中断注入偏移量，并强制调用原生安全 goTo 接口完成闭环补偿。

```cpp
#else    
      // at the home position
      initStartPosition();

      // =========================================================
      // --- [核心修改] 寻零后执行坐标欺骗，并自动触发 GOTO 补偿 ---
      // =========================================================
      // 1. 读取网页下发的偏移量 (度)
      float offsetDeg1 = nv.readFloat(EE_homeOffsetAxis1);
      float offsetDeg2 = nv.readFloat(EE_homeOffsetAxis2);

      // 2. 数据防呆清洗 (防止未初始化 EEPROM 读出乱码导致电机暴走)
      if (isnan(offsetDeg1) || offsetDeg1 > 360.0 || offsetDeg1 < -360.0) offsetDeg1 = 0.0;
      if (isnan(offsetDeg2) || offsetDeg2 > 360.0 || offsetDeg2 < -360.0) offsetDeg2 = 0.0;

      // 3. 挂起中断，执行最高优先级的坐标系静默平移
      cli();
      long offsetSteps1 = (long)((double)offsetDeg1 * axis1Settings.stepsPerMeasure);
      long offsetSteps2 = (long)((double)offsetDeg2 * axis2Settings.stepsPerMeasure);

      posAxis1 -= offsetSteps1;
      posAxis2 -= offsetSteps2;
      targetAxis1.part.m -= offsetSteps1;
      targetAxis2.part.m -= offsetSteps2;
      sei();

      // 4. 恢复安全保护机制
      safetyLimitsOn = true;
      atHome = true;

      // 5. 核心魔法：如果存在偏差，立刻唤醒闭环 GOTO，平滑补偿至物理绝对零点！
      if (offsetSteps1 != 0 || offsetSteps2 != 0) {
          trackingState = TrackingNone; // 确保不进入恒星追踪逻辑
          goTo(homePositionAxis1, homePositionAxis2, homePositionAxis1, homePositionAxis2, PierSideEast);
      }
      // =========================================================
    #endif
  }
```
---

## 💡 总结与建议

经过上述修改，基于 OnStep 的谐波赤道仪如同加上了“物理级”的安全外骨骼。从开机通电的瞬间到未回零前的任何误操作，都被死死按在安全的边界内，且完美兼顾了 ASCOM 驱动和 LX200 协议的底层脾气。建议搭配 N.I.N.A. 等上位机软件使用时，养成开机首选 **Find Home** 的良好习惯，享受丝滑且安全的星空探索。