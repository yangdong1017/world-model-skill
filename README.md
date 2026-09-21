# 世界模型

一个用于整理跨窗口对话、语音转写、个人思考和真实决策的 Codex Skill。

它不会试图替用户定义“你是谁”，而是把散落在不同对话中的真实表达整理成一个可追溯、允许矛盾、能够持续更新的世界模型。

## 它解决什么问题

大量有价值的判断会自然出现在聊天、语音和临时思考中。如果这些内容没有被整理，它们往往随着窗口增加而逐渐失去关联。

“世界模型”将这些材料分层保存：

1. **原始来源层：** 保留用户消息原文和来源，便于以后复查。
2. **原子观点层：** 只提取真正包含思想、判断、选择、偏好或预测的内容。
3. **模型推断层：** 保存 AI 根据多条证据提出的解释，并与用户明确说过的话严格分开。
4. **主题总结层：** 按商业、人性、成长、技术、关系等主题呈现用户思想的当前状态和历史变化。

普通操作请求会保留在原始来源层，但通常不会被包装成人生观点。

## 核心原则

- 只有用户本人的原话、选择、明确认同和自述行为，才能作为描述用户的直接证据。
- AI 的回答只能用于理解上下文，不能冒充用户观点。
- 附件、录音转写和他人发言属于外部来源，除非用户明确认同、反对或采用。
- 每条归纳观点都必须能够追溯到用户原话。
- 直接证据、归纳观点和模型推断分开保存。
- 保留矛盾和思想变化，不强行制造一个永远自洽的人格。
- 默认低打扰，不要求用户填写表格或逐条确认。
- 不保存密码、访问令牌、私钥、认证 Cookie 或完整金融账号。

## 功能

- 整理当前对话窗口
- 增量扫描尚未处理的历史对话
- 导入语音转写、笔记和对话导出
- 提取用户自己的原子观点
- 区分明确立场、试探性思考、情绪、问题和模型推断
- 追踪观点的强化、修正、取代和历史变化
- 记录不同观点之间的张力，而不是简单判定自相矛盾
- 按主题重建用户对世界的理解
- 审计误归因、重复记录、证据缺失和过期推断

## 安装

### 使用 Skill Installer

在 Codex 中调用 `$skill-installer`，让它从下面的 GitHub 仓库安装：

```text
https://github.com/yangdong1017/world-model-skill
```

这是私有仓库时，需要当前环境已经拥有相应的 GitHub 访问权限。

### 手动安装

将仓库克隆到用户级 Skill 目录，并确保最终目录名为 `personal-world-model`：

```powershell
git clone https://github.com/yangdong1017/world-model-skill "$HOME\.agents\skills\personal-world-model"
```

如果 Codex 没有立即显示新 Skill，请重启 Codex。

官方 Skill 说明：[Build skills](https://learn.chatgpt.com/docs/build-skills)

## 调用方式

技术调用标识为：

```text
$personal-world-model
```

示例：

```text
$personal-world-model 整理这个窗口，并更新我的世界模型。
```

```text
$personal-world-model 扫描尚未处理的历史对话。
```

```text
$personal-world-model 根据过去的表达，整理我对商业和人性的理解发生了哪些变化。
```

## 第一次运行

本 Skill 不预设数据库目录。

第一次正式整理内容时，它会先询问用户希望把数据库保存在哪里。在用户明确回复之前，不会创建数据库目录，也不会写入个人数据。

用户确认后，Skill 会：

1. 将位置规范化为绝对路径；
2. 把路径和配置时间写入 `SKILL.md` 的“数据库固定配置”；
3. 初始化数据库；
4. 开始整理用户指定范围内的内容。

后续运行会固定使用该位置，不会自行切换目录。

> 注意：配置后的 `SKILL.md` 会包含本机数据库绝对路径。再次分享或上传 Skill 前，应检查并移除个人路径和个人数据。

## 哪些内容会进入模型

| 内容 | 保留原文 | 进入观点模型 |
|---|---:|---:|
| 用户明确表达的判断、信念和价值取向 | 是 | 是 |
| 用户的决定、预测和工作方法 | 是 | 是 |
| 用户对外部观点的明确认同或反对 | 是 | 是 |
| 当时的情绪和试探性假设 | 是 | 视语境分类 |
| 打开文件、修改格式、询问进度等普通操作请求 | 是 | 通常否 |
| AI 助手的建议 | 仅作为上下文 | 否 |
| 他人录音、附件和引用材料 | 作为外部来源 | 不直接进入 |

## 数据库结构

数据库与 Skill 安装目录分开保存，避免更新 Skill 时覆盖个人数据。

```text
world-model-data/
|-- manifest.json
|-- state/
|   `-- threads.json
|-- sources/
|   `-- user-messages.jsonl
|-- claims/
|   `-- claims.jsonl
|-- inferences/
|   `-- inferences.jsonl
|-- tensions/
|   `-- tensions.jsonl
|-- worldview/
|-- influences/
|-- decisions/
`-- timeline.md
```

详细字段定义见 [`references/store-schema.md`](references/store-schema.md)。

## Skill 目录

```text
world-model-skill/
|-- SKILL.md
|-- README.md
|-- agents/
|   `-- openai.yaml
`-- references/
    |-- extraction-rules.md
    `-- store-schema.md
```

- [`SKILL.md`](SKILL.md)：核心行为规则和数据库固定配置
- [`references/extraction-rules.md`](references/extraction-rules.md)：观点提取、归因和更新规则
- [`references/store-schema.md`](references/store-schema.md)：数据库目录与字段结构
- [`agents/openai.yaml`](agents/openai.yaml)：Skill 界面名称、说明和调用策略

## 隐私与分享

- 仓库只包含 Skill 规则，不包含任何用户对话或世界模型数据库。
- 建议将世界模型数据库保存在受控的本地目录中。
- 不要把已配置的本机绝对路径、原始对话或个人模型直接提交到公开仓库。
- 将仓库设为公开以前，应重新检查 `SKILL.md` 的数据库固定配置和全部提交历史。

## 当前状态

- Skill 名称：世界模型
- 技术标识：`personal-world-model`
- 调用方式：`$personal-world-model`
- 数据库位置：首次使用时由用户指定
- 自动调用：已启用

## 作者

[yangdong1017](https://github.com/yangdong1017)
