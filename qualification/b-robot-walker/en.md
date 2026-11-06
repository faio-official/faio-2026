## Task

Yerniyaz and Gaukhar built a two-legged robot with six motors. A small neural network controls it: it takes 18 numbers as input (the robot's state and the requested speed) and outputs 6 numbers (motor signals). Your task is to find weights for this network so that the robot walks at the requested speed without falling. You submit only the weights; the jury runs them in the simulator.

## Environment

The environment is a two-legged robot in a physics simulator (Walker2d from Gymnasium / MuJoCo). It runs in steps: it takes an action, advances the robot's physics by one step (0.008 seconds), and returns a new observation and a reward. The implementation is `environment.py` in the starter kit.

- Observation : 18 numbers: positions and velocities of the robot's parts (17 numbers) and the target speed v_target.
- Action : 6 numbers in [−1, 1], one per motor.
- Target speed is drawn at the start of an episode, v_target ∼ Uniform(0.8, 5.0) m/s, and stays fixed until the episode ends.
- Episode ends when the robot falls, or after 1250 steps (10 seconds). The return of an episode is the sum of rewards, maximum 1312.5.

**Reward per step:**

```
r = exp(−4 (v − v_target)²) + 0.05 − 0.001 (a₁² + ⋯ + a₆²)
```

- The first term is for speed: it equals 1 when the current speed v matches the target and decreases as the difference grows.
- 0.05 is for every step on which the robot has not fallen.
- The last term is an energy penalty: the sum of squares of the six motor signals a₁, …, a₆.
- The maximum reward per step is about 1.05.

## Policy

The action is computed from the observation exactly like this (file `policy.py`):

```
def act(obs, w):
    x = (obs - w["obs_mean"]) / w["obs_std"]
    x = np.clip(x, -10.0, 10.0)
    x = np.tanh(x @ w["W1"] + w["b1"])   # 18 -> 64
    x = np.tanh(x @ w["W2"] + w["b2"])   # 64 -> 64
    a = x @ w["W3"] + w["b3"]            # 64 -> 6
    return np.clip(a, -1.0, 1.0)
```

`W1` … `b3` are the weights of the 18 → 64 → 64 → 6 network (5766 numbers). `obs_mean`, `obs_std` are observation-normalization constants. The action depends only on the current observation.

## Starter kit

Download: **[starter kit](https://drive.google.com/drive/folders/1xog0M-X71a1wzzrH7LLG5MOuZA3N-lh4?usp=sharing)**

- environment.py : the environment.
- policy.py : loads the weights and computes the action, exactly as the jury does.
- example.py : runs one episode and saves a speed plot and a GIF.
- example_submission.npz : an example submission file.

## Submission format

One file **submission.npz**: a NumPy archive with exactly 8 arrays of type float32:

| Array | Shape |
| --- | --- |
| `W1` | (18, 64) |
| `b1` | (64,) |
| `W2` | (64, 64) |
| `b2` | (64,) |
| `W3` | (64, 6) |
| `b3` | (6,) |
| `obs_mean` | (18,) |
| `obs_std` | (18,) |

Requirements: all numbers are finite (no NaN or Inf), all values of `obs_std` are greater than zero, the file contains no other arrays, and the file size is at most 1 MB.

A file that does not match this format gets 0 points.

## Evaluation

The system runs the policy in **50 hidden episodes** (seeds, hence target speeds and initial states, are unknown), the same for all participants. You may submit several times: **the best score** among all submitted files counts.

Let R be the mean return over these episodes:

```
score = 100 · min(1, R / 1312.5)
```

## Useful links

- OpenAI Spinning Up in Deep RL
- Gymnasium: Walker2d
