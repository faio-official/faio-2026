# The Mysterious Shooting Ride

## Story

One day, you are on The Mysterious Shooting Ride, an interactive dark ride at an amusement park, for the *n*th time. You are so tired of aiming at the targets by eye every time that you decide to hand the job over to your robot assistant.

Your robot assistant should process the video from a camera at the front of the cart and aim at the target. Build the vision system behind the robot so that you can enjoy the scenery of the classic game scenes along the ride and still beat your friends on the ride's leaderboard!

Here are the key characteristics of the ride:

1. The ride is built so that exactly one target is visible at all times.
2. There is a 0.3-second delay between pressing the 'shoot' button and the target actually being hit. During this delay, the ride keeps moving the same way it did in the previous frames.
3. There are exactly three targets in the game:
   - a folder icon with an error sign in the middle
   - a mesh of four triangles joined into a single shape
   - a rectangle filled with green, blue, and purple shades that eerily resemble a spectrogram you once saw in an audio processing course

## Dataset Description

To develop and test your robot assistant, you are given video segments recorded on the ride. Each segment is 2.5 seconds long and has 25 frames, numbered 0 to 24 inclusive (10 frames per second, so the 0.3-second delay equals 3 frames). The test set contains 800 segments; these are the ones you submit predictions for. The train set contains 799 segments.

For each test segment, your system must predict the pixel to aim at. The shot is fired at the end of the segment, at frame 24, and because of the 0.3-second delay it hits at frame 27. Your prediction must therefore land on the target as it appears in frame 27. Frames 25, 26, and 27 of each video are NOT given to you. No ground truth labels are provided for either the train or the test set.

You are also given a cropped image of each of the three targets, one target per file, in the targets directory.

[Download task-a.zip (170.8 MB)](https://contest.faio.kz/problems/01a173ba-7706-7429-804b-246a4525486f/public/task-a.zip)

```bash
wget https://contest.faio.kz/problems/01a173ba-7706-7429-804b-246a4525486f/public/task-a.zip
```

## Submission

Each team submits one CSV file with a header row followed by 800 rows, one per test video. Each row gives exactly one (y, x) coordinate pair: the point where you want to shoot.

Coordinates are integer pixel positions in the video's original resolution, with the origin (0, 0) at the top-left corner: y is the row (counted downwards) and x is the column (counted to the right).

The file uses the following format:

```
filename,y,x
video1200.mp4,100,100
video1201.mp4,100,100
...
```

## Evaluation

Your solution is scored on both how often you hit the target and how close to its center you hit. A shot is a success if the predicted point lies on any pixel of the target's mask in frame 27. A miss scores 0 points. A successful shot scores between 100 and 50 points: the score decreases linearly with the distance *D* from the predicted point to the center of the target, adjusted for the video's resolution, from 100 at the center down to 50 at *D* = 32 pixels or more.

```python
D = distance_to_center * sqrt(image_width * image_height / 640 / 360)
S = 1.0 - 0.5 * np.clip(D / 32.0, 0.0, 1.0)
success = 1 if prediction in target else 0
score = S * success * 100
```

The final score is the average score over all test videos. Note that masks of rotated or zoomed targets are interpolated at the object boundaries, so for a small number of pixels near the edges the actual result may differ from the formula above.

## Important Considerations

- **Public/private split.** The score shown during the round is computed on 50% of the test set. The final result is computed on the remaining 50% and announced after the round ends.

**Submission limit.** For each problem, you can submit at most once per minute.