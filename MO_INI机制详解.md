# MO INI 机制详解

> 基于 Mental Omega 3.3.6 (Ares 引擎) 的 INI 关键机制整理

---

## 一、NavalTargeting / LandTargeting 详解

控制单位武器对海军和陆地目标的攻击行为。

### 值含义

| 值 | NavalTargeting | LandTargeting |
|----|---------------|---------------|
| 0 | 不能攻击海军 | 主武器打陆地 |
| 1 | 副武器攻击海军 | 副武器打陆地 |
| 2 | 主武器攻击海军 | 主武器对地不可用，副武器打陆地 |
| 3 | 主副武器都能打海军 | — |
| 5 | **可攻击潜艇** | — |
| 6 | 可攻击潜艇（另一模式） | — |

### 实战示例

```ini
; 原版战列舰 — 主炮打地，副炮打海
[HCRUIS]
NavalTargeting=0
LandTargeting=2
; 结果：Primary 只能打陆地，Secondary 只能打海军

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

## 四、辐射场双层配置

辐射场需同时在武器和弹头中配置：

```ini
; 武器层
[130mm]
RadLevel=200          ; 辐射强度

; 弹头层
[ARTYHE_HX]
Radiation=yes         ; ★ 必须！否则辐射场不可见
RadLevel=200          ; 辐射强度
CellSpread=2          ; 辐射扩散范围（格）
```

- 仅设 `RadLevel` 而不设 `Radiation=yes` → 辐射场不显示
- `CellSpread` 控制辐射场扩散半径

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

## 八、LandTargeting 详解

| 值 | 含义 |
|----|------|
| 0 | Primary 打陆地，Secondary 正常使用 |
| 1 | Primary 正常使用，Secondary 打陆地 |
| 2 | Primary 不能打陆地，Secondary 打陆地+海军 |
| 3+ | 不常用 |

---

## 九、实战经验总结

| 问题 | 根因 | 解决 |
|------|------|------|
| 单位模型透明 | Turret=yes + Spawner虚拟武器 | Turret=no |
| 子单位模型透明 | Owner 不含母体阵营 | 追加阵营到子单位 Owner |
| 调用子单位崩溃 | AircraftTypes 编号不连续 | 不克隆子单位，直接复用原版 |
| 辐射场不显示 | 弹头缺少 Radiation=yes | 武器+弹头双层配置 |
| 单位不在列表 | 阵营名拼写错误 | 检查 Owner/RequiredHouses 拼写 |
