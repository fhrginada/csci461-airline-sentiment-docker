# CSCI461 Big Data - Assignment #1
## Dockerized Data, Text Analysis and Machine Learning

**University:** Nile University
**Course:** CSCI461 - Introduction to Big Data
**Semester:** Fall 2026
**Dataset:** Twitter US Airline Sentiment
**Docker Hub image:** [`maysoun1106/bd-a1`](https://hub.docker.com/r/maysoun1106/bd-a1)

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Results at a Glance](#2-results-at-a-glance)
3. [Dataset](#3-dataset)
4. [Project Structure](#4-project-structure)
5. [Requirements](#5-requirements)
6. [Quick Start](#6-quick-start)
7. [Docker Implementation](#7-docker-implementation)
8. [Useful Docker Commands](#8-useful-docker-commands)
9. [Output Files](#9-output-files)
10. [Docker Hub](#10-docker-hub)
11. [Running the Published Docker Image](#11-running-the-published-docker-image)
12. [Reproducibility](#12-reproducibility)
13. [Screenshots](#13-screenshots)
14. [Team](#14-team)

---

## 1. Project Overview

This project analyzes the Twitter US Airline Sentiment dataset to explore customer opinions about major US airlines.

The workflow covers data analysis and preprocessing, text analysis, supervised sentiment classification (TF-IDF + Logistic Regression) and unsupervised clustering (K-Means). Everything is executed inside a Docker container, so the results can be reproduced on any machine.

```
Dockerfile -> Custom Image -> Container -> JupyterLab -> Data Cleaning -> Sampling
-> Text Analysis -> TF-IDF -> Classification + K-Means -> Persistent Outputs -> Docker Hub
```

### Objectives

- Explore and understand the dataset.
- Handle missing values and duplicate records.
- Apply data cleaning, transformations and feature creation.
- Perform simple random, stratified and cluster sampling.
- Analyze text data, word frequencies and correlations between numerical text features.
- Build a supervised sentiment classifier using TF-IDF.
- Apply K-Means clustering and interpret the clusters.
- Package the environment with Docker and publish the image on Docker Hub.

---

## 2. Results at a Glance

**Data:** 14,485 tweets after removing duplicate tweet IDs (63% negative, 21% neutral, 16% positive).

### Supervised classification (TF-IDF + Logistic Regression)

The data was split 80% / 20% (train / test), **stratified** by sentiment. The TF-IDF vectorizer was fitted on the training set only, so no information from the test set leaks into the model.

| Model | Accuracy | Macro-F1 |
|---|---|---|
| Multinomial Naive Bayes (unigrams) | 0.722 | 0.552 |
| Logistic Regression (unigrams) | 0.796 | 0.719 |
| **Logistic Regression (tuned: bigrams, balanced classes, C=2)** | 0.792 | **0.739** |

- A model that always predicts "negative" scores **0.627** accuracy, so the classifier is clearly learning from the text.
- The final model was chosen by **macro-F1** (not accuracy), because the classes are imbalanced. Hyperparameters were tuned with 5-fold stratified cross-validation on the training set only.
- Negative tweets are classified best (F1 0.87). Neutral is the hardest class (F1 0.62) and is mostly confused with negative.
- **Limitations:** TF-IDF ignores word order, context and sarcasm; crowd-sourced labels are noisy (especially neutral); some top words are dataset artifacts (e.g. a promotional hashtag); results come from a single week of tweets (Feb 2015).

![Confusion matrix](output/figures/confusion_matrix.png)

### K-Means clustering (k = 8)

- Representation: TF-IDF (stop words removed) -> TruncatedSVD (50 components) -> L2 normalization.
- k was chosen by an experiment (k = 2 to 12, inertia, silhouette score, smallest cluster size, and reading the top terms), not fixed to 3.
- Silhouette score: **0.09**. It is low for every k because tweets are short and sparse, so clusters overlap.
- The clusters follow **topics** (cancellations, customer service, stuck at the gate, thank-you tweets), not sentiment, so they do not match the labels directly (ARI 0.025). K-Means is unsupervised and groups by shared vocabulary.

![Cluster vs labels](output/figures/cluster_vs_labels.png)

Full details: [`output/model_metrics.txt`](output/model_metrics.txt) and [`output/cluster_summary.txt`](output/cluster_summary.txt).

---

## 3. Dataset

**Dataset:** Twitter US Airline Sentiment

**Source:** [Kaggle - Twitter US Airline Sentiment](https://www.kaggle.com/datasets/crowdflower/twitter-airline-sentiment)

The dataset contains tweets about major US airlines with sentiment labels (negative / neutral / positive), the airline, and the reason for negative sentiment.

- Records: 14,640 (14,485 after removing duplicate tweet IDs)
- Columns: 15
- Format: CSV, stored as `data/Tweets.csv`

**If the dataset file is missing:** download `Tweets.csv` from the Kaggle link above and place it in the `data/` folder.

---

## 4. Project Structure

```text
bd-a1/
├── Dockerfile
├── requirements.txt
├── README.md
├── .gitignore
├── data/
│   └── Tweets.csv
├── notebook/
│   └── analysis.ipynb
├── output/
│   ├── cleaned_data.csv
│   ├── cleaned_text.csv
│   ├── model_metrics.txt
│   ├── cluster_summary.txt
│   └── figures/
│       ├── cluster_vs_labels.png
│       ├── confusion_matrix.png
│       ├── correlation_heatmap.png
│       ├── group_comparison_neg_vs_pos.png
│       ├── kmeans_k_selection.png
│       ├── model_comparison.png
│       ├── sampling_comparison.png
│       └── word_frequency_top20.png
└── screenshots/
    ├── 01_docker_build.png
    ├── 02_docker_images.png
    ├── 03_docker_ps.png
    ├── 04_jupyterlab.png
    ├── 05_docker_ps_all.png
    └── 06_docker_inspect.png
```

*Jupyter may automatically create `.ipynb_checkpoints/` folders. They are ignored by `.gitignore`.*

---

## 5. Requirements

- Docker Desktop (or Docker Engine) with a running Docker daemon.
- Nothing else: Python and all libraries are installed inside the image.

The Python libraries (listed in `requirements.txt` and installed during the image build) are:

pandas, numpy, matplotlib, seaborn, scikit-learn, nltk, wordcloud, jupyterlab.

---

## 6. Quick Start

```bash
# 1. Get the project
git clone <YOUR-GITHUB-REPO-URL>
cd bd-a1

# 2. Build the image
docker build -t bd-a1:latest .

# 3. Run the container (bind-mounting the project folder)
docker run -d --name bd-a1-container -p 8888:8888 -v "${PWD}:/home/jovyan/bd-a1" bd-a1:latest

# 4. Get the JupyterLab URL with its token
docker logs bd-a1-container
```

Then open `http://localhost:8888`, open `notebook/analysis.ipynb`, and choose **Kernel -> Restart Kernel and Run All Cells**.

> **Shell note:** `"${PWD}"` works in PowerShell, Linux and macOS. In Windows Command Prompt (cmd) use `"%cd%"` instead.

---

## 7. Docker Implementation

### 7.1 Custom Docker Image

The custom image is built from the official Jupyter SciPy Notebook image:

`quay.io/jupyter/scipy-notebook:latest`

The Dockerfile:

- Sets the working directory to `/home/jovyan/bd-a1`.
- Installs the Python packages from `requirements.txt`.
- Creates the project folders (`data/`, `notebook/`, `output/figures/`).
- Copies the dataset and the notebook into the image.
- Exposes port `8888`.
- Starts the Jupyter server when the container runs.

Build the image from the project root:

```bash
docker build -t bd-a1:latest .
```

### 7.2 Image vs. Container

A **Docker image** is a read-only, packaged template that contains the environment, dependencies and project files. It is made of layers (one per Dockerfile instruction) and does not run by itself.

A **Docker container** is a running instance created from an image. You can create many containers from one image.

Here, the image provides the Jupyter environment, and the container runs JupyterLab so the notebook can be executed.

### 7.3 Running the Container

```bash
docker run -d --name bd-a1-container -p 8888:8888 -v "${PWD}:/home/jovyan/bd-a1" bd-a1:latest
```

- `-d` runs the container in the background (detached mode).
- `--name bd-a1-container` gives the container a name.
- `-p 8888:8888` maps a host port to a container port (see 7.4).
- `-v "${PWD}:/home/jovyan/bd-a1"` bind-mounts the project folder (see 7.5).

### 7.4 Port Mapping

`-p 8888:8888` follows the format **HOST_PORT:CONTAINER_PORT**.

The first number is the port on your computer, the second is the port inside the container where JupyterLab listens. This lets you open JupyterLab from your browser at `http://localhost:8888`. If port 8888 is already in use on your computer, change only the first number, for example `-p 8890:8888`, and open `http://localhost:8890`.

### 7.5 Bind Mount and Persistent Output

```bash
-v "${PWD}:/home/jovyan/bd-a1"
```

This is a **bind mount**: it connects a folder on the host to a folder in the container, so both see the same files. Everything the notebook saves under `output/` (CSV files, metrics, figures) is written directly to the host, and stays there even if the container is stopped or removed. (A *volume*, in contrast, is storage managed by Docker itself and is not a normal folder on the host.)

### 7.6 Accessing JupyterLab

Open `http://localhost:8888`. If a token is requested, get it from the logs:

```bash
docker logs bd-a1-container
```

Use the URL (or token) shown in the logs.

---

## 8. Useful Docker Commands

| Purpose | Command |
|---|---|
| Build the image | `docker build -t bd-a1:latest .` |
| List images | `docker images` |
| Run the container | `docker run -d --name bd-a1-container -p 8888:8888 -v "${PWD}:/home/jovyan/bd-a1" bd-a1:latest` |
| List running containers | `docker ps` |
| List all containers | `docker ps -a` |
| View logs | `docker logs bd-a1-container` |
| Inspect the container | `docker inspect bd-a1-container` |
| Stop the container | `docker stop bd-a1-container` |
| Start it again | `docker start bd-a1-container` |
| Remove the container | `docker rm bd-a1-container` |

---

## 9. Output Files

The `output/` directory contains:

| File | Description |
|---|---|
| `cleaned_data.csv` | Dataset after cleaning and transformations |
| `cleaned_text.csv` | Tweets with the `cleaned_text` column and numerical text features |
| `model_metrics.txt` | Supervised model results: comparison, final metrics, confusion matrix, interpretation, limitations |
| `cluster_summary.txt` | K-Means results: cluster sizes, silhouette score, top terms, interpretation, comparison with labels |

The `output/figures/` directory contains:

| Figure | Description |
|---|---|
| `correlation_heatmap.png` | Correlation between numerical text features |
| `word_frequency_top20.png` | 20 most frequent words |
| `group_comparison_neg_vs_pos.png` | Negative vs. positive tweet comparison |
| `sampling_comparison.png` | Original vs. sampled sentiment distributions |
| `confusion_matrix.png` | Confusion matrix of the final classifier |
| `model_comparison.png` | Comparison of the classifiers |
| `kmeans_k_selection.png` | Elbow and silhouette experiment for choosing k |
| `cluster_vs_labels.png` | Real sentiment labels inside each cluster |

---

## 10. Docker Hub

**Docker Hub username:** `maysoun1106`
**Repository:** `maysoun1106/bd-a1`
**Tag:** `latest`
**URL:** https://hub.docker.com/r/maysoun1106/bd-a1

To tag and push the image (after `docker login`):

```bash
docker tag bd-a1:latest maysoun1106/bd-a1:latest
docker push maysoun1106/bd-a1:latest
```

To pull it:

```bash
docker pull maysoun1106/bd-a1:latest
```

---

## 11. Running the Published Docker Image

To reproduce the project from the published image (this was verified by a second team member):

1. Pull the image:

   ```bash
   docker pull maysoun1106/bd-a1:latest
   ```

2. Get the project folder (extract the submitted ZIP, or clone the GitHub repository) and open a terminal inside it.

3. Run the container:

   ```bash
   docker run -d --name bd-a1-container -p 8888:8888 -v "${PWD}:/home/jovyan/bd-a1" maysoun1106/bd-a1:latest
   ```

4. Open `http://localhost:8888` and, if needed, get the token with `docker logs bd-a1-container`.

**Note:** the project folder must exist on the host before running, because the bind mount places the local folder over the project folder inside the image.

---

## 12. Reproducibility

The Docker image contains the whole Python environment, so the analysis does not depend on the host machine.

To verify:

1. Start the container (Section 6 or 11).
2. Open `notebook/analysis.ipynb` in JupyterLab.
3. Select **Kernel -> Restart Kernel and Run All Cells**.
4. Wait until all cells finish without errors.
5. Check that the files listed in Section 9 are present in `output/` on the host.

All random steps (train/test split, sampling, K-Means, SVD) use `random_state=42`, so the results are the same on every run.

---

## 13. Screenshots

The `screenshots/` folder documents the Docker workflow and the execution of the project.

| Evidence | File |
|---|---|
| Successful `docker build` | `screenshots/01_docker_build.png` |
| `docker images` | `screenshots/02_docker_images.png` |
| `docker ps` (running container) | `screenshots/03_docker_ps.png` |
| JupyterLab with the notebook | `screenshots/04_jupyterlab.png` |
| `docker ps -a` | `screenshots/05_docker_ps_all.png` |
| `docker inspect` | `screenshots/06_docker_inspect.png` |

![Docker build](screenshots/01_docker_build.png)
![JupyterLab](screenshots/04_jupyterlab.png)

---

## 14. Team

| Member | Responsibility |
|---|---|
| Member 1 (Mysoun Ahmed) | Docker environment, README and Docker Hub |
| Member 2 (Malak Ayman) | Data understanding, cleaning and sampling |
| Member 3 (Doha waleed) | Text features, correlation and text analysis |
| Member 4 (Nada Hassan) | Machine learning: classification, K-Means and model improvement |


