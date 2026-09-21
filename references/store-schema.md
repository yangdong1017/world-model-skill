# 世界模型数据库结构

创建、迁移、校验或直接编辑“世界模型”数据库时，读取本参考文件。

## 首次初始化前提

数据库目录没有默认值。首先读取 `SKILL.md` 中的“数据库固定配置”。只有同时满足以下条件，才能创建本页所列目录和文件：

- 配置状态已经是“已配置”；
- 数据库绝对路径已经写入 `SKILL.md`；
- 用户明确选择了该位置；
- 该位置可访问并且可以安全写入。

如果仍然显示“未配置”，必须先询问用户，并在收到答复后把规范化的绝对路径写回 `SKILL.md`。不得先创建临时数据库，也不得使用工作区、用户目录或 Codex 目录作为隐含默认值。

## 目录结构

```text
personal-world-model/
|-- manifest.json                 数据库清单
|-- state/
|   `-- threads.json              对话处理进度
|-- sources/
|   `-- user-messages.jsonl       用户原始消息
|-- claims/
|   `-- claims.jsonl              归纳后的原子观点
|-- inferences/
|   `-- inferences.jsonl          模型推断
|-- tensions/
|   `-- tensions.jsonl            观点之间的张力或表面矛盾
|-- worldview/
|   |-- index.md                  世界观主题索引
|   `-- <topic-slug>.md           单个主题总结
|-- influences/
|   `-- <person-or-source-slug>.md 外部人物或来源的影响
|-- decisions/
|   `-- decisions.jsonl           重要决策及后续结果
`-- timeline.md                   思想变化时间线
```

只创建实际需要的文件，不必预先建立空的可选目录。

所有文件使用 UTF-8 编码。JSONL 文件每一行保存一个完整的 JSON 对象。新增的原始来源记录只能追加，不能覆盖；更新衍生记录时，也必须保留被取代版本的历史信息。

## 数据库清单 `manifest.json`

```json
{
  "schema_version": 1,
  "created_at": "ISO-8601 时间",
  "updated_at": "ISO-8601 时间",
  "timezone": "IANA 时区名称",
  "owner_label": "user",
  "store_purpose": "对用户真实表达进行可追溯的长期建模"
}
```

清单文件只描述数据库本身，不要在其中保存人格结论或用户观点。

## 原始消息记录

`sources/user-messages.jsonl` 的必需字段：

```json
{
  "record_type": "source_message",
  "id": "src_<稳定编号>",
  "source_type": "codex_task",
  "task_id": "能够取得时保存稳定的任务编号",
  "task_title": "任务标题，只作为来源信息，不作为观点证据",
  "timestamp": "能够取得时保存 ISO-8601 时间",
  "captured_at": "采集时的 ISO-8601 时间",
  "user_text": "用户原话，敏感凭据必须脱敏",
  "content_hash": "由任务范围和规范化原文生成的稳定内容指纹",
  "context_note": "理解指代所需的最少中立背景",
  "attachments": []
}
```

导入语音转写时，应额外记录说话人身份和作者归属。其他人的录音应进入外部来源或 `influences/`，不能写入 `user-messages.jsonl` 冒充用户原话。

## 原子观点记录

`claims/claims.jsonl` 的必需字段：

```json
{
  "record_type": "claim",
  "id": "claim_<稳定编号>",
  "normalized_claim": "一条能够独立判断的原子观点",
  "kind": "explicit_belief",
  "stance": "endorsed",
  "stability": "current",
  "confidence": 0.9,
  "topics": ["主题"],
  "scope": "该观点适用的条件或范围",
  "source_ids": ["src_..."],
  "verbatim_excerpts": ["用户准确原话"],
  "first_seen_at": "首次出现的 ISO-8601 时间",
  "last_seen_at": "最近出现的 ISO-8601 时间",
  "related_claim_ids": [],
  "tension_ids": [],
  "supersedes": [],
  "status": "active"
}
```

`kind`（观点类型）允许使用：

- `explicit_belief`：明确相信的观点
- `tentative_hypothesis`：试探性假设
- `value_preference`：价值或偏好
- `observation`：观察和判断
- `emotion_state`：当时的情绪状态
- `decision`：已经作出或准备作出的决定
- `prediction`：对未来的预测
- `question`：尚未解决的问题
- `endorsement`：对外部观点的认同
- `rejection`：对外部观点的反对
- `self_narrative`：对自己的叙述
- `working_method`：工作或思考方法

`stance`（立场）允许使用：

- `endorsed`：认同
- `tentative`：暂时倾向或仍在探索
- `rejected`：反对
- `ambivalent`：同时存在支持与保留
- `unclear`：目前无法确认

`stability`（稳定程度）允许使用：

- `momentary`：当时状态
- `contextual`：只在特定情境成立
- `recurring`：在多个独立情境反复出现
- `current`：目前明确持有
- `superseded`：已被后来的观点取代
- `unknown`：证据不足

`confidence` 表示“提取和归因是否可靠”，不表示该观点在客观世界中是否正确。同一场对话中反复说同一句话，不等于获得了多份独立证据。

## 模型推断记录

`inferences/inferences.jsonl` 中的内容永远不能与直接观点混为一谈：

```json
{
  "record_type": "inference",
  "id": "inf_<稳定编号>",
  "hypothesis": "使用克制措辞描述的模型推断",
  "confidence": 0.45,
  "evidence_for": ["claim_..."],
  "evidence_against": [],
  "conditions": "这一模式出现的条件",
  "what_would_change_it": "哪些未来证据会使该推断发生变化",
  "status": "active",
  "created_at": "ISO-8601 时间",
  "updated_at": "ISO-8601 时间"
}
```

## 张力记录

当不同语境可能解释两种立场时，用“张力”记录，不要直接宣布用户自相矛盾：

```json
{
  "record_type": "tension",
  "id": "tension_<稳定编号>",
  "claim_ids": ["claim_a", "claim_b"],
  "description": "对表面冲突的中立描述",
  "possible_contextual_resolution": "可选的情境解释，必须标明只是推断",
  "status": "open",
  "created_at": "ISO-8601 时间"
}
```

## 决策记录

当用户作出重要选择，或者回顾过去的重要选择时使用：

```json
{
  "record_type": "decision",
  "id": "decision_<稳定编号>",
  "decision": "已经或准备作出的选择",
  "made_at": "ISO-8601 时间",
  "source_ids": ["src_..."],
  "stated_reasons": [],
  "assumptions": [],
  "predicted_outcomes": [],
  "reversal_conditions": [],
  "observed_outcomes": [],
  "review_status": "unreviewed"
}
```

不得编造用户没有说过的理由、假设、退出条件或结果。

## 对话处理状态

`state/threads.json` 是一个以任务编号为键的 JSON 对象。每个任务可以记录：

```json
{
  "last_processed_turn_id": "能够取得时保存回合编号",
  "last_processed_timestamp": "最后处理到的 ISO-8601 时间",
  "last_content_hash": "最后一条内容指纹",
  "processed_at": "本次处理时间",
  "status": "complete"
}
```

只有当本批次所有相关记录都成功保存以后，才能推进处理状态。

## 世界观主题总结

Markdown 总结只是根据原始数据生成的阅读视图，不是事实来源。每个实质性判断都要引用观点编号，例如：

```markdown
用户目前认为，在作出不可逆决定时，保留选择权比追求短期效率更重要。[claim_abc123]
```

每份主题总结应明确分开：

- 当前明确立场；
- 试探性观点和尚未解决的问题；
- 历史变化；
- 张力和反面证据；
- 明确标注的模型推断。

只重建受到新增或修订记录影响的主题总结。
