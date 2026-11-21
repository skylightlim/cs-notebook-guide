# Machine Learning (WIA1006), Python

> Teaching computers to learn patterns from data instead of being explicitly programmed.

**Topics:** [Intro to ML](#1-introduction-to-machine-learning) · [Data Pre-processing](#2-data-pre-processing) · [Regression](#3-regression-linear--logistic) · [Decision Trees](#4-decision-trees) · [Neural Networks](#5-neural-networks) · [Bayes, Markov Models & HMMs](#6-bayes-theorem-markov-models--hmms) · [Clustering](#7-clustering) · [PCA](#8-principal-component-analysis-pca) · [SVM](#9-support-vector-machines-svm) · [GMM & EM](#10-gaussian-mixture-models-gmm--expectation-maximization-em) · [Reinforcement Learning](#11-reinforcement-learning-rl) · [Genetic Algorithms](#12-genetic-algorithms-ga) · [ML Workflow](#ml-project-workflow)

---

## 1. Introduction to Machine Learning

**Tom Mitchell's definition:** a program **learns** from experience **E** with respect to task **T** and performance measure **P** if its performance at T, measured by P, improves with E.

*Example: checkers.* T = playing checkers, P = % of games won, E = playing games against itself.
*Example: spam filter.* T = classify emails, P = % correctly classified, E = a dataset of labelled emails.

> 🐶 **Analogy:** traditional programming = writing exact rules ("if the email contains 'lottery', mark spam"). Machine learning = showing the computer thousands of examples and letting it work out the rules, like training a puppy with treats instead of a manual.

```
Traditional:  Data + Rules  → Program → Answers
ML:           Data + Answers → Learning → Rules (model)
```

**AI ⊃ ML ⊃ Deep Learning.**

### Types of learning
| Type | Data | Goal | Example |
|---|---|---|---|
| **Supervised** | inputs **with labels** | predict labels for new inputs | house price (regression), spam/not spam (classification) |
| **Unsupervised** | inputs **without labels** | find structure | customer segmentation (clustering) |
| **Reinforcement** | rewards from interacting with an environment | learn the best actions | game-playing AI, robots |

**Classification** = predicting a category; **regression** = predicting a number.

### Designing a learning system (checkers example)
1. **Training experience:** direct or indirect feedback? Who picks the examples? Does the training distribution match the test distribution?
2. **Target function:** what exactly to learn, e.g. `V(b)` = the value of board state b.
3. **Representation:** e.g. linear: `V̂(b) = w₀ + w₁x₁ + … + w₆x₆` (x's = number of pieces, kings, threatened pieces…).
4. **Learning algorithm:** e.g. LMS weight update `wᵢ ← wᵢ + η (V_train(b) − V̂(b)) xᵢ`. Training values can come from the next state's estimate (**temporal difference learning**).

**Measuring performance:** accuracy, precision, recall, error rate, on **test data the model has never seen**.

---

## 2. Data Pre-processing

Real data is messy: **missing values, outliers, inconsistent formats, different scales**. "Garbage in, garbage out."

> 🥗 **Analogy:** washing and chopping vegetables before cooking. Even the best chef (algorithm) can't make a good dish with dirty ingredients.

### 7 steps
```python
# 1. Acquire the dataset   2. Import libraries
import numpy as np, pandas as pd, matplotlib.pyplot as plt
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import LabelEncoder, OneHotEncoder, StandardScaler, MinMaxScaler
from sklearn.model_selection import train_test_split

# 3. Import the dataset
df = pd.read_csv("student.csv")
X = df.iloc[:, :-1]; y = df.iloc[:, -1]

# 4. Handle missing values
df.isnull().sum()                     # find them
df = df.dropna()                      # option A: drop rows
df["age"] = df["age"].fillna(df["age"].mean())   # option B: fill with mean/median/mode

# 5. Encode categorical data
df["gender"] = LabelEncoder().fit_transform(df["gender"])   # Male→1, Female→0
df = pd.get_dummies(df, columns=["city"])                   # one-hot: city_KL, city_Penang…

# 6. Feature scaling
X_scaled = StandardScaler().fit_transform(X)   # mean 0, std 1:  z = (x − μ)/σ
X_mm     = MinMaxScaler().fit_transform(X)     # range 0–1:      (x − min)/(max − min)

# 7. Split train/test (e.g. 80/20 Pareto, 70/30)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

**Why one-hot instead of label encoding for cities?** Label encoding (KL=0, Penang=1, JB=2) implies JB > Penang > KL, which is meaningless. One-hot avoids fake ordering.

**Why scale?** Salary (0–100,000) would drown out age (0–100) in distance-based algorithms like k-means and SVM.

**Binning/bucketing:** turn a continuous value into categories (marks → A/B/C).

---

## 3. Regression (Linear & Logistic)

### Linear regression
Fit a straight line (or hyperplane) through the data: `h(x) = θ₀ + θ₁x₁ + … + θₙxₙ = θᵀx`

**Cost function** (mean squared error): `J(θ) = ½ Σ (h(x⁽ⁱ⁾) − y⁽ⁱ⁾)²`

> 📈 **Analogy:** drawing the "best-fit line" through scattered points by hand. Regression does it mathematically by minimising the total squared distance from the points to the line.

### Gradient descent
Repeatedly nudge θ in the direction that reduces the cost:
```
θⱼ := θⱼ − α · ∂J/∂θⱼ
LMS (Widrow-Hoff) update rule:  θⱼ := θⱼ + α (y⁽ⁱ⁾ − h(x⁽ⁱ⁾)) xⱼ⁽ⁱ⁾
```
- **α (learning rate):** too big → overshoots; too small → very slow.
- **Batch gradient descent:** use **all** examples for each update (accurate but slow on big data).
- **Stochastic/incremental gradient descent:** update after **each** example (faster, noisier).

> ⛰️ **Analogy:** you're on a foggy mountain and want to reach the valley. Feel the slope under your feet and take a step downhill. Repeat. α is your step size.

```python
from sklearn.linear_model import LinearRegression
model = LinearRegression().fit(X_train, y_train)
print(model.coef_, model.intercept_, model.score(X_test, y_test))  # R²
```

**Locally weighted linear regression:** for each prediction, fit a line that weights nearby training points more heavily (`w⁽ⁱ⁾ = exp(−(x⁽ⁱ⁾ − x)² / 2τ²)`). It's non-parametric (keeps all the data).

### Logistic regression (for classification!)
Despite the name, it's used for **yes/no classification**. It squashes the output into a probability between 0 and 1 with the **sigmoid**:
```
h(x) = σ(θᵀx) = 1 / (1 + e^(−θᵀx))
predict 1 if h(x) ≥ 0.5, else 0
```
```python
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression().fit(X_train, y_train)
clf.predict_proba(X_test)
```
*Example:* predict whether a student passes (1) or fails (0) from hours studied.

---

## 4. Decision Trees

A **decision tree** classifies by asking a series of questions about the features.

```
           Outlook?
         /    |     \
     Sunny  Overcast  Rain
       |       |        |
   Humidity?  YES     Wind?
    /    \           /    \
  High  Normal    Strong  Weak
   NO    YES        NO     YES
```
> 🎮 **Analogy:** the game "20 Questions". A good player asks the question that eliminates the most possibilities first.

**Properties:** easy to interpret, handles discrete and continuous features, robust to noise and missing values, represents any boolean function (disjunctions of conjunctions).

### Choosing the best split: entropy and information gain (ID3 algorithm)
**Entropy** = how "mixed" or impure a set is:
```
H(S) = −Σ pᵢ log₂ pᵢ
```
- All the same class → H = 0 (pure)
- 50/50 split → H = 1 (maximally mixed)

**Information gain** = how much entropy drops after splitting on attribute A:
```
Gain(S, A) = H(S) − Σᵥ (|Sᵥ|/|S|) · H(Sᵥ)
```
**ID3:** at each node, pick the attribute with the **highest information gain**; repeat on each branch until the nodes are pure or no attributes are left.

*Worked example:* 14 days, 9 "Play = Yes", 5 "No":
`H = −(9/14)log₂(9/14) − (5/14)log₂(5/14) ≈ 0.940`
If splitting on Outlook gives Gain = 0.246 (the highest), Outlook becomes the root.

**Inductive bias:** ID3 prefers **shorter trees** with high-gain attributes near the root (Occam's razor).

### Overfitting
The tree memorises noise in the training data → great training accuracy, bad test accuracy.
> 📝 Like a student memorising past-year answers word for word instead of understanding the concepts. They fail when the questions change slightly.

**Prevention (pruning):**
- **Pre-pruning:** stop growing early (max depth, min samples per leaf).
- **Post-pruning / reduced-error pruning:** grow the full tree, then remove nodes if that doesn't hurt accuracy on a **validation set**.
- **Cross-validation (k-fold):** split the data into k parts and rotate which part is used for validation, so no training data is "wasted".

```python
from sklearn.tree import DecisionTreeClassifier
tree = DecisionTreeClassifier(criterion="entropy", max_depth=4).fit(X_train, y_train)
```

---

## 5. Neural Networks

Inspired by the brain: **biological neurons** receive signals through dendrites, sum them in the cell body, and "fire" through the axon if a threshold is reached.

### Artificial neuron (perceptron)
```
x₁ ─w₁─┐
x₂ ─w₂─┼─► Σ (wᵢxᵢ) + b ─► activation f ─► output
x₃ ─w₃─┘
```
> 🗳️ **Analogy:** a committee vote. Each input is a member's opinion, the weight is how much you trust that member, and the bias is your default leaning. If the weighted total passes a threshold, the decision is "yes".

**Activation functions:**
| Function | Formula | Notes |
|---|---|---|
| Step | 1 if z ≥ 0 else 0 | original perceptron |
| **Sigmoid** | `1/(1+e⁻ᶻ)` | output 0–1; can cause "vanishing gradients" |
| Tanh | `(eᶻ−e⁻ᶻ)/(eᶻ+e⁻ᶻ)` | output −1 to 1 |
| **ReLU** | `max(0, z)` | fast, the most popular in deep learning |
| Softplus | `ln(1+eᶻ)` | a smooth ReLU |

**Types of networks:** single-layer perceptron (only linearly separable problems; it **can't** learn XOR), **multilayer perceptron (MLP)** with hidden layers, recurrent networks (feedback loops), convolutional networks (images).

**Learning techniques:** supervised, unsupervised, reinforcement.

### Training an MLP: backpropagation
1. **Forward pass:** compute outputs layer by layer.
2. Compute the **error** at the output (target − actual).
3. **Backward pass:** propagate the error back through the layers using the chain rule, working out how much each weight contributed.
4. **Update weights:** `w ← w + η · δ · input` (gradient descent).
5. Repeat for many **epochs**.

> 🎯 **Analogy:** a basketball team loses a game. The coach traces back who contributed most to each mistake (backprop) and gives each player specific feedback (weight updates).

**Strengths:** learns complex non-linear patterns, tolerates noise, many applications (image and speech recognition, medical diagnosis).
**Problems:** slow training, local minima, "black box" (hard to interpret), overfitting, choosing the architecture.

```python
from sklearn.neural_network import MLPClassifier
mlp = MLPClassifier(hidden_layer_sizes=(10, 5), activation="relu", max_iter=500).fit(X_train, y_train)
```

---

## 6. Bayes' Theorem, Markov Models & HMMs

### Bayes' theorem
```
P(A|B) = P(B|A) · P(A) / P(B)
posterior = likelihood × prior / evidence
```
*Example:* 1% of people have a disease. The test is 90% accurate for sick people and has a 5% false-positive rate. You test positive. Chance you're sick?
`P(D|+) = 0.9×0.01 / (0.9×0.01 + 0.05×0.99) = 0.009 / 0.0585 ≈ 15%`. Surprisingly low, because the disease is rare!

**Naive Bayes classifier:** assumes features are independent given the class: `P(class | x₁…xₙ) ∝ P(class) Π P(xᵢ | class)`. Simple, fast, and great for spam filtering.

### Markov models
A sequence of **states** where the next state depends **only on the current state** (the Markov property, "memoryless").
```
       0.3            0.8
      ┌──┐           ┌──┐
      ▼  │   0.7     ▼  │
     Rain ─────────► Dry
          ◄─────────
              0.2
Initial: P(Rain) = 0.4, P(Dry) = 0.6
```
**P(Dry, Dry, Rain, Rain)** = `P(Dry) · P(Dry|Dry) · P(Rain|Dry) · P(Rain|Rain) = 0.6 × 0.8 × 0.2 × 0.3 = 0.0288`

> 🎲 **Analogy:** a board game where your next move only depends on the square you're on now, not how you got there.

### Hidden Markov Models (HMM)
The states are **hidden**; you only see **observations** that each state emits with some probability.
- **Transition probabilities** `aᵢⱼ`, **emission probabilities** `bᵢ(o)`, **initial probabilities** `πᵢ`.

> 🧥 **Analogy:** you're locked indoors and can't see the weather (hidden state), but you can see whether your friend brings an umbrella (observation). From a week of umbrella observations you infer the likely weather.

**Three main problems:**
| Problem | Question | Algorithm |
|---|---|---|
| **Evaluation** | how likely is this observation sequence? | **Forward** algorithm (trellis, dynamic programming) |
| **Decoding** | what's the most likely hidden state sequence? | **Viterbi** algorithm |
| **Learning** | what model parameters best explain the data? | **Baum-Welch** (an EM algorithm) |

**Applications:** speech recognition, handwriting/character recognition, word recognition, gene sequencing.

---

## 7. Clustering

**Clustering** = grouping similar data points **without labels** (unsupervised).

> 🧺 **Analogy:** sorting a pile of laundry into groups without being told the categories. You naturally group by colour or type.

**Applications:** customer segmentation, document grouping, image segmentation, anomaly detection.

**Similarity / distance measures:** Euclidean `√Σ(xᵢ−yᵢ)²`, Manhattan `Σ|xᵢ−yᵢ|`, cosine similarity.

### K-means
1. Choose **k** and pick k initial centroids (randomly).
2. **Assign** each point to the nearest centroid.
3. **Update** each centroid to the mean of its assigned points.
4. Repeat 2–3 until the assignments stop changing (**convergence**).

```python
from sklearn.cluster import KMeans
km = KMeans(n_clusters=3, n_init=10).fit(X)
km.labels_, km.cluster_centers_
```
Weaknesses: you must choose k (use the **elbow method**), it's sensitive to initialisation and outliers, and it assumes round clusters.

### Hierarchical clustering
- **Agglomerative (bottom-up):** start with every point as its own cluster; repeatedly merge the two closest clusters.
- **Divisive (top-down):** start with one cluster and split.
- The result is a **dendrogram** (tree); "cut" it at any height to get the number of clusters you want.
- Linkage: single (closest pair), complete (farthest pair), average.

### Spectral clustering (graph partitioning)
Treat the data as a graph (similarity = edge weight) and cut it into groups with weak connections between them.
1. Build a similarity matrix and the graph **Laplacian**.
2. Compute its eigenvectors.
3. Run k-means on the rows of the eigenvector matrix.

It handles odd-shaped clusters (e.g. two interlocking rings) where k-means fails.

---

## 8. Principal Component Analysis (PCA)

**Problem:** high-dimensional data (hundreds of features) is slow, hard to visualise, and suffers from the "curse of dimensionality".

**PCA** finds new axes (**principal components**) along which the data **varies the most**, then keeps only the top few.

> 📸 **Analogy:** photographing a 3D object. You choose the camera angle that shows the most detail (maximum variance) so a 2D photo still captures most of the information.

**Steps:**
1. Standardise the data (mean 0).
2. Compute the **covariance matrix**.
3. Find its **eigenvectors** (directions) and **eigenvalues** (amount of variance).
4. Sort by eigenvalue; keep the top k eigenvectors.
5. Project the data onto them.

```python
from sklearn.decomposition import PCA
pca = PCA(n_components=2)
X2 = pca.fit_transform(X_scaled)
pca.explained_variance_ratio_   # e.g. [0.72, 0.18] → 90% of the info kept in 2 dims
```

### Application: Eigenfaces (face recognition)
Each face image (e.g. 100×100 = 10,000 pixels) is a vector. PCA on many faces gives **eigenfaces**, the "building block" faces. Any face ≈ average face + a weighted sum of a few eigenfaces. To recognise a face, compare its weights with the stored faces' weights.

---

## 9. Support Vector Machines (SVM)

An SVM finds the **separating line/hyperplane with the maximum margin** between two classes.

```
  o  o            x  x
 o  o  ┆    ┃    ┆  x  x
  o    ┆    ┃    ┆ x
       ┆◄──►┃◄──►┆
       margin    (support vectors sit on the dashed lines)
```
> 🛣️ **Analogy:** building the widest possible road between two villages. The houses closest to the road (**support vectors**) decide where it goes; houses further away don't matter.

**Hyperplane:** `wᵀx + b = 0`. Margin width = `2/‖w‖`, so maximising the margin = **minimising ‖w‖²** subject to `yᵢ(wᵀxᵢ + b) ≥ 1`.

**Soft margin:** real data overlaps, so allow some mistakes using slack variables ξᵢ. Parameter **C** sets the trade-off: big C = few mistakes but a narrow margin (risk of overfitting); small C = a wider margin with more mistakes.

### The kernel trick
If the data isn't linearly separable, map it to a higher dimension where it is, *without* computing that mapping explicitly.
> 🎈 **Analogy:** red and blue marbles mixed on a table can't be split with a straight ruler. Flick the table so the red ones (in the middle) fly higher, and now a flat sheet can separate them in 3D.

Common kernels: linear, polynomial `(xᵀz + 1)ᵖ`, **RBF/Gaussian** `exp(−γ‖x − z‖²)`, sigmoid.

```python
from sklearn.svm import SVC
svm = SVC(kernel="rbf", C=1.0, gamma="scale").fit(X_train, y_train)
```
**Applications:** text classification, image recognition, bioinformatics, handwriting recognition.

---

## 10. Gaussian Mixture Models (GMM) & Expectation-Maximization (EM)

**Hard clustering** (k-means): each point belongs to exactly one cluster.
**Soft clustering** (GMM): each point has a **probability** of belonging to each cluster.

A **GMM** models the data as a mix of K Gaussian (bell-curve) distributions, each with a mean μₖ, covariance Σₖ, and mixing weight πₖ.

> 🍬 **Analogy:** a jar of sweets made by 3 different factories, each with its own typical size and colour. Looking at one sweet, you can say "70% likely factory A, 30% factory B".

### EM algorithm (how a GMM is trained)
1. **Initialise** μ, Σ, π (e.g. from k-means).
2. **E-step (Expectation):** for each point, compute the "responsibility" of each Gaussian (the probability the point came from it).
3. **M-step (Maximization):** update μ, Σ, π using those responsibilities as weights.
4. Repeat until the log-likelihood stops improving.

**Relationship to k-means:** k-means is like EM with hard 0/1 responsibilities and equal, spherical covariances.
**General EM:** used whenever there are hidden (latent) variables; it's guaranteed not to decrease the likelihood (proof uses **Jensen's inequality**).

```python
from sklearn.mixture import GaussianMixture
gmm = GaussianMixture(n_components=3).fit(X)
gmm.predict_proba(X[:5])     # soft memberships
```

---

## 11. Reinforcement Learning (RL)

An **agent** learns by interacting with an **environment**: it takes **actions**, receives **rewards**, and tries to maximise the total reward over time.

```
        action aₜ
Agent ───────────► Environment
  ▲                    │
  └── state sₜ₊₁, reward rₜ₊₁ ──┘
```
> 🐕 **Analogy:** training a dog. Sit → treat (+ reward). Jump on the couch → "No!" (− reward). Over time the dog learns which actions in which situations lead to treats.

**Applications:** AlphaGo, game AI, robotics, self-driving cars, recommendation systems.

### Key concepts
| Term | Meaning |
|---|---|
| **MDP** (Markov Decision Process) | states S, actions A, transition probabilities P(s'\|s,a), rewards R, discount γ |
| **Policy** π(s) | the agent's strategy: which action to take in each state |
| **Return** | total future reward: `Gₜ = rₜ₊₁ + γrₜ₊₂ + γ²rₜ₊₃ + …` |
| **Discount factor γ** (0–1) | how much future rewards matter (γ near 0 = short-sighted) |
| **Value function** V(s) | expected return starting from s and following π |
| **Q-function** Q(s,a) | expected return taking action a in s, then following π |
| Episodic vs continuing tasks | has an end (a game) vs goes on forever (stock trading) |

**Optimal policy:** choose the action with the highest Q: `π*(s) = argmaxₐ Q*(s,a)`.

### Q-learning
```
Q(s,a) ← Q(s,a) + α [ r + γ · maxₐ' Q(s',a') − Q(s,a) ]
```
"Adjust my estimate toward (reward now + best estimated future)."
**Exploration vs exploitation:** sometimes try random actions (ε-greedy) to discover better options, instead of always picking the current best.

```python
Q = np.zeros((n_states, n_actions))
for episode in range(1000):
    s = env.reset(); done = False
    while not done:
        a = np.random.randint(n_actions) if np.random.rand() < eps else np.argmax(Q[s])
        s2, r, done = env.step(a)
        Q[s, a] += alpha * (r + gamma * np.max(Q[s2]) - Q[s, a])
        s = s2
```

---

## 12. Genetic Algorithms (GA)

An optimisation method inspired by **natural selection** ("survival of the fittest").

> 🧬 **Analogy:** breeding racing pigeons. Keep the fastest birds, breed them together, occasionally get a random mutation, and over generations the flock gets faster.

### Steps
1. **Initialise** a random **population** of candidate solutions (**chromosomes**, often bit strings).
2. **Evaluate** each with a **fitness function**.
3. **Select** parents (fitter = more likely to be chosen: roulette wheel, tournament).
4. **Crossover:** combine parts of two parents.
   ```
   Parent A: 1101|0010      Child 1: 1101|1100
   Parent B: 0110|1100  →   Child 2: 0110|0010
   ```
5. **Mutation:** randomly flip some bits (keeps diversity): `11011100 → 11001100`.
6. Replace the population; repeat until good enough or the generation limit is reached.

**Uses:** scheduling (exam timetables!), routing, design optimisation, feature selection. Good when the search space is huge and you don't have gradients.

---

## ML project workflow
```
1. Define the problem  →  2. Collect data  →  3. Pre-process (clean, encode, scale)
→  4. Split train/test  →  5. Choose & train a model  →  6. Evaluate (accuracy, confusion matrix, cross-validation)
→  7. Tune hyperparameters  →  8. Deploy & monitor
```

| Problem | Try first |
|---|---|
| Predict a number | Linear regression, decision tree regressor |
| Yes/no or categories | Logistic regression, decision tree, SVM, neural network, naive Bayes |
| Group unlabelled data | K-means, hierarchical, GMM |
| Too many features | PCA |
| Sequences (speech, text) | HMM |
| Learn by trial and error | Reinforcement learning |
| Optimise a complex objective | Genetic algorithm |

---

### ✅ Quick self-check
1. Supervised or unsupervised: grouping news articles by topic with no labels? *(unsupervised, clustering)*
2. What does entropy = 0 mean? *(the set is pure, all one class)*
3. Why can't a single perceptron learn XOR? *(XOR isn't linearly separable)*
4. Which HMM algorithm finds the most likely hidden state sequence? *(Viterbi)*
5. What are support vectors? *(the training points closest to the decision boundary)*
6. What's the difference between k-means and GMM? *(hard vs soft cluster assignment)*
