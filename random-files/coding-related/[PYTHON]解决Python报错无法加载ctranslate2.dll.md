> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0QVvHh4tCj2

# you asked

message time: 2026-09-17 23:18:13

![image](https://hunyuan.tencent.com/api/resource/download?resourceId=99483188e1b977cb37aaf61c859626c0_91)

解决这个文件

---

# yuanbao response

这个报错表明在运行 Python 程序时，系统无法加载 `ctranslate2.dll` 文件。错误代码 `0xc0e90002`（STATUS_SYSTEM_INTEGRITY_POLICY_VIOLATION）通常意味着该 DLL 文件缺失了必要的依赖、与当前系统不兼容、文件已损坏，或者被 Windows 的安全策略（如智能应用程序控制）拦截[citation:3][citation:5]。

结合你的路径（`miniconda3\Lib\site-packages\ctranslate2\`），这是你在 Conda 环境中安装的 `ctranslate2` 库出了问题。建议按以下优先级步骤进行排查和解决：

### 方法一：安装/修复 Visual C++ 运行库（最常见原因）
`ctranslate2.dll` 依赖 Microsoft Visual C++ 运行库（如 `MSVCP140.dll` 和 `VCRUNTIME140.dll`）。如果系统缺少这些依赖，就会报映像损坏的错误。
1. 前往微软官网，下载最新版的 **Visual C++ Redistributable for Visual Studio 2015-2022**。
2. 建议将 **x64** 和 **x86** 版本都下载并安装一遍[citation:2]。
3. 安装完成后，重启电脑，再次运行你的 Python 程序测试。

### 方法二：关闭 Windows“智能应用程序控制”
错误码 `0xc0e90002` 经常出现在 Windows 11 中，这是因为系统严格的“智能应用程序控制 (Smart App Control)”或 Defender 安全策略将非官方签名的 `ctranslate2.dll` 误判并拦截了[citation:3][citation:5]。
1. 点击 Win + I 打开**设置**。
2. 进入 **隐私和安全性** > **Windows 安全中心** > **应用和浏览器控制**。
3. 找到 **智能应用程序控制设置 (Smart App Control)**，将其**关闭**[citation:3][citation:5]。
4. 重启电脑后再试。

### 方法三：重新安装 ctranslate2 库（针对 Conda 环境）
该 DLL 文件可能是在安装时下载了不兼容的版本或文件本身已损坏[citation:8]。
1. 打开你的 **Anaconda Prompt**（或终端）。
2. 激活你报错的那个 Conda 环境（如果不在 base 环境的话）：
   ```bash
   conda activate 你的环境名称
   ```
3. 卸载当前损坏的库：
   ```bash
   pip uninstall ctranslate2
   ```
4. 清理缓存后重新安装（建议指定与你项目兼容的版本，或者直接安装最新版）：
   ```bash
   pip install ctranslate2 --force-reinstall --no-cache-dir
   ```
   *(注：如果你使用的是 CUDA 加速，请确保安装的 `ctranslate2` 版本与你的 CUDA 版本匹配；如果是 CPU 版，直接安装即可。)*

### 方法四：修复系统文件
如果上述方法无效，可能是 Windows 系统核心文件受损。
1. 在底部搜索栏搜索 `cmd`，右键点击“命令提示符”，选择**以管理员身份运行**。
2. 输入以下命令并回车[citation:3][citation:6]：
   ```bash
   sfc /scannow
   ```
3. 等待扫描和修复完成后，继续输入以下命令并回车[citation:3][citation:6]：
   ```bash
   DISM /Online /Cleanup-Image /RestoreHealth
   ```
4. 修复完毕后重启电脑。

**💡 补充建议**：
如果你最近刚安装或更新了某个依赖翻译功能（如 Whisper、机器翻译等）的软件，请确保该软件所在的文件夹没有被杀毒软件（如 360、火绒等）隔离或误杀[citation:3]。如果报错在开机时出现，可以按 `Ctrl+Shift+Esc` 打开任务管理器，在“启动”选项卡中禁用相关的 Python 后台启动项[citation:2]。