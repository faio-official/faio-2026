**Inverse text normalization** (ITN) is the task of converting text from spoken form into written form. A speech recognizer produces the spoken form, while readers and every downstream system need the written one.

ITN deals with *semiotic classes* — numbers, dates, times, currency amounts, abbreviations, addresses and the like, whose written form differs from the spoken one:

```
двенадцать сорок пять           ->  12:45
он бес мың                      ->  15 000
айгуль собачка гмайл точка кз   ->  aigul@gmail.kz
кока кола                       ->  Coca-Cola
```

Before anything can be converted, you must decide **which words belong to a semiotic class at all, and to which one**. Miss a span and the transcript stays unreadable. Mark a spurious one and you get a transcript that looks right and is wrong: a system that turns an ordinary word into a number or a name invents a fact, and nobody reading the transcript afterwards can tell.

In this problem you do that first step, and only that step.

You are given a sentence as a sequence of tokens. Assign every token a label in **BIO** form:

- `O` — the token belongs to no span;
- `B-<CLASS>` — the token begins a span of class `CLASS`;
- `I-<CLASS>` — the token continues the span begun by the previous token.

A *span* is a maximal group of consecutive tokens that a person would write down as one entry of one class.

There are eight classes:

| class | what it is | what a writer would produce |
| --- | --- | --- |
| `CARDINAL` | a plain count | `15 000` |
| `ORDINAL` | a position in a sequence | `17th` |
| `DECIMAL` | a fractional number | `1.5` |
| `DATE` | a day of a month, or a year | `23rd` |
| `TIME` | a clock time | `5:30` |
| `MEASURE` | an amount with a unit — money and percentages included | `3 l`, `15 000 tenge`, `40%` |
| `EMAIL` | a dictated e-mail address | `aigul@gmail.kz` |
| `WHITELIST` | a name, technical term or abbreviation | `Coca-Cola`, `attention`, `СМС` |

The written forms are shown so you understand what each class *is*; you never produce them.

The label is decided by what surrounds the word, not by the word itself. Every example below contains the same word — and in the first one it is not a number at all:

```
адалдық ақыры бір көрінеді деп айтты
O       O     O   O        O   O
```

```
говорил двадцать   три        раза
O       B-CARDINAL I-CARDINAL O
```

```
встречаемся в три    тридцать
O           O B-TIME I-TIME
```

```
выйдем через три       минут
O      O     B-MEASURE I-MEASURE
```

```
мы используем ллама       три         в проде
O  O          B-WHITELIST I-WHITELIST O O
```

```
жүз       жиырма    келілік   қап
B-MEASURE I-MEASURE I-MEASURE O
```

A unit or currency word belongs to the span; a counted object does not. Names stay outside.

## Input format

Both the training and the test set contain three kinds of sentences:

- entirely in Kazakh;
- entirely in Russian;
- mixed, with Kazakh and Russian words inside one sentence.

Tokens are already split, sentences are independent. The input contains **no digits, no punctuation and no capital letters** — all of that is what a writer would add. Only **Cyrillic** is used: there are no Latin letters or other scripts, and foreign names are written the way they are pronounced.

Three files are given:

- [train.csv](https://drive.google.com/file/d/1jyYB7SnchA_gtAFUVaLBI2lP5v9A3zvm/view?usp=drive_link) — columns `sent_id`, `token_id`, `token`, `label`; 738 843 sentences;
- [test.csv](https://drive.google.com/file/d/13Rcuy7onRSeL6QdJg-wFNy8rG8ZLLaIa/view?usp=drive_link) — columns `sent_id`, `token_id`, `token`; 146 797 sentences;
- [sample_submission.csv](https://drive.google.com/file/d/1Ze5euv6nVH2Xd6rB4u3sMOLftQw-4rKF/view?usp=drive_link) — a sample solution, every label `O`.

Columns:

- `sent_id` — sentence identifier;
- `token_id` — position of the token inside its sentence, starting at `0`;
- `token` — the token itself;
- `label` — `O`, `B-<CLASS>` or `I-<CLASS>` with one of the eight classes. The target variable.

## Output format

Submit a single CSV file named `solution.csv`: a header and exactly one row per token of `test.csv`, in any order.

```
sent_id,token_id,label
t089845,0,O
t089845,1,O
t089845,2,O
```

An `I-` label that does not continue a span of the same class is read as the start of a new span.

A solution is **rejected as a format error** and earns no score for the attempt if a row for some token of `test.csv` is missing or appears twice, if the `label` column is missing or named differently, or if a `label` value is outside the allowed set.

## Scoring

Solutions are scored by **span-level F1 with exact boundaries**. A span counts as correct only if the answer key contains a span with the same first token, the same last token and the same class. Partial overlap earns nothing.

Let `TP` be the number of correctly found spans, `FP` the number of spans in your solution that are absent from the key, and `FN` the number of spans in the key that are absent from your solution. Then

```
precision = TP / (TP + FP)
recall    = TP / (TP + FN)
F1        = 2 · precision · recall / (precision + recall)
```

If `TP = 0`, then `F1 = 0`.

The score of an attempt is `100 · F1`, rounded to two decimal places. A solution of all `O` scores `0.00`; a solution matching the key scores `100.00`.
