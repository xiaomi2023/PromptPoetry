![MiYago LOGO.png](image/MiYago%20LOGO.png)

# PromptPoetry

<p>
中文 |
<a href="README.md">English</a>
</p>

## 介绍

快速发展的大语言模型（LLMs）在诗歌生成领域存在广泛应用，但是利用已有模型为诗歌生成任务合成训练数据可能会导致多样性退化或引入已有模型的偏见。

鉴于此，本数据集收集了真实的英文诗歌，并为每首诗合成了自然语言生成 prompt，以更好地训练和评估模型"根据指定指令生成诗歌"的能力。

每条样本都是一次完整的两轮对话：用户用自然语言描述想要的诗，助手回复对应的原文。

## 数据组成和结构

### 文件

本数据集包含两个JSON Lines 数据文件。

| 文件 | 条目数 |
| --- | --- |
| `data/poems_queries/poems_queries_multilingual.jsonl` | 9,291 |
| `data/Poetry-Foundation-Poems/Poetry-Foundation-Poems.jsonl` | 13,745 |

合计 23,036 条样本。

### 字段结构

每行是一个独立的 JSON 对象（JSON Lines 格式）：

```json
{
  "conversations": [
    {
      "role": "user",
      "content": "Write a haiku or short poem about the theme of loneliness. Use the image of childhood as a source of warmth to contrast with the present feeling of isolation."
    },
    {
      "role": "assistant",
      "content": "06 Haiku Alone\n\nloneliness\nhis childhood\nwarms him up\n..."
    }
  ]
}
```

- **`conversations`**：两轮对话，分别为 user 指令和 assistant 返回的诗歌原文。
- **`language`**：prompt 的语言标签，取值为 ISO 639-1 语言代码（如 `ja`、`ru`）。英文与中文 prompt 无此字段；混合多种外语的样本以列表形式给出。

### 数据样例

#### 英文 prompt

```json
{
  "conversations": [
    {"role": "user", "content": "Write a poem titled \"Objects Used to Prop Open a Window\"."},
    {"role": "assistant", "content": "Objects Used to Prop Open a Window\n\nDog bone, stapler,\ncribbage board, garlic press\nbecause this window is loose—lacks\nsuction, lacks grip.\n..."}
  ]
}
```

#### 非英文 prompt

```json
{
  "conversations": [
    {"role": "user", "content": "Écris un court poème anglais, cynique, sur le fait de porter trop de croix, avec peu d'espoir, et des gens qui prient un dieu auquel ils ne croient pas."},
    {"role": "assistant", "content": "All Alone With Out Hope 0\n\nSO many crosses,\nso little hope, so many\npuppets, praying for\nsome thing they dont\neven believe in...."}
  ],
  "language": "fr"
}
```

### 语言分布

`poems_queries_multilingual.jsonl` 的 prompt 语言构成：

| 语言 | 代码 | 条目数 |
| --- | --- | --- |
| 英语 | （无字段） | 7,320 |
| 中文 | （无字段） | 862 |
| 法语 | `fr` | 100 |
| 德语 | `de` | 88 |
| 俄语 | `ru` | 85 |
| 日语 | `ja` | 84 |
| 波兰语 | `pl` | 82 |
| 意大利语 | `it` | 81 |
| 葡萄牙语 | `pt` | 79 |
| 阿拉伯语 | `ar` | 77 |
| 泰语 | `th` | 76 |
| 越南语 | `vi` | 75 |
| 西班牙语 | `es` | 69 |
| 印地语 | `hi` | 69 |
| 印尼语 | `id` | 67 |
| 韩语 | `ko` | 66 |
| 其他（丹麦、保加利亚、孟加拉、挪威、克罗地亚、罗马尼亚、威尔士、索马里、希腊） | `da` `bg` `bn` `nb`/`no` `hr` `ro` `cy`/`so` `el` | 9 |
| 混合语言 prompt | `ja`+`de`、`es`+`cy` | 2 |

单一语言标签共 16 种，另有 2 条混合语言条目。英文与中文 prompt 不带 `language` 字段，故不计入上述已标注条目。

`Poetry-Foundation-Poems.jsonl` 基本全为英文，仅 21 条带非英文标签（`fr` 7、`de` 7、`es` 5、`ca` 1、`es`+`tl` 1）。

## 构建

### Poetry-Foundation-Poems.jsonl

原始数据来自 [`suayptalha/Poetry-Foundation-Poems`](https://huggingface.co/datasets/suayptalha/Poetry-Foundation-Poems/) 提供的 `PoetryFoundationData.csv`（对 Poetry Foundation 网站的抓取结果）。转换时按固定模板拼出 prompt：

```
Write a poem titled "标题" about tag1, tag2, tag3.
```

标签取该诗的前 3 个 `tag` 字段，无标签时只保留标题。`assistant` 回复为 `标题\n\n诗歌正文`，逐字取自 CSV。

### poems_queries_multilingual.jsonl

原始数据为 HuggingFace 数据集 [`AJMC2002/poems`](https://huggingface.co/datasets/AJMC2002/poems)（`poems.parquet`），外加人工撰写的自由文本写作指令文件 `1.txt`~`5.txt`。

1. **构建**：人工撰写的 query 去重后按**标题**与 `parquet` 中的原诗匹配，匹配成功者构成样本。
2. **多语种扩展**：按约 21% 的比例抽样，调用 minimax/minimax-m3 模型，把英文 query 翻译为 15 个目标语种。

### 清洗
所有数据均进行了细致的清洗，包括修复乱码、提出空正文行等。

## 使用方式

数据以 JSON Lines 格式发布，直接读取即可：

```python
import json

with open("data/poems_queries_multilingual.jsonl", encoding="utf-8") as f:
    for line in f:
        if line.strip():
            sample = json.loads(line)
            conversations = sample["conversations"]
            # conversations[0]["content"] 为 prompt
            # conversations[1]["content"] 为诗歌原文
            # sample.get("language") 为 prompt 的 ISO 639-1 语种标签（若有）
```

## 问题和局限

欢迎报告问题或提出建议。

## MiYago Datasets

[MiYago Datasets](https://huggingface.co/collections/Mikoris/miyago-datasets)是一个专为提升模型创意写作和角色扮演能力而生的数据集 Collection。

## 许可协议

本数据集采用 [Apache License 2.0](LICENSE) 许可。
