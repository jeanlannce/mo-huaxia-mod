# Mental Omega 华夏版（Huaxia Faction）

基于 **Mental Omega 3.3.6**（Ares 引擎）的华夏阵营（Huaxia）规则修改包。

> ⚠️ 本仓库**仅包含文本规则文件与说明文档**，不含《红色警戒 2：尤里的复仇》或 Mental Omega 的游戏本体、美术资源。使用前请自备正版游戏与 Mental Omega 3.3.6。

## 内容

| 文件 | 说明 |
|---|---|
| `rulesmo.ini` | 主规则文件（已修正历史拼写问题，华夏专属修改在此基础上的子文件覆盖） |
| `rulesmo_huaxia.ini` | **华夏专属子文件**（核心修改：克隆单位/建筑、辐射武器、阵营权限锁定） |
| `rulesmo_override.ini` | 华夏禁用单位覆盖（最后加载） |
| `MO_INI机制详解.md` | INI 关键机制整理|
| `MO_INI修改经验.md` | 零基础修改手册 + 实战案例 + 健康检查流程 |

## 安装方法

将三个 .ini 文件放入游戏根目录（覆盖同名文件）

## 主要特性

- 创建独立阵营华夏（程世涛版核武CN）
- 为华夏添加了各方的部分单位，克隆缸（保留原工业工厂）以及华夏阵营完整科技链
- 华夏单位价格提高
- 原版共享建筑对华夏的显式禁用（ForbiddenHouses=Huaxia），避免与原版单位重复

## 致谢

- [Mental Omega](https://mentalomega.com/) - 本 mod 的基础
- [ModEnc](https://modenc.renegadeprojects.com/) - RA2/YR INI 权威参考
- [Ares](https://ares.strategy-x.com/) - 扩展引擎
