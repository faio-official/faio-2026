<div align="center">
    <img src="https://contest.faio.kz/problems/01a17411-4760-7785-80c8-1b27d8d5d32d/public/image-c.png" alt="hid-dim-v3" border="0" width="700">
</div>

You stand before an ancient artifact — the Cube of Reflections.

According to legend, those imprisoned within the cube cannot escape by ordinary means: the space inside is distorted, shattered into countless mirrored layers and hidden dimensions. To open a passage to the outside world, one must uncover the true structure of this space.

The sages of the past left behind only one rule: if you can **compress the space to its hidden essence** and then **reconstruct it without significant distortion**, the cube will recognize that you have uncovered its nature and open the way out.

You are given a dataset $X \in \mathbb{R}^{N \times 128}$, representing observations of the space inside the cube.

Your task is to develop an algorithm that finds the most compact representation of the objects while allowing the original data to be accurately reconstructed.

You must implement a class `Compressor` with three methods:

- `fit(X_train)` — studies the structure of the space using the training data;
- `encode(X)` — compresses the objects into a representation of dimension $k$;
- `decode(Z)` — reconstructs the original 128 features from the compressed representation.

Here, $1 \le k \le 128$.

Your goal is to find a representation with the smallest possible dimensionality that still allows the space to be reconstructed accurately enough to open the path to freedom.

The training dataset is available in the file `train.csv`.

Each row represents one sample with 128 features.

The evaluation system automatically creates an instance of the `Compressor` class and calls the methods `fit`, `encode`, and `decode`.

Data is passed as two-dimensional NumPy arrays.

Reading from standard input is not required.

Submit a single Python file implementing the `Compressor` class.

The class must contain the following methods:

- `fit(X_train)`
- `encode(X)`
- `decode(Z)`

The `encode` method must return an array $Z \in \mathbb{R}^{N \times k}$ of type `float32`.

The `decode` method must return the reconstructed data $\hat X \in \mathbb{R}^{N \times 128}$.

The number and order of samples must be preserved.

All returned values must be finite.

Writing to standard output is not required.

Reconstruction quality is evaluated using Normalized Mean Squared Error (NMSE):

$$
\operatorname{NMSE}(X,\hat X) =
\frac{\|X-\hat X\|_F^2}
{\|X-\mathbf{1}\mu_{\mathrm{train}}^{T}\|_F^2},
$$

where $\hat X$ is the reconstructed data, $\mu_{\mathrm{train}}$ is the vector of mean feature values of the training dataset, and $\mathbf{1}$ is a vector of ones.

The final score is calculated as follows:

$$
S =
\begin{cases}
100\left(1-\dfrac{k}{128}\right), & \operatorname{NMSE}\leq 0.01,\\
0, & \operatorname{NMSE}>0.01.
\end{cases}
$$

Here, $k$ is the dimensionality of the compressed representation.

The smaller $k$ is, the higher the score, provided that the reconstruction error does not exceed 0.01.

Evaluation is performed on a hidden test dataset.

The maximum possible score is 100.

Constraints:

- Allowed libraries: NumPy, SciPy, scikit-learn, and the Python standard library.

The `decode` method may only use the compressed representation and the parameters learned during training.

Transferring the original test data outside the compressed representation is prohibited.

**Task Materials**

The training dataset and Python code templates are available at the following link:

[Download c-data.zip](https://contest.faio.kz/problems/01a17411-4760-7785-80c8-1b27d8d5d32d/public/c-data.zip)
```bash
wget https://contest.faio.kz/problems/01a17411-4760-7785-80c8-1b27d8d5d32d/public/c-data.zip
```

**Submission limit.** For each problem, you can submit at most once per minute.