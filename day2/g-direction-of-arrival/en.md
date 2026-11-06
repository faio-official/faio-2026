# Story
Picture a service robot standing in a room ready to take requests from humans.  
Before the robot can respond, it has to solve two problems: work out **where** the voice is coming from, so it can turn to face the person, and work out **what** that person actually said.

The robot hears the room through a ring of **four microphones** mounted on its head.  
Because the microphones sit a few centimetres apart, a voice reaches each of them at a slightly different moment, and those tiny timing differences reveal the direction the sound arrived from.

You are given a collection of such recordings. Each one is 30 seconds of four-channel audio in which exactly one person speaks.  
Your mission is to recover, for every recording, the **direction of arrival** of that speaker's voice and a **transcript** of what they said.

# Formal Problem Statement

Every item in the dataset is **one 30-second, four-channel recording** made by the robot's microphone array in a simulated room.  
The room has random dimensions; the robot stands and faces a random direction inside it.  
The four microphones are arranged on a circle of **16 cm diameter** at angles $0^\circ$, $90^\circ$, $180^\circ$, $270^\circ$ in the robot's own frame, all at the same height ($1.2$ m).  
The channels in each file are ordered to match those four bearings.

![Top view of the robot's four-microphone array and the azimuth convention: microphone 0 points straight ahead ($0^\circ$), and the source bearing $\theta$ (`azimuth_rel_robot_deg`) increases counter-clockwise — $90^\circ$ left, $180^\circ$ behind, $270^\circ$ right.]

<div align="center">
    <img src="https://contest.faio.kz/problems/01a17418-df96-7552-901a-c0e0757173db/public/diagram.png" alt="hid-dim-v3" border="0" width="500">
</div>

Inside every recording there is:

- **exactly one speech source** — a person speaking English, placed at a random angle around the robot. Their speech starts at a random moment and fits entirely within the 30 seconds;
- **ambient noise** — background noise that fills the whole clip (sometimes includes very far-away background speech, which is not part of the transcript);
- **upto 3 point-noise sources** — localized sounds (machines, instruments, everyday events) placed, like the speaker, at their own directions around the robot, and at random times.

For every recording you must predict two things:

- `azimuth_rel_robot_deg` — the **direction of arrival** of the speech source, as a bearing in the robot's frame, a real number in $[0, 360)$. 

   - $0^\circ$ is the direction of microphone 0 (the robot's reference / "straight ahead").
   - $90^\circ$ is to the robot's left.
   - $180^\circ$ behind it.
   - $270^\circ$ to its right. 
   
   The angle increases **counter-clockwise**.

   The angle is **circular**, so $359^\circ$ and $1^\circ$ are only $2^\circ$ apart. 
   
   Only the horizontal bearing is scored — elevation and distance are not required.

- `text` — a transcript of what the speaker said. You do NOT need to worry about punctuation and register (upper vs lower case). Grader script will substitute all punctuation with space and convert all letters to lowercase. 

# Input

The data is organised into a **development** split (with ground truth, for building and checking your solution) and a **test** split (without ground truth, the one you submit predictions for):

```
data/
  audio/val/val_000000.flac, val_000001.flac, ...    # 4-channel, 16 kHz, 30 s
  audio/test/test_000000.flac, ...                   # 4-channel, 16 kHz, 30 s
  val.tsv          # development labels
  test.tsv         # test items (audio only)
```

All audio is **4-channel FLAC, 16 kHz, exactly 30 seconds**. Channel $k$ is microphone $k$ (bearings $0/90/180/270^\circ$).

`val.tsv` is a tab-separated file with a header and the following columns:

- `audio` — path to the recording, relative to the data directory (e.g. `audio/val/val_000000.flac`).
- `azimuth_rel_robot_deg` — the ground-truth direction of arrival of the speech, in $[0, 360)$.
- `text` — the ground-truth transcript of the speaker.

`test.tsv` has the **same format but only the `audio` column**; the other two are what you must predict.

The dataset and helpful code can be accessed here: [Download data.zip (3.52 GB)](https://contest.faio.kz/problems/01a17418-df96-7552-901a-c0e0757173db/public/data.zip)

```bash
wget https://contest.faio.kz/problems/01a17418-df96-7552-901a-c0e0757173db/public/data.zip
```

# Provided Tools

To get you started, the download includes ready-to-use wrappers under `src/` (with their models under `data/models/`). All of them work on **16 kHz mono** audio.

- **`SileroVAD`** (`src/vad/silero.py`) — a neural voice-activity detector. `speech_timestamps(audio)` returns the speech spans in a clip as `[{"start", "end"}, ...]` (sample indices). More accurate, slightly heavier.
- **`WebRTCVAD`** (`src/vad/webrtc.py`) — a lightweight WebRTC voice-activity detector with the same `speech_timestamps(audio)` interface. Faster, and a handy fallback when the neural VAD finds nothing.
- **`FastConformer`** (`src/transcriber/fastconformer.py`) — an English Speech-to-text model (NVIDIA NeMo FastConformer hybrid RNNT/CTC, `stt_en_fastconformer_hybrid_large_pc`). `transcribe_array(audio)` returns the transcript of a mono clip. May take a minute or two to initialize but runs fast afterwards, even on CPU.

# Output

Participants must submit a single tab-separated file named **solution.tsv**, with a header and exactly these three columns:

```
audio	azimuth_rel_robot_deg	text
audio/test/test_000000.flac	272.1797	yeah she is a nurse she worked in infectious disease for ten years
audio/test/test_000001.flac	237.7833	the only thing stopping him from being governor for life
...
```

`azimuth_rel_robot_deg` is your predicted bearing, a real number in $[0, 360)$;  
`text` is your predicted transcript;  
The file should contain **one row for every clip in `test.tsv`**;

**Important:** submit only the `solution.tsv` file; source code is not required. Rows are matched to the ground truth by their `audio` path, so that column must be copied verbatim from `test.tsv`.

# Evaluation

Each of the two predictions is turned into a score in $[0, 1]$ (higher is better), and the final score is their average on a 0–100 scale.  
Both are computed over the **scored clips**: the public or the private part of the test set (see *Public and private leaderboard* below).

**Direction of arrival.** For a test clip with predicted bearing $p$ and true bearing $g$ (both in degrees), the error is the wrap-around (circular) distance

$$\mathrm{err} = \min\!\big(\,|p-g| \bmod 360,\ \ 360 - (|p-g| \bmod 360)\,\big) \in [0, 180]$$

so that pointing the wrong way around the circle is penalised correctly.  
Let $\mathrm{MAE}$ be the mean of $\mathrm{err}$ over all scored clips. The direction score is

$$\text{score}_\text{doa} = \max\!\left(0,\ 1 - \frac{\mathrm{MAE}}{90}\right) \in [0, 1]$$

i.e. a perfect bearing scores $1$, and an average error of $90^\circ$ or more scores $0$. $90^\circ$ is what random guessing averages, so only predictions better than chance earn points.

**Transcription.** Reference and hypothesis are normalised the same way — lower-cased, punctuation replaced by spaces, and runs of whitespace collapsed — and then scored by **word error rate** (WER).

WER counts the fewest number of word edits that turn your transcript into the reference: **substitutions** $S$ (a wrong word), **deletions** $D$ (a reference word you missed) and **insertions** $I$ (an extra word you added).  
The edits are counted per clip, then summed over all scored clips and divided by the total number of reference words $N$ in those clips, so longer clips weigh more:

$$\mathrm{WER} = \frac{S + D + I}{N}$$

For example, reference `the cat sat on the mat` ($N = 6$) vs. hypothesis `the cat sit on mat today` has one substitution (`sat`→`sit`), one deletion (the second `the`) and one insertion (`today`), so $\mathrm{WER} = 3/6 = 0.5$. A perfect transcript has $\mathrm{WER} = 0$; an empty one has $\mathrm{WER} = 1$.

The transcription score is

$$\text{score}_\text{asr} = \max(0,\ 1 - \mathrm{WER}) \in [0, 1]$$

(clamped at $0$, since insertions can push $\mathrm{WER}$ above $1$).

**Final score.**

$$\text{score} = 100 \cdot \frac{\text{score}_\text{doa} + \text{score}_\text{asr}}{2}$$

Rows are matched to the test set by their `audio` value, so the row order and the exact row count do not matter — just give every clip in `test.tsv` one valid row.  
Anything missing or invalid falls back to the **worst case** for that clip rather than being thrown away, so skipping the hard clips never helps:

- a clip you leave out is scored as a $180^\circ$ direction error **and** an empty transcript;
- a blank or non-numeric azimuth is scored as a $180^\circ$ direction error;
- an empty (or missing) transcript is scored as $\mathrm{WER} = 1$.

Extra rows whose `audio` is not in `test.tsv` are ignored, and an azimuth outside $[0, 360)$ is wrapped back into range (taken modulo $360$).  
If submission has a missing or misnamed column — every clip falls to its worst case and the score drops towards $0$.

**Public and private leaderboard.**  
The test set is split at random into two fixed parts that are not disclosed.  
During the contest, the score you see is computed on a **public 40%** of the test clips.  
The **final result** is computed on the remaining **private 60%** and will be announced after the contest ends.  
Always submit predictions for **every** clip in `test.tsv`: you cannot tell which clips are public, and every clip you skip counts as a worst case in whichever part it belongs to.

# Example

**Input (`val.tsv`), one row, shown column by column:**

```
audio                : audio/val/val_000000.flac
azimuth_rel_robot_deg: 282.1706
text                 : thus they were finally sentenced to criminal law
```

The matching `audio/val/val_000000.flac` is a 30-second, 4-channel clip in which this sentence is spoken from about $282^\circ$ (to the robot's right-and-slightly-ahead), over room reverberation and background noise.

**Input (`test.tsv`):** the same format, with only the `audio` column.

**Output example (`solution.tsv`):**

```
audio	azimuth_rel_robot_deg	text
audio/test/test_000000.flac	280.5	thus they were finally sentenced to criminal law
audio/test/test_000001.flac	134.2	three singles were released from the album
...
```
**Submission limit.** For each problem, you can submit at most once per minute.