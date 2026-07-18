> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0Osa66bSDyK

# you asked

message time: 2026-07-18 21:53:28

有没有办法把微软 Outlook 邮件导成 Markdown 的格式？这样方便汇总起来，存成 Markdown 文件，放到人工智能里，做成一个个人资料库，方便和人工智能互动。

---

# yuanbao response

这个需求现在挺常见，微软自己都下场了——**MarkItDown**（微软官方开源，GitHub 12 万星）原生支持 Outlook 邮件转 MD，是最省事的入口。下面按你的使用强度分几档给方案。

## 🎯 首选：微软 MarkItDown（单封 / 少量，最稳）

微软开源的 `markitdown` 明确把 **Outlook 邮件** 列在支持格式里，和 PDF/Word/Excel 同级[citation:4]。

```bash
pip install 'markitdown[all]'
markitdown 邮件.msg -o 邮件.md
# 也支持 .eml、甚至直接从 PST 里的邮件抽
```

优点是**元数据保留得干净**（发件人、日期、主题、正文层级），出来的 MD 直接能扔进 Obsidian / Logseq / 喂给 AI。对你"做个人资料库"的场景来说，这是门槛最低、结构保留最好的一条路。

---

## 📦 整库倒入：PST → MD（适合历史邮件批量进 AI）

如果你是想把几年累积的 PST 一次性洗成 MD 灌进 AI，**convert-psts** 这个项目就是为"AI-friendly"做的[citation:12]：

```bash
# 只要元数据（token 省）
python -m pst_converter.cli 存档.pst
# 带正文
python -m pst_converter.cli 存档.pst --full
# 保留线程关系（message_id / in_reply_to），对 AI 理解对话链很重要
python -m pst_converter.cli 存档.pst --include-threading
# 输出 json / csv / md 都行
python -m pst_converter.cli 存档.pst -f all
```

> 💡 `--include-threading` 这个参数建议你开着，否则 AI 拿到的是散邮件，不知道谁回复谁。

同档位的还有 **Total Outlook Converter**（付费 GUI，批量 PST/OST 直出 MD，保留文件夹结构）[citation:5]，不想碰命令行的可以走这个。

---

## 🖱 不想敲命令：unmsg（GUI 拖拽）

`unmsg` 是专门把 `.msg` → Markdown/HTML/JSON/EML 的小工具，有桌面 GUI，拖进去点 Convert 就行，附件和内联图会一起抽出来[citation:8]。纯本地、无遥测，Python ≥3.11 可 `pip install unmsg`，也有 Windows 安装包。适合"偶尔存几封进资料库"的节奏。

---

## ⚡ 进阶：Outlook 里一键存进 Obsidian（VBA 宏）

如果你用 Obsidian 做资料库，GitHub 上有个 **Outlook2Obsidian** 宏，Alt+F11 贴进去，在 Outlook 里选中邮件一键存成 MD 进 Vault，自带 **YAML frontmatter**（subject/from/date/to），还会自动在 Obsidian 里打开[citation:2]。

frontmatter 大致长这样，对 AI 检索很友好：

```yaml
---
title: 原邮件主题
from: 张三 <zhangsan@xx.com>
to: ...
date: 2024-01-15
---
```

局限：仅 Windows 版 Outlook 桌面端，不支持附件和会议邀请。

---

## 📥 自动化路线：Power Automate（但有个坑）

PA 可以做"新邮件 → 存 SharePoint / OneDrive"，但**PA 原生不输出 .md**，只能存 .eml 或把正文转纯文本。想变 MD 得在 PA 后面再接一步（比如 Office Script 或 Flow 里调 markitdown）。所以如果你是"存量邮件"场景，PA 不太对口；如果是"今后新邮件自动归档进资料库"，可以 PA 存 .eml → 定时脚本刷成 .md 进 Vault。

---

## 🍜 给你的资料库搭法建议（少走弯路）

> 喂 AI 前这几件事提前想好，比选型更重要：

1. **frontmatter 必须留**：发件人、日期、主题、邮件类型（普通/回复/转发），否则 AI 半年后分不清谁是谁
2. **线程别拆散**：用 `--include-threading` 或 Outlook2Obsidian 自带的 YAML，让 AI 能追对话链
3. **附件单独处理**：邮件里的 PDF/图片，markitdown 也能转（图片走 OCR + EXIF，PDF 走 Azure Doc Intel）[citation:4]，但建议附件和邮件本体分开存、用链接关联，不然 token 爆得快
4. **命名 `{date}_{from}_{subject}.md`**，按发件人 / 项目再建一层文件夹，AI 检索和人工回溯都舒服

---

**我的推荐组合**：日常零散存 → `markitdown` 命令行（一行搞定）；历史整库 → `convert-psts --full --include-threading`；Obsidian 用户直接上 Outlook2Obsidian 宏。

你是想先把历史 PST 洗一遍，还是主要管往后新收到的邮件？方向不同，工具链可以再缩一下。

---
Powered by [AI Exporter](https://saveai.net)