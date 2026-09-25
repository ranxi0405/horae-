# Horae 修改记录

## 项目
- Fork 自 Horae 1.15.1，原作者 SenriYuki
- 本地路径：C:\Users\admin\Documents\GitHub\horae-
- 部署路径：I:\AI\SillyTavern-1.19.0\public\scripts\extensions\third-party\SillyTavern-Horae\
- GitHub：https://github.com/ranxi0405/horae-

## 已完成的改动

### P0 NPC 四层匹配
- _findMentionedNpcs()：全名 / 别名 / 姓氏+敬称 / 关系反查
- NPC filter 里使用 mentionedNpcs.has(name)
- commit: 92b1528

### P1 动态物品检索 + 材料判定
- _getDynamicItemContext()：14 条 actionRules
- isAccessible：储物袋 + 贴身位置 + 当前场景
- _injectMaterialGuard：材料判定注入
- commit: 92b1528

### P2 物品归属判定
- _getHolderGuardText()：防止 AI 把商店物品写成玩家持有
- commit: aea9fec

### iPad 向量记忆修复（关键）
- 病因：iPadOS Standalone PWA 上 Worker 内动态 import 跨域 CDN 会永久 pending
- 修复：本地化 transformers.min.js (877KB) + Worker 相对路径 import
- commit: c5a478d

## 部署流程
1. 在 Git 仓库改文件
2. git commit + git push
3. 拷贝以下文件到 I: 盘对应位置：
   - core/vectorManager.js
   - utils/embeddingWorker.js
   - utils/lib/transformers.min.js（lib 目录需手动创建）
4. 重启 SillyTavern
5. 浏览器 Ctrl+Shift+R 强刷

## 常用调试命令
- window.horaeDebugPrompt        看动态注入内容
- window.horaeDebugChat          看全部注入
- window.Horae.getLatestState()  看当前状态

