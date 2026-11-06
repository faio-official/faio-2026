# Who Wrote This? Adilkhan's AI Detective Lab

<div align="center">
    <img src="https://contest.faio.kz/problems/01a173cc-3954-7414-b408-9e67fc86af22/public/poster.png" alt="hid-dim-v3" border="0" width="500">
</div>

Adilkhan has just become a professor at a newly opened AI university. Now he spends his evenings reading homework, wondering why half the class has suddenly developed a fondness for numbered lists.

His students are allowed to use AI assistants, and they certainly do. They argue about which one writes better code, which one explains mathematics properly, and which one can be trusted with a question about Kazakhstan. Adilkhan starts to wonder whether he could recognize these assistants from their answers.

One afternoon, he brings the idea to his PhD student, Darkhan.

"Suppose I give you a question and an AI's answer. Could you tell me which model wrote it?"

"You could just ask the students," Darkhan says.

"I could. But I was hoping you'd build something."

Darkhan agrees to try. He asks ten models the same set of questions and saves their answers, replacing the model names with codenames. At first, he thinks he can tell them apart by eye. A few pages later, he is much less sure.

He now has a labeled collection to learn from and 500 more answers to identify. His next meeting with Adilkhan is approaching. Help him build a classifier that can do better than his guesses.

## Your task

Given an English prompt and an AI-generated response, predict which of ten anonymous models produced that response. This is a multiclass classification problem. You submit predictions, not training code.

The possible labels are `Aster`, `Birch`, `Cedar`, `Dune`, `Ember`, `Frost`, `Grove`, `Harbor`, `Iris`, and `Juniper`. Codenames are arbitrary class labels; their spelling does not describe a model's identity or abilities.

## Data

- `train.csv`: 1,000 labeled examples with columns `prompt`, `response`, and `model`. There are 100 examples per model. The training set contains 100 distinct prompts, each answered by all ten models.
- `test.csv`: 500 unlabeled examples with columns `prompt` and `response`. You receive the entire test file from the start.
- `sample_submission.csv`: an example of the required submission structure. Its predictions are placeholders.

The files use UTF-8 CSV. Text can contain commas, quotation marks, and embedded newlines, so use a CSV parser rather than splitting lines manually. 

Responses have been cleaned to remove common Markdown formatting and collapse whitespace. For answers longer than 100 words, only a centered, consecutive 100-word excerpt is provided; shorter answers are kept in full. Words are counted by whitespace. An excerpt may begin or end in the middle of a sentence. The same preprocessing is applied to both training and test responses; the original prompts are unchanged.

## Submission format

Upload a UTF-8 CSV file containing exactly one column named `model` and exactly 500 prediction rows after the header. Use the codenames exactly as written, including capitalization.

Prediction row 1 must correspond to data row 1 of `test.csv`, prediction row 2 to data row 2, and so on. Preserve the original test-row order. Do not add an index, prompt ID, category, probability, or any other column.

For example, the start of a submission could be:

```csv
model
Aster
Cedar
Juniper
```

This illustration shows only three predictions; your submitted file must contain all 500. Invalid labels, missing or extra rows, and incorrect columns are rejected.

## Scoring and leaderboards

Your score is classification accuracy multiplied by 100:

`score = 100 × correctly predicted rows / scored rows`.

During the contest, the public leaderboard uses a 40% of the test, while other 60% reserved for private test.

After the contest, final evaluation uses the remaining 60% private subset. The final score is the private score alone, NOT an average of the public and private scores. The membership of both subsets and all test labels remain hidden during the contest.

Every submission must predict all 500 test rows. A strong public score is useful feedback, but Darkhan needs a classifier that generalizes beyond the examples used for that feedback.

[Download data.zip (396.4 KB)](https://contest.faio.kz/problems/01a173cc-3954-7414-b408-9e67fc86af22/public/data.zip)

```bash
wget https://contest.faio.kz/problems/01a173cc-3954-7414-b408-9e67fc86af22/public/data.zip
```

**Submission limit.** For each problem, you can submit at most once per minute.