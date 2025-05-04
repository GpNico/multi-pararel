# Multi‑ParaRel

**Multi‑ParaRel** is an open‑source, multilingual prompt‑and‑triplet dataset released alongside the paper [**“Qualifying Knowledge and Knowledge Sharing in Multilingual Models”** (TMLR 2025)](https://openreview.net/forum?id=hnpB3SRbZj).
It extends the original **ParaRel** English‑only prompt set to **10 languages** and pairs every prompt with Wikidata‑based subject–relation–object (S‑R‑O) triplets from **mLAMA**.
The goal is to let you probe, fine‑tune, or evaluate multilingual language models on factual knowledge with thousands of natural‑language paraphrases per relation.

---

## 1 Quick overview

| Key fact                | Value                                                                                  |
| ----------------------- | -------------------------------------------------------------------------------------- |
| **Languages**           | Catalan, Danish, Dutch, English, French, German, Italian, Portuguese, Spanish, Swedish |
| **Relations**           | 38 Wikidata properties (e.g. *capital of*, *occupation*, *located in*)                 |
| **Avg. prompts / rel.** | 9 – 19 (language‑dependent)                                                            |
| **Total prompts**       | ≈ 6 400                                                                                |
| **Triplets / split**    | Up to 10 k per language · **train / dev**                                              |
| **Source corpora**      | ParaRel prompts + mLAMA triplets                                                       |
| **Translation model**   | Meta **Seamless M4T‑Large** (`facebook/seamless-m4t-large`)                            |

---

## 2 Dataset in depth

### 2.1 Creation pipeline

1. **Template instantiation**
     • Take an English ParaRel template, e.g. *“The capital of \[X] is \[Y]”.*
     • Fill **\[X]** and **\[Y]** with an English mLAMA triplet (*Great Britain → London*).

2. **Machine translation**
   Translate the fully instantiated sentence with Seamless M4T into the target language.

3. **Placeholder recovery**
   Align with the *target‑language* version of the same triplet and swap entities back to **\[X]**, **\[Y]**.

4. **Quality voting**
   For each English template, sample **30** triplets → 30 translations.
   Keep the highest‑voted candidate that is (i) autoregressive, (ii) unique, (iii) in the top‑5.

5. **Manual spot‑check**
   Native French & Spanish speakers label 300 random prompts: **FR 88 % / ES 78 %** fluent; remainder weird or ungrammatical.

### 2.2 Statistics

| Language   | Avg. prompts / relation |
| ---------- | ----------------------- |
| Catalan    | 19                      |
| Danish     | 15                      |
| Dutch      | 17                      |
| English    | 19                      |
| French     | 14                      |
| German     | 9                       |
| Italian    | 19                      |
| Portuguese | 19                      |
| Spanish    | 19                      |
| Swedish    | 16                      |



---

## 3 Repository layout & how to use

```
multi-pararel/
├── prompts/
│   ├── en/
│   │   ├── P17.jsonl
│   │   └── …
│   └── fr/
│       └── P17.jsonl
└── triplets/
    ├── en/
    │   └── P17/
    │       ├── train.jsonl
    │       └── dev.jsonl
    └── fr/
        └── P17/
            ├── train.jsonl
            └── dev.jsonl
```

### 3.1 Prompt files

*Path:* `prompts/<lang>/<relation>.jsonl`
Each line is a distinct template using placeholders **\[X]** and **\[Y]**:

```json
{"pattern": "The country [X] is located in is [Y]."}
{"pattern": "Is [X] located in [Y]?"}
```

### 3.2 Triplet files

*Path:* `triplets/<lang>/<relation>/{train,dev}.jsonl`
Each line is a Wikidata fact aligned with the same relation—ready to be slotted into any prompt.

```json
{
  "sub_uri":"Q3300448",
  "obj_uri":"Q96",
  "sub_label":"Ozumba",
  "obj_label":"Mexico",
  "index":509,
  "lineid":510,
  "uuid":"5a2d6d86-4086-4d6f-b64c-e9fe3afdd355"
}
```

### 3.3 Loading in Python

```python
import json, pathlib
root = pathlib.Path('multi-pararel')

# English prompts for relation P17 (country/location)
prompts = [json.loads(l)['pattern']
           for l in open(root / 'prompts/en/P17.jsonl')]

# Corresponding training triples
triples = [json.loads(l)
           for l in open(root / 'triplets/en/P17/train.jsonl')]

print(prompts[0])
print(triples[0]['sub_label'], '→', triples[0]['obj_label'])
```
