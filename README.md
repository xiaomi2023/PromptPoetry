![MiYago LOGO.png](image/MiYago%20LOGO.png)

# PromptPoetry

<p>
<a href="README_CN.md">中文</a> |
English
</p>

## Introduction

Large language models (LLMs) are widely used for poetry generation, but synthesizing training data for that task from existing models can lead to reduced diversity or introduce the biases already present in those models.

To address this, this dataset collects real English-language poems and pairs each one with a natural-language generation prompt, so as to better train and evaluate models on the ability to "generate a poem from a given instruction."

Every sample is a two-turn conversation: the user describes the desired poem in natural language, and the assistant replies with the corresponding original poem.

## Dataset Composition and Structure

### Files

The dataset contains two JSON Lines data files.

| File | Entries |
| --- | --- |
| `data/poems_queries/poems_queries_multilingual.jsonl` | 9,291 |
| `data/Poetry-Foundation-Poems/Poetry-Foundation-Poems.jsonl` | 13,745 |

23,036 samples in total.

### Field Structure

Each line is a standalone JSON object (JSON Lines format):

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

- **`conversations`** — the two dialogue turns, being the user instruction and the original poem returned by the assistant.
- **`language`** — the language tag of the prompt, given as an ISO 639-1 language code (e.g. `ja`, `ru`). English and Chinese prompts have no such field; a sample mixing several foreign languages carries a list.

### Data Samples

#### English prompt

```json
{
  "conversations": [
    {"role": "user", "content": "Write a poem titled \"Objects Used to Prop Open a Window\"."},
    {"role": "assistant", "content": "Objects Used to Prop Open a Window\n\nDog bone, stapler,\ncribbage board, garlic press\nbecause this window is loose—lacks\nsuction, lacks grip.\n..."}
  ]
}
```

#### Non-English prompt

```json
{
  "conversations": [
    {"role": "user", "content": "Écris un court poème anglais, cynique, sur le fait de porter trop de croix, avec peu d'espoir, et des gens qui prient un dieu auquel ils ne croient pas."},
    {"role": "assistant", "content": "All Alone With Out Hope 0\n\nSO many crosses,\nso little hope, so many\npuppets, praying for\nsome thing they dont\neven believe in...."}
  ],
  "language": "fr"
}
```

### Language Distribution

The prompt language composition of `poems_queries_multilingual.jsonl`:

| Language | Code | Entries |
| --- | --- | --- |
| English | (no field) | 7,320 |
| Chinese | (no field) | 862 |
| French | `fr` | 100 |
| German | `de` | 88 |
| Russian | `ru` | 85 |
| Japanese | `ja` | 84 |
| Polish | `pl` | 82 |
| Italian | `it` | 81 |
| Portuguese | `pt` | 79 |
| Arabic | `ar` | 77 |
| Thai | `th` | 76 |
| Vietnamese | `vi` | 75 |
| Spanish | `es` | 69 |
| Hindi | `hi` | 69 |
| Indonesian | `id` | 67 |
| Korean | `ko` | 66 |
| Others (Danish, Bulgarian, Bengali, Norwegian, Croatian, Romanian, Welsh, Somali, Greek) | `da` `bg` `bn` `nb`/`no` `hr` `ro` `cy`/`so` `el` | 9 |
| Mixed-language prompts | `ja`+`de`, `es`+`cy` | 2 |

There are 16 single-language tags, plus 2 mixed-language entries. English and Chinese prompts carry no `language` field, so they are not counted among the tagged entries.

`Poetry-Foundation-Poems.jsonl` is almost entirely English; only 21 entries carry a non-English tag (`fr` 7, `de` 7, `es` 5, `ca` 1, `es`+`tl` 1).

## Construction

### Poetry-Foundation-Poems.jsonl

The source data comes from the `PoetryFoundationData.csv` provided by [`suayptalha/Poetry-Foundation-Poems`](https://huggingface.co/datasets/suayptalha/Poetry-Foundation-Poems/) (a scrape of the Poetry Foundation website). During conversion the prompt is assembled from a fixed template:

```
Write a poem titled "TITLE" about tag1, tag2, tag3.
```

The tags are the poem's first 3 `tag` fields; a poem without tags keeps the title only. The `assistant` reply is formatted as `TITLE\n\nPOEM`, taken verbatim from the CSV.

### poems_queries_multilingual.jsonl

The original poems are first scraped from the dataset [AJMC2002/poems](https://huggingface.co/datasets/AJMC2002/poems), then the model minimax-m3 is called to generate diverse and multilingual user prompts, in order to strengthen the model's ability to follow multilingual instructions.

### Cleaning

All the data has been carefully cleaned, including repairing mojibake, removing empty-body lines, etc.

## Usage

The data is released in JSON Lines format and can be read directly:

```python
import json

with open("data/poems_queries_multilingual.jsonl", encoding="utf-8") as f:
    for line in f:
        if line.strip():
            sample = json.loads(line)
            conversations = sample["conversations"]
            # conversations[0]["content"] is the prompt
            # conversations[1]["content"] is the original poem
            # sample.get("language") is the ISO 639-1 prompt tag, if any
```

## Issues and Limitations

Please report any issues or suggestions.

## MiYago Datasets

[MiYago Datasets](https://huggingface.co/collections/Mikoris/miyago-datasets) is a dataset collection built specifically to improve the creative writing and roleplay capabilities of models.

## License

This dataset is licensed under the [Apache License 2.0](LICENSE).
