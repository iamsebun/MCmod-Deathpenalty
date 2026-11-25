[README.md](https://github.com/user-attachments/files/35413549/README.md)
# MCmod-Deathpenalty
这是一个简单的minecraft死亡惩罚模组，目前支持Fogre1.20.1版本，由DSH制作
# 死亡惩罚（deathpenalty）

Forge 1.20.1 模组。存档内第一次死亡不惩罚，之后 1 小时内的连续死亡会逐级加重惩罚；牛奶无法解除，
只能用投掷物「圣灵祝福」净化。

- modId：`deathpenalty`
- 版本：0.2.0（Forge 47.3.0 / Minecraft 1.20.1 / Java 17）
- 贴图：模组图标与原版风格的临时贴图一律借用原版素材（下界之星 / 凋零骷髅头 / 信标），
  个人绘制的美术贴图留待后续替换。

## 玩法

### 惩罚等级（从 1 小时内第一次死亡开始计时，每次死亡重新计时）
| 等级 | 触发 | 惩罚时长 | 复活后生命 | 饥饿值 | 药水效果 |
| --- | --- | --- | --- | --- | --- |
| I | 1 小时内第 1 次死亡 | 3 分钟 | 锁 10 点（不可恢复） | 4 点（可恢复） | 饥饿 I 3min、缓慢 I / 虚弱 I / 挖掘疲劳 I 各 1min、反胃 5s |
| II | 1 小时内第 2 次死亡 | 5 分钟 | 锁 5 点（不可恢复） | 4 点（可恢复） | 饥饿 I 5min、缓慢 I / 虚弱 I / 挖掘疲劳 I 各 90s、反胃 5s |
| III | 1 小时内第 3 次及以后 | 10 分钟 | 锁 2 点（不可恢复） | 4 点（可恢复） | 饥饿 II 5min、缓慢 II / 虚弱 II / 挖掘疲劳 III 各 90s、反胃 10s；额外赋予 1 小时「意志薄弱」 |

- 存档内**第一次死亡完全豁免**：不计数、不开始计时、不给任何惩罚。
- 每次死亡都会重置 1 小时窗口；超过 1 小时没死，下次死亡回到惩罚 I。
- 生命上限不变，惩罚期间血量被硬锁在 10 / 5 / 2：任何治疗（药水、金苹果、自然回血）都会被压回去，到期自动解锁。
- 惩罚效果每 10 tick 重新补一次，因此**牛奶喝不掉**；只有「圣灵祝福」能清。
- 惩罚生效时**不再往聊天框发提示**，改由物品栏效果 + HUD 呈现。

### 屏幕 HUD
- 惩罚期间在**物品栏上方**显示一行**红底黑字**：`死亡惩罚 <罗马数字> · <倒计时>`（如 `死亡惩罚 II · 4:37`）。
- 倒计时结束后或被圣灵祝福净化后，HUD 立即消失（数据源就是同步到客户端的「死亡惩罚」效果）。
- 身上带「圣灵祝福」时同一位置下方再加一行**金底黑字**：`圣灵庇护 · <倒计时>`（方便看赦免还剩多久）。
- 位置可用命令调整，并会写回配置文件：
  - `/deathpenalty hud` 查看当前偏移
  - `/deathpenalty hud <x> <y>`（像素；**+x 向右、+y 向下**，负数向上）
  - `/deathpenalty hud reset` 回到默认（屏幕水平居中、物品栏正上方）

### 状态效果
| 效果 | id | 图标 | 说明 |
| --- | --- | --- | --- |
| 死亡惩罚 | `deathpenalty:death_penalty` | 凋零骷髅头 | 惩罚期间可见，等级 = 惩罚 I/II/III，时长 = 剩余惩罚时间 |
| 意志薄弱 | `deathpenalty:fragile_will` | 下界之星 | 生命上限线性下调 I 75% / II 50% / III 25% / IV 15% / V 10%；牛奶无效 |
| 圣灵祝福（增益） | `deathpenalty:spirit_blessing` | 信标 | 1 小时，期间可赦免一次死亡惩罚 |

- 意志薄弱效果描述：`诅咒加深，灵魂虚弱。`
- 圣灵祝福效果描述：`大天使的祝福，魂灵得以复苏`
- 死亡惩罚 III 触发时附带 1 小时意志薄弱；在意志薄弱期间每再死一次 +1 级（最多 V 级），升一级刷新时长。

### 圣灵祝福（Spirit Blessing）
- 投掷物；落地后在坠落点展开半径 5 格的圆形粒子阵（约 3 秒）。
- 落地瞬间结算一次：范围内的**所有玩家**（水平 5 格、垂直不限）：
  - 有死亡惩罚 → 解除惩罚、清空连续死亡累计与计时、消除意志薄弱，提示 `天使降福，魂灵充盈`；
  - **没有**死亡惩罚 → 获得 1 小时「圣灵祝福」效果（可赦免一次死亡惩罚）。
- 赦免：带「圣灵祝福」效果时死亡，该次死亡**不给惩罚、不计数、也不开启累计计时**，效果随即消耗。
- 获取：
  1. **远古城市（ancient city）宝箱**，15% 概率出现，每次 1–2 个；
  2. 有序合成（3×3）：
     ```
     金锭   下界之星  金锭
     钻石   回响碎片  钻石
     空     金锭      空
     ```
- 物品 id：`deathpenalty:spirit_blessing`（`/give @s deathpenalty:spirit_blessing 1`）
- 物品描述：`大天使的祝福，魂灵得以复苏`（灰色斜体）

## 配置（`config/deathpenalty-common.toml`）
| 键 | 默认 | 说明 |
| --- | --- | --- |
| `general.enabled` | true | 总开关 |
| `general.chain_window_minutes` | 60 | 连续死亡累计窗口（也是意志薄弱时长） |
| `general.reset_chain_on_blessing` | true | 圣灵祝福是否重置累计计时 |
| `blessing.radius` | 5 | 圣灵祝福半径（格） |
| `blessing.loot_chance_percent` | 15 | 远古城市宝箱出现概率 |
| `blessing.max_per_chest` | 2 | 每个宝箱最多几个 |
| `hud.enabled` | true | 是否显示惩罚 / 庇护 HUD |
| `hud.x_offset` | 0 | HUD 水平偏移（像素，正数向右） |
| `hud.y_offset` | 0 | HUD 垂直偏移（像素，**正数向下、负数向上**） |

## 调试命令（需要权限 2）
- `/deathpenalty status` 查看当前状态（等级、剩余时间、意志薄弱等级、累计次数、圣灵祝福）
- `/deathpenalty apply 1|2|3` 立刻施加对应等级惩罚
- `/deathpenalty reset` 清空所有惩罚与累计（保留「已死过一次」标记）
- `/deathpenalty firstdeath` 恢复成「存档首次死亡」状态（用于重新测试豁免逻辑）
- `/deathpenalty hud [<x> <y> | reset]` 调整 HUD 位置并写回配置

## 开发
- 构建：`gradlew build` → `build/libs/deathpenalty-0.2.0.jar`
- 测试：`gradlew runGameTestServer` → **17 个 GameTest 用例**：惩罚 I/II/III 数值、硬锁血、牛奶无效、
  到期解锁、连续死亡升级、窗口重置、意志薄弱叠加与倍率、可见惩罚效果与牛奶后补回、圣灵祝福效果
  （1 小时 / 赦免一次 / 消耗）、HUD 布局计算、合成表注册、战利品池注入。
  结构模板为 `src/gametest/gameteststructures/deathpenaltygametests.empty.snbt`，
  由 build.gradle 在跑测试前复制进 `run/world/gameteststructures/`（1.20.1 只从世界目录读 `.snbt`）。

## 截图（真机 dev 客户端，1.20.1-Forge 47.3.0）
| 文件 | 内容 |
| --- | --- |
| `docs/screenshots/hud-tier2.png` | 惩罚 II 的 HUD（红底黑字 `死亡惩罚 II · 4:57`）+ 锁血 5 点 |
| `docs/screenshots/effects.png` | 物品栏效果列：饥饿 II / 死亡惩罚 III（凋零骷髅头）/ 意志薄弱（下界之星） |
| `docs/screenshots/blessing.png` | 圣灵祝福落地的金色粒子阵 + `天使降福，魂灵充盈` |
| `docs/screenshots/blessing-hud.png` | 无惩罚时受祝福：金底 `圣灵庇护 · 57:49` |
