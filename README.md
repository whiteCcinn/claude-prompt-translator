# Claude Prompt Translator

分别配置输入语言、发给 Claude 的语言、Claude 回复语言和最终输出语言。翻译可使用外部模型 API 或本地 Codex CLI，支持 `en / jp / cn / fr`，输入还支持自动识别。

**默认：中文输入 → 英文提示词 → Claude 英文回复 → 中文输出。** 每一段语言都可修改，日常仍然直接运行 `claude`。

从 **0.3.0** 起，默认只在界面显示译文，模型历史保留 Claude 原始回复（默认英文）。后续请求不会因为输出翻译而读到中文译文。

```text
你的输入 -> 输入翻译 -> Claude -> 输出翻译 -> 最终输出
  中文       英文       英文回复     中文       中文

输入翻译与输出翻译共用你选择的 API / 本地 Codex CLI
```

## toy 的实现原理手稿

![toy 手绘 Claude Prompt Translator 实现原理](assets/architecture/principle.png)

[可缩放 SVG](assets/architecture/principle.svg) · [可编辑 Excalidraw](assets/architecture/principle.excalidraw)

手稿展示输入翻译的 API 模式；当前版本也支持 Codex CLI 和独立的输出翻译。

## 视频演示

[![Claude 多语言Prompt 转换 — Toy张](https://i.ytimg.com/vi/VBJLDWzfLpI/hqdefault.jpg)](https://www.youtube.com/watch?v=VBJLDWzfLpI)

[在 YouTube 观看：Claude 多语言Prompt 转换](https://www.youtube.com/watch?v=VBJLDWzfLpI)

本页继续说明怎么安装和接入；实际使用效果见视频。

## 1. 安装插件

先确认 Claude Code 可运行。双向翻译在 **Claude Code 2.1.316** 上验证，使用同版本或更新版本。

在终端执行：

```bash
claude plugin marketplace add whiteCcinn/claude-prompt-translator
claude plugin install prompt-translator@claude-prompt-translator
```

然后运行 `claude`。已有会话可以执行 `/reload-plugins` 加载新安装的插件。

也可以在安装时直接选择 Codex CLI，并指定日文输出：

```bash
claude plugin install prompt-translator@claude-prompt-translator \
  --config backend=codex-cli \
  --config input_language=cn \
  --config language=en \
  --config claude_language=en \
  --config output_language=jp
```

## 2. 首次配置：选择翻译方式

安装后的首次配置和后续修改都使用 Claude Code 原生插件配置。在 Claude Code 中执行：

```text
/plugin configure prompt-translator@claude-prompt-translator
```

先选 **Translation backend**，再配置下面四段语言，最后按模式填写对应字段并保存。Claude Code 的登录继续按你原来的方式使用。

| Translation backend | 需要配置 |
| --- | --- |
| `api`（默认） | API URL、Translation model、API Key |
| `codex-cli` | 已安装并登录的 Codex；Codex 选项默认选择“跟随本地配置” |

选择 `codex-cli` 时，API Key 可以留空，其余 API 字段不参与翻译。安装时未配置也可以稍后再打开这个入口。

### API 模式（默认）

主要字段使用原生选择项；选择“手动输入”后，在紧邻的“手动值”行填写内容。手动值行在未使用时可以留空；选中手动输入却未填值，会停止翻译并提示补齐。这里的 API Key 属于翻译服务。

| 字段 | 怎么填 |
| --- | --- |
| **API URL** | 选择默认 GLM 地址，或选“手动输入”并在下一行填写完整的 `chat/completions` 地址 |
| **Translation model** | 选择 `glm-5.3-flash`，或选“手动输入”并在下一行填写模型 ID |
| **API Key source** | 默认“自动读取凭据”；选“手动输入”时，仅使用下一行填写的 Key |
| **API Key** | 该翻译接口的密钥，保留原生遮罩输入并存入安全存储 |

默认提供 GLM 接入值，只需填写自己的 Key：

```text
API URL:           https://open.bigmodel.cn/api/coding/paas/v4/chat/completions
Translation model: glm-5.3-flash
Reasoning effort:  low
```

换成其他服务时，将 **API URL** 和 **Translation model** 都选为“手动输入”，再填写各自的手动值。如果接口不接受 `reasoning_effort`，把 **Reasoning effort** 选为 `default`，插件就不会发送这个字段。

接入自己的网关或本地模型，请看 [模型接入说明](docs/integration.md)。

### Codex CLI 模式

先确认在启动 Claude Code 的同一环境中，Codex CLI 已安装、可运行且已登录：

```bash
codex --version
codex login status
```

在插件配置中选择：

```text
Translation backend:     codex-cli
Codex executable:        跟随本地配置
Codex translation model: 跟随本地配置
Codex reasoning effort:  跟随本地配置
Codex Fast mode:         跟随本地配置
```

插件会调用 `codex exec`，复用本地 Codex 的登录态和服务配置。**Codex executable** 的“跟随本地配置”使用 `PATH` 中的 `codex`；若 Claude Code 找不到命令，选择“手动输入”，在 **Codex executable · 手动值** 填写可执行文件的绝对路径，只填路径，不加引号或参数。

**Codex translation model** 的顺序为“跟随本地配置” → `gpt-6.1-sol / gpt-6.1-terra / gpt-6-astra / gpt-6-luna` → “手动输入”。预设按 2026-11-01 的 [Codex 官方模型说明](https://learn.chatgpt.com/docs/models) 整理；具体可用性取决于账号和服务。自定义或新增模型可以选择“手动输入”，再填写 **Codex model · 手动值**。

翻译较慢时，可以保持模型“跟随本地配置”，先把 **Codex reasoning effort** 设为 `low` 或 `medium` 比较耗时；该选项也支持单独一行填写手动等级。这些 CLI 选项同时影响输入和输出翻译，不会修改本地 Codex 配置；原有 **Reasoning effort** 仍只用于 API，默认保持 `low`。

**Codex Fast mode** 可单独选择“跟随本地配置”、`on`（请求 Fast）或 `off`（关闭本次翻译的 Fast）。Fast 与模型、推理等级互相独立，可以组合使用 Fast 和 `low` / `medium`；实际可用性取决于模型、账号和服务端，Fast 会增加用量消耗。详见 [OpenAI Fast 模式说明](https://learn.chatgpt.com/docs/agent-configuration/speed#fast-mode)。

安装时也可以指定翻译推理等级：

```bash
claude plugin install prompt-translator@claude-prompt-translator \
  --config backend=codex-cli \
  --config codex_reasoning_effort=low
```

希望翻译同时请求 Fast，可在安装命令中再加 `--config codex_fast_mode=on`，或稍后通过 `/plugin configure prompt-translator@claude-prompt-translator` 修改 **Codex Fast mode**。

更新前已填写的自定义模型、路径和 API 地址会继续沿用；首次在会话内打开配置时，插件会补齐对应的选择项。明确选择“跟随本地配置”或某个预设后，保留的手动值不再参与翻译。API Key 仍由原生安全存储管理，不填充占位密钥。

每次翻译启动一个临时 Codex 会话，超时为 120 秒。提示框会显示 `Codex CLI`；输入翻译失败时停止提交，界面翻译失败时显示重试按钮，不会自动回退到 API。Codex 仍通过自己的配置访问模型服务，本地 CLI 模式不代表离线推理。更多说明见 [Codex CLI 接入](docs/integration.md#codex-cli-模式)。

## 3. 配置语言链路并检查

在 `/config` 或插件配置中分别设置：

| 配置项 | 用途 | 默认值 |
| --- | --- | --- |
| **Input language** | 你输入的语言；可选 `auto` 自动识别 | `cn`，中文 |
| **Claude prompt language** | 输入翻译后交给 Claude 的语言 | `en`，英文 |
| **Claude reply language** | 要求 Claude 回复正文使用的语言 | `same-as-prompt`，默认随提示词使用英文 |
| **Output language** | Claude 回复翻译后，你最终读到的语言 | `cn`，中文 |

语言代码为 `cn`（简体中文）、`en`（英文）、`jp`（日文）、`fr`（法文）。输入语言和输出语言均可自由选择。

例如，中文输入、日文输出，只需将 **Output language** 设为 `jp`，其他保持默认：

```text
中文输入 → 英文提示词 → Claude 英文回复 → 日文输出
```

也可以把输入设为日文、Claude 回复设为法文、最终输出设为英文；四段配置互相独立。将 **Output language** 设为 `same-as-claude`，或与 Claude 回复语言设为相同值，就直接显示回复，省去输出翻译。

**Claude reply language** 通过指令引导正文语言，不控制内部思考语言。**Output translation mode** 决定译文出现在哪里：

| 选项 | 界面 | 模型历史 / `claude -p` |
| --- | --- | --- |
| `display-only`（默认） | 显示译文 | 保留 Claude 原始回复，默认英文 |
| `conversation`（兼容旧版） | 显示译文 | 写入译文；无界面输出也会翻译 |

希望 Claude 尽量接触英文，保持提示词和回复语言为 `en`，使用 `display-only`。旧会话已经写入的中文历史不会自动清除；请在新会话中使用这一设置。其他中文入口及尚未处理的范围见 [上下文语言边界](docs/context-language-boundaries.md)。

提交一句普通文本，例如“请简要介绍这个项目”。输入框上方会显示：

```text
+---------------------------------------------+
| 输入翻译中...  1.2 秒 · 英文 (en) · glm-5.3-flash |
| 中文 -> 英文                                 |
+---------------------------------------------+
> 
```

成功后显示译文预览，3 秒后收起。失败时显示原因并停止这次提交，原文不会被悄悄放行。命令、空输入和以 `! / #` 开头的输入直接放行。

默认模式在回复生成结束后，按当前可见的助手文本块翻译；生成和翻译期间显示等待状态。代码、路径、URL 会保护并还原，纯代码和代码边界不完整的块保留原文；工具参数、工具结果和思考块不参与翻译。失败提示和重试按钮只出现在界面，模型历史保持原样。

显示缓存只保存在内存，重绘不会重复翻译同一文本块；重载插件或恢复会话后，可见正文可能重新翻译。未打开的后台回复不触发显示翻译；主动查看子代理对话时，可见助手正文也可以翻译。

输出翻译也会在输入框上方显示独立进度，例如：

```text
输出翻译中...  0.8 秒 · 中文 (cn) · Codex CLI
英文 → 中文
```

这里的计时从开始调用翻译后端算起，不包含 Claude 生成回复的时间。完成提示显示 3 秒，取消或会话结束时清理进度。同语言直出和纯工具响应不显示输出翻译提示。

查看安装状态：

```bash
claude plugin list
```

## 后续按需切换

随时运行 `/plugin configure prompt-translator@claude-prompt-translator` 修改翻译后端和四段语言，也可以在 `/config` 中修改。

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
- 即使输入已是提示词目标语言，也会调用翻译后端，并要求原文返回。Claude 回复与最终输出配置为同一种语言时，省去输出翻译。
- 输入和输出翻译均使用所选后端。默认按可见文本块调用翻译服务，同一回复的多个文本块可能产生多次调用；`conversation` 模式按模型响应翻译。
- 默认的界面译文不写入模型历史。只有显式选择 `conversation` 才同时改写对话记录；回复语言由指令引导。
- 输入和输出的动态进度框用于交互会话。`claude -p` 默认保留原始回复且不调用输出翻译；需要原来的无界面翻译行为时，选择 `conversation`。

## 更多说明

- [模型接入、环境变量与故障排查](docs/integration.md)
- [上下文语言边界](docs/context-language-boundaries.md)
- [开发和验证](CONTRIBUTING.md)
- [MIT License](LICENSE)
- [Claude Code 插件安装](https://code.claude.com/docs/en/plugins/install)
- [Claude Code Mods API](https://code.claude.com/docs/en/plugins/mods/api)
