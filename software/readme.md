# OnStep 谐波赤道仪安全回零与限位脱困增强版

---

## 📝 摘要 (Summary)

本项目以 [原始 OnStep 4.24](https://github.com/hjd1964/OnStep/tree/release-4.24) 为对照基准，说明本地固件针对 MaxESP3 谐波赤道仪做出的实际修改

修改围绕三条主线展开，开机后先建立保持力矩并确认位置，运动中保留安全中断与反向脱困能力，驱动细分切换时保持双轴同步

具体改进包括开机电机保持、位置可信状态、GOTO 与 Tracking 安全锁、物理限位反向脱困、三阶段自动回零、GOTO 中断分类、客户端状态修复以及 MaxESP3 TMC2209 共享细分切换

本文只展示本地代码相对原始 OnStep 4.24 新增或改进的实现，不重复粘贴未修改的上游代码

---

## 📌 背景与痛点 (Pain Points)

原始 OnStep 4.24 是面向多种赤道仪和经纬仪的通用固件。将它用于当前 MaxESP3 谐波赤道仪时，还需要处理下面这些实际问题

1. **开机可能溜车**

   谐波赤道仪通常没有传统离合结构。镜筒失衡时，如果驱动器在上电初期没有保持力矩，机械结构可能在重力作用下滑落

2. **上电位置不能直接信任**

   开环步进系统断电后无法证明机械位置没有变化。Home 传感器启用时，开机后的坐标只能在真实回零完成后重新建立

3. **未回零时自动动作风险较高**

   N.I.N.A、ASCOM 或其他 LX200 客户端可能在连接后发送 GOTO、Sync、Park 或 Tracking 指令。如果当前位置不可信，自动动作可能造成错误指向、绕线或机械碰撞

4. **安全中断后不能恢复成正常到达**

   GOTO 因限位、驱动故障或回零失败而停止时，不能继续走正常完成流程，更不能在错误坐标上自动开始 Tracking

5. **触发物理限位后仍需要反向脱困**

   如果限位触发后锁死所有手动方向，设备会停在开关上无法退出。如果完全不限制方向，又可能继续向危险侧运动

6. **Home 开关位置不一定是真实机械零位**

   传感器安装位置与目标零位可能存在角度偏差，需要在快速搜索和慢速精找后继续执行可配置的双轴偏置

7. **回零失败不能伪装成成功**

   传感器未触发、阶段超时或电机启动失败时，系统必须保留位置恢复要求，不能只因为回零状态机结束就开放 GOTO

8. **MaxESP3 的共享模式引脚需要双轴同步处理**

   MaxESP3 的 Axis1 与 Axis2 共用 M0 和 M1。独立模式 TMC2209 如果按原始双轴独立逻辑切换细分，可能出现硬件细分已经改变而软件步距没有同步的问题

9. **首次连接可能误报 Home**

   `atHome` 只能说明内部状态位于 Home，不能单独证明当前机械位置可信。客户端状态必须同时检查位置恢复状态

10. **有无 Home 传感器都需要可恢复路径**

    有传感器时可以自动 Find Home。没有传感器时则需要保留坐标 Home 和人工 Set Home 的恢复方式，不能被新增安全锁永久限制

---

## ✨ 核心功能特性 (Key Features)

1. **开机电机保持**

   上电后使能双轴驱动器，使用跟踪细分和跟踪电流保持当前位置，但不启动 Tracking，也不声明已经完成 Home

2. **独立的位置可信状态**

   使用 `mountPositionTrusted` 与 `positionRecoveryRequired` 区分正常使用、仅允许坐标回零和位置完全不可信三种状态

3. **自动命令统一进入安全检查**

   GOTO、Sync、Park 和开启 Tracking 都依赖 `positionReady()`，不会因为命令入口不同而绕过回零要求

4. **未回零仍允许手动移动**

   手动 East、West、North、South 不依赖位置可信状态，方便用户找零、调整姿态和处理机械风险

5. **物理限位反向脱困**

   限位触发后记录危险方向，只拒绝继续撞击的一侧，允许相反方向手动退出

6. **三阶段自动回零**

   自动 Home 依次执行快速搜索、慢速精找和零位偏置，双轴全部停稳后才重建坐标

7. **ST4 长按一键回零**

   同时长按 East 和 West 超过 3 秒可启动快速回零，并通过按键锁避免残留输入干扰状态机

8. **GOTO 中断分类**

   区分普通停止、物理硬停止和位置丢失，按风险等级决定是否停止 Tracking 和要求重新 Home

9. **客户端状态修复**

   `:GU#` 只有在 `atHome` 与 `positionReady()` 同时成立时才返回 `H`。被拒绝的 `:Te#` 会立即回复失败，不再让客户端等待通信超时

10. **MaxESP3 TMC2209 共享细分支持**

    双轴共享 M0 和 M1 时统一判断 GOTO 模式，在可安全切换的位置同步更新硬件模式与软件步距

11. **配置校验与中文说明**

    新增开机策略、回零速度、偏置角度和共享细分规则校验，并在 `Config.h` 与 MaxESP3 引脚文件中补充中文说明

---

## 🛠️ 代码修改与源码对照 (Code Modification Guide)

下表汇总本地代码相对原始 OnStep 4.24 的 20 项修改。后文按相同编号给出修改代码和功能说明

| 编号 | 修改主题 | 主要文件 | 相对原始 OnStep 4.24 的变化 | 修改带来的功能 |
| --- | --- | --- | --- | --- |
| 1 | 开机保持与位置确认 | `Config.h` `OnStep.ino` | 新增开机回零要求和电机保持策略 | 上电后保持双轴但不启动 Tracking |
| 2 | 位置可信模型 | `OnStep.ino` | 新增位置可信状态 恢复状态和 GOTO 中断类型 | 区分正常使用 坐标回零恢复和位置丢失 |
| 3 | ST4 一键回零 | `Guide.ino` | 新增长按 East 与 West 启动回零的按键逻辑 | 无需上位机即可触发自动回零 |
| 4 | GOTO 与 Sync 安全锁 | `Goto.ino` | 将 `positionReady()` 纳入 GOTO 和 Sync 校验 | 位置恢复前禁止自动指向和同步 |
| 5 | Tracking 失败反馈 | `Command.ino` | 开启 Tracking 前检查位置并立即返回结果 | 避免错误启动和客户端通信超时 |
| 6 | Park 安全锁 | `Park.ino` | Park 前增加位置可信检查 | 防止使用错误坐标执行驻留 |
| 7 | Guide 反向脱困 | `Guide.ino` | 按限位方向拦截 Guide 并保留反向移动 | 阻止继续撞限位同时允许退出危险位置 |
| 8 | 物理限位方向记录 | `OnStep.ino` | 记录双轴危险方向并判断是否已经逃离限位 | 实现硬停止和反向移动后的自动解锁 |
| 9 | Home 状态修复 | `Command.ino` | `:GU#` 同时检查 `atHome` 和 `positionReady()` | 避免客户端首次连接误报已经回零 |
| 10 | 三阶段自动回零 | `Home.ino` | 新增慢速精找和零位偏置阶段 | 从找到传感器提升为恢复真实机械零位 |
| 11 | Home 参数配置 | `Config.h` `Home.ino` | 回零速度 超时和双轴偏置改为可配置 | 便于适配不同传动比和传感器位置 |
| 12 | 回零失败保护 | `Home.ino` | 阶段超时或电机启动失败时保留位置锁 | 防止失败流程被当作回零成功 |
| 13 | 回零成功判定 | `Home.ino` | 只在 `FH_DONE` 中重新建立可信位置 | 确保全部阶段完成后才开放自动动作 |
| 14 | 无传感器恢复路径 | `Home.ino` | `HOME_SENSE` 关闭时保留坐标 Home | 无传感器配置仍可完成位置恢复 |
| 15 | 人工 Set Home 恢复 | `Home.ino` | Set Home 后统一清除中断并恢复可信位置 | 提供明确的人工位置恢复入口 |
| 16 | 坐标 Home 中断保护 | `MoveTo.ino` | 检查坐标 Home 是否完整结束 | 中断的回零不会错误解除位置锁 |
| 17 | GOTO 分类收尾 | `Goto.ino` `MoveTo.ino` | 按正常到达 普通停止 硬停止和位置丢失分别处理 | 防止安全中断后自动进入 Tracking |
| 18 | 停止等级标记 | `OnStep.ino` | `stopSlewingAndTracking()` 按停止来源写入中断等级 | 按风险决定是否要求重新回零 |
| 19 | MaxESP3 引脚与校验 | `Pins.MaxESP3.h` `Validate.MaxESP3.h` `Validate.h` `Validate.TMC2209.h` | 重整引脚定义并增加共享模式配置校验 | 让固件配置匹配本地 MaxESP3 硬件 |
| 20 | TMC2209 双轴细分切换 | `Timer.ino` `StepMode.ino` | 共享 M0 和 M1 时同步切换双轴硬件模式与软件步距 | 在 Tracking 与 GOTO 间切换时保持双轴坐标一致 |

### 1. 开机自动使能电机并要求位置确认

- **对照基准**：原始 OnStep 4.24 没有独立的开机回零要求和电机保持选项
- **修改文件**：`Config.h`、`OnStep.ino`
- **修改逻辑**：新增三个彼此独立的策略开关，并在启动流程中分别处理位置状态和驱动器保持

```cpp
// Config.h
#define HOME_REQUIRED_ON_BOOT          ON
#define HOME_REQUIRED_AFTER_LIMIT      ON
#define MOTOR_HOLD_ON_BOOT             ON
```

```cpp
// OnStep.ino
#if HOME_REQUIRED_ON_BOOT == ON
  #if MOUNT_TYPE != ALTAZM
    #if HOME_SENSE == OFF
      requireCoordinateHomeRecovery();
    #else
      invalidatePositionReference();
    #endif

    gotoAbortState = GOTO_ABORT_NONE;
    trackingState = TrackingNone;
    lastTrackingState = TrackingNone;
    abortTrackingState = TrackingNone;
    safetyLimitsOn = false;
  #endif
#endif

#if MOTOR_HOLD_ON_BOOT == ON
  #if MOUNT_TYPE != ALTAZM
    enableStepperDrivers();
    axis1DriverTrackingMode(false);
    axis2DriverTrackingMode(false);
  #endif
#endif
```

**修改带来的功能**

开机后电机立即获得保持力矩。Tracking 保持关闭，有 Home 传感器时位置被标记为不可信，没有 Home 传感器时只保留返回坐标 Home 的能力

---

### 2. 新增位置可信状态与 GOTO 中断分类

- **对照基准**：原始 OnStep 4.24 主要依赖 `atHome`、`trackingState` 和 `generalError`，没有独立的位置可信模型
- **修改文件**：`OnStep.ino`
- **修改逻辑**：新增位置状态、恢复状态和 GOTO 中断类型

```cpp
enum GotoAbortState {
  GOTO_ABORT_NONE,
  GOTO_ABORT_STOPPED,
  GOTO_ABORT_HARD_STOP,
  GOTO_ABORT_POSITION_LOST
};

GotoAbortState gotoAbortState = GOTO_ABORT_NONE;
bool gotoStartTrackingOnSuccess = false;
bool mountPositionTrusted = true;
bool positionRecoveryRequired = false;
```

```cpp
bool positionReady() {
  return mountPositionTrusted && !positionRecoveryRequired;
}

bool positionHomeReturnOnly() {
  return mountPositionTrusted && positionRecoveryRequired;
}

void requireCoordinateHomeRecovery() {
  mountPositionTrusted = true;
  positionRecoveryRequired = true;
  gotoStartTrackingOnSuccess = false;
}

void invalidatePositionReference() {
  mountPositionTrusted = false;
  positionRecoveryRequired = true;
  gotoStartTrackingOnSuccess = false;
}

void completePositionRecovery() {
  mountPositionTrusted = true;
  positionRecoveryRequired = false;
  gotoAbortState = GOTO_ABORT_NONE;
  gotoStartTrackingOnSuccess = false;
  clearPhysicalLimitState();
}
```

**修改带来的功能**

停止运动不再等于位置可信。限位、故障、回零失败和正常停止可以进入不同的恢复路径，所有自动动作共用同一套位置判断

---

### 3. 复用 ST4 接口实现实体按键一键回零

- **对照基准**：原始 OnStep 4.24 的 East 与 West 长按用于 Alt Mode，没有 3 秒 Find Home 入口
- **修改文件**：`Guide.ino`
- **修改逻辑**：增加 3 秒组合键和释放锁，保留原有 2 秒 Alt Mode

```cpp
static bool homingLockout = false;

if (homingLockout) {
  if (!st4e.isDown() && !st4w.isDown() &&
      !st4n.isDown() && !st4s.isDown()) {
    homingLockout = false;
  }
  return;
}

if (trackingState != TrackingMoveTo && !waitingHome) {
  if (st4e.isDown() && st4w.isDown()) {
    if (st4e.timeDown() > 3000 && st4w.timeDown() > 3000) {
      homingLockout = true;
      soundBeep();
      altModeA = false;
      stopGuideAxis1();
      stopGuideAxis2();
      stopSlewingAndTracking(SS_ALL_FAST);
      goHome(true);
      return;
    }
  }
}
```

**修改带来的功能**

不连接电脑也能启动自动回零。触发后会清理 Alt Mode 和当前运动，全部按键松开后才恢复 ST4 输入，避免残留按压干扰回零

---

### 4. 统一拦截未恢复状态下的 GOTO 与 Sync

- **对照基准**：原始 `validateGoto()` 只检查 Park、电机使能、运动状态和驱动故障
- **修改文件**：`Goto.ino`
- **修改逻辑**：在所有 GOTO 与 Sync 共用的校验函数中加入 `positionReady()`

```cpp
CommandErrors validateGoto() {
  if (parkStatus != NotParked)                 return CE_SLEW_ERR_IN_PARK;
  if (!axis1Enabled)                           return CE_SLEW_ERR_IN_STANDBY;
  if (trackingSyncInProgress())                return CE_MOUNT_IN_MOTION;
  if (trackingState == TrackingMoveTo)         return CE_GOTO_ERR_GOTO;
  if (guideDirAxis1 || guideDirAxis2)          return CE_MOUNT_IN_MOTION;
  if (faultAxis1 || faultAxis2)                return CE_SLEW_ERR_HARDWARE_FAULT;
  if (!positionReady())                        return CE_SLEW_ERR_IN_STANDBY;
  return CE_NONE;
}
```

**修改带来的功能**

`:MS#`、Sync 和其他自动指向入口都无法绕过位置检查。位置未恢复时返回 standby，上游原有的地平线、双轴范围、中天和墩侧校验继续保留

---

### 5. Tracking 指令被拦截时立即回复失败

- **对照基准**：原始 `:Te#` 没有位置可信检查
- **修改文件**：`Command.ino`
- **修改逻辑**：只拦截开启 Tracking，不影响停止 Tracking，并保留布尔回复

```cpp
if (command[0] == 'T' && parameter[0] == 0) {
  const bool trackingBlockedUntilRecovery =
    command[1] == 'e' && !positionReady();

  if (trackingBlockedUntilRecovery) {
    commandError = CE_SLEW_ERR_IN_STANDBY;
  } else {
    // 原始 Tracking 命令继续处理
  }
}
```

**修改带来的功能**

位置不可信时 `:Te#` 不会启动恒星跟踪。命令处理器会立即返回 `0`，避免 N.I.N.A 或 ASCOM 一直等待到通信超时

---

### 6. Park 位置增加可信状态检查

- **对照基准**：原始 OnStep 4.24 在设备停止且驱动无故障时即可保存 Park 位置
- **修改文件**：`Park.ino`
- **修改逻辑**：保存 Park 前增加位置可信检查

```cpp
CommandErrors setPark() {
  if (parkStatus == ParkFailed)         return CE_PARK_FAILED;
  if (parkStatus == Parked)             return CE_PARKED;
  if (isSlewing())                      return CE_MOUNT_IN_MOTION;
  if (faultAxis1 || faultAxis2)         return CE_SLEW_ERR_HARDWARE_FAULT;
  if (!positionReady())                 return CE_SLEW_ERR_IN_STANDBY;

  // 原始 Park 保存逻辑继续执行
}
```

**修改带来的功能**

未回零或位置失信时不能保存错误的 Park 坐标，避免下一次启动从错误位置恢复

---

### 7. Guide 层只锁危险方向并允许反向脱困

- **对照基准**：原始 Guide 层遇到 `ERR_LIMIT_SENSE` 时没有物理限位方向锁
- **修改文件**：`Guide.ino`
- **修改逻辑**：Axis1 与 Axis2 分别检查危险方向，并为相反方向建立脱困例外

```cpp
bool escapingPhysicalLimitAxis1 = false;

#if LIMIT_SENSE != OFF
  if (direction == 'e' && Axis1_LimitLock == 1)
    return CE_SLEW_ERR_OUTSIDE_LIMITS;

  if (direction == 'w' && Axis1_LimitLock == -1)
    return CE_SLEW_ERR_OUTSIDE_LIMITS;

  escapingPhysicalLimitAxis1 =
    generalError == ERR_LIMIT_SENSE &&
    ((direction == 'e' && Axis1_LimitLock == -1) ||
     (direction == 'w' && Axis1_LimitLock == 1));
#endif
```

```cpp
bool escapingPhysicalLimitAxis2 = false;

#if LIMIT_SENSE != OFF
  if (direction == 'n' && Axis2_LimitLock == 1)
    return CE_SLEW_ERR_OUTSIDE_LIMITS;

  if (direction == 's' && Axis2_LimitLock == -1)
    return CE_SLEW_ERR_OUTSIDE_LIMITS;

  escapingPhysicalLimitAxis2 =
    generalError == ERR_LIMIT_SENSE &&
    ((direction == 'n' && Axis2_LimitLock == -1) ||
     (direction == 's' && Axis2_LimitLock == 1));
#endif
```

**修改带来的功能**

危险方向在 Guide 底层被拒绝，串口、ST4 和其他入口都不能绕过。相反方向可以越过 `ERR_LIMIT_SENSE` 完成脱困

---

### 8. loop2() 增加物理限位方向记录与双轴逃离判断

- **对照基准**：原始 OnStep 4.24 对 LimitPin 进行两次读取，确认触发后直接调用 `SS_LIMIT`
- **修改文件**：`OnStep.ino`
- **修改逻辑**：记录当前运动方向，结合轴位置推断静止触发方向，并判断双轴是否都在远离限位

```cpp
#if LIMIT_SENSE != OFF
  byte limit_reading = digitalRead(LimitPin);
  unsigned long currentTime = millis();

  if (limit_reading == LIMIT_SENSE_STATE) {
    lastLimitTriggerTime = currentTime;

    if (isHoming()) {
      Axis1_LimitLock = 0;
      Axis2_LimitLock = 0;
      return;
    }

    delay(2);
    if (digitalRead(LimitPin) == LIMIT_SENSE_STATE) {
      int currentMotionDir1 = 0;
      int currentMotionDir2 = 0;

      if (guideDirAxis1 == 'e') currentMotionDir1 = 1;
      else if (guideDirAxis1 == 'w') currentMotionDir1 = -1;

      if (guideDirAxis2 == 'n') currentMotionDir2 = 1;
      else if (guideDirAxis2 == 's') currentMotionDir2 = -1;

      if (trackingState == TrackingMoveTo) {
        if (targetAxis1.part.m < posAxis1) currentMotionDir1 = 1;
        else if (targetAxis1.part.m > posAxis1) currentMotionDir1 = -1;

        if (targetAxis2.part.m > posAxis2) currentMotionDir2 = 1;
        else if (targetAxis2.part.m < posAxis2) currentMotionDir2 = -1;
      }
```

```cpp
      if (Axis1_LimitLock == 0) {
        if (currentMotionDir1 != 0 && trackingState == TrackingMoveTo) {
          Axis1_LimitLock = currentMotionDir1;
        } else {
          long threshold = 500L;
          if (posAxis1 > threshold) Axis1_LimitLock = -1;
          else if (posAxis1 < -threshold) Axis1_LimitLock = 1;
          else if (currentMotionDir1 != 0)
            Axis1_LimitLock = currentMotionDir1;
        }
      }

      if (Axis2_LimitLock == 0) {
        if (currentMotionDir2 != 0 && trackingState == TrackingMoveTo) {
          Axis2_LimitLock = currentMotionDir2;
        } else {
          long threshold = 500L;
          if (posAxis2 > threshold) Axis2_LimitLock = 1;
          else if (posAxis2 < -threshold) Axis2_LimitLock = -1;
          else if (currentMotionDir2 != 0)
            Axis2_LimitLock = currentMotionDir2;
        }
      }

      const bool axis1MovingIntoLimit =
        Axis1_LimitLock != 0 && currentMotionDir1 == Axis1_LimitLock;

      const bool axis2MovingIntoLimit =
        Axis2_LimitLock != 0 && currentMotionDir2 == Axis2_LimitLock;

      const bool hasEscapeMotion =
        currentMotionDir1 != 0 || currentMotionDir2 != 0;

      const bool isEscaping =
        hasEscapeMotion && !axis1MovingIntoLimit && !axis2MovingIntoLimit;

      if (!isEscaping) {
        generalError = ERR_LIMIT_SENSE;
        stopGuideAxis1();
        stopGuideAxis2();
        stopSlewingAndTracking(SS_LIMIT_PHYSICAL);
      }
    }
  } else if (currentTime - lastLimitTriggerTime > 500) {
    if (guideDirAxis1 == 0 && trackingState != TrackingMoveTo)
      Axis1_LimitLock = 0;

    if (guideDirAxis2 == 0 && trackingState != TrackingMoveTo)
      Axis2_LimitLock = 0;
  }
#endif
```

**修改带来的功能**

GOTO 与手动移动都能记录危险侧。双轴中只要仍有一轴朝限位运动就执行停止，限位释放并停稳 500 毫秒后自动清除方向锁

---

### 9. 修复客户端首次连接误报已经回零

- **对照基准**：原始 `:GU#` 只根据 `atHome` 返回 `H`
- **修改文件**：`Command.ino`
- **修改逻辑**：Home 状态必须同时满足位置可信

```cpp
if (atHome && positionReady()) reply[i++] = 'H';
```

**修改带来的功能**

上电后即使 `atHome` 保留启动值，客户端也不会收到可信 Home。只有 Find Home 或 Set Home 完成并解除恢复锁后才返回 `H`

---

### 10. 将自动回零扩展为三阶段状态机

- **对照基准**：原始 OnStep 4.24 只有快速搜索、慢速精找和完成状态
- **修改文件**：`Home.ino`
- **修改逻辑**：增加第二次停稳状态和零位偏置状态

```cpp
enum findHomeModes {
  FH_OFF,
  FH_FAST,
  FH_IDLE,
  FH_SLOW,
  FH_IDLE2,
  FH_OFFSET,
  FH_DONE
};
```

```text
FH_FAST   快速寻找 Home 开关
FH_IDLE   等待双轴停止并进入慢速精找
FH_SLOW   慢速再次寻找 Home 开关
FH_IDLE2  等待双轴停止并进入零位偏置
FH_OFFSET 按配置角度移动到真实零位
FH_DONE   重建坐标并解除恢复锁
```

**修改带来的功能**

Home 开关位置不再被直接当作最终机械零位。传感器负责提供重复定位基准，第三阶段再补偿安装偏差

---

### 11. Home 速度、超时和双轴偏置改为可配置

- **对照基准**：原始快速阶段固定 8 档并按 180 度计算超时，慢速阶段固定 7 档和 30 秒超时
- **修改文件**：`Config.h`、`Home.ino`
- **修改逻辑**：速度档位与偏置角度由配置决定，阶段超时按角度范围换算

```cpp
// Config.h
#define HOME_FAST_RATE                  9
#define HOME_SLOW_RATE                  7
#define HOME_OFFSET_AXIS1            -1.5
#define HOME_OFFSET_AXIS2               1
#define HOME_OFFSET_RATE                8
```

```cpp
// Home.ino 快速阶段
double secPerDeg = 3600.0 / (double)guideRates[HOME_FAST_RATE];
findHomeTimeout = millis() +
  (unsigned long)(secPerDeg * 360.0 * 1000.0);

e = startGuideAxis1(a1, HOME_FAST_RATE, 0, false);
if (e == CE_NONE)
  e = startGuideAxis2(a2, HOME_FAST_RATE, 0, false, true);
```

```cpp
// Home.ino 慢速阶段
double secPerDeg = 3600.0 / (double)guideRates[HOME_SLOW_RATE];
findHomeTimeout = millis() +
  (unsigned long)(secPerDeg * 6.0 * 1000.0);

e = startGuideAxis1(a1, HOME_SLOW_RATE, 0, false);
if (e == CE_NONE)
  e = startGuideAxis2(a2, HOME_SLOW_RATE, 0, false, true);
```

```cpp
// Home.ino 偏置阶段
double secPerDeg = 3600.0 / (double)guideRates[HOME_OFFSET_RATE];

char dir1 = HOME_OFFSET_AXIS1 > 0 ? 'e' : 'w';
unsigned long duration1 =
  (unsigned long)(fabs(HOME_OFFSET_AXIS1) * secPerDeg * 1000.0);

char dir2 = HOME_OFFSET_AXIS2 > 0 ? 'n' : 's';
unsigned long duration2 =
  (unsigned long)(fabs(HOME_OFFSET_AXIS2) * secPerDeg * 1000.0);
```

**修改带来的功能**

第一阶段保持约 360 度搜索余量，第二阶段保持约 6 度精找范围。修改速度后超时保护仍与机械角度对应，双轴偏置可以分别设置方向和角度

---

### 12. 回零阶段超时或启动失败时保留位置锁

- **对照基准**：原始回零失败会停止流程，但没有独立的位置可信状态需要维护
- **修改文件**：`Home.ino`
- **修改逻辑**：第一、第二、第三阶段失败都退出 Home，并确保位置仍然不可信

```cpp
if ((long)(millis() - findHomeTimeout) > 0L ||
    (guideDirAxis1 == 0 && guideDirAxis2 == 0)) {

  if ((long)(millis() - findHomeTimeout) > 0L)
    generalError = ERR_LIMIT_SENSE;

  safetyLimitsOn = true;
  invalidatePositionReference();
  gotoAbortState = GOTO_ABORT_NONE;
  findHomeMode = FH_OFF;
}
```

```cpp
if (e1 != CE_NONE || e2 != CE_NONE) {
  findHomeMode = FH_OFF;
  safetyLimitsOn = true;
  generalError = ERR_LIMIT_SENSE;
  stopSlewingAndTracking(SS_ALL_FAST);
  VLF("MSG: Homing phase 3 failed");
  return;
}
```

```cpp
if (e != CE_NONE) {
  findHomeMode = FH_OFF;
  safetyLimitsOn = true;
  stopSlewingAndTracking(SS_ALL_FAST);
}
```

**修改带来的功能**

传感器未触发、搜索超时或偏置启动失败时不会继续进入 `FH_DONE`。GOTO 与 Tracking 仍保持锁定，用户可以重新 Find Home 或人工 Set Home

---

### 13. 只有 FH_DONE 才重新建立可信位置

- **对照基准**：原始完成状态只负责设置起始位置和 `atHome`
- **修改文件**：`Home.ino`
- **修改逻辑**：完成三阶段并停稳后统一清理恢复状态和限位方向锁

```cpp
if (findHomeMode == FH_DONE &&
    guideDirAxis1 == 0 && guideDirAxis2 == 0) {

  findHomeMode = FH_OFF;

  #if AXIS2_TANGENT_ARM == ON
    trackingState = abortTrackingState;
    cli();
    targetAxis2.part.m = 0;
    targetAxis2.part.f = 0;
    posAxis2 = 0;
    sei();
  #else
    initStartPosition();
    atHome = true;
  #endif

  completePositionRecovery();
  safetyLimitsOn = true;
  abortGoto = 0;
  lastTrackingState = TrackingNone;
  abortTrackingState = TrackingNone;
  generalError = ERR_NONE;
}
```

**修改带来的功能**

只有快速搜索、慢速精找和零位偏置全部成功后才开放自动动作。完成时同时清除旧限位锁、GOTO 中断和错误状态

---

### 14. HOME_SENSE 关闭时保留坐标 Home 恢复路径

- **对照基准**：新增位置安全锁后，无传感器设备需要一条不会被 `positionReady()` 永久阻断的回零路径
- **修改文件**：`Home.ino`
- **修改逻辑**：位置仍可信但要求恢复时，只允许原生坐标 Home 使用这份坐标

```cpp
#if HOME_SENSE == OFF
  const bool homeMayOverridePark = positionHomeReturnOnly();
#else
  const bool homeMayOverridePark = !positionReady();
#endif

if (parkStatus == Parked && homeMayOverridePark) {
  parkStatus = NotParked;
  nv.write(EE_parkStatus, parkStatus);
}
```

```cpp
#if HOME_SENSE == OFF
  const bool coordinateHomeSafe =
    parkStatus == NotParked &&
    !trackingSyncInProgress() &&
    trackingState != TrackingMoveTo &&
    guideDirAxis1 == 0 && guideDirAxis2 == 0 &&
    !faultAxis1 && !faultAxis2;

  if (e == CE_SLEW_ERR_IN_STANDBY &&
      coordinateHomeSafe && positionHomeReturnOnly()) {
    enableStepperDrivers();
    e = CE_NONE;
  }
#endif
```

**修改带来的功能**

关闭 Home 传感器时，设备仍能依靠可信的开环坐标返回 Home。若位置已经完全不可信，则必须由用户把机械结构放回零位后执行 Set Home

---

### 15. Set Home 成为明确的人工恢复入口

- **对照基准**：原始 Set Home 会重建起始位置，但不需要处理新增的位置恢复状态
- **修改文件**：`Home.ino`
- **修改逻辑**：用户确认机械零位后统一恢复位置可信并清理安全残留

```cpp
CommandErrors setHome() {
  if (isSlewing()) return CE_MOUNT_IN_MOTION;

  reactivateBacklashComp();
  initStartupValues();
  initStartPosition();
  safetyLimitsOn = true;
  StepperModeTrackingInit();

  // 原始 PEC 与 Park 状态处理继续执行

  completePositionRecovery();
  abortGoto = 0;
  generalError = ERR_NONE;
  return CE_NONE;
}
```

**修改带来的功能**

有无 Home 传感器都可以通过 Set Home 人工确认机械位置。该命令会解除位置恢复锁，但不会自动寻找零位，执行前必须由用户确认姿态正确

---

### 16. 坐标 Home 被中断时不解除恢复状态

- **对照基准**：原始 `MoveTo.ino` 在坐标 Home 结束后恢复 Tracking 并设置 `atHome`
- **修改文件**：`MoveTo.ino`
- **修改逻辑**：先检查 Home 运动是否被安全状态中断，再决定是否完成恢复

```cpp
if (homeMount) {
  const GotoAbortState completedHomeState = gotoAbortState;

  if (completedHomeState != GOTO_ABORT_NONE) {
    homeMount = false;
    trackingSyncSeconds = 0;
    trackingState = TrackingNone;
    lastTrackingState = TrackingNone;
    gotoAbortState = GOTO_ABORT_NONE;
    gotoStartTrackingOnSuccess = false;
    SiderealClockSetInterval(siderealInterval);
    setDeltaTrackingRate();
  } else {
    if (parkClearBacklash() == -1) return;
    soundAlert();
    trackingState = lastTrackingState;
    lastTrackingState = TrackingNone;
    SiderealClockSetInterval(siderealInterval);
    homeMount = false;
    if (AXIS2_TANGENT_ARM == OFF) atHome = true;
    completePositionRecovery();
    safetyLimitsOn = true;
  }
}
```

**修改带来的功能**

无传感器坐标 Home 只有正常完成时才恢复位置可信。被限位或其他安全状态中断时保持 Tracking 关闭，并继续要求用户恢复位置

---

### 17. GOTO 完成时按中断类型分别收尾

- **对照基准**：原始 `MoveTo.ino` 在 GOTO 结束后统一恢复 `lastTrackingState` 并报告完成
- **修改文件**：`Goto.ino`、`MoveTo.ino`
- **修改逻辑**：启动前记录一次性 Tracking 请求，结束后按正常完成、普通停止、物理停止和位置丢失分别处理

```cpp
// Goto.ino
const bool requestTrackingAfterSuccess =
  trackingState == TrackingNone &&
  timeWasSet && dateWasSet &&
  positionReady() &&
  parkStatus == NotParked &&
  !isHoming();

CommandErrors result =
  goTo(Axis1, Axis2, Axis1Alt, Axis2Alt, thisPierSide);

if (result == CE_NONE)
  gotoStartTrackingOnSuccess = requestTrackingAfterSuccess;
```

```cpp
// MoveTo.ino
if (completedGotoState == GOTO_ABORT_POSITION_LOST ||
    !positionReady()) {

  trackingState = TrackingNone;
  lastTrackingState = TrackingNone;
  if (generalError == ERR_NONE) generalError = ERR_LIMIT_SENSE;

} else if (completedGotoState == GOTO_ABORT_HARD_STOP) {

  trackingState = TrackingNone;
  lastTrackingState = TrackingNone;

} else {

  trackingState = lastTrackingState;
  lastTrackingState = TrackingNone;

  if (completedGotoState == GOTO_ABORT_NONE && startTracking)
    trackingState = TrackingSidereal;

  if (completedGotoState == GOTO_ABORT_NONE &&
      trackingState == TrackingSidereal) {
    trackingSyncSeconds = 5;
  }
}

gotoAbortState = GOTO_ABORT_NONE;
gotoStartTrackingOnSuccess = false;
```

**修改带来的功能**

安全中断不会再走正常 GOTO done 流程。只有正常到达才允许自动 Tracking 和 5 秒坐标同步窗口，避免出现失败 GOTO 后的幽灵 Tracking

---

### 18. stopSlewingAndTracking() 标记安全中断等级

- **对照基准**：原始函数只负责停止对应运动，没有位置恢复等级
- **修改文件**：`OnStep.ino`
- **修改逻辑**：驱动故障、物理限位和普通软件停止分别写入不同的 GOTO 中断类型

```cpp
if (ss == SS_LIMIT_HARD) {
  invalidatePositionReference();
  safetyLimitsOn = false;

  if (trackingState == TrackingMoveTo)
    gotoAbortState = GOTO_ABORT_POSITION_LOST;
}

if (ss == SS_LIMIT_PHYSICAL) {
  gotoStartTrackingOnSuccess = false;

#if HOME_REQUIRED_AFTER_LIMIT == ON
  #if HOME_SENSE == OFF
    requireCoordinateHomeRecovery();
  #else
    invalidatePositionReference();
  #endif

  safetyLimitsOn = false;
  if (trackingState == TrackingMoveTo)
    gotoAbortState = GOTO_ABORT_POSITION_LOST;
#else
  if (trackingState == TrackingMoveTo)
    gotoAbortState = GOTO_ABORT_HARD_STOP;
#endif
}

if (trackingState == TrackingMoveTo) {
  gotoStartTrackingOnSuccess = false;

  if (ss != SS_ALL_FAST && gotoAbortState == GOTO_ABORT_NONE)
    gotoAbortState = GOTO_ABORT_STOPPED;

  if (!abortGoto) abortGoto = StartAbortGoto;
}
```

**修改带来的功能**

更严重的位置丢失状态不会被后续普通停止覆盖。`HOME_REQUIRED_AFTER_LIMIT` 可以决定物理限位后是否强制重新 Home

---

### 19. 标记 MaxESP3 共享模式引脚并增加编译校验

- **对照基准**：原始 MaxESP3 定义了相同的 M0 与 M1 引脚，但没有声明独立模式驱动器需要双轴同步切换
- **修改文件**：`src/pinmaps/Pins.MaxESP3.h`、`src/pinmaps/Validate.MaxESP3.h`、`Validate.h`、`src/sd_drivers/Validate.TMC2209.h`
- **修改逻辑**：调整当前控制板引脚，显式标记共享模式引脚，并要求 TMC2209 双轴配置一致

```cpp
// Pins.MaxESP3.h 当前控制板引脚
#define Aux2                  2
#define Aux3                 21
#define Aux4                 22
#define Aux5                 25
#define Aux6                 15
#define Aux7                 39
#define Aux8                 12

#define PpsPin               36
#define LimitPin           Aux7

#define Axis1_EN              4
#define Axis1_STEP           18
#define Axis1_DIR            19
#define Axis1_HOME         Aux5

#define Axis2_EN         SHARED
#define Axis2_STEP           27
#define Axis2_DIR            26
#define Axis2_HOME         Aux6
```

```cpp
// Pins.MaxESP3.h
#define Axis1_M0 13
#define Axis1_M1 14
#define Axis2_M0 13
#define Axis2_M1 14

#define AXIS12_DRIVER_MODE_PINS_SHARED
```

```cpp
// Validate.h
#if defined(AXIS12_DRIVER_MODE_PINS_SHARED) && \
    (AXIS1_DRIVER_MODEL == TMC2209 || AXIS2_DRIVER_MODEL == TMC2209)

  #if AXIS1_DRIVER_MODEL != TMC2209 || AXIS2_DRIVER_MODEL != TMC2209
    #error "Configuration (Config.h): shared M0/M1 pins require TMC2209 on both Axis1 and Axis2."
  #endif

  #if AXIS1_DRIVER_MICROSTEPS != AXIS2_DRIVER_MICROSTEPS
    #error "Configuration (Config.h): shared TMC2209 M0/M1 pins require equal Axis1/Axis2 tracking microsteps."
  #endif

  #if AXIS1_DRIVER_MICROSTEPS_GOTO != AXIS2_DRIVER_MICROSTEPS_GOTO
    #error "Configuration (Config.h): shared TMC2209 M0/M1 pins require equal Axis1/Axis2 Goto microsteps."
  #endif

  #if MODE_SWITCH_BEFORE_SLEW != OFF
    #error "Configuration (Config.h): shared TMC2209 M0/M1 pins require on-the-fly mode switching."
  #endif

  #define AXIS12_TMC2209_MODE_SHARED
#endif
```

**修改带来的功能**

Home、Limit、PPS、Axis1 Enable 和 Axis1 Dir 已按当前控制板重新分配。对应的引脚占用检查也同步修改。双轴驱动型号和细分不一致时会在编译阶段报错，避免生成可以启动但运动比例错误的固件。当前仓库默认驱动仍为 `TMC5160_QUIET`，TMC2209 共享逻辑需要按实际硬件启用

---

### 20. MaxESP3 TMC2209 双轴同步切换细分

- **对照基准**：原始 Timer 逻辑分别判断 Axis1 与 Axis2 的 GOTO 模式，并在两个电机中断中独立切换
- **修改文件**：`Timer.ino`、`StepMode.ino`
- **修改逻辑**：任一轴需要 GOTO 细分时双轴共同进入 GOTO 模式，切换前等待两个轴都处于可表示位置

```cpp
// Timer.ino
gotoRateAxis1 =
  thisTimerRateAxis1 < AXIS1_DRIVER_SWITCH_RATE ||
  thisTimerRateAxis2 < AXIS2_DRIVER_SWITCH_RATE;

gotoRateAxis2 = gotoRateAxis1;
```

```cpp
bool axis1Ready =
  (!inbacklashAxis1 && posAxis1 == (long)targetAxis1.part.m) ||
  ((posAxis1 + blAxis1) % axis1StepsGoto == 0);

bool axis2Ready =
  (!inbacklashAxis2 && posAxis2 == (long)targetAxis2.part.m) ||
  ((posAxis2 + blAxis2) % axis2StepsGoto == 0);

if (!axis1Ready || !axis2Ready) return;

a1CLEAR;
a2CLEAR;

if (gotoRateAxis1) {
  axis1DriverGotoFast();
  gotoModeAxis1 = true;
  gotoModeAxis2 = true;
} else {
  axis1DriverTrackingFast();
  gotoModeAxis1 = false;
  gotoModeAxis2 = false;
}
```

```cpp
// StepMode.ino
#ifdef AXIS12_TMC2209_MODE_SHARED
  stepAxis1 = axis1StepsGoto;
  stepAxis2 = axis2StepsGoto;
#endif
```

**修改带来的功能**

一个轴先减速时不会单独改变另一个轴的硬件细分。双轴硬件模式与软件步距保持一致，MaxESP3 独立模式 TMC2209 可以使用 64 细分跟踪和 32 细分 GOTO

---

## 🔁 运行状态流程 (Runtime Flow)

### 正常开机流程

```text
上电
  ↓
MOTOR_HOLD_ON_BOOT 使能双轴驱动器
  ↓
Tracking 保持关闭
  ↓
HOME_REQUIRED_ON_BOOT 要求位置确认
  ↓
手动方向键仍可移动
  ↓
GOTO、Sync、Park 和 Tracking 被锁定
  ↓
执行 Find Home 或人工确认后的 Set Home
  ↓
positionReady() 返回 true
  ↓
开放自动指向与跟踪
```

### 自动 Find Home 流程

```text
客户端 Find Home
或 ST4 East 与 West 长按 3 秒
  ↓
FH_FAST 快速寻找 Home 开关
  ↓
FH_IDLE 等待双轴停止
  ↓
FH_SLOW 慢速精找 Home 开关
  ↓
FH_IDLE2 等待双轴停止
  ↓
FH_OFFSET 执行双轴零位偏置
  ↓
FH_DONE 重建起始坐标
  ↓
completePositionRecovery()
  ↓
允许 GOTO 与 Tracking
```

### 限位触发流程

```text
LimitPin 稳定触发
  ↓
读取手动方向或 GOTO 目标方向
  ↓
记录 Axis1_LimitLock 与 Axis2_LimitLock
  ↓
判断双轴是否都在远离限位
  ↓
仍有轴朝危险方向运动
  ↓
停止 Guide、GOTO 与 Tracking
  ↓
根据 HOME_REQUIRED_AFTER_LIMIT 标记位置恢复要求
  ↓
允许相反方向手动脱困
  ↓
Find Home 或 Set Home 后恢复自动动作
```

---

## ✅ 当前代码行为总表

| 功能场景 | 当前行为 |
| --- | --- |
| 开机 | 双轴电机保持，Tracking 关闭，不自动建立 Home |
| 未 Home 前手动移动 | 允许 |
| 未 Home 前 GOTO 与 Sync | 返回 standby |
| 未 Home 前开启 Tracking | 立即返回失败 |
| 未 Home 前保存 Park | 拒绝 |
| 客户端首次连接 | 只有位置可信时才报告 `H` |
| ST4 East 与 West 长按 3 秒 | 启动快速 Find Home |
| 第一阶段 Home | 按配置速度搜索约 360 度范围 |
| 第二阶段 Home | 按配置速度精找约 6 度范围 |
| 第三阶段 Home | 按双轴角度和速度执行零位偏置 |
| Home 任一阶段失败 | 停止流程并保留位置恢复锁 |
| Home 完成 | 重建坐标并开放自动动作 |
| Set Home | 人工确认当前位置为机械零位并恢复位置可信 |
| HOME_SENSE 关闭 | 可信坐标可用于返回 Home，完全失信时需要 Set Home |
| 物理限位触发 | 记录危险方向并停止危险运动 |
| 限位后继续向危险侧移动 | 拒绝 |
| 限位后向相反方向移动 | 允许脱困 |
| 限位释放并停稳 500 毫秒 | 清除方向锁 |
| 驱动器硬故障 | 位置标记为不可信并停止运动 |
| GOTO 普通停止 | 不报告正常到达，不启动新的 Tracking |
| GOTO 位置丢失 | Tracking 关闭并要求 Home 或 Set Home |
| MaxESP3 TMC2209 共享细分 | 双轴同步切换硬件模式和软件步距 |

---

## 💡 总结与建议

本地固件相对原始 OnStep 4.24 的重点不是增加更多自动动作，而是让每个自动动作都建立在可信位置上

- 开机只保持电机，不默认相信坐标
- GOTO、Sync、Park 和 Tracking 共用位置安全检查
- 手动方向保留，便于找零和处理机械风险
- 限位后只锁危险方向，允许反向脱困
- Home 失败不会误报成功
- GOTO 安全中断不会恢复成正常到达
- MaxESP3 TMC2209 共享细分由双轴同步处理

当前仓库带有一套具体硬件配置

```text
PINMAP                    MaxESP3
MOUNT_TYPE                GEM
TIME_LOCATION_SOURCE      DS3231
AXIS1_DRIVER_MODEL        TMC5160_QUIET
AXIS2_DRIVER_MODEL        TMC5160_QUIET
AXIS1_DRIVER_MICROSTEPS   64
AXIS2_DRIVER_MICROSTEPS   64
AXIS1_DRIVER_MICROSTEPS_GOTO 32
AXIS2_DRIVER_MICROSTEPS_GOTO 32
SLEW_RATE_BASE_DESIRED    6.0 度每秒
AXIS1_LIMIT               -180 到 180 度
AXIS2_LIMIT               -90 到 90 度
```

这套配置只适用于当前设备。刷写前应重新确认每度步数、驱动电流、电机方向、Home 极性、Limit 极性、回零偏置、软件范围和 MaxESP3 引脚。尤其不要把当前 Axis1 的正负 180 度直接视为其他赤道仪的安全范围

推荐第一次测试时卸下镜筒或保持可随时断电，先低速验证双轴方向，再验证 Home 和 Limit。回零期间普通 `LimitPin` 处理会让位于 Home 状态机，因此传感器极性、回零方向和机械余量必须在正式使用前确认

触发限位后先向相反方向手动脱困，再执行 Find Home。只有在人工确认赤道仪已经位于定义的机械零位时才使用 Set Home

固件继续使用 OnStep 的 LX200 派生命令集，可以配合 OnStep App、ASCOM、N.I.N.A、Stellarium、SkySafari、INDI 和其他兼容客户端使用

感谢 Howard Dutton 和 OnStep 社区维护原始项目。本地固件是 OnStep 4.24 的衍生版本，继续遵循 GNU General Public License，许可说明见 [LICENSE.txt](./LICENSE.txt)
