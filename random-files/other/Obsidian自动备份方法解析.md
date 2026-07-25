> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0P10Ppuv3UP

# you asked

message time: 2026-07-25 12:34:32

如何自动备份 Obsidian？

---

# yuanbao response

Obsidian 的仓库本质上就是一个 Markdown 文件夹，**最简单的备份方式 = 定时把整个文件夹复制一份到异地**。但官方明确提醒过一句关键的话：**"同步 ≠ 备份"**——iCloud、OneDrive、Dropbox、Obsidian Sync 这些服务会把你的误删、错误编辑同样同步到所有设备，它们防不了"数据损坏或误删"[citation:2]。

所以真正可靠的自动备份，要分两层来做：**本地/局域网定时副本 + 远程版本化备份**。下面按"省事程度"从低到高给你三套方案，挑一套适合你的即可。

## 方案一：Local Backup 插件（最简单，零命令行）

适合不想碰 Git、只要"自动留底"的用户。

1. Obsidian → 设置 → 第三方插件 → 浏览 → 搜 **Local Backup** → 安装并启用[citation:15]
2. 插件设置里关键点：
   - **输出路径**：指向一个**外置硬盘**或**网盘同步文件夹**（Dropbox/OneDrive 的本地目录都行），这样即使电脑硬盘挂了也有副本
   - **Backup on startup / Backup on quit**：开启，每次开关 Obsidian 自动打一个 zip 包
   - **Interval backups**：开启，建议 ≥ 10 分钟（太频繁会吃 CPU 和磁盘 I/O）[citation:15]
   - **备份保留期**：设 5-7 天，避免无限堆积
3. 如果想排除 `.trash`、`*.mp4` 等大文件/临时文件，在 Excluded directories 里用通配符配置[citation:15]

> 💡 这套方案的优势是"设完就忘"，缺点是**没有版本历史**——每个 zip 都是当下全量快照，回滚粒度取决于备份频率。

## 方案二：Obsidian Git 插件（推荐，带完整版本历史）

这是 Obsidian 圈最主流的自动备份方式，**每一次保存都是一次 commit，可以回到任意时间点**[citation:5]。官方帮助页也把它列为社区备份插件的首选[citation:2]。

### 准备工作

先在 GitHub/GitLab/Gitee 建一个**私有仓库**（不要公开，笔记是隐私数据）。

### 初始化仓库

在 Obsidian 仓库根目录打开终端：

```bash
git init
git remote add origin git@github.com:你的用户名/obsidian-vault.git
# 创建 .gitignore，排除易变/不必要文件
cat > .gitignore <<EOF
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/cache/
.trash/
.DS_Store
EOF
git add .
git commit -m "init vault"
git push -u origin main
```

`workspace.json` 必须排除——它记录窗口布局，多设备间会冲突且没价值[citation:4][citation:5]。

### 安装并配置 Obsidian Git

1. 社区插件商店搜 **Obsidian Git**（作者 Vinzent03）→ 安装启用
2. 关键配置[citation:5][citation:14]：

| 设置项 | 推荐值 | 说明 |
|---|---|---|
| Auto commit-and-sync interval | **10 分钟** | 自动提交+同步间隔 |
| Auto commit-and-sync after stopping file edits | **开启** | 停笔后才备份，避免编辑冲突 |
| Auto push interval | 10 分钟 | 自动推送到远程 |
| Auto pull interval | 10 分钟 | 多设备时拉取最新 |
| Pull on startup | **开启** | 启动时先拉最新 |
| Push on commit-and-sync | **开启** | 提交后立即推送 |
| Commit message | `vault: {{date}}` | 带时间戳 |

3. 认证建议用 **SSH Key** 而不是 HTTPS 令牌，一劳永逸免输密码[citation:4]：

```bash
ssh-keygen -t ed25519 -C "your@email.com"
# 把 ~/.ssh/id_ed25519.pub 内容贴到 GitHub → Settings → SSH Keys
```

配置完后基本零感知，Obsidian 在后台默默每 10 分钟推一次。

> ⚠️ 注意：用 Git 做备份意味着你的笔记会存储在外部托管平台上（GitHub/GitLab），如果是高度敏感内容，请用 Gitee 私有仓库或自托管的 Gitea[citation:2]。

## 方案三：系统级定时任务（不依赖 Obsidian 运行）

方案一和二的局限是：**必须 Obsidian 在运行才能触发**。如果你希望"哪怕 Obsidian 没开，备份也在跑"，就要用操作系统自带的定时任务。

### 🪟 Windows：任务计划程序

写一个 `sync_notes.bat`：

```batch
@echo off
cd /d "D:\Your\Obsidian\Vault\Path"
git pull origin main
git add .
git commit -m "计划任务自动同步: %date% %time%"
git push origin main
```

然后：开始菜单搜"任务计划程序" → 创建基本任务 → 触发器选"每天"或"当用户登录时" → 操作选"启动程序" → 指向这个 .bat 文件[citation:8]。

如果只是想做本地副本而不用 Git，也可以用 PowerShell 脚本压缩打包 + 任务计划触发[citation:17]。

### 🍎 macOS：launchd（不要用 cron）

Mac 上 cron 在电脑睡眠时会跳过执行，苹果官方推荐用 launchd[citation:13]。写一个 plist 放到 `~/Library/LaunchAgents/`：

```xml
<key>ProgramArguments</key>
<array>
  <string>/usr/bin/rsync</string>
  <string>-razu</string>
  <string>--delete-after</string>
  <string>/Users/你的用户名/Documents/Obsidian/</string>
  <string>外接硬盘或远程服务器:/path/to/backup/</string>
</array>
<key>StartInterval</key>
<integer>3600</integer>
<key>RunAtLoad</key>
<true/>
```

`StartInterval` 单位是秒，3600 = 每小时备份一次[citation:13]。

## 推荐的"黄金组合"

按业界 **3-2-1 备份原则**（3 份副本、2 种介质、1 份异地）[citation:3][citation:12]，最稳的组合是：

1. **主仓库**：电脑本地，日常使用
2. **Local Backup 插件**：每小时打 zip 包 → 输出到外接 SSD（本地介质）
3. **Obsidian Git**：每 10 分钟 commit+push → GitHub 私有仓库（异地+版本历史）

这样即便出现"误删文件 + 外接盘损坏"的双重打击，你还能从 GitHub 的历史 commit 里捞回任意一天的状态。

> 📌 **避坑提醒**：
> - 不要把仓库直接放在 iCloud/Dropbox 的同步目录里同时又用 Git 管理，两套机制同时写同一个文件极易产生冲突文件[citation:9]
> - `.gitignore` 务必排除 `workspace.json`，否则多设备间 90% 的冲突都来自它[citation:4]
> - 如果用 Gitee 替代 GitHub，国内推送速度会快很多，适合大仓库[citation:18]

按技术熟悉度选：完全新手选**方案一**，愿意花半小时配置选**方案二**（最推荐），运维思维强选**方案三**。

---
Powered by [AI Exporter](https://saveai.net)