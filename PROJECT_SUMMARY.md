# Horae 改造项目总结

## 一、背景

- 基础版本：Horae 1.15.1（原作者 SenriYuki）
- Fork 仓库：https://github.com/ranxi0405/horae-
- 使用场景：《问道长生》中文修仙游戏，主角"冉汐"
- 设备：Windows 电脑 + iPad（局域网访问）

**关键前提**：SillyTavern 聊天记录注入已关闭  AI 看不到玩家历史指令，这是触发所有改造的根本原因。

## 二、改造目标

1. 降低 Token 消耗（不再无差别注入全部物品/NPC）
2. Token 缓存稳定命中（固定规则与动态数据分离）
3. AI 状态准确（避免 AI 因缺状态自行猜测）
4. 跨设备一致（iPad 与电脑行为一致）

## 三、与原版的核心差异

| # | 改动 | 原版 | 改后 |
|---|---|---|---|
| 1 | 玩家指令提取 | getChat() | eventData.chat（最终 prompt 链路） |
| 2 | Horae 规则注入 | 与动态数据混在一起 | 稳定规则独立插到 [Start a new Chat] 定位符后 |
| 3 | 剧情轨迹注入 | 混在数据 prompt | 独立注入（separate 模式） |
| 4 | 动态物品检索 | importance 驱动 | USER 输入语义驱动 + 14 条 actionRules |
| 5 | NPC 注入 | 只看当前场景 | 四层匹配（全名/别名/姓氏+敬称/关系反查） |
| 6 | 材料判定 | 无 | 硬约束："不得编造材料硬凑成功" |
| 7 | 物品归属判定（P2） | 无 | 硬约束："未购买物品 holder 写店名" |
| 8 | 向量记忆加载 | CDN 动态 import | 本地相对路径 import（iPad 兼容） |

## 四、三大目标达成

| 目标 | 原版 | 改后 | 关键改动 |
|---|---|---|---|
| 降 Token | 全部物品/关键物品常驻 | 按需注入 | #4 |
| 缓存命中 | 整块 prompt 变化 | 稳定前缀 + 动态后缀分离 | #1 #2 #3 |
| 跨设备一致 | iPad 5 分钟超时 | 电脑+iPad 都成功 | #8 |
| AI 准确度 | 缺状态就猜 | 材料判定 + 归属判定 | #6 #7 |

## 五、commit 历史

- c402eef  docs: 补充 FIX_NOTES.md
- c5a478d  fix: iPad 向量记忆本地化 + 完成 P0-P2 改造
- aea9fec  feat: P2 物品归属判定
- 8048a20  backup: patch 前基线快照
- 7287a30  feat: index.js 配套改动（动态 USER 输入提取等）
- 92b1528  feat: 动态物品/NPC 检索 + 材料判定 + 四层 NPC 匹配

## 六、关键约定

1. 主角名硬编码：'冉汐' 在 holderIsUser 白名单
2. 宽松单字匹配：actionRules 用 /丹|药|炉/ 不用完整词组
3. 一对多关系不反查：师兄/师弟等不参与 _findMentionedNpcs
4. 宁多勿漏：宽松匹配优于严格匹配
5. Token 敏感：任何新增注入需谨慎评估

## 七、遗留与暂缓

| 项目 | 状态 | 原因 |
|---|---|---|
| P3 远处物品段 | 暂缓 | 增 800-1200 token/回合 |
| 模型权重本地化 | 暂缓 | 目前电脑+iPad 都能拉到 |

## 八、常用调试命令

- window.horaeDebugPrompt        看动态注入内容
- window.horaeDebugChat          看全部注入
- window.Horae.getLatestState()  看当前状态
- window.Horae.getChat()[N].horae_meta  看某条消息原始 meta

## 九、一句话总结

因为关闭了酒馆聊天记录，AI 看不到玩家指令，所以必须：
1. 改玩家指令提取位置（从最终 prompt 链路取）
2. 改 Horae 规则注入位置（稳定/动态分离，提缓存命中）
3. 改物品/NPC 筛选策略（从"重要/最近"改为"玩家意图驱动"）
4. 改向量记忆加载方式（从 CDN 改为本地，解决 iPad 兼容）

最终实现：Token 消耗下降、缓存命中率提升、跨设备稳定、AI 状态准确。
