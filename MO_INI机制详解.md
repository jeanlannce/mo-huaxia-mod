# MO INI 机制详解

> 基于 Mental Omega 3.3.6 (Ares 引擎) 的 INI 关键机制整理

---

## 一、NavalTargeting / LandTargeting 详解

控制单位武器对海军和陆地目标的攻击行为。
**（2026-08-18 依据 ModEnc 权威值修正，旧表有误导）**

### NavalTargeting 值含义（ModEnc 权威）

| 值 | 含义 | 说明 |
|----|------|------|
| 0 | 不能攻击**未显形的潜艇**；普通海军用主武器正常攻击 | 默认值（"UNDERWATER_NEVER"） |
| 1 | **副武器**攻击潜艇（主武器正常打普通海军） | "UNDERWATER_SECONDARY" |
| 2 | **只能用主武器**攻击潜艇，不能攻击其他任何目标 | "UNDERWATER_ONLY" |
| 3 | 对有机(Organic=yes)/非自然(Unnatural=yes)单位用**副武器**，其他用主武器 | "ORGANIC_SECONDARY" |
| 4 | 对有机/悬浮(Hover)单位用**主武器**，其他用副武器 | "SEAL_SPECIAL" |
| 5 | **主武器攻击所有海军（含潜艇）**——反潜舰标准配置 | "NAVAL_ALL" |
| 6 | **不能攻击水面/水中单位**（防空炮对海无效） | "NAVAL_NONE" |
| 7 | 与 5 无实质区别（Westwood 注释误导，勿用） | — |

> ⚠️ 常见误解纠正：值 0 不是"不能攻击海军"（普通海军照样打）；值 6 不是"另一种反潜"，恰恰相反是**禁用对海攻击**。

### LandTargeting 值含义（ModEnc 权威）

| 值 | 含义 |
|----|------|
| 0 | 陆地正常（主武器打陆地） |
| 1 | **不能攻击陆地**（纯防空/纯对海） |
| 2 | **只能副武器打陆地**（主武器不能打陆地） |

### 实战示例（已按权威修正注释）

```ini
; 原版战列舰 — 主炮打海军（不含潜艇），副炮打陆地
[HCRUIS]
NavalTargeting=0
LandTargeting=2
; 结果：Primary 打普通海军，Secondary 打陆地

; 剑鱼反潜舰 — 主炮即可打潜艇
[SWORD]
NavalTargeting=5
; 结果：Primary 可攻击所有海军单位包括潜艇

; 海狼炮艇 — 主炮打陆+反潜，副炮防空
[SWLFHX]
NavalTargeting=5
LandTargeting=0
; 结果：Primary 打陆地+海军/潜艇，Secondary 正常防空
```

---

## 二、Sensors 反隐/反潜探测

```ini
Sensors=yes          ; 启用探测器
SensorsSight=9       ; 探测范围（格）
```

- 可探测隐形单位、潜水单位、伪装单位
- 独立于 `Sight`（视野），不影响正常视野范围
- 驱逐舰 DEST、剑鱼 SWORD 等反潜单位均配备此功能

---

## 三、Spawn 子机系统（Ares）

### 基本配置

```ini
Spawns=ASW                      ; 子单位类型
SpawnsNumber=1                  ; 子单位数量
SpawnRegenRate=300              ; 子单位重生时间（帧）
SpawnReloadRate=150             ; 子单位重新装填时间（帧）
NoSpawnAlt=yes                  ; 子单位出击后母体切换模型状态
Experience.SpawnOwnerModifier=75%  ; 子单位获得母体经验的百分比
```

### 关键注意事项

1. **Turret=yes + Spawner 武器 = 模型透明**
   - `Turret=yes` 时引擎需要为每个武器槽渲染炮管/炮口动画
   - Spawner 武器（如 ASWLauncher）是虚拟武器，无炮口动画
   - 引擎找不到渲染数据 → 整个炮塔/模型渲染崩溃
   - **解决方案**：改用 `Turret=no`（参考驱逐舰 DEST）

2. **Spawned 子单位的 Owner 渲染**
   - 子单位诞生时根据 `Owner` 列表匹配阵营调色板
   - 如果 Owner 不含母体阵营 → 子单位模型透明
   - **解决方案**：在子单位定义中追加母体阵营到 Owner 列表

3. **AircraftTypes 编号连续性**
   - `[AircraftTypes]` 编号必须连续，不能通过 `[#include]` 子文件单独编号
   - 克隆子单位会破坏编号连续性 → 崩溃

---

## 四、辐射场机制（2026-08-18 依据 ModEnc/RA2DIY 权威修正）

**核心：辐射场只由武器层 `RadLevel` 产生**（ModEnc：RadLevel "Applicable to: Weapons"；RA2DIY：辐射代码必须写在武器主体）。

```ini
; ✅ 正确写法 — 武器层 RadLevel 是产生辐射场的唯一途径
[130mm]
Damage=90
Warhead=ARTYHE_HX
RadLevel=200          ; 武器层：辐射强度（产生辐射场，走 [RadSite] 结算）

; 弹头层（注意！）
[ARTYHE_HX]
Radiation=yes         ; 弹头伤害类型标记：ImmuneToRadiation 单位免疫此弹头（≠辐射场开关！）
CellSpread=2          ; 爆炸范围
```

**重要澄清（旧版"双层配置"说法有误，勿照抄）**：
- ❌ **弹头层 `RadLevel` 无效**（引擎不读，纯冗余）——130mm 的辐射实际来自武器层 RadLevel=200
- ❌ `Radiation=yes` **不是辐射场显示开关**——官方作用（ModEnc 原文）："ImmuneToRadiation=yes 的单位不能被此弹头伤害"
- ✅ 武器层 `RadLevel` **单独即可产生辐射场**（如辐射工兵副武器 RadEruptionWeapon 只有武器层 RadLevel=400）
- 辐射场伤害由 `[RadSite]` 弹头结算（Verses 修正），可用 `Versus.hx_*` 覆盖（华夏辐射抗性机制）

---

## 五、阵营权限三级控制

```ini
Owner=Huaxia                              ; 能看到此单位
RequiredHouses=Huaxia                     ; 能建造此单位
ForbiddenHouses=USSR,Latin,Chinese,...   ; 明确排除其他阵营
```

**规则：三者缺一不可。** 少写一个，其他阵营就能造你的专属单位。

常见错误：阵营名拼错（如 `Huavia` 而不是 `Huaxia`）→ 单位不在建造列表。

---

## 六、Verses 弹头伤害对照

Verses 标准 11 位顺序：`none, flak, plate, light, medium, heavy, wood, steel, concrete, special_1, special_2`

### 海军相关护甲

| 护甲类型 | Verses 位 | 典型单位 |
|----------|----------|---------|
| light (s_seamon) | 第4位 | 海豚、鱿鱼 |
| medium | 第5位 | 大多数潜艇、中小型舰艇 |
| heavy (s_heavy) | 第6位 | 战列舰、重型舰艇 |

### 反潜弹头示例

```ini
; 剑鱼主炮 — 65% 反潜
[SwordfishWH]
Verses=...,65%,...

; 海狼主炮 — 80% 反潜（更强）
[ARTYHE_HX]
Verses=...,80%,...

; 驱逐舰反潜机 — 100% 反潜
[ASWSplash]
Verses=...,100%,...
```

---

## 七、Turret 机制

| Turret | 效果 |
|--------|------|
| `yes` | 炮塔独立旋转，每根炮管需对应炮口动画 |
| `no` | 无炮塔，武器从车体中心发射，靠车体转向瞄准 |

- 海军单位常用 `Turret=no`（驱逐舰、护卫舰等），靠船体转向
- 多武器+Turret=yes 时，每个武器槽都需要有效的炮口动画（FLH）
- Spawner 虚拟武器（ASWLauncher）没有炮口动画 → 必须 Turret=no

---

## 八、LandTargeting 详解（2026-08-18 依据 ModEnc 修正）

| 值 | 含义 |
|----|------|
| 0 | 陆地正常（主武器打陆地） |
| 1 | **不能攻击陆地**（防空炮 NAFLAK 用此值 + NavalTargeting=6 → 纯防空） |
| 2 | **只能副武器打陆地**（主武器不能打陆地；战列舰 HCRUIS 用此值 → 主炮打海、副炮打陆） |

> ⚠️ 旧版误写"1=Primary正常使用，Secondary打陆地"——**错误**。值 1 是"不能打陆地"（LAND_NOT_OKAY）。

---

## 九、实战经验总结

| 问题 | 根因 | 解决 |
|------|------|------|
| 单位模型透明 | Turret=yes + Spawner虚拟武器 | Turret=no |
| 子单位模型透明 | Owner 不含母体阵营 | 追加阵营到子单位 Owner |
| 调用子单位崩溃 | AircraftTypes 编号不连续 | 不克隆子单位，直接复用原版 |
| 辐射场不生效 | `RadLevel` 写在弹头层（无效），武器层缺失 | RadLevel 移至武器层（见第四节） |
| 单位不在列表 | 阵营名拼写错误 | 检查 Owner/RequiredHouses 拼写 |
