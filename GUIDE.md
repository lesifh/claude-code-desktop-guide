# Windows 11 操作手册：一键部署 Claude Code，使用 AionUi 桌面界面连接 DeepSeek

核对日期：2026 年 10 月 30 日。

适用于普通 Intel / AMD x64 的 Windows 11 电脑。ARM Windows 不建议直接套用本文的一键安装器：当前源码的架构检测及安装包选择没有充分区分 ARM64。

本手册根据项目发布页、安装源码和官方说明整理，尚未完成实机安装验证。软件版本变化时，以实际界面为准。

## 一、最终使用方式

**打开 AionUi → 选择 Claude Code → 打开项目文件夹 → 用中文提出任务。**

各部分的分工：

| 部分 | 用途 |
| --- | --- |
| 一键部署工具 | 帮助安装 Node.js、Claude Code、CC Switch |
| CC Switch | 设置 Claude Code 使用的 DeepSeek 地址、Key 和模型 |
| Claude Code | 负责读写文件、执行命令和完成任务 |
| DeepSeek | 提供模型推理，费用计入 DeepSeek API 账户 |
| AionUi | 提供独立桌面窗口，连接本机 Claude Code |

**新建会话时选 Claude Code。** AionUi 的“内置 Agent / Built-in Agent”是另一套执行系统，不是本文要配置的方案。外部 Agent 使用各自的认证和配置。[AionUi 接入说明](https://github.com/iOfficeAI/AionUi/wiki/ACP-Setup)

## 二、准备下载与账户

在日常使用的 Windows 账户下操作，准备以下内容：

1. [一键部署项目发布页](https://github.com/K1evL/claudecode-setup/releases)。当前发布包为 `claudecode-setup.zip`。
2. [CC Switch 发布页](https://github.com/farion1231/cc-switch/releases/latest)。安装器未成功安装 CC Switch 时使用。
3. [AionUi 官方下载页](https://www.aionui.com/download/)。普通电脑选 Windows x64 安装包。
4. [DeepSeek 开放平台](https://platform.deepseek.com/)。创建自己的 API Key，并确认 API 账户有可用余额或额度。

注意：AionUi 当前 GitHub 发布说明写明安装包已不再放在 GitHub，需从官网获取；旧教程中的“在 GitHub Assets 找 EXE”不一定适用。[AionUi 发布说明](https://github.com/iOfficeAI/AionUi/releases/latest)

妥善保管 Key，后续在本机填写。不要放到项目文件、聊天截图或公开仓库。

## 三、运行一键部署工具

### 3.1 下载并解压

1. 打开一键部署项目的 [Releases 页面](https://github.com/K1evL/claudecode-setup/releases)。
2. 展开 **Assets**，下载 `claudecode-setup.zip`，不是 Source code 源码包。
3. 右键 ZIP →“全部解压”，例如解压到“下载”中的 `claudecode-setup` 文件夹。
4. 在解压后的文件夹找到 `ClaudeCode-Setup.exe`，保留压缩包里与它配套的文件，例如 `.exe.config`，不要只移走 EXE。

这是第三方社区安装器，并非 Anthropic 官方安装程序。当前源码会配置 npm 镜像、全局安装目录和 Path，并可能修改 PowerShell 执行策略。已有 Node.js 开发环境的电脑先记录原配置；已有重要 CC Switch 配置时先备份。[安装脚本](https://github.com/K1evL/claudecode-setup/blob/main/src/scripts/install.ps1)

### 3.2 安装

1. 右键 `ClaudeCode-Setup.exe` → **以管理员身份运行**。
2. 核对程序来源后确认 Windows 权限提示。
3. 等待安装完成，保留末尾报告和日志位置。时间取决于网络，下载较慢时可能超过项目说明的几分钟。
4. 退出安装器，关闭已有终端，再重开一个普通 PowerShell。

管理员权限仅用于此安装器；以后运行 AionUi、CC Switch 和 Claude Code 使用普通权限。

### 3.3 独立验证安装结果

在 PowerShell 逐行执行：

```powershell
node --version
npm.cmd --version
claude --version
where.exe claude
```

前三条应显示版本，最后一条应显示 Claude Code 入口位置。

**不要仅凭安装器显示“完成”就认定所有组件成功。** 当前源码会在部分安装失败时继续生成报告，需要以上面的真实结果为准。[安装模块](https://github.com/K1evL/claudecode-setup/blob/main/src/modules/CcSwitchInstaller.psm1)

若 `claude` 找不到，先重启 Windows 后重试。仍失败时，如果 Node/npm 正常，可在普通 PowerShell 补装：

```powershell
npm.cmd install -g @anthropic-ai/claude-code --registry=https://registry.npmjs.org
```

然后重开终端执行 `claude --version`。如果 npm 报 Node 版本不支持，先从 [Node.js 官网](https://nodejs.org/)升级到当前 LTS，再重试，不要忽略版本错误继续配置。

本流程通过 CC Switch 填写 Key，**不需要执行安装器 README 中的 `claude set-key` 或 `cc-switch config set` 示例**；这些不是本文采用的配置方法。

### 3.4 检查 Git

当前查看到的一键安装主流程没有安装 Git 的步骤，因此单独验证：

```powershell
git --version
```

如未安装，从 [Git 官方下载页](https://git-scm.com/install/windows)下载 x64 安装程序，通常保留默认选项。安装后重开终端，再验证版本。Git 为项目版本管理和 Claude Code 的 Bash 工具提供支持。

## 四、通过 CC Switch 配置 DeepSeek

### 4.1 打开或补装 CC Switch

在桌面或开始菜单打开 CC Switch。若没有安装成功，从 [官方发布页](https://github.com/farion1231/cc-switch/releases/latest)下载普通 Windows 的 `.msi` 安装包，完成安装。

### 4.2 添加并启用 DeepSeek

1. 左侧选择 **Claude Code**。
2. 使用 **直连**模式；如已开启路由或聚合，选择“回到直连”。
3. 点击 **添加供应商 / +**，搜索 **DeepSeek**，选择厂商官方预设。
4. 名称填 `DeepSeek 官方`，Key 填自己的 DeepSeek API Key。
5. 按下表核对模型配置，保存后点击卡片上的 **启用**。

| 字段 | 内容 |
| --- | --- |
| Base URL | `https://api.deepseek.com/anthropic` |
| API 格式（如有） | Anthropic / Anthropic Messages |
| 默认模型 | `deepseek-flash[1m]` |
| Opus 映射 | `deepseek-flash[1m]` |
| Sonnet 映射 | `deepseek-flash[1m]` |
| Haiku 映射 | `deepseek-flash` |
| 子代理模型（可添加） | `deepseek-flash` |

此处使用核对日的 [DeepSeek 官方 Claude Code 接入参数](https://api-docs.deepseek.com/guides/coding_agents/)。未来模型调整时跟随该文档。三档名称只是映射入口，实际调用的是表中的 DeepSeek 模型。

URL 包含 `/anthropic`，不要再自行追加 `/v1/messages`。本方案不需要启用本地路由。

### 4.3 可核对的配置示例

若需要补充字段，在 CC Switch 的供应商配置编辑区域使用下面的 `env`。占位内容替换为真实 Key；有其他配置时合并，不要覆盖整份已有设置。

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "替换为自己的DeepSeek_API_Key",
    "ANTHROPIC_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-flash",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-flash",
    "CLAUDE_CODE_EFFORT_LEVEL": "max"
  }
}
```

字段按 [DeepSeek 官方指南](https://api-docs.deepseek.com/guides/coding_agents/)列出。`max` 是其示例中的思考强度。

如果 Claude Code 首次启动仍要求 Claude 官方登录，退出后，在 **CC Switch → 设置 → 应用配置 → Claude Code** 开启 **跳过 Claude Code 初次安装确认**，再启动。它只处理首次引导，API 仍需有效 Key。[CC Switch 设置说明](https://github.com/farion1231/cc-switch/blob/main/docs/user-manual/zh/1-getting-started/1.5-settings.md)

## 五、先验证 Claude Code 本身

先通过这一关，再安装连接 AionUi，避免同时排查两层问题。

在普通 PowerShell 运行：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\Documents\AI-Test"
Set-Location "$env:USERPROFILE\Documents\AI-Test"
claude
```

若提示信任目录，核对确实是刚建的测试目录后继续。在对话框输入：

> 请只回复“连接成功”，暂时不要运行命令或读写文件。

正常回复后，再输入：

> 请在当前文件夹创建 hello.txt，内容为“你好，DeepSeek！”，然后读取这个文件并展示内容。

核对操作目标后允许。打开“文档 → AI-Test”，确认文件存在且内容正确。再查看 DeepSeek 平台是否有相应用量，可能存在显示延迟。

完成后退出 Claude Code，再开一个全新的 PowerShell 重试对话。新窗口仍能调用，说明配置已经持久保存，而不只是当前终端的临时变量生效。

## 六、安装 AionUi 并选择 Claude Code

### 6.1 安装桌面客户端

1. 打开 [AionUi 官方下载页](https://www.aionui.com/download/)，选择 Windows x64 安装包。
2. 运行安装程序，按提示完成。
3. 用日常账户、普通权限启动 AionUi。首次使用按实际界面选择中文。

### 6.2 连接已安装的 Claude Code

1. 启动后，在首页或新建会话区域查看检测到的 Agent。
2. 选择 **Claude Code**。AionUi 官方说明支持扫描 Path 检测本机 `claude` 命令。
3. 选择工作文件夹：刚才的“文档 → AI-Test”。
4. 新建会话。确认该会话绑定的是 **Claude Code**。
5. 如果出现模型列表，核对当前选择对应 DeepSeek 配置。不要随意改为其他模型名称。

这里不用在 AionUi 的“设置 → 模型”里另建一个内置 Agent 的 DeepSeek 配置；外部 Claude Code 的配置由 Claude Code / CC Switch 管理。按钮位置随版本变化，以界面实际显示的 Claude Code 入口为准。[外部 Agent 说明](https://github.com/iOfficeAI/AionUi/wiki/ACP-Setup)

### 6.3 在图形界面验收

输入：

> 请读取当前文件夹中的 hello.txt，并原样展示内容。暂时不要修改文件。

收到正确内容后，再输入：

> 请在当前文件夹创建 gui-test.txt，写入“这是通过 AionUi 调用 Claude Code 完成的测试”，并读取它确认内容。

核对权限提示和路径后允许。确认文件真实存在，不只看聊天中是否说“完成”。

**通过标准：Claude Code 会话身份正确、能回复、能读取已有文件、能创建新文件，并且退出 AionUi 后从桌面重新打开仍可使用。**

## 七、AionUi 找不到 Claude Code

按顺序检查：

1. 完全退出 AionUi，包括系统托盘中的后台进程，再重新打开。
2. 如刚安装环境或改过 Path，先重启 Windows。
3. 在普通 PowerShell 执行：

```powershell
where.exe claude
claude --version
```

如果命令行也失败，先修复第 3 节。若只在 AionUi 中检测不到，从同一个能运行 `claude` 的 PowerShell 启动 AionUi，检查是否是桌面进程尚未继承新 Path。

启动方法：右键 AionUi 快捷方式 → 属性，查看实际程序的“目标”。在 PowerShell 使用实际 EXE 路径，例如：

```powershell
Start-Process -FilePath '替换为快捷方式中实际的AionUi.exe完整路径' -WindowStyle Hidden
```

先退出旧 AionUi，否则命令可能只激活原有进程。这里的示例路径必须替换；不要原样复制占位文字。

不要直接把 `claude` 加进自定义 ACP Agent 就假定能用；裸 CLI 和 ACP 适配器的启动方式并不相同，应优先使用 AionUi 内置的 Claude Code 集成。[检测与适配说明](https://github.com/iOfficeAI/AionUi/wiki/ACP-Setup)

## 八、命令行能用，但 AionUi 提示未认证

项目中有 Windows + DeepSeek 出现 `Agent requires authentication` 的反馈，维护者要求检查版本和变量是否只设置在终端里。因此这套组合需要实机验收，不能保证所有版本都开箱即用。[相关问题记录](https://github.com/iOfficeAI/AionUi/issues/3095)

先做以下检查：

1. 确认选的是 Claude Code 会话。
2. 确认 CC Switch 已启用 DeepSeek，而非只保存了一张卡。
3. 确认第 5 节“新开终端仍能调用”已通过。
4. 确认 AionUi 与 CC Switch、Claude Code 使用同一 Windows 用户。
5. 如果电脑以前设置过 `CLAUDE_CONFIG_DIR`，核对它是否指向不同配置目录。
6. 记录当前版本后升级 AionUi，必要时升级 Claude Code，再重新测试。

### 8.1 诊断性尝试：把已保存配置传给 AionUi

以下是排查环境变量传递问题的方法，不是保证修复所有认证错误的官方补丁。

先完全退出 AionUi。新开普通 PowerShell，运行下面的代码：它从当前用户默认 Claude Code 配置读取指定连接变量，只放入此 PowerShell 及其子进程，不打印 Key、不写入系统变量。

```powershell
$claudeSettingsPath = Join-Path $env:USERPROFILE '.claude\settings.json'
$claudeSavedSettings = Get-Content -LiteralPath $claudeSettingsPath -Raw | ConvertFrom-Json
if ($claudeSavedSettings.env.ANTHROPIC_BASE_URL -ne 'https://api.deepseek.com/anthropic') {
    throw '当前默认配置不是预期的 DeepSeek 直连地址，请先在 CC Switch 中启用 DeepSeek。'
}
if ([string]::IsNullOrWhiteSpace($claudeSavedSettings.env.ANTHROPIC_AUTH_TOKEN)) {
    throw '当前配置缺少 DeepSeek Token，请先补全供应商配置。'
}
$claudeConnectionNames = @(
    'ANTHROPIC_BASE_URL',
    'ANTHROPIC_AUTH_TOKEN',
    'ANTHROPIC_MODEL',
    'ANTHROPIC_DEFAULT_OPUS_MODEL',
    'ANTHROPIC_DEFAULT_SONNET_MODEL',
    'ANTHROPIC_DEFAULT_HAIKU_MODEL',
    'CLAUDE_CODE_SUBAGENT_MODEL',
    'CLAUDE_CODE_EFFORT_LEVEL'
)
foreach ($connectionName in $claudeConnectionNames) {
    $connectionValue = $claudeSavedSettings.env.$connectionName
    if (-not [string]::IsNullOrWhiteSpace($connectionValue)) {
        [Environment]::SetEnvironmentVariable($connectionName, [string]$connectionValue, 'Process')
    }
}
```

随后在同一个窗口，用第 7 节的方法启动真实路径的 AionUi，再新建 Claude Code 会话测试。

如果这样可以调用，但桌面双击仍失败，就说明日常启动的配置传递还没解决，暂不能算安装验收通过。应检查该版本的集成配置，或携带遮盖 Key 的版本信息、错误和日志向 AionUi 项目反馈。

如果仍提示认证失败，不要为此填写一个虚假的 Claude Key、删除全部配置或关闭权限控制。保留能正常工作的 Claude Code 配置，先把 AionUi 的适配错误查清。

## 九、其他常见问题

| 现象 | 处理方法 |
| --- | --- |
| CC Switch 没有快捷方式 / 安装失败 | 从 CC Switch 官方发布页补装，验证能打开。 |
| 安装器卡住或报下载失败 | 看安装日志中失败的组件，检查网络；不要反复重跑覆盖已经正常的组件。 |
| `claude.ps1` 被执行策略阻止 | 尝试 `claude.cmd --version`；不要为了这一点继续降低全系统执行策略。 |
| Git Bash 找不到 | 安装 Git；确有 `C:\Program Files\Git\bin\bash.exe` 时，可在 CC Switch 供应商 env 增加 `CLAUDE_CODE_GIT_BASH_PATH` 指向实际路径。 |
| 401 / Key 无效 | 核对 DeepSeek Key 完整、没有空格，确认当前供应商已启用。 |
| 余额不足 | 到 DeepSeek 开放平台检查 API 余额。 |
| 404 | 检查地址是否为 `https://api.deepseek.com/anthropic`，协议是否为 Anthropic。 |
| 400 / 模型不存在 | 根据 DeepSeek 官方当前指南更新模型名和各档映射。 |
| CC Switch 检测连通成功，聊天失败 | 连通检测不代表 Key 和模型有效，必须实际发消息。 |
| AionUi 模型页配好了 DeepSeek，但 Claude Code 会话没变化 | 模型页主要配置 AionUi 自己的模型；回 CC Switch 检查 Claude Code 的供应商。 |
| 修改配置后仍用旧模型 | 完全退出旧会话及 AionUi，检查旧环境变量和项目模型设置，再重启测试。 |

## 十、日常使用

1. 双击桌面 AionUi。
2. 新建 **Claude Code** 会话。
3. 选择要处理的项目文件夹。
4. 输入具体任务，例如：“先阅读项目，告诉我怎么运行，暂时不要修改文件。”
5. 查看文件修改与命令请求，再允许操作。
6. 更换 API Key 或模型时，在 CC Switch 修改、保存并启用，再重开 AionUi 会话。

不要把不同的项目都放在同一个会话里。重要项目在修改前先备份或提交 Git。

## 十一、安装验收清单

- [ ] Node/npm、Claude Code 能显示版本，Git 可用。
- [ ] 普通用户终端可以找到 `claude`。
- [ ] CC Switch 已启用 DeepSeek 直连配置。
- [ ] 新开终端中的 Claude Code 对话、文件读写成功。
- [ ] AionUi 能检测并选择 Claude Code。
- [ ] AionUi 中对话、读取 hello.txt、创建 gui-test.txt 成功。
- [ ] 从桌面重新打开 AionUi 后仍然可用，无需依赖临时终端变量。
- [ ] 能够在 DeepSeek 平台查看余额和用量。
- [ ] 已记录 AionUi、Claude Code、CC Switch 的可用版本，Key 已妥善保管。

## 参考链接

- [一键部署项目](https://github.com/K1evL/claudecode-setup)
- [一键部署 ZIP 发布页](https://github.com/K1evL/claudecode-setup/releases)
- [CC Switch 发布页](https://github.com/farion1231/cc-switch/releases/latest)
- [DeepSeek Claude Code 接入指南](https://api-docs.deepseek.com/guides/coding_agents/)
- [AionUi 官方下载](https://www.aionui.com/download/)
- [AionUi 外部 Agent 连接说明](https://github.com/iOfficeAI/AionUi/wiki/ACP-Setup)
- [AionUi DeepSeek 认证问题记录](https://github.com/iOfficeAI/AionUi/issues/3095)
