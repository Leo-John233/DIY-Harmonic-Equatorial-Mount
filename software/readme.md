# OnStep 谐波赤道仪深度安全与控制优化补丁 (For N.I.N.A. & APP)

## 📝 摘要 (Summary)

本次提交对 OnStep 底层控制逻辑与 LX200 通讯协议进行了深度定制与重构。主要针对 谐波赤道仪 的物理特性（无物理离合、极易溜车、高减速比反冲）以及上位机软件（N.I.N.A. / ASCOM / 手机 APP）的协同痛点，引入了 开机电子抱闸防溜车、协议级静默拦截锁、限位智能脱困机制 以及大角度寻零偏置与全自动闭环对齐等核心功能，彻底解决了误操作导致的设备撞台、蓝牙数据堵塞卡死等致命问题。

---

## 📌 背景与痛点 (Pain Points)

在使用原生 OnStep 驱动谐波赤道仪时，面临以下严重的软硬件协同问题：

1. **断电/待机溜车砸毁风险**：谐波减速器不具备物理离合，静摩擦力极小。若开机未建立绝对坐标时误触指令，或设备两端严重不平衡，极易导致断电溜车砸毁设备

2. **LX200 协议通讯堵塞 (Log Flooding)**：在拦截未回零的危险指令时，若粗暴使用 return 掐断通讯或频繁触发 CE_PARKED 错误，会导致底层日志系统疯狂刷屏，撑爆 ESP32 的蓝牙缓存，致使 APP 按键指令严重卡顿（极高延迟）

3. **N.I.N.A. 串口崩溃与幽灵跟踪**：强行返回非标准字符串会导致 ASCOM 驱动解析失败 (Parsing Error)；而拦截 GOTO 后，天文软件的预处理逻辑往往会强行唤醒恒星跟踪（幽灵跟踪）

4. **原生寻零偏置严重受限**：原生 Home Offset 采用角秒单位，补偿范围极小（不到 1°），而 DIY谐波赤道仪的光电传感器由于本身结构，导致实际回零位误差往往超过 1°，导致原生补偿直接失效，同时，原生寻零结束后不会自动回到绝对零点，操作割裂

5. **核心逻辑解耦与传感器宏绑定 (Sensor Macro Binding)**：为了保证固件的普适性，开机防撞锁机制实现了与硬件配置的动态智能绑定，完美兼容了带有或不带有限位传感器的赤道仪

---

## ✨ 核心功能特性 (Key Features)
1. **开机防溜车电子抱闸**：通电瞬间自动使能电机并进入静止状态，利用步进电机的保持转矩（Holding Torque）充当电子抱闸，防止重力滑落

2. **复用 ST4 物理一键回零**：外接实体按键长按 3 秒即可触发寻找零位（Find Home），附带硬件级防抖与电机脉冲屏蔽锁，防止回零过程被打断

3. **LX200 协议深度静默拦截**：   
  - (1) GOTO 伪装拦截：开机未回零前拦截 GOTO，向 N.I.N.A. 返回标准3(Standby) 状态，优雅报错并强制关闭误唤醒的跟踪
  - (2) Tracking 防堵塞：未回零前拒绝开启跟踪，标准回复 0 (失败)，消除 APP 死等造成的蓝牙断连
  - (3) 方向键静默锁死：未回零或触发限位时，伪装成无错误 (CE_NONE) 并不予回复，实现零延迟、无日志溢出的丝滑手控拦截
4. **限位智能脱困(Smart Escape)**：撞击限位后仅触发单次急停报警（防刷屏），智能记录并仅锁死撞击方向，允许向反方向安全逃离

5. **寻零闭环全自动到位**：支持通过网页 UI 输入以“度 (Degrees)”为单位的无限位直观偏移量，寻零结束瞬间完成 EEPROM 读取与坐标静默篡改，并自动无缝唤醒 GOTO 引擎，平滑移
动至绝对物理零位

6. **动态宏判断解耦**：在 Globals.h 中，开机状态锁变量 systemHasHomed 直接与 Config.h 中的 HOME_SENSE 宏进行绑定

---

## 🛠️ 代码修改与源码对照 (Code Modification Guide)

### 1. 开机自动使能电机防止“溜车”滑落

- **文件**: `OnStep.ino` (初始化逻辑段)
- **修改逻辑**: 在开机瞬间强制使能步进电机驱动器，产生保持转矩，充当电子抱闸，防止重力滑落。注意，这里不直接进入 Tracking 状态，而是保持在 TrackingNone，确保电机通电但不产生任何运动指令

```cpp
// --- [修改开始] 开机防滑落抱死 ---
    trackingState = TrackingNone;  // 系统进入Active状态，但不发脉冲，电机绝对静止
    enableStepperDrivers();        // 物理激活驱动器，拉低 EN 引脚产生电子抱闸
// --- [修改结束] ---
```

### 2. 复用 ST4 接口实现物理“一键回零”

- **文件**: `OnStep.ino` -> `void ST4()` 函数
- **修改逻辑**: 巧妙利用闲置的 ST4 导星接口，外接物理按键(同时短接 East 和 West 引脚至 GND),加入 `homingLockout` 防抖锁

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

- **文件**: `Command.ino` 搜索 `if (command[1] == 'S' && parameter[0] == 0)` (处理 `:MS#` 指令处)
- **修改逻辑**: 伪装成待机状态 (`3`)，优雅弹窗，关闭跟踪，且不产生报错日志

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

### 4. 规范拦截开启跟踪消除闲置高延迟

- **文件**: `Command.ino` 搜索 `if (command[0] == 'T'` 下方的 `if (command[1] == 'e') {` (即 `:Te#` 指令)
- **修改逻辑**: 强制规范回复 `0`，防止 APP 死等导致蓝牙崩溃

```cpp
// --- [新增] 必须标准回复 0，否则 APP 死等导致蓝牙崩溃！ ---
          if (!systemHasHomed) {
              reply[0] = '0'; // 明确告诉 APP 被拒绝了
              reply[1] = 0; 
              boolReply = false; 
              supress_frame = true; 
              commandError = CE_NONE; // 保持静默，不刷日志
              // 必须用 else 包裹原逻辑，不能直接 return 导致跳过发送步骤
          } else {
             // ... [保留原有 trackingState 逻辑] ...
```

### 5. 拦截手动方向键实现零延迟静默锁死

- **文件**: `Command.ino` 搜索 `:Me#` 或 `Move Telescope East or West`
- **修改逻辑**: 剥离 `return`，使用 `CE_NONE` 欺骗日志系统，实现静默锁死

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


### 6. 优化智能脱困与单次急停防刷屏

- **文件**:`OnStep.ino` 搜索 `loop2()` 中限位判断代码
- **修改逻辑**: 移除易导致意外锁死的坐标推断逻辑，增加单次报错锁防刷屏

```cpp
#if LIMIT_SENSE != OFF
    byte limit_reading = digitalRead(LimitPin);
    unsigned long currentTime = millis(); 
    
    // 请确保这三个变量在外部定义为了全局变量，或者在这里加上 static 关键字
    static unsigned long lastLimitTriggerTime = 0;
    static int Axis1_LimitLock = 0;
    static int Axis2_LimitLock = 0;

    // [检测到限位触发]
    if (limit_reading == LIMIT_SENSE_STATE) {
      
      delayMicroseconds(50); 
      if (digitalRead(LimitPin) == LIMIT_SENSE_STATE) {
        
        lastLimitTriggerTime = currentTime;

        // =========================================================
        // 1. 【最高权限】回零模式保护拦截
        // =========================================================
        // 使用全局函数 isHoming() 如果在寻找原点时撞击物理限位，执行最高级别硬刹车
        if (isHoming()) {
            generalError = ERR_LIMIT_SENSE;
            stopSlewingAndTracking(SS_LIMIT_HARD); // 强行切断电机脉冲
            return; // 立即退出 Home.ino 的 checkHome() 
        }

        // =========================================================
        // 2. 统一方向定义
        // =========================================================
        int currentMotionDir1 = 0;
        int currentMotionDir2 = 0;

        // --- Axis 1 (RA) ---
        if (guideDirAxis1 == 'e') currentMotionDir1 = 1;       
        else if (guideDirAxis1 == 'w') currentMotionDir1 = -1; 
        else if (trackingState == TrackingMoveTo) {            
             if (targetAxis1.part.m < posAxis1) currentMotionDir1 = 1; 
             else if (targetAxis1.part.m > posAxis1) currentMotionDir1 = -1; 
        }

        // --- Axis 2 (DEC) ---
        if (guideDirAxis2 == 'n') currentMotionDir2 = 1;       
        else if (guideDirAxis2 == 's') currentMotionDir2 = -1; 
        else if (trackingState == TrackingMoveTo) {            
             if (targetAxis2.part.m > posAxis2) currentMotionDir2 = 1; 
             else if (targetAxis2.part.m < posAxis2) currentMotionDir2 = -1; 
        }

        // =========================================================
        // 3. 智能记录锁死方向 
        // =========================================================
        // --- Axis 1 智能判断 ---
        if (Axis1_LimitLock == 0) {
            if (currentMotionDir1 != 0 && trackingState == TrackingMoveTo) {
                Axis1_LimitLock = currentMotionDir1;
            } else {
                long threshold = 500L; 
                if (posAxis1 > threshold) Axis1_LimitLock = -1;       
                else if (posAxis1 < -threshold) Axis1_LimitLock = 1;  
                else if (currentMotionDir1 != 0) Axis1_LimitLock = currentMotionDir1; 
            }
        }

        // --- Axis 2 智能判断 ---
        if (Axis2_LimitLock == 0) {
            if (currentMotionDir2 != 0 && trackingState == TrackingMoveTo) {
                Axis2_LimitLock = currentMotionDir2;
            } else {
                long threshold = 500L;
                if (posAxis2 > threshold) Axis2_LimitLock = 1;        
                else if (posAxis2 < -threshold) Axis2_LimitLock = -1; 
                else if (currentMotionDir2 != 0) Axis2_LimitLock = currentMotionDir2;
            }
        }

        // =========================================================
        // 4. 逃离判断 (Escape Logic)
        // =========================================================
        bool isEscaping = false;
        
        if (Axis1_LimitLock != 0 && currentMotionDir1 != 0 && currentMotionDir1 != Axis1_LimitLock) isEscaping = true;
        if (Axis2_LimitLock != 0 && currentMotionDir2 != 0 && currentMotionDir2 != Axis2_LimitLock) isEscaping = true;

        // =========================================================
        // 5. 执行急停
        // =========================================================
        if (!isEscaping) {
            generalError = ERR_LIMIT_SENSE;
            stopGuideAxis1(); 
            stopGuideAxis2();
            stopSlewingAndTracking(SS_LIMIT_HARD);
        }
      } 
    } else {
       // [限位松开]
       if (currentTime - lastLimitTriggerTime > 500) {
           if (guideDirAxis1 == 0 && trackingState != TrackingMoveTo) Axis1_LimitLock = 0;
           if (guideDirAxis2 == 0 && trackingState != TrackingMoveTo) Axis2_LimitLock = 0;
       }
    }
#endif
```


### 7. 修改 Globals.h (添加全局状态锁与传感器绑定)

- **文件**：打开`Globals.h`
- **修改逻辑**: 拉到 `Globals.h` 文件的最底部（在最后一行）

```cpp
// =========================================================
// --- [自定义] 限位锁死与强制回零状态 ---
int Axis1_LimitLock = 0;
int Axis2_LimitLock = 0;
unsigned long lastLimitTriggerTime = 0; 

// 传感器动态宏绑定
#if HOME_SENSE != OFF
  bool systemHasHomed = false; // 启用了传感器：开机上锁，必须手动回零
#else
  bool systemHasHomed = true;  // 未启用传感器：开机直接解锁
#endif
// =========================================================
```


### 8. 分配寻零偏移量 EEPROM 存储地址

- **文件**: `Constants.h`
- **修改逻辑**: 在 `Constants.h` 中为双轴偏移量分配 EEPROM 存储地址
```cpp
// =========================================================
// --- [核心修改] 新增：寻零偏移量 EEPROM 地址 ---
// =========================================================
// 利用 GSB 尾部的绝对安全空闲空间，完美避开所有系统内置参数
#define EE_homeOffsetAxis1         GSB+182 // 占用 4 bytes (float)
#define EE_homeOffsetAxis2         GSB+186 // 占用 4 bytes (float)
```


### 9. 网页端 UI 直观输入偏移量 (度)

- **文件**: `AxisTile.cpp`
- **修改逻辑**: 在 Axis 1 和 Axis 2 的参数配置界面中，增加以“度”为单位的寻零偏移量输入框，并支持多语言适配(并配合 `Command.ino` 中的 `:SHO1 / :SHO2` 指令解析存入 `EEPROM`)

```cpp
// ================== 新增：轴1 寻零偏移量输入框 (支持翻译) ==================
// 利用字符串拼接： "<br/><label>" + "与零位偏移:" + " (Deg):</label>..."
#if HOME_OFFSET_AUTO_GOTO == ON
sprintf_P(temp, PSTR("<br/>%s <input type='number' name='ho1' step='0.01' style='width:5em;' placeholder='0.0'>&deg;,"), L_HOME_OFFSET);
data.concat(temp);
www.sendContentAndClear(data);
#endif
// ================== 新增：轴2 寻零偏移量输入框 (支持翻译) ==================
#if HOME_OFFSET_AUTO_GOTO == ON
sprintf_P(temp, PSTR("<br/>%s <input type='number' name='ho2' step='0.01' style='width:5em;' placeholder='0.0'>&deg;,"), L_HOME_OFFSET);
data.concat(temp);
www.sendContentAndClear(data);
#endif
// ========================================================================
```


### 10. 寻零闭环坐标篡改与自动 GOTO 补偿

- **文件**: `Home.ino` -> `checkHome()` 的 `FH_DONE` 分支
- **修改逻辑**: 在寻零结束事件中，注入 EEPROM 读取的偏移量，静默篡改坐标，并强制调用原生 GOTO 引擎实现全自动闭环对齐

```cpp
// =========================================================
      // --- [核心修改] 寻零后执行坐标欺骗，并自动触发 GOTO 补偿 ---
      // =========================================================
      float offsetDeg1 = nv.readFloat(EE_homeOffsetAxis1);
      float offsetDeg2 = nv.readFloat(EE_homeOffsetAxis2);

      if (isnan(offsetDeg1) || offsetDeg1 > 360.0 || offsetDeg1 < -360.0) offsetDeg1 = 0.0;
      if (isnan(offsetDeg2) || offsetDeg2 > 360.0 || offsetDeg2 < -360.0) offsetDeg2 = 0.0;

      cli(); // 挂起中断，执行最高优先级的坐标系静默平移
      long offsetSteps1 = (long)((double)offsetDeg1 * axis1Settings.stepsPerMeasure);
      long offsetSteps2 = (long)((double)offsetDeg2 * axis2Settings.stepsPerMeasure);
      posAxis1 -= offsetSteps1; posAxis2 -= offsetSteps2;
      targetAxis1.part.m -= offsetSteps1; targetAxis2.part.m -= offsetSteps2;
      sei();

      safetyLimitsOn = true;
      atHome = true;

      // 如果存在偏差，立刻唤醒闭环 GOTO，平滑补偿至物理绝对零点！
      if (offsetSteps1 != 0 || offsetSteps2 != 0) {
          trackingState = TrackingNone; 
          goTo(homePositionAxis1, homePositionAxis2, homePositionAxis1, homePositionAxis2, PierSideEast);
      }
```


### 11. 修改 `Command.ino` (协议深度静默拦截)

- **文件**: `Command.ino`
- **修改逻辑**： 搜索 `if (command[1] == 'h')` (这是处理 `:Sh#` 指令的地方)，找到它对应的 `} else` 结尾，紧接着它的下面，插入自定义的 `:SHO` 指令：

```cpp
// ========================== 【自定义寻零偏移量接收】 =======================
      if (command[1] == 'H' && parameter[0] == 'O') {
        int axis = parameter[1] - '0';
        float val = atof(&parameter[2]);
        if (axis == 1) { 
            nv.writeFloat(EE_homeOffsetAxis1, val);
            boolReply = false;
        } else if (axis == 2) { 
            nv.writeFloat(EE_homeOffsetAxis2, val);
            boolReply = false;
        } else {
            commandError = CE_CMD_UNKNOWN;
        }
      } else
// =======================================================================
```

---

## 💡 总结与建议 
经过上述深度修改，基于 OnStep 构建的谐波赤道仪如同加上了“物理级”的安全外骨骼,从开机通电的瞬间到未回零前的任何误操作，都被死死限制在安全的边界内，且完美兼顾了 ASCOM 驱动和 LX200 协议的底层特性(零延迟、无报错) <br>
&emsp;&emsp; **日常使用建议**：搭配 N.I.N.A. 等上位机软件使用时，养成开机首选 Find Home (寻找原点) 的良好习惯，享受丝滑且绝对安全的星空探索之旅