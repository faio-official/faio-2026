## Story

The **Localization team** at a large telecom company is choosing a machine translation engine for Kazakh$\leftrightarrow$Russian customer support. To pick one, they ran a blind human evaluation: for every source sentence, two competing systems produced a translation, the two outputs were shown side by side as **A** and **B**, and a human linguist decided which one was better — or that neither was.

Two linguists did all of this work. Call them **person1** and **person2**. They worked from the same guidelines on the same material, yet as the results came in the team lead noticed something uncomfortable: the two of them did *not* judge alike. The same pair of translations could earn a shrug from one and a rejection from the other. Where one linguist saw two acceptable options, the other saw two failures.

Before the company spends a year’s budget on the winning engine, the team lead wants to know how much of the verdict came from the translation and how much came from the person typing it. And there is a practical problem in the way: a migration of the evaluation platform corrupted part of the log, and for a large block of judgements **the column recording who made them was lost**.

The recovered records are in `train.csv`. The damaged ones are in `test.csv`. Your mission is to restore the missing attribution: for every judgement in the damaged block, say who made it.

## Formal Problem Statement

Every row of the dataset is **one human judgement of one pair of translations**. It contains the sentence that was translated, the reference translation, the two competing translations shown as A and B, and the verdict the linguist recorded — which side they preferred and the quality score they gave to each side. The identity of the systems that produced the two translations is not disclosed.

- `train.csv`: judgements with the `annotator` column intact (`person1` or `person2`).
- `test.csv`: judgements with the `annotator` column removed. For each row you must estimate the probability that it was made by `person2`.

The two linguists contributed an equal number of judgements, in both the training data and the test data.

The dataset can be accessed here:

- [Train dataset](https://drive.google.com/file/d/1S1PRuzxM6b2SGnTGqi4UxtjD8wElHZ3F/view?usp=drive_link)
- [Test dataset](https://drive.google.com/file/d/1hh6O-j6G5yyoxg9Ex448AqpBBKwhQ_KL/view?usp=drive_link)

Useful links:

- [Feature extraction](https://scikit-learn.org/stable/modules/feature_extraction.html)
- [Column Transformer with Mixed Types](https://scikit-learn.org/stable/auto_examples/compose/plot_column_transformer_mixed_types.html)
- [Ensembles](https://scikit-learn.org/stable/modules/ensemble.html)

## Input

`train.csv` and `test.csv` share the following columns.

- `id` — row identifier.
- `direction` — `kk->ru` or `ru->kk`.
- `original` — the source sentence.
- `reference` — a human reference translation of the source sentence.
- `translation_1` — the translation shown to the linguist as **A**.
- `translation_2` — the translation shown to the linguist as **B**.
- `preference` — the verdict. One of:
  - `a` — translation A is better;
  - `b` — translation B is better;
  - `tie` — the two are of equal quality;
  - `both_bad` — neither translation is acceptable.
- `rating_a`, `rating_b` — the quality score the linguist gave to translation A and to translation B, on a scale of `0`, `1`, `2`, where `2` is best. A small number of these values are missing.

`train.csv` has one additional column:

- `annotator` — `person1` or `person2`, the linguist who made the judgement. This is the target.

## Output

Participants must submit only a CSV file named **solution.csv**, with the following format:

```
id,probability
0,0.83
1,0.12
2,0.50
...
```

`probability` is your estimated probability, a real number in the interval $[0, 1]$, that the judgement in that row was made by **person2**. A value close to $1$ means you are confident it was `person2`; a value close to $0$ means you are confident it was `person1`.

**Important:** Submit only the `solution.csv` file. Source code is not required. The file must contain exactly one row for every row of `test.csv`, and the order of `id` values in `solution.csv` must strictly follow the order in `test.csv`, starting from `0` up to the highest ID.

## Evaluation

- Your predictions will be evaluated using **ROC-AUC** (the area under the receiver operating characteristic curve), computed with `person2` as the positive class.
- ROC-AUC measures how well your probabilities *rank* the rows, not whether they cross any particular threshold. Submitting only the values $0$ and $1$ is allowed but discards all information about your confidence.
- A submission that assigns the same value to every row scores an AUC of exactly $0.5$.
- The final competition score is the ROC-AUC placed on a $0$–$100$ scale, rounded to two decimal places: $$\text{score} = \text{AUC} \times 100$$
- A submission that cannot be read is rejected as a formatting error and receives no score. This happens when the number of rows does not match `test.csv`, when the `id` values are not in the same order as `test.csv`, when the `probability` column is missing or misnamed, or when any value is empty, non-numeric, or outside $[0, 1]$.

## Example

**Input (`train.csv`), one row, shown column by column:**

```
id           : 0
direction    : ru->kk
original     : Спокойной ночи, до завтра.
reference    : Қайырлы түн, ертеңге дейін.
translation_1: Қайырлы түн, ертеңкеге дейін.
translation_2: Қайырлы түн, ертеңге дейін.
preference   : b
rating_a     : 0
rating_b     : 2
annotator    : person1
```

**Input (`test.csv`):** the same columns, without `annotator`.

**Your Output (`solution.csv`):**

```
id,probability
0,0.83
1,0.12
2,0.50
```
