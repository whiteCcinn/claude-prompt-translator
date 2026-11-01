# Claude Prompt Translator

把提示词翻译成指定语言，再交给 Claude Code。翻译可使用外部模型 API 或本地 Codex CLI，默认英文，支持 `en / jp / cn / fr`。

**接入只需要：安装插件 → 选择翻译方式 → 选目标语言。** 日常仍然直接运行 `claude`。

```text
你的输入 -> API / 本地 Codex CLI -> 译文 -> Claude Code
                     |
                     +-> 翻译进度、耗时、译文预览
```

## toy 的实现原理手稿

![toy 手绘 Claude Prompt Translator 实现原理](assets/architecture/principle.png)

[可缩放 SVG](assets/architecture/principle.svg) · [可编辑 Excalidraw](assets/architecture/principle.excalidraw)

手稿展示 API 模式；Codex CLI 模式在同一提交钩子内调用本地进程翻译。

## 视频演示

[![Claude 多语言Prompt 转换 — Toy张](https://i.ytimg.com/vi/VBJLDWzfLpI/hqdefault.jpg)](https://www.youtube.com/watch?v=VBJLDWzfLpI)

[在 YouTube 观看：Claude 多语言Prompt 转换](https://www.youtube.com/watch?v=VBJLDWzfLpI)

本页继续说明怎么安装和接入；实际使用效果见视频。

## 1. 安装插件

先确认 Claude Code 可运行。本项目在 **Claude Code 2.1.291** 上验证，使用同版本或更新版本。

在终端执行：

```bash
claude plugin marketplace add cfrs2005/claude-prompt-translator
claude plugin install prompt-translator@claude-prompt-translator
```

然后运行 `claude`。已有会话可以执行 `/reload-plugins` 加载新安装的插件。

也可以在安装时直接选择 Codex CLI 和目标语言（以下参数已在 Claude Code 2.1.316 验证）：

```bash
claude plugin install prompt-translator@claude-prompt-translator \
  --config backend=codex-cli \
  --config language=en
```

## 2. 首次配置：选择翻译方式

安装后的首次配置和后续修改都使用 Claude Code 原生插件配置。在 Claude Code 中执行：

```text
/plugin configure prompt-translator@claude-prompt-translator
```

先选 **Translation backend**，再选 **Target language**，最后按模式填写对应字段并保存。Claude Code 的登录继续按你原来的方式使用。

| Translation backend | 需要配置 |
| --- | --- |
| `api`（默认） | API URL、Translation model、API Key |
| `codex-cli` | 已安装并登录的 Codex；Codex executable 通常保持 `codex` |

选择 `codex-cli` 时，API Key 可以留空，其余 API 字段不参与翻译。安装时未配置也可以稍后再打开这个入口。

### API 模式（默认）

填写下面三项。这里的 API Key 属于翻译服务。

| 字段 | 怎么填 |
| --- | --- |
| **API URL** | 完整的 OpenAI 兼容 `chat/completions` 地址，不能只填域名或 `/v1` |
| **Translation model** | 翻译接口支持的模型 ID |
| **API Key** | 该翻译接口的密钥，输入会遮罩并存入 Claude Code 的安全存储 |

默认提供 GLM 接入值，只需填写自己的 Key：

```text
API URL:           https://open.bigmodel.cn/api/coding/paas/v4/chat/completions
Translation model: glm-5.3-flash
Reasoning effort:  low
```

换成其他服务时，同时修改 **API URL** 和 **Translation model**。如果接口不接受 `reasoning_effort`，把 **Reasoning effort** 选为 `default`，插件就不会发送这个字段。

接入自己的网关或本地模型，请看 [模型接入说明](docs/integration.md)。

### Codex CLI 模式

先确认在启动 Claude Code 的同一环境中，Codex CLI 已安装、可运行且已登录：

```bash
codex --version
codex login status
```

在插件配置中填写：

```text
Translation backend: codex-cli
Codex executable:    codex
```

插件会调用 `codex exec`，复用本地 Codex 的登录态、模型和服务配置，无需填写翻译 API URL、模型或 API Key。若 Claude Code 找不到命令，将 **Codex executable** 改为 Codex 可执行文件的绝对路径，只填路径，不加参数。

每次翻译启动一个临时 Codex 会话，超时为 120 秒。提示框会显示 `Codex CLI`；失败时停止提交，不会自动回退到 API。Codex 仍通过自己的配置访问模型服务，本地 CLI 模式不代表离线推理。更多说明见 [Codex CLI 接入](docs/integration.md#codex-cli-模式)。

## 3. 选择语言并检查

在 `/config` 中找到本插件的 **Target language**：

| 代码 | 目标语言 |
| --- | --- |
| `en` | 英文，默认 |
| `jp` | 日文 |
| `cn` | 简体中文 |
| `fr` | 法文 |

语言选项会同时控制提示词翻译和 Claude 的回复语言。修改后自动生效。

提交一句普通文本，例如“请简要介绍这个项目”。输入框上方会显示：

```text
+---------------------------------------------+
| 翻译中...  1.2 秒 · 英文 (en) · glm-5.3-flash |
| 自动识别 -> 英文                             |
+---------------------------------------------+
> 
```

成功后显示译文预览，3 秒后收起。失败时显示原因并停止这次提交，原文不会被悄悄放行。命令、空输入和以 `! / #` 开头的输入直接放行。

查看安装状态：

```bash
claude plugin list
```

## 后续按需切换

随时运行 `/plugin configure prompt-translator@claude-prompt-translator` 修改 **Translation backend**。也可以在 `/config` 中找到该选项，选择 `api` 或 `codex-cli`。

配置保存后由 Claude Code 重新加载插件，后续提交使用新的翻译方式；已填写的另一种模式的配置会保留。日常直接运行 `claude`。

## 更新和停用

```bash
claude plugin marketplace update claude-prompt-translator
claude plugin update prompt-translator@claude-prompt-translator
```

更新后在当前会话运行 `/reload-plugins`。

```bash
claude plugin disable prompt-translator@claude-prompt-translator
```

停用后重新加载插件或重新打开 Claude Code。

## 接口与数据

- API 模式须使用 OpenAI 兼容接口，并返回 `choices[0].message.content`。
- API 模式使用翻译服务的密钥和额度；Codex CLI 模式使用本地 Codex 的认证和服务配置。
- 路径、URL、行内代码和完整代码块会先替换成标记，翻译后原样放回。标记缺失时停止提交。
- 即使输入已是目标语言，也会调用所选翻译后端，并要求原文返回。
- Claude 的回复语言通过随提交附带的指令引导；插件不翻译 Claude 已生成的回答。
- 动态提示框用于 Claude Code 终端的交互会话。`claude -p` 同样翻译，但不显示提示框。

## 更多说明

- [模型接入、环境变量与故障排查](docs/integration.md)
- [开发和验证](CONTRIBUTING.md)
- [MIT License](LICENSE)
- [Claude Code 插件安装](https://code.claude.com/docs/en/plugins/install)
- [Claude Code Mods API](https://code.claude.com/docs/en/plugins/mods/api)
