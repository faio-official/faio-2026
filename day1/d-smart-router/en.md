## Statement

Adilkhan and Darkhan are building a personal data protection system. Before documents are
processed, the system masks names, phone numbers, addresses and other personal data using a set of
rules and a named entity recognition (NER) model. Sometimes the system misses personal data.
A large language model could double-check the documents, but the budget covers only
**10 % of them**.

Build a **router**: for every document output a **risk score** — a single finite number, the
larger the higher the priority for the extra check. The documents with the highest scores are sent
for review. The more documents with misses land in the selected 10 %, the better; pay special
attention to **critical** personal data.

You do not need to fix the masking — only to set priorities.

### Categories

| Class | What it covers |
|---|---|
| `USER` | first name, last name, username |
| `ADDR` | country, city, region, county, street address, postal code, coordinates |
| `MAIL` | e-mail address |
| `TEL` | phone number, fax number |
| `IBAN` | account number, IBAN, SWIFT/BIC, bank routing number |
| `DOC` | licence, certificate, medical record or insurance policy number |
| `CARD` | payment card number, CVV |
| `IIN` | IIN, other national identifier, tax number |

Closely related data types are merged into one class for simplicity. **Critical classes:** `IIN`,
`CARD`, `TEL`, `MAIL`, `IBAN`. `has_critical_leak = 1` implies `has_leak = 1`.

A **miss** is an entity that is not fully covered by masks (it remained in the text completely or
in part).

### Data

Texts in Russian, Kazakh, English and with mixed languages, in roughly equal shares.

All files come in one archive: [download (907.1 MB)](https://contest.faio.kz/problems/01a17404-65fa-734a-bddd-c6e3b53845bb/public/task-d.zip)

```bash
wget https://contest.faio.kz/problems/01a17404-65fa-734a-bddd-c6e3b53845bb/public/task-d.zip
```

| file | documents | contents |
|---|---|---|
| `train.jsonl` | 25 002 | input, gold `pii` annotation, both labels | |
| `test.jsonl` | 15 001 | input only | |
| `sample_submission.csv` | — | example submission | |
| `metric.py` | — | metric script for self-evaluation | |
| GLiNER | — | base `gliner_multi-v2.1` model with its tokenizer | |
`train.jsonl` contains 2 079 documents with misses (8.3 %), of which 837 have critical ones
(3.3 %). Split your own validation set out of `train.jsonl`.

You may train your solution on this data and on the bundled GLiNER model — the same model
that served as the NER layer of the masking system, but in its base version. Instructions for
running it without internet access are inside the model archive.

---

## Input format

JSON Lines, UTF-8, one object per line.

| Field | Description |
|---|---|
| `document_id` | document identifier |
| `lang` | `ru`, `kk`, `en` or `mixed` |
| `text` | original text |
| `masked_text` | text after masking: every entity found is replaced with `[CLASS]` |
| `cascade_entities` | **what the masking system found**: an array of objects with `start`, `end` (character offsets in `text`, half-open interval), `type` (class), `source` (`"rule"` or `"ner"`), `confidence` (NER confidence, or `null` for rules) |
| `pii` | **gold annotation** with `start`, `end`, `label` — only in `train.jsonl` |
| `has_leak`, `has_critical_leak` | 0/1 labels — only in `train.jsonl` |

**Do not confuse the two arrays:** `cascade_entities` holds the system's predictions, and it is
exactly their errors that you must predict; `pii` is the gold annotation and is absent from the
test set.

---

## Output format

`predictions.csv` in UTF-8, with a header:

```
document_id,risk_score
39703d9ea01c9e58,0.871
1a2b3c4d5e6f7890,0.014
```

Exactly one row per test document, no duplicate or unknown identifiers. `risk_score` must be a
finite number (decimal point as the separator); `NaN` and `Inf` are not allowed. The score need not
be a probability — only the ordering matters. Row order is arbitrary.

A file that does not match the format scores **0 points**.

---

## Examples

Two lines of the input file (texts shortened):

```
{"document_id": "a1", "lang": "ru", "text": "Клиент Айгерим Сапарова, тел. +7 701 000 00 00.", "masked_text": "Клиент [USER], тел. [TEL].", "cascade_entities": [{"start": 7, "end": 23, "type": "USER", "source": "ner", "confidence": 0.97}, {"start": 30, "end": 46, "type": "TEL", "source": "rule", "confidence": null}]}
{"document_id": "a2", "lang": "kk", "text": "Байланыс: aigerim [at] mail [dot] kz, ЖСН 000000000000.", "masked_text": "Байланыс: aigerim [at] mail [dot] kz, ЖСН [IIN].", "cascade_entities": [{"start": 42, "end": 54, "type": "IIN", "source": "rule", "confidence": null}]}
```

Submission:

```
document_id,risk_score
a1,0.05
a2,0.93
```

In `a2` the e-mail address is spelled out in words (`[at]`, `[dot]`), the rule did not recognise it
and it stayed in the masked text — that is a critical miss. In `a1` everything is masked.
The example illustrates the format only; the particular scores in it are not checked.

---

## Notes

**Document selection.** With N documents in the test set, `K = floor(0.1 × N)` documents with the
highest scores are sent for review; for N = 15 001 that is K = 1 500. Ties are broken by
`document_id` in ascending order as strings.

**Public / private.** The score shown during the round is computed on 30 % of the test set; the
final score is computed on the remaining 70 % and announced after the round. The two may differ
noticeably.

---

## Scoring

Let **A** be the test documents with `has_leak = 1`, **C** those with `has_critical_leak = 1`, and
**S** the top K documents selected by the router. `|X|` denotes the number of documents and
`∩` the intersection.

```
R_all      = |S ∩ A| / |A|
R_critical = |S ∩ C| / |C|
score      = 100 × (0.5 × R_all + 0.5 × R_critical)
```

Both quantities are computed on the same set S — there is no separate budget for critical misses.
A document counts once in each quantity regardless of how many entities were missed in it.
The maximum score is 100.

**Example.** The test set has 10 000 documents and 1 000 go for review. There are 200 documents
with misses in total, 40 of them critical. Among the selected ones there are 140 with misses and
20 with critical misses:

```
R_all      = 140 / 200 = 0.70
R_critical =  20 /  40 = 0.50
score      = 100 × (0.5 × 0.70 + 0.5 × 0.50) = 60
```

**Submission limit.** For each problem, you can submit at most once per minute.