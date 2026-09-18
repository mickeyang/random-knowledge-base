> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0QTy6Q1FAX2

# you asked

message time: 2026-09-16 19:01:57

如何在opencode中使用icompify的api

---

# yuanbao response

在 OpenCode 里接 iCompify（积算科技 Token 平台）一般走“OpenAI 兼容自定义 Provider”。iCompify 官方已把调用地址升级为：

- OpenAI 兼容：`https://api.icompify.com/v1` [citation:2]
- Anthropic 兼容：`https://api.icompify.com/anthropic` [citation:2]
- 模型列表：`https://api.icompify.com/v1/models` [citation:2]
- 老地址 `maas.icompify.com:32206 / :32788` 已弃用，别再用。 [citation:2]

下面按最常用 OpenAI 兼容方式配。

## 1. 拿到 iCompify API Key 和模型名
1. 登录 iCompify 控制台（新地址 `https://maas.icompify.com` 或你签约的管理台），创建/复制 API Key。
2. 查可用模型：
```bash
curl https://api.icompify.com/v1/models \
  -H "Authorization: Bearer $ICOMPIFY_API_KEY"
```
返回的 `id` 就是后面 `models` 里要填的模型名（不同账号套餐模型可能不同，以返回为准）。

## 2. 设环境变量（推荐，别把 key 写进 json）
```bash
export ICOMPIFY_API_KEY="sk-..."      # 临时
```
长期可写进 `~/.bashrc`/`~/.zshrc` 或项目 `.env`。

## 3. 写 OpenCode 配置
全局：`~/.config/opencode/opencode.json`（macOS/Linux），Windows：`C:\Users\你的用户\.config\opencode\opencode.json`，或项目根目录 `opencode.json`。

OpenAI 兼容示例：
```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "icompify": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "iCompify",
      "options": {
        "baseURL": "https://api.icompify.com/v1",
        "apiKey": "{env:ICOMPIFY_API_KEY}"
      },
      "models": {
        "替换成实际模型id": {
          "name": "iCompify 默认模型",
          "limit": { "context": 128000, "output": 8192 }
        }
      }
    }
  },
  "model": "icompify/替换成实际模型id"
}
```
要点：
- `baseURL` 用 `.../v1`，不要写到 `/chat/completions`。 [citation:3]
- `npm` 用 `@ai-sdk/openai-compatible` 接大多数兼容网关。 [citation:6]
- `models` 下 key 用 iCompify `/v1/models` 返回的 id；`limit` 按该模型实际上下文/输出改。
- 想接多个模型就继续往 `models` 里加。

如果你要用 iCompify 的 Anthropic 兼容端点（部分 Claude 类模型/代理场景）：
```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "icompify-anthropic": {
      "npm": "@ai-sdk/anthropic",
      "name": "iCompify Anthropic",
      "options": {
        "baseURL": "https://api.icompify.com/anthropic",
        "apiKey": "{env:ICOMPIFY_API_KEY}"
      },
      "models": {
        "替换成实际模型id": { "name": "iCompify Claude 兼容" }
      }
    }
  },
  "model": "icompify-anthropic/替换成实际模型id"
}
```

## 4. 交互方式（可选）
不手写 json 也行：
```bash
opencode auth login   # 选 Other，输入 provider id 如 icompify，贴 key
# 或进 TUI 打 /connect -> Other
```
然后再把 `opencode.json` 里的 `baseURL/models` 补上。 [citation:3][citation:6]

## 5. 启动并验证
```bash
opencode          # 进项目目录启动
/models           # TUI 里应看到 icompify/xxx
```
或指定默认模型直接跑：
```bash
opencode --model icompify/替换成实际模型id
```

排错：
- `/models` 看不到 → 检查 provider id 和 json 里一致、`baseURL` 带 `/v1`、key 环境变量已 export、网络能通 `api.icompify.com`。
- 401/403 → key 无效或没该模型权限；用 `/v1/models` 验证。
- 超时/老地址报错 → 换成 `https://api.icompify.com/v1`，别用 `:32788`。 [citation:2]
- 工具调用不正常 → 确认 iCompify 该模型支持 OpenAI tool_calls；可在 model 项加 `"tool_call": true`（部分 OpenCode 版本支持）。

如果你把 iCompify 控制台里“模型列表”贴出来，我可以直接按真实模型 id 给你写一份完整 `opencode.json`。

---

# you asked

message time: 2026-09-16 19:17:02

![image](https://hunyuan.tencent.com/api/resource/download?resourceId=9aeed5e85e09f5636f2a9d705421b476_91)

output : 8192 什么意思？


对应截图的那个元素？

---

# yuanbao response

**`output: 8192` 的含义：**
在 OpenCode 配置中，`limit.output`（输出限制）代表**模型单次回复最多能生成的 Token 数量**[citation:1]。它是 OpenCode 客户端用来管控成本和安全的“预算闸门”[citation:4]。如果模型生成的回复达到这个数值，就会被强制截断[citation:5]。

**对应截图中的元素：**
它对应截图中参数名称为 **`max_tokens`** 的那一行（建议值为 `16384`，最大值为 `65536`）[citation:5]。

**两者的区别与配置建议：**
- **名称区别**：`output` 是 OpenCode 配置文件里的字段名（对应 API 请求里的 `max_tokens` 或 `max_output_tokens`）[citation:2]。
- **数值区别**：上一轮配置示例中写的 `8192` 是一个保守的常用值（适合代码解释或小修改）[citation:5]。但根据你当前截图中 iCompify API 的参数表，该模型支持更大的输出长度（建议值 `16384`，最大可达 `65536`）。
- **调整建议**：既然截图中 `max_tokens` 的建议值是 `16384`，你可以把 OpenCode 配置里的 `output` 改成 `16384`，这样模型能给出更完整的长回复（不容易被截断）。如果确实需要极长的输出（比如生成长篇文档），可以设得更高（不超过 `65536`）。

补充：截图里被截断的 `presence_pena...` 和 `frequency_pen...` 分别是“存在惩罚”和“频率惩罚”参数，用于控制模型回复的重复度，与 `output` 无关。

---

# you asked

message time: 2026-09-16 19:18:40

"env:ICOMPIFY_API_KEY"

什么意思？我需要加入电脑的系统还是用户变量？

---

# yuanbao response

`{env:ICOMPIFY_API_KEY}` 的意思是：OpenCode 不从 JSON 里读明文 key，而是启动时间去操作系统环境变量里取名叫 `ICOMPIFY_API_KEY` 的值。这样 key 不会写进配置文件、不容易误提交。

## 一、放“系统变量”还是“用户变量”？

单人大概率用**用户变量**就够了。

- **用户变量**：只对你当前 Windows 账号生效。你自己在终端/桌面启动 OpenCode、VS Code 终端，都能读到。安全、不需要管理员，推荐。
- **系统变量**：整台电脑所有账号都生效。仅当你有这几种情况才用：
  - OpenCode 用 Windows 服务 / 计划任务启动（用别的账号运行）
  - 多人共用一台机器都要调 iCompify
  - 某些后台 daemon 不走你的桌面会话

普通开发机：用户变量即可。

## 二、Windows 图形界面添加（最省事）

1. 开始菜单搜“环境变量” → 点“编辑系统环境变量”
2. 弹窗里点「环境变量(N)...」
3. 上半部分“用户变量”区域 → 新建
   - 变量名：`ICOMPIFY_API_KEY`
   - 变量值：`sk-xxxxxxxx`（iCompify 控制台拿到的真实 key）
4. 确定保存，**关掉已开的终端/OpenCode 重新打开**（环境变量不会自动注入已运行程序）
5. 验证：
   - CMD：`echo %ICOMPIFY_API_KEY%`
   - PowerShell：`$env:ICOMPIFY_API_KEY`

> 如果改的是“系统变量”，需要管理员权限；改完同样要重开终端。

## 三、只在本次终端临时用（不写系统）

CMD：
```cmd
set ICOMPIFY_API_KEY=sk-xxxx
opencode
```
PowerShell：
```powershell
$env:ICOMPIFY_API_KEY="sk-xxxx"
opencode
```
bash（WSL/Linux/macOS）：
```bash
export ICOMPIFY_API_KEY="sk-xxxx"
opencode
```
关掉终端就失效，适合测试。

## 四、写进项目 .env（替代系统变量）

OpenCode 项目根目录放 `.env`：
```env
ICOMPIFY_API_KEY=sk-xxxx
```
JSON 里仍写 `{env:ICOMPIFY_API_KEY}`，启动 OpenCode 时会读。注意 `.env` 别提交到 git（加进 `.gitignore`）。

## 五、配置里怎么写才对

```json
"options": {
  "baseURL": "https://api.icompify.com/v1",
  "apiKey": "{env:ICOMPIFY_API_KEY}"
}
```
- 变量名大小写要一致：`ICOMPIFY_API_KEY`，JSON 里写 `{env:ICOMPIFY_API_KEY}`
- 别加引号、别加空格：`"{env:ICOMPIFY_API_KEY}"` 整体作为字符串值可以，但里面不要写成 `{ env:XXX }`
- 设完变量后必须重启 OpenCode，否则它启动时没读到就是空 key，会 401

如果你愿意，我可以按“Windows 单用户 + 全局 OpenCode 配置”给你列一遍从建变量到 `opencode.json` 可复制的完整步骤。