# Keyboard Activities

<div align="center">
    <img src="https://contest.faio.kz/problems/01a173fe-d24c-725c-8008-09a77c82c58a/public/task-b.png" alt="hid-dim-v3" border="0" width="700">
</div>

## 1. Story

Two people sit in the same office. One is writing a program, while the other is writing an article. Their screens are hidden, and all you can hear is the sound of their keyboards.

Could you tell who is doing what?

In this challenge, you'll explore whether keyboard sounds contain enough clues to distinguish coding from ordinary writing.

## 2. Formal Problem Statement

Every row of the training dataset `train_labels.csv` corresponds to **one recording of a person working with a keyboard** of around 20 seconds. It contains the identifier of the audio file, which is stored in the `train/` directory, and the true label, either `coding` or `writing`.

For every audio file in the `test/` directory, you must predict its keyboard activity and label it `coding` or `writing`.

The two activities constitute an **unequal** number of audio files in both the training data and the test data.

The dataset and a sample submission can be accessed here:

[Download task-b.zip (1.33 GB)](https://contest.faio.kz/problems/01a173fe-d24c-725c-8008-09a77c82c58a/public/task-b.zip)

```bash
wget https://contest.faio.kz/problems/01a173fe-d24c-725c-8008-09a77c82c58a/public/task-b.zip
```

## 3. Input

`train_labels.csv` has the following columns.

- `id` — audio identifier.
- `target` — `coding` or `writing`, the correct audio label for the audio file.

There is **no** `test.csv`.

## 4. Output

Participants must submit only a CSV file named **solution.csv**, with the following format:

```
id,target
0,writing
1,coding
2,writing
...
```

`target` is your predicted label, either `coding` or `writing`, depending on what your model thinks the recorded person is doing on the keyboard in the corresponding audio file.

**Important:** Submit only the `solution.csv` file. Source code is not required. The file must contain exactly one row for every audio file in `test/`, and the order of `id` values in `solution.csv` must follow the order from `0` up to the highest ID.

## 5. Evaluation

- Your predictions will be evaluated using **Macro F1 Score.**
- Macro F1 Score measures how well your model balances precision and recall across classes, giving each class equal weight. A model can perform well overall while performing poorly on minority classes, which Macro F1 makes visible.
- A submission that assigns the same value to every row scores a Macro F1 of **less than** $0.5$.
- The final competition score is the Macro F1 placed on a $0$–$100$ scale, rounded to two decimal places:

  $$
  \text{score} = \text{Macro F1} \times 100
  $$

- A submission that cannot be read is rejected as a formatting error and receives no score. This happens when the number of rows does not match the number of files in the `test` folder, when the `target` column is missing or misnamed, or when any `target` value is empty or not among `coding` or `writing`.
- The score you see during the competition is evaluated on $50\%$ of the test set. The final result is evaluated on the remaining $50\%$ and will be announced after the end of the competition.

## 6. Example

**Input (`train_labels.csv`):**

```
id,target
0,coding
1,coding
2,coding
3,writing
...
```

**Your Output (`solution.csv`):**

```
id,target
0,writing
1,coding
2,writing
...
```
**Submission limit.** For each problem, you can submit at most once per minute.