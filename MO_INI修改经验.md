# Mental Omega INI 修改经验

> 面向新手的零基础教程式手册。每一步都附带"为什么这么做"的解释。
>
> 权威参考：[ModEnc - RA2/YR INI 标签文档](https://modenc.renegadeprojects.com/)

---

## 一、基础概念

### 1. 什么是 INI 文件？

Mental Omega（心灵终结）是《红色警戒2：尤里的复仇》的模组。游戏的所有单位、武器、弹头数据都保存在 `.ini` 文件中。

| 文件 | 作用 | 位置 |
|---|---|---|
| `rulesmo.ini` | **主规则文件**：单位属性、武器、弹头、科技树等 | 打包在游戏目录的 `.mix` 文件中，需用 XCC Mixer 提取 |
| `rulesmo_huaxia.ini` | **华夏专属子文件**：通过 `[#include]` 由主文件加载 | 放在游戏根目录即可（Ares 的 include 功能） |
| `artmo.ini` | 美术资源（模型、动画）引用 | 同样打包在 `.mix` 中 |

### 2. 阵营系统核心概念

游戏中每个可建造的单位/建筑都需要通过以下标签指定"谁能建"：

```
Owner=USSR,Latin,Chinese,Huaxia    ← 哪些阵营可以拥有这个单位
RequiredHouses=Huaxia              ← 哪些阵营可以建造（更严格的限制）
ForbiddenHouses=USSR,Latin,Chinese ← 哪些阵营被禁止建造
```

**三者关系**：
- `Owner` 是基本门槛——不在 Owner 列表里的阵营连看都看不到
- `RequiredHouses` 是白名单——只有列表里的阵营能建
- `ForbiddenHouses` 是黑名单——列表里的阵营不能建

> **新手常犯错误**：给华夏专属单位设置了 `Owner=Huaxia`，但忘了在 `ForbiddenHouses` 里排除其他阵营。这样苏联、拉丁同盟、中国也能造你的专属单位！

### 3. 武器 → 弹头 引用链

每个攻击性单位都有一条引用链：

```
[单位]
  Primary=某武器        ← 主武器
  Secondary=某武器      ← 副武器（如对空）

[某武器]
  Damage=数值           ← 每次命中的基础伤害
  ROF=数值              ← 射击间隔（帧数）
  Range=数值            ← 射程（格子）
  Warhead=某弹头        ← 引用弹头

[某弹头]
  Verses=百分比,百分比,...  ← 对 11 种护甲的伤害百分比
  CellSpread=数值          ← 爆炸扩散范围
  ...
```

**最终伤害计算**：`Damage` × `Verses[目标护甲]%` = 实际伤害

---

## 二、常用标签速查

### 武器标签

| 标签 | 含义 | 示例 | 新手说明 |
|---|---|---|---|
| `Damage` | 每次命中基础伤害 | `Damage=43` | 最终伤害 = Damage × 弹头Verses百分比 |
| `ROF` | Rate of Fire，射击间隔（帧） | `ROF=25` | 1帧≈1/15秒，ROF=15≈每秒1发，ROF=1=最快射速 |
| `Burst` | 每次开火连射几发 | `Burst=2` | Burst=2 + ROF=1 = 每秒射出30发 |
| `Range` | 攻击距离（格子） | `Range=7` | 约等于屏幕上的格子数 |
| `MinimumRange` | 最小攻击距离 | `MinimumRange=3` | 3格以内无法攻击，防止贴脸 |
| `Warhead` | 引用的弹头名称 | `Warhead=FlakGuyATWH_HX` | 指向弹头节，决定伤害修正比 |
| `Projectile` | 弹体类型 | `Projectile=InvisibleHigh` | 激光武器常用 InvisibleHigh |

### 弹头标签

| 标签 | 含义 | 示例 | 新手说明 |
|---|---|---|---|
| `Verses` | 对11种基础护甲的伤害% | `Verses=30%,40%,50%,...` | 顺序固定：none,flak,plate,light,medium,heavy,wood,steel,concrete,sp1,sp2 |
| `CellSpread` | 爆炸/溅射扩散格数 | `CellSpread=.3` | 越小越集中 |
| `Radiation` | 是否产生辐射场 | `Radiation=yes` | **必须配合武器层的 RadLevel 才生效！** |
| `RadLevel` | 辐射强度 | `RadLevel=200` | 数值越高辐射越强 |
| `Versus.xxx` | 对特定护甲的修正 | `Versus.defense=45%` | 覆盖 Verses 中的对应值 |

### 11 种基础护甲对应关系

| 序号 | 护甲名 | 典型单位 |
|---|---|---|
| 1 | none | 基础步兵（动员兵、美国大兵） |
| 2 | flak | 防弹步兵（防空兵、辐射工兵） |
| 3 | plate | 重甲步兵（磁爆步兵、狂兽人） |
| 4 | light | 轻型车辆（恐怖机器人、IFV） |
| 5 | medium | 中型车辆（犀牛坦克） |
| 6 | heavy | 重型车辆（天启坦克） |
| 7 | wood | 木制建筑（兵营、电厂） |
| 8 | steel | 钢制建筑（战车工厂） |
| 9 | concrete | 混凝土建筑（围墙、碉堡） |
| 10 | special_1 | 特殊（恐怖机器人、Drone） |
| 11 | special_2 | 特殊（围墙等） |

### 建筑标签

| 标签 | 含义 | 示例 | 说明 |
|---|---|---|---|
| `BuildCat` | 建筑分类 | `BuildCat=Combat` | Combat=防御建筑, Tech=科技建筑, Resource=资源建筑 |
| `Prerequisite` | 建造前提 | `Prerequisite=NACNST,NATEKHX` | 需要建造厂+原子核心 |
| `TechLevel` | 科技等级 | `TechLevel=9` | 1-3=T1, 4-7=T2, 8-10=T3 |
| `Power` | 电力消耗 | `Power=-150` | 负数=耗电，正数=发电 |
| `Strength` | 生命值 | `Strength=1650` | 耐久度 |
| `Armor` | 护甲类型 | `Armor=defense_b` | 决定承受伤害时的修正 |
| `Image` | 使用的模型 | `Image=FACOMP` | 复用时指向原版建筑的模型名 |
| `Primary` | 主武器 | `Primary=NeutralizerCutterHX` | 防御建筑的攻击武器 |

---

## 三、添加新阵营的核心步骤

### 1. `[Countries]` 注册

- 序号必须从0开始连续编号，不能中断
- **绝对禁止**覆盖或重排前9个主阵营的索引，否则遭遇战/LAN地图会崩溃
- 格式：`序号=阵营名`

```ini
[Countries]
0=UnitedStates
1=Europeans
...
6=Huaxia
...
```

### 2. `[Sides]` 注册

- 将新阵营加入对应阵营的列表
- 格式：`阵营名=国家1,国家2,...`

```ini
[Sides]
;Allies
GDI=UnitedStates,Europeans,Pacific
;Soviets
Nod=USSR,Latin,Chinese,Huaxia
;Epsilon
ThirdSide=PsiCorps,ScorpionCell,Headquaters
```

### 3. `[阵营名]` 节定义

- **节名必须与 `[Countries]` 中的名称完全一致**（区分大小写！）
- 关键字段：`Side=`、`SmartAI=`、`Multiplay=`、`Suffix=`、`Prefix=`
- `ListIndex=` 需设为唯一值

### 4. 建筑 Owner 全覆盖（**最容易遗漏**）

- 所有新阵营可建造的建筑，必须在 `Owner=` 列表中包含新阵营
- 缺少 Owner 的后果：
  - **NACNST（建造厂）缺少 → 立即失败**（MCV 部署后无法生成基地）
  - 其他建筑缺少 → 无法建造该建筑
- 修复方式：在 `rulesmo_huaxia.ini` 中逐一覆盖关键建筑的 Owner：

```ini
[NACNST]
Owner=USSR,Latin,Chinese,Huaxia

[NAHAND]
Owner=USSR,Latin,Chinese,Huaxia

[NAWEAP]
Owner=USSR,Latin,Chinese,Huaxia
; ... 其余关键建筑同理
```

### 5. `[#include]` 文件包含机制（Ares）

- Ares 支持 `+=` 语法追加包含文件
- **文件名拼写必须与引用完全一致**
- 在 `rulesmo.ini` 末尾声明：

```ini
[#include]
+=rulesmo_huaxia.ini
```

---

## 四、常用操作流程

### 流程 A：克隆步兵/载具单位

> 示例：为华夏克隆防空兵（FLAKT → FLAKTHX）

**第 1 步：在 `rulesmo.ini` 中找到原版单位**

搜索 `[FLAKT]`，记录完整定义。

**第 2 步：在 `rulesmo_huaxia.ini` 中创建克隆体**

```ini
[FLAKTHX]
Image=FLAKT                    ← 复用原版模型
UIName=NAME:FLAKT              ← 共享显示名称
Name=Huaxia Flak Trooper       ← 改成华夏名称
Owner=Huaxia                   ← 只属于华夏
RequiredHouses=Huaxia
ForbiddenHouses=USSR,Latin,Chinese  ← 禁止其他阵营
Prerequisite=NAHAND            ← 兵营即可建造
; ... 其余标签复制原版
```

**第 3 步：在原版单位上加 `ForbiddenHouses=Huaxia`**

防止原版单位和克隆体同时出现。

```ini
[FLAKT]
ForbiddenHouses=Huaxia
```

**第 4 步：在 `[InfantryTypes]` 或 `[VehicleTypes]` 注册**

在注册列表末尾追加新单位名。

### 流程 B：克隆建筑（如 T3 防御塔）

> 示例：为华夏克隆焚风粒子中和器（FACOMP → NACOMPHX）

**第 1 步：搜索原版建筑**

在 `rulesmo.ini` 中搜索目标建筑的名称（如 "Neutralizer"），找到 `[FACOMP]` 节的完整定义。

**第 2 步：在原版建筑上加 `ForbiddenHouses=Huaxia`**

在 `rulesmo.ini` 中找到 `[FACOMP]`，在 `Owner=` 行下方添加：

```ini
[FACOMP]
Owner=Europeans,UnitedStates,...Guild3
ForbiddenHouses=Huaxia         ← 新增此行
```

**第 3 步：克隆武器和弹头**

这是**重要的一步**——不能直接引用原版武器，必须克隆一份。为什么？因为未来如果想单独调参，直接改华夏版的武器即可，不影响原版。

在 `rulesmo_huaxia.ini` 末尾添加：

```ini
; 克隆武器
[NeutralizerCutterHX]
Damage=12
ROF=1
Burst=2
Range=14
MinimumRange=3
Projectile=InvisibleHigh
Speed=100
Report=RamwagonAttack
Warhead=NeutralizerWeldWH_HX   ← 关键：引用克隆的弹头
Bright=no
LaserInnerColor=92,235,175
LaserOuterColor=0,0,0
LaserOuterSpread=0,0,0
LaserDuration=15
IsLaser=yes
Anim=LOMUZZLE
IsBigLaser=true

; 克隆弹头
[NeutralizerWeldWH_HX]
CellSpread=.3
PercentAtMax=.25
Wall=yes
Wood=yes
Sparky=yes
Verses=50%,50%,50%,90%,80%,70%,65%,50%,60%,30%,100%
; ... 其余标签全量复制
```

**第 4 步：创建建筑定义**

在武器和弹头之后，添加建筑定义：

```ini
[NACOMPHX]
Image=FACOMP                    ← 复用焚风模型
UIName=NAME:FACOMP
Name=Huaxia Neutralizer
BuildCat=Combat
Strength=1650
Armor=defense_b
TechLevel=9
Prerequisite=NACNST,NATEKHX    ← 华夏建造厂 + 原子核心
Owner=Huaxia
RequiredHouses=Huaxia
ForbiddenHouses=USSR,Latin,Chinese
Primary=NeutralizerCutterHX    ← 引用克隆的武器
ElitePrimary=NeutralizerCutterHX
; ... 其余标签复制原版，注意引用链完整
```

**第 5 步：在 `[BuildingTypes]` 注册**

在 `rulesmo_huaxia.ini` 末尾追加：

```ini
[BuildingTypes]
880=NACOMPHX
```

### 流程 C：修改武器伤害/弹头护甲修正

**修改伤害/射速**（以粒子中和器为例）：

```ini
[NeutralizerCutterHX]
Damage=15        ← 12 → 15
ROF=1            ← 射击间隔（帧），1=最快
Burst=2          ← 每次连射2发
```

修改后的理论 DPS：15 × 2 × 15 = 450（对 100% 护甲目标）。

**修改弹头护甲修正比**：

找到弹头节的 `Verses=` 行：

```ini
[NeutralizerWeldWH_HX]
;           none  flak  plate light medium heavy wood steel concrete sp1 sp2
Verses=50%,  50%,  50%,  90%,  80%,   70%,  65%, 50%,  60%,     30%,100%
```

- 每个百分比对应一种护甲（共 11 种，顺序固定）
- `50%` 表示只造成 50% 伤害
- 修改时要确认目标单位的护甲类型（见"11 种基础护甲对应关系"）

### 流程 D：修改辐射场

**关键规则**：辐射场需要弹头 + 武器**双层**配置，缺一不可！

```ini
; ✅ 正确做法
[某武器]
RadLevel=200             ← 武器层：必须有

[某弹头]
Radiation=yes            ← 弹头层：必须有
RadLevel=200             ← 弹头层：必须有
CellSpread=1             ← 辐射扩散范围（格数）
```

如果只配弹头不配武器 `RadLevel`，辐射场不会出现——这是最常见的辐射场不生效原因。

---

## 五、故障排查

### 遭遇战"立即失败"

| 优先级 | 检查项 | 说明 |
|---|---|---|
| 1 | NACNST 的 Owner | 建造厂没有新阵营，MCV 部署后立即失败 |
| 2 | 阵营名一致性 | `[Countries]`、`[Sides]`、`[阵营节]` 三处名称是否完全一致（区分大小写） |
| 3 | 文件实际被加载 | `[#include]` 引用的文件名是否存在 |
| 4 | BaseUnit/MCV | MCV 的 Owner 是否包含新阵营 |
| 5 | 起始步兵 | `AllowedToStartInMultiplayer=yes` 的步兵 Owner 是否包含新阵营 |

### 单位/建筑不显示

| 常见原因 | 排查方法 |
|---|---|
| `Owner` 没有新阵营 | 搜索该单位的 Owner 行，确认含你的阵营名 |
| `Owner` 阵营名拼写错误 | **对比正常工作的单位**，逐字节检查 Owner= 后的名称是否与阵营节 `[阵营名]` 完全一致 |
| `ForbiddenHouses` 误禁了新阵营 | 搜索该单位的 ForbiddenHouses，确认不含你的阵营 |
| `Prerequisite` 前置建筑不可用 | 顺着 Prerequisite 往上查，每个前置建筑的 Owner 都检查 |
| `TechLevel` 超出允许范围 | 遭遇战科技等级限制可能屏蔽高 TechLevel 单位 |
| `[#include]` 文件名不匹配 | 文件名大小写、拼写须与 include 声明完全一致 |

---

## 六、自查清单

- [ ] `[Countries]`、`[Sides]`、`[阵营节]` 三处阵营名拼写一致？
- [ ] `[#include]` 引用文件名与实际文件名一致？
- [ ] NACNST（建造厂）的 `Owner` 包含新阵营？
- [ ] NAHAND（兵营）的 `Owner` 包含新阵营？
- [ ] NAWEAP（战车厂）的 `Owner` 包含新阵营？
- [ ] NAPOWR（电厂）的 `Owner` 包含新阵营？
- [ ] NAREFN（矿场）的 `Owner` 包含新阵营？
- [ ] 所有克隆单位的 `Owner`/`RequiredHouses`/`ForbiddenHouses` 正确？
- [ ] 原版单位已添加 `ForbiddenHouses=新阵营` 避免重复？
- [ ] 武器引用的弹头名称与克隆弹头一致？
- [ ] 辐射武器：武器和弹头都配置了 `RadLevel`？

**（2026-08 华夏版全面检查补充）**

- [ ] 覆盖共享建筑 `Owner` 前，已核对原版 `Prerequisite` 是否为阵营专属建筑（判断窄列表覆盖是否安全）？
- [ ] 未出现 `Owner` 与 `ForbiddenHouses` 自相矛盾（同一阵营既在白名单又在黑名单，导致其失去原版建造权）？
- [ ] 覆盖的节在原版 `rulesmo.ini` 中真实存在（不是 `NADEPT` 这类仅作为 `Image=` 模型名出现的名称）？
- [ ] 超武列表（`SW.Designators`/`SW.Inhibitors`/`SW.AITargeting` 等）未引用不存在的建筑？
- [ ] 最新 `debug.log` 无新增 `[Developer error]` / `Failed to parse`？

**（2026-08-18 拼写地雷拆雷补充）**

- [ ] 全部被加载 INI 中 `Huavia` 已清零（grep 三文件 = 0，有意的历史说明文字除外）？
- [ ] 华夏不能造的原版单位使用**显式** `ForbiddenHouses=Huaxia`，不依赖拼写错误"碰巧挡住"？
- [ ] 废弃的克隆定义（未注册的 `*HX` 节）已删除或已注册，无"半成品"残留？
- [ ] 修改后已跑"健康检查六步法"（A16）？

---

## 七、实战案例记录

### 案例 1：Huaxia 阵营初始修复 (2026-07-19)

**问题**：进入遭遇战立即显示失败。

**根因**：
1. `rulesmo.ini` 的 `[#include]` 引用的文件名与实际文件不一致（历史拼写错误，详见附录引言）
2. 阵营节名与 `[Countries]` 注册的阵营名存在拼写不一致
3. NACNST 的 `Owner=USSR,Latin,Chinese` 缺少 Huaxia

**修复**：
1. 修正 `[#include]` → `+=rulesmo_huaxia.ini`
2. 全局替换错误拼写（含 v 的 `Huavia`）为正确拼写 `Huaxia`（含 x）
3. 逐一覆盖苏联关键建筑的 Owner

### 案例 2：Flak Trooper 缺失 (2026-07-19)

**问题**：华夏阵营不显示 Flak Trooper。

**根因**：建筑覆盖中 NAHAND 的 Owner 缺少 Huaxia → FLAKTHX 的 Prerequisite=NAHAND 不可用 → 单位不显示。

**修复**：
1. 从稳定备份恢复 `rulesmo_huaxia.ini`
2. 修正 `[#include]` 引用
3. 矫正 3 处阵营名拼写错误（`Huavia` → `Huaxia`）

### 案例 3：克隆 T3 粒子中和器 NACOMPHX (2026-07-20)

**需求**：为华夏创建与焚风粒子中和器 (FACOMP) 功能相同的 T3 防御建筑。

**步骤回顾**：
1. 在 `rulesmo.ini` 中找到 `[FACOMP]`，记录完整定义
2. 在原版 FACOMP 添加 `ForbiddenHouses=Huaxia`
3. 克隆武器 `[NeutralizerCutter]` → `[NeutralizerCutterHX]`，Warhead 指向克隆弹头
4. 克隆弹头 `[NeutralizerWeldWH]` → `[NeutralizerWeldWH_HX]`
5. 克隆建筑 `[FACOMP]` → `[NACOMPHX]`，修改 Owner/Prerequisite/Primary
6. 在 `[BuildingTypes]` 注册 `880=NACOMPHX`

**后续微调**：
- Damage 12→15（DPS 360→450）
- Verses 对步兵/钢制建筑 30%/30%/25%/30% → 50%/50%/50%/50%

### 案例 4：战列舰 HCRUISHX / 海狼炮艇 SWLFHX 不显示 (2026-07-20)

**问题**：其他华夏单位（SUBHX、EMPRHX 等）均正常显示，唯独新增的战列舰和海狼炮艇在建造列表中看不到。

**根因**：`Owner=` 和 `RequiredHouses=` 的阵营名拼写错误——把正确拼写 `Huaxia`（含 **x**）写成了错误拼写 `Huavia`（含 **v**）。`Huavia` 是一个不存在的阵营，游戏引擎无法将这两个单位分配给任何玩家。

```
阵营定义:  [Huaxia]        ← 正确（x）
工作单位:  Owner=Huaxia     ← 正确（x）
故障单位:  Owner=Huavia     ← 错误（v）→ 阵营不存在！
```

**教训**：`Owner` / `RequiredHouses` / `ForbiddenHouses` 中的阵营名必须与 `[Countries]` 和阵营节 `[阵营名]` **逐字节一致**。一个字母的差异就会导致单位完全不可见。当"部分新单位正常、部分不正常"时，优先对比正常与异常单位的 Owner 行。

### 案例 5：战列舰搭载反潜机 (2026-07-20)

**需求**：为华夏战列舰 HCRUISHX 添加盟军驱逐舰的反潜机生成能力。

**过程**：
1. 将 `Secondary`/`EliteSecondary` 改为 `ASWLauncher`，添加 `Spawns=ASW` / `SpawnsNumber=1` / `SpawnRegenRate=300` / `SpawnReloadRate=150` / `NoSpawnAlt=yes`
2. 修改 `LandTargeting=0`（主炮对地）、`NavalTargeting=1`（副武器对潜艇）
3. Osprey 模型透明 → 尝试克隆 ASW→ASWHX（含全套武器/弹体/弹头），但 `[AircraftTypes]` 编号不连续导致崩溃
4. 最终方案：**不克隆**，直接在 `rulesmo_huaxia.ini` 中覆盖原版 ASW 的 Owner 追加 `Huaxia`

**崩溃根因**：`[AircraftTypes]` 不能通过 `[#include]` 子文件另行编号。原版 ASW 注册在 `rulesmo.ini` 的 AircraftTypes 中（编号 7），子文件中新增 `883=ASWHX` 不连续，引擎初始化时找不到该类型。

**模型透明根因**：`Turret=yes` + `Secondary=ASWLauncher`（无炮口动画的 Spawner 武器）→ 炮塔渲染系统无法渲染第二根炮管→整个模型透明。改为 `Turret=no` 解决（参照驱逐舰 DEST）。

**最终成果**：
- `[ASW] Owner=Europeans,UnitedStates,Pacific,Huaxia` 覆盖（只加不删）
- 战列舰 `Turret=no`、`NavalTargeting=1`、`LandTargeting=0`
- 反潜机完全复用原版，无克隆开销

---

## 八、线上"心灵终结ini"修改资源汇总

| # | 来源 | 链接 | 内容 |
|---|---|---|---|
| 1 | B站专栏 | https://m.bilibili.com/opus/735483926653436025 | 心灵终结 3.3.6 高级修改教程 |
| 2 | B站专栏 | https://m.bilibili.com/opus/652682845436772361 | 小白 INI 修改记录（解锁渗透单位等） |
| 3 | B站专栏 | https://www.bilibili.com/read/cv9773786/ | 调出隐藏单位教程 |
| 4 | hflchs.com | https://hflchs.com/2296.html | RULES 单位篇代码中文版 + 单位全代码 |
| 5 | ModEnc | https://modenc.renegadeprojects.com/ | 权威 RA2/YR INI 标签参考 |

---

## 九、工具与资源

| 工具 | 用途 | 获取方式 |
|---|---|---|
| XCC Mixer | 从 `.mix` 文件中提取 `rulesmo.ini` 等 | 网络搜索下载 |
| RA红警语言编辑器 | 修改单位名称、台词（.csf 文件） | B站有免费资源 |
| Notepad++ / VS Code | INI 文本编辑 | 免费开源 |
| Ares DLL | 扩展引擎功能（支持 `[#include]` 等） | https://ares.strategy-x.com/ |

---

## 十、核心原则（记住这 7 条）

1. **克隆而不修改原版**：所有华夏单位的武器、弹头都克隆一份到 `rulesmo_huaxia.ini`，不动原版数据
2. **Owner 三级防护**：Owner + RequiredHouses + ForbiddenHouses 三重锁定，确保只有华夏能用
3. **引用链完整性**：建筑 → 武器 → 弹头，每一环的名称必须对应正确
4. **辐射双层配置**：武器和弹头都要写 `RadLevel`
5. **先备份再修改**：每次修改前保留一份可用的备份版本
6. **名称逐字节一致**：`Owner`、`RequiredHouses`、`ForbiddenHouses` 中的阵营名必须与 `[Countries]` 及阵营节名完全一致，一字之差就会导致单位不可见
7. **显式锁定，不靠拼写错误**：华夏不能造的原版单位用显式 `ForbiddenHouses=Huaxia` 声明；Owner 里无效拼写"碰巧挡住"不是防护，是定时炸弹（见 A14/A15）

---

## 附录：全面检查与配置经验（2026-08-17 华夏版检查沉淀）

> 本文档早期案例（2026-07）中的阵营名拼写经历过一次统一修正：**由错误拼写（`Huavia`，含 v）修正为 `Huaxia`（含 x）**，子文件为 **`rulesmo_huaxia.ini`**。正文已按正确拼写书写；案例 4 因教学需要保留两个拼写对比。以下经验以正确拼写书写。

### A1. 共享建筑 Owner 窄列表覆盖的"前置判定法"（最重要的认知升级）

**现象**：`rulesmo_huaxia.ini` 中大量共享建筑（NAPOWR/NAHAND/NAREFN/NAWEAP/NAAIR/NAYARD/NARADR/NAIRON/NAWALL/NABNKR 等）被写成窄列表：

```ini
[NAHAND]
Owner=USSR,Latin,Chinese,Huaxia
```

而原版这些建筑的 Owner 是**全 14 国列表**（含盟军、厄普西隆）。乍看之下这种整行替换会让盟军/厄普西隆失去建造权，构成"覆盖危机"。

**但实际无破坏**。判断依据：**这类建筑的原版 `Prerequisite` 全部基于苏联专属建筑 `NACNST`**（例如 `Prerequisite=NACNST`、`Prerequisite=PROC2,POWER,NACNST`）。盟军/厄普西隆不可能拥有 `NACNST`，所以它们原版就无法建造这些建筑——Owner 里的全列表只是 MO 的"共享写法"，窄列表覆盖不改变任何阵营的实际建造能力。

**可复用判定法（覆盖 Owner 前先回答）**：
1. 该建筑原版 `Prerequisite=` 中的前置建筑，是否**全部属于同一阵营**？
   - 是（如全是苏联建筑）→ 窄列表覆盖安全，其他阵营本来就不能造
   - 否（含跨阵营前置，如 `GAPILE,GACNST`）→ 必须先查哪些阵营能满足前置，再决定 Owner 列表
2. 原版 Owner 里除本阵营外，还有哪些阵营**实际能造**（前置满足）？保留它们，只追加新阵营。

**结论**：覆盖共享建筑 Owner 前，**先查 Prerequisite，不要凭 Owner 全列表就断定会误伤**。

### A2. `Owner` 与 `ForbiddenHouses` 自相矛盾陷阱

**错误示例**（本次发现的 NAFLAK 覆盖）：

```ini
[NAFLAK]
Owner=USSR,Latin,Chinese,Huaxia
ForbiddenHouses=USSR,Latin,Chinese
```

`Owner` 里保留苏军三家，`ForbiddenHouses` 又把苏军三家禁掉 → **结果只有华夏能造**，苏军三家失去原版防空炮。这种"先留再禁"的写法是历史拷贝残留，极易误导。

**判定法**：检查每个覆盖节，`Owner` 与 `ForbiddenHouses` 是否出现同一阵营名。出现即意味着"这个阵营被 Owner 允许、又被 ForbiddenHouses 禁止"——要么删 ForbiddenHouses，要么从 Owner 移除。

**原则**：想给华夏做"专属强化版"（更强的武器/更高价格），**不要改原版建筑**，按 A3 克隆新建筑；原版保持不动，其他阵营的原版能力零影响。

### A3. 克隆建筑必须"全量复制属性"，`Image=` 只复用模型

**重要认知**：`Image=NAFLAK` 只引用模型文件，**不继承任何属性**。克隆建筑必须把原版建筑的全部标签完整重写，只修改差异化项（`Owner`/`Prerequisite`/武器/`Cost`/`TechLevel`）。漏写任一标签（如 `Turret=yes`、`Power=-50`、`ThreatPosed=40`）都会导致行为异常。

**完整模板**（NAFLAKHX，克隆 NAFLAK 的华夏专属防空炮）：

```ini
[NAFLAKHX]
Image=NAFLAK                    ← 复用原版模型
UIName=NAME:NAFLAK              ← 共享显示名称
Name=Huaxia Flak Cannon
BuildCat=Combat
Strength=800
Armor=defense
TechLevel=5
Prerequisite=NAHAND,NACNST
Adjacent=7
Sight=10
Owner=Huaxia
RequiredHouses=Huaxia
ForbiddenHouses=USSR,Latin,Chinese
AIBasePlanningSide=1
Cost=900
Bounty.Value=180
Bounty=no
Bounty.Display=yes
BaseNormal=no
Points=50
Power=-50
Crewed=no
Primary=FlakFake
Secondary=FlakWeaponHX          ← 克隆武器（射程 15，原版 12）
ElitePrimary=FlakFake
EliteSecondary=FlakWeaponHX
LandTargeting=1
NavalTargeting=6
Capturable=false
Explosion=GBLDEXP1,...,GBLDEXP10
DebrisAnims=DBRIS4LG,DBRIS4SM,DBRIS6LG
MaxDebris=4
MinDebris=2
ThreatPosed=40
IsBaseDefense=yes
Powered=yes
ROT=14
Turret=yes
TurretAnim=NAFLAKTUR
TurretAnimIsVoxel=false
TurretAnimY=0
TurretAnimZAdjust=-40
HasStupidGuardMode=false
WorkingSound=PowerOn
NotWorkingSound=PowerOff
Drainable=yes
AntiInfantryValue=0
AntiArmorValue=0
AntiAirValue=75
VeteranAbilities=STRONGER
EliteAbilities=SELF_HEAL,FIREPOWER
Trainable=yes
HasRadialIndicator=true
VoiceSelect=FlakCannon2Select
ImmuneToEMP=no
ImmuneToPsionics=no
Insignia.Veteran=defvet
Insignia.Elite=defeli
EVA.VeteranPromoted=EVA_DefenseUpgraded
EVA.ElitePromoted=EVA_DefenseUpgraded
Promote.VeteranSound=UpgradeDefenseVeteran
Promote.EliteSound=UpgradeDefenseElite
```

**注册**：在 `rulesmo_huaxia.ini` 的 `[BuildingTypes]` 中接续注册（本次用 `1159=NAFLAKHX`，紧接 1157/1158 之后）。

### A4. 删除覆盖键/覆盖节 = 恢复原版

Ares 的规则合并是**按键覆盖**（section 级 merge），不是整节替换。因此：

- 在 `rulesmo_huaxia.ini` 中**删除某个键** → 原版该键的值自动生效
- **删除整个覆盖节**（如删除 `[NAFLAK]` 节）→ 原版该建筑完全恢复原样

**复用场景**：本次恢复 NAFLAK 共享时，直接删除整个 `[NAFLAK]` 覆盖节，原版（`Owner` 含全列表 + `FlakWeapon` + `Cost=700`）即恢复。当时原版 `Owner` 末尾挂着无效拼写 `Huavia`，华夏"碰巧"不能造原版 NAFLAK，正好避免与 NAFLAKHX 重复。⚠️ **注意：这个"碰巧"是地雷，不是设计**——2026-08-18 已全面清理该拼写（见 A14），华夏不能造原版 NAFLAK 现在靠"Owner 不含 Huaxia + 显式 `ForbiddenHouses=Huaxia`"双保险（见 A15）。

### A5. 超武目标引用残留排查（SW.Designators 等）

**现象**（本次修复）：游戏加载报 `[Developer warning]Failed to parse INI file content: [DrakuvSpecial]SW.Designators=NAPRISHX`。原因是超武节里引用了**不存在的建筑** `NAPRISHX`（华夏早期想做磁暴线圈克隆、中途废弃，引用却留在超武列表里）。

**排查法**：在全部 INI 文件中 grep 所有 `SW.Designators=` / `SW.Inhibitors=` / `SW.AITargeting=` 行，逐个确认被引用的建筑 ID 都有定义节（`[该ID]` 存在）。同时反向检查：所有 `*HX` 结尾的 ID 必须同时出现在注册表或定义节中。

**修复**：删除不存在的引用即可（`NAPRIS,NAPRISHX` → `NAPRIS`）。

### A6. 无效覆盖排查（覆盖前先 grep 原版确认建筑存在）

**现象**：`rulesmo_huaxia.ini` 中 `[NASENT]`、`[NADEPT]`、`[NASILO]` 三个覆盖节，在 MO 3.3.6 中**根本不存在对应建筑**：

- `NADEPT` 只是 `NADRON`（苏联维修起重机）的 `Image=NADEPT` 模型名，不是建筑 ID
- `NASENT`/`NASILO` 完全不存在

覆盖不存在的节**无效果也无报错**（Ares 静默忽略），但会误导后人以为"已给某建筑加了 Owner"。

**排查法**：覆盖任何节前，先在 `rulesmo.ini` 中 grep `^\[节名\]` 确认其真实存在。发现无效覆盖直接删除。

### A7. debug.log 健康检查法（区分"必修"与"固有"）

修改后必查游戏目录 `debug\debug.log`，按级别分类处理：

| 级别 | 报错内容 | 处理 |
|---|---|---|
| **必修** | `[Developer error]`、`Failed to parse INI file content` | 配置错误，必须修复 |
| **必修** | `[Developer error][XXXHX]SpeedType is invalid!` | 克隆单位 SpeedType 未写，必须补齐 |
| **固有** | `[Developer fatal]Pointer 00000000 declared change to both X AND Y` | MO 3.3.6 建筑注册超 512 的固有现象（`Counter attempted to overflow`），**对比旧版日志**：旧版同样存在则非 mod 引入，无需处理 |
| **无害** | `Unrecognized UI action value: show` | 游戏内置，忽略 |

**判定原则**：拿"修改前/旧版"日志做基线对比，只处理**新增**的 error。注意看日志头部的 mod 版本号，区分新旧目录的日志。

### A8. 虚拟前置（GenericPrerequisites）与科技链完整性

```ini
[GenericPrerequisites]
SOVWEAP=NAWEAP,NAWEAPB,NAFIST
SOVTECH=NATECHR,NATECHC,NATEK,NATEKHX
```

`SOVWEAP`/`SOVTECH` 是 Ares 的**虚拟前置**，会展开为多个真实建筑。关键收益：`SOVTECH` 已含 `NATEKHX`（华夏原子核心）→ **克隆单位只要前置写了 `SOVTECH` 或 `NATEKHX`，华夏科技链就天然完整**，无需逐级排查前置 Owner。

**复用**：给华夏克隆单位配前置时，优先沿用原版写法（含虚前置），省去大量 Owner 排查。

### A9. 克隆定义的位置约定

克隆单位的**注册**可放在 `rulesmo.ini`（InfantryTypes 151–159、VehicleTypes 364–375），**定义**应统一放 `rulesmo_huaxia.ini`。历史遗留：`[RACCHX]` 的完整定义直接写在 `rulesmo.ini`（1976 行起），功能正常（后加载子文件可覆盖）但违反约定。**新增克隆定义一律放子文件**，避免日后维护时"定义散落主文件"的混乱。

### A10. 克隆武器的弹头约定（轻微）

规范做法：克隆武器同时克隆弹头（如 `SentinelWH_HX`），保证未来可独立调参。本次 `FlakWeaponHX` 复用了原版 `FlakCannonWH` 弹头，属于轻微违规——**当前不影响功能**，仅当需要独立调整防空弹头属性时才需拆分。

### A11. 克隆防御建筑不会自动进入 AI 建造列表

克隆建筑（如 `NAFLAKHX`）即使 `AIBasePlanningSide=1`，也不会自动加入 `[AI] BuildAA` 列表——**华夏 AI 不会主动建造它**（玩家可正常建造）。如需 AI 防空，需在 `[AI]` 中补充（注意 `BuildAA` 整行覆盖会影响所有阵营，需谨慎处理）。

### A12. 实战案例 6：NAPRISHX 超武残留 (2026-08-17)

**问题**：加载报 `Failed to parse INI file content: [DrakuvSpecial]SW.Designators=NAPRISHX`。
**根因**：超武 `DrakuvSpecial` 的 `SW.Designators` 引用了不存在的 `NAPRISHX`（废弃的华夏磁暴线圈克隆残留）。
**修复**：`SW.Designators=NAPRIS,NAPRISHX` → `SW.Designators=NAPRIS`。
**验证**：grep 全部 INI，确认 `NAPRISHX` 无任何残留引用。

### A13. 实战案例 7：NAFLAK 独占化 → 克隆专属版 (2026-08-17)

**问题**：华夏版将共享防空炮 NAFLAK 变成华夏独占（`ForbiddenHouses=USSR,Latin,Chinese`），苏军三家失去防空能力，且武器/价格被改。
**决策**：采用"克隆专属版"方案——原版恢复共享，华夏用专属克隆。
**实施**：
1. 删除 `rulesmo_huaxia.ini` 中整个 `[NAFLAK]` 覆盖节 → 原版恢复（苏军三家可造，华夏不能造原版，无重复；当时依赖原版 Owner 里无效拼写 `Huavia` 挡住，2026-08-18 起改为显式 `ForbiddenHouses=Huaxia` 锁定，见 A14/A15）
2. 新建 `[NAFLAKHX]`（A3 模板），使用 `FlakWeaponHX`（射程 15）、`Cost=900`
3. 注册 `1159=NAFLAKHX`
4. 验证依赖：`FlakFake`/`FlakWeaponHX`/`FlakCannonWH` 均存在，无悬挂引用
**教训**：需要"华夏专属强化版"时走克隆路线（A3），不要改原版共享建筑（A2）。

---

## 附录二：2026-08-18 新增经验（拼写地雷拆雷 + 健康检查体系）

### A14. Huavia 拼写地雷与全面拆雷（⭐ 最重要的认知升级）

**现象**：`rulesmo.ini` 中曾存在 **126 处 `Huavia`**（错误拼写，正确为 `Huaxia`）：
- 122 处是 `Owner=Europeans,...,Guild3,Huavia`（全 12 国列表末尾挂无效拼写），波及**全阵营 120+ 个原版建筑**（盟军 GAPILE/GAPOWR、苏军 NAFLAK/NAPRIS/TESLA、厄普西隆 YAPOWR/YAGGUN、焚风 **FACOMP**/FAGUAR 等）
- 2 处 `Name=Huavia ...`（HCRUISHX/SWLFHX 内部名）+ 2 处注释

**为什么是地雷**：`Huavia` 无效 → 华夏"碰巧"不能造这些原版建筑。但一旦有人做"全局替换 `Huavia`→`Huaxia`"（案例 1/2 的修复模式！），**120+ 个原版建筑瞬间全部解锁给华夏**——华夏将能造盟军/厄普西隆/焚风的所有基础建筑与超武，阵营体系直接崩溃。

**处理**（2026-08-18 已执行，备份 `rulesmo.ini.bak_20260818`）：
```bash
sed -i 's/,Huavia//g' rulesmo.ini   # 删除 Owner 末尾无效项，恢复原版全 12 国列表
sed -i 's/Huavia/Huaxia/g' rulesmo.ini  # 修正 Name/注释
```
验证：三个被加载文件（rulesmo.ini / rulesmo_huaxia.ini / rulesmo_override.ini）`Huavia` 清零；`rulesmo_huaxia.ini` 后加载覆盖的共享建筑（NAHAND/NAYARD/NAPRIS 等）不受影响，华夏建造权零变化。

**教训**：**Owner 里"碰巧挡住"的无效拼写不是防护，是定时炸弹**。防误改要靠显式 `ForbiddenHouses`（见 A15），不靠拼写错误。排查法：grep 全部 INI 的 `Huavia`，出现即处理（除非是有意的历史说明文字）。

### A15. 原版单位显式禁用：ForbiddenHouses 保险（NAFLAK / SAPC 案例）

**需求**：确保华夏不能建造"原版防空炮 NAFLAK"和"野牛运输艇"（= `[SAPC]`，Name=**Zubr Transport**，Zubr 即野牛；Prerequisite=NAYARD、Image=TRS）。

**现状判断**：这两个原版单位的 Owner 里本来就没有有效 `Huaxia`（NAFLAK 只有无效 `Huavia`），华夏原本就造不了——**但"原本造不了"依赖拼写错误，不安全**（见 A14）。

**做法**：延续 `rulesmo_huaxia.ini` 开头的"原版苏联单位禁用区"模式（E2/FLAKT/SUB/DRED/DBOAT 同款写法），显式追加：

```ini
; 海军禁用区（DBOAT 之后）
[SAPC]
ForbiddenHouses=Huaxia

; NAFLAK 注释之后
[NAFLAK]
ForbiddenHouses=Huaxia
```

**要点**：
- `ForbiddenHouses=Huaxia` 只禁华夏，苏军/盟军/厄普西隆/焚风的原版建造权零影响
- 华夏自己的克隆版（NAFLAKHX、LCRFHX 等）不受影响
- 即使未来有人做"全局替换 Huavia→Huaxia"，显式 ForbiddenHouses 依然拦截——这就是"显式锁定"与"拼写错误挡路"的本质区别
- **识别小技巧**：遇到中文俗称（如"野牛运输艇"）先在 `rulesmo.ini` 搜 `Name=` 找英文名（Zubr=Bison 系），再定位 ID（SAPC）

### A16. INI 健康检查六步法（修改后必跑）

用脚本合并解析三个规则文件（rulesmo.ini → rulesmo_override.ini → rulesmo_huaxia.ini，按键级合并模拟 Ares），依次检查：

1. **拼写变体扫描**：统计所有含 `Huax` 的 token，确认只有 `Huaxia`（+ [Countries] 注释/UIName）
2. **注册一致性**（崩溃高危）：VehicleTypes/InfantryTypes/BuildingTypes/AircraftTypes 中每个注册 ID 必须有定义节 `[该ID]`
3. **HX 单位归属**：所有注册的 `*HX` 单位 Owner 必须为 `Huaxia`
4. **悬挂引用**：Primary/Secondary/Elite\*/Warhead/Image/Prerequisite/Spawns——注意：
   - 武器/弹头**不注册**，直接节引用（`Primary=某武器` 需存在 `[某武器]` 节）
   - `Image=` 引用的是模型名（SHP/VXL），无规则节**正常**（如 `Image=SEAL`、`Image=BSHIP`；除非克隆体与原版 Image 不一致）
   - `SOVWEAP`/`SOVTECH` 是 `[GenericPrerequisites]` 虚前置；`PROC2`/`POWER`/`BARRACKS` 是引擎内置虚前置——都不是节，**别误报**
   - `Primary=none` 是合法值
   - ⚠️ **解析坑**：节头可带注释（`[RuptureBeam] ; scavenger rupture beam`），正则必须用 `^\[(.+?)\]\s*(;.*)?$` 匹配，否则漏节导致误报（本次 `RuptureBeam`/`Nanofiber1Weapon` 即因此被误判为悬挂）
5. **超武引用**：只查 `SW.Designators` / `SW.Inhibitors` 引用的建筑是否存在；**其他 `SW.*` 键的值是动画/声音/枚举/数值，不是建筑引用，不要误报**
6. **debug.log 基线对比**：只处理**新增**的 `[Developer error]`/`Failed to parse`；`BuildingTypes > 512`（MO 固有）和 `Unrecognized UI action: show`（内置）是已知无害项

**2026-08-18 检查结论**：六步全过——零拼写错误、零注册缺失、23 个 HX 单位归属全对、零悬挂武器/弹头、超武无残留、日志无新增 error。

### A17. 废弃克隆定义清理（ARMAHX 案例）

**现象**：`[ARMAHX]`（华夏犰狳，克隆 ARMA）有完整定义（Image=ARMA、Primary=ReaperCannonHX、Owner=Huaxia）但**未注册**——实际注册的是 `370=ARMAHX2`。引擎不加载未注册节，**无害**，但属于 A6 类"废弃残留"，会误导后人以为华夏有 ARMAHX 单位。

**清理前必须确认**：
1. grep `ARMAHX[^2]` 确认无任何引用（注册表/其他单位 Primary 等）
2. **节内武器是否被共享**：`ReaperCannonHX`/`ReaperCannonHXE` 被 ARMAHX2 继续使用 → **只删单位节，武器保留**

**删除方式**：`rulesmo_huaxia.ini` 含 GBK 中文注释，**用二进制按节边界删除**（`[ARMAHX]` 行 → 下一个 `[` 节头前，连同上方紧邻的注释行），不要用文本模式（会破坏编码）。删除后验证：残留只剩 ARMAHX2 相关引用；文件行数减少正确。

**教训**：克隆定义中途废弃时，要么注册、要么整节删除，不要留"半成品"。

### A18. 原版 Owner/ForbiddenHouses 重叠 ≠ 错误（区分原版写法与华夏覆盖）

**现象**：健康检查发现 9 处 `Owner` 与 `ForbiddenHouses` 重叠（DESO/YURI/FV/HTK/BOREK/DTRUCK/BUZZ/RACC/NAWEAPB）——例如 `[FV] Owner=Europeans,UnitedStates,Pacific` + `ForbiddenHouses=Europeans,Pacific`（实际仅美国能造）。

**判定**：这与 A2 的"华夏覆盖自相矛盾"是**两回事**——
- **原版节的"Owner 全列表 + Forbidden 排除"是 MO 的合法设计**（用差集精确控制谁能造），游戏一直正常运行，**不要误改**
- 只有**华夏自己写的覆盖节**出现重叠（先留再禁）才是 A2 的错误

**检查前先区分节来源**：该节是否在 `rulesmo_huaxia.ini` 中被覆盖？没有 → 原版写法，跳过。

**另**：`[MadBlastStartAI] Warhead=MadAIWH` 的弹头无定义节——**原版 MO 遗留**（Mental Omega 1.1 旧目录同样写法），游戏正常，不要动。
