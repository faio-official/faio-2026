# Lecture Shuffle

<div align="center">
    <img src="https://contest.faio.kz/problems/01a173de-b790-7144-991c-d7cd96dc7b92/public/cover.png" alt="hid-dim-v3" border="0" width="900">
</div>

## 1. Story

An online university is turning its recorded lectures into short video lessons. Every lecture was cut into 30-second clips, and a few seconds between neighbouring clips were trimmed away so that each clip starts cleanly. During a move to a new video platform, the index that stored the **order** of the clips was lost. The clips of each lesson are still grouped together, but nobody knows any more which clip comes first.

The platform team asks you to restore the order automatically. For some lessons the order survived: these are in `train`. For the others it was lost: these are in `test`.

## 2. Formal Problem Statement

The data consists of **puzzles**. A puzzle is a set of **6 to 12 video clips** cut from one continuous stretch of one lecture:

- every clip is exactly **30 seconds** long, with picture and sound;
- between two clips that follow each other in the lecture, a piece of **5 to 30 seconds** was removed; the lengths of the removed pieces are not given;
- the clips of a puzzle are given in a random order, under random ids.

For every clip of every test puzzle, predict its **position** in the lecture: `0` for the clip recorded first, `1` for the next one, and so on up to $K-1$ for a puzzle of $K$ clips.

The lectures are in English, at university level, by many different lecturers and on many subjects. They come in many formats: slides, handwriting on paper or a tablet, whiteboards, a lecturer talking to the camera, live coding. Some lecturers appear in both `train` and `test`; others appear only in `test`.

The dataset can be accessed here:

[Download the dataset (626.4 MB)](https://contest.faio.kz/problems/01a173de-b790-7144-991c-d7cd96dc7b92/public/data.zip)

```bash
wget https://contest.faio.kz/problems/01a173de-b790-7144-991c-d7cd96dc7b92/public/data.zip
```

## 3. Input

- `train/puzzles.csv`, one row per clip:
  - `puzzle_id` — the puzzle the clip belongs to;
  - `clip_id` — the clip's file name in `train_clips.zip`;
  - `position` — the clip's position in its puzzle (`0` = first);
  - `lecturer` — an anonymous code; clips with the same code were recorded by the same lecturer.
- `test/puzzles.csv`: the columns `puzzle_id` and `clip_id` only.
- `train_clips.zip` and `test_clips.zip`: one file `clips/<clip_id>.mp4` per clip (360p H.264 video, mono AAC audio, 30 seconds).

The training set has 69 puzzles (650 clips); the test set has 100 puzzles (882 clips).

In some videos a small black box covers an on-screen counter, such as a slide number or a clock. This is deliberate: those counters would give the order away.

## 4. Output

Participants must submit only a CSV file named **solution.csv**, with the following format:

```
clip_id,position
c_5d1e7a20,3
c_a93f04c8,0
c_17b6e2f5,5
...
```

- The file must contain exactly one row for every row of `test/puzzles.csv`, in the same order.
- `position` must be an integer. Within each puzzle of $K$ clips, every value from `0` to $K-1$ must be used exactly once.

**Important:** Submit only the `solution.csv` file. Source code is not required.

## 5. Evaluation

- A puzzle with $K$ clips has $K(K-1)/2$ pairs of clips. A pair counts as **correct** if your positions put the two clips in the same relative order as in the lecture. The puzzle's score is the share of correct pairs:

  $$
  \text{puzzle score} = \frac{\text{correct pairs}}{K(K-1)/2}
  $$

  This equals $(1+\tau)/2$, where $\tau$ is Kendall's rank correlation between your order and the true one.

- The final score is the mean puzzle score, placed on a $0$–$100$ scale and rounded to two decimal places:

  $$
  \text{score} = 100 \times \frac{1}{N}\sum_{i=1}^{N} \text{puzzle score}_i
  $$

  A random order scores about $50$, the reversed order scores $0$, and the correct order scores $100$.

- During the competition the leaderboard shows the score on a **public** part of the test set, about 30% of the puzzles. The final ranking uses the remaining **private** part. You do not know which puzzles are public.
- Ties are broken by the share of **neighbouring** clips placed correctly: for each clip, whether the clip recorded right after it is placed directly after it in your order, averaged over the puzzles.
- A submission that cannot be read is rejected as a formatting error and receives no score. This happens when the number of rows does not match `test/puzzles.csv`, when the `clip_id` values are not in the same order as in `test/puzzles.csv`, when the `position` column is missing or misnamed, or when a position is empty, not an integer, out of range, or repeated within a puzzle.

## 6. Example

**Input (`train/puzzles.csv`), one puzzle of six clips:**

```
puzzle_id,clip_id,position,lecturer
p0007,c_3f9a01b2,4,L03
p0007,c_8d22e0c5,0,L03
p0007,c_c41a7d93,2,L03
p0007,c_0b6e7a91,5,L03
p0007,c_77f2b6e0,1,L03
p0007,c_e2d95c14,3,L03
```

**Input (`test/puzzles.csv`):** the same layout without `position` and `lecturer`.

**Your Output (`solution.csv`):**

```
clip_id,position
c_5d1e7a20,3
c_a93f04c8,0
c_17b6e2f5,5
...
```

**Scoring one puzzle:** a puzzle of 6 clips has 15 pairs. If your order is right except that two neighbouring clips are swapped, exactly one pair is wrong and the puzzle scores $14/15 \approx 0.933$. If your order is exactly reversed, every pair is wrong and the puzzle scores $0$.

## 7. Important Considerations

- **Public/private split.** The score participants see during the round is computed on about 30% of the test set (29 of the 100 test puzzles). The final result is computed on the remaining 70% (71 puzzles) and announced after the round ends.

**Submission limit.** For each problem, you can submit at most once per minute.