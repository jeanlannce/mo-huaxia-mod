# Mental Omega 华夏版 · 规则修改（Huaxia Faction）

基于 **Mental Omega 3.3.6**（Ares 引擎）的华夏阵营（Huaxia）规则修改包。

> ⚠️ 本仓库**仅包含文本规则文件与说明文档**，不含《红色警戒 2：尤里的复仇》或 Mental Omega 的游戏本体、美术资源。使用前请自备正版游戏与 Mental Omega 3.3.6。

## 内容

| 文件 | 说明 |
|---|---|
| `rulesmo.ini` | 主规则文件（已修正历史拼写问题，华夏专属修改在此基础上的子文件覆盖） |
| `rulesmo_huaxia.ini` | **华夏专属子文件**（核心修改：克隆单位/建筑、辐射武器、阵营权限锁定） |
| `rulesmo_override.ini` | 华夏禁用单位覆盖（最后加载） |
| `MO_INI机制详解.md` | INI 关键机制整理（NavalTargeting / Spawn 子机 / 辐射双层配置等） |
| `MO_INI修改经验.md` | 零基础修改手册 + 实战案例 + 健康检查流程 |

## 安装方法

1. 将三个 `.ini` 文件放入游戏根目录（覆盖同名文件，**先备份原文件**）
2. 启动游戏，选择华夏阵营（Huaxia）

## 主要特性

- 华夏阵营完整科技链（NATEKHX 原子核心、NAFLAKHX 防空炮、LCRFHX 运输艇等克隆单位）
- 辐射主题武器体系（武器层 + 弹头层双层配置）
- 原版共享建筑对华夏的显式禁用（`ForbiddenHouses=Huaxia`），避免与原版单位重复
- 2026-08-18 全面清理 `Huavia` 拼写地雷（恢复原版 Owner 列表，杜绝"全局替换误解锁"风险）

## 修改规范（要点）

- **克隆而不改原版**：华夏单位/武器/弹头均克隆到 `rulesmo_huaxia.ini`
- **Owner 三级防护**：`Owner` + `RequiredHouses` + `ForbiddenHouses`
- **显式锁定，不靠拼写错误**：原版单位禁用一律显式 `ForbiddenHouses=Huaxia`
- 每次修改后运行"健康检查六步法"（见 `MO_INI修改经验.md` 附录二 A16）

## 致谢

- [Mental Omega](https://mentalomega.com/) - 本 mod 的基础
- [ModEnc](https://modenc.renegadeprojects.com/) - RA2/YR INI 权威参考
- [Ares](https://ares.strategy-x.com/) - 扩展引擎
