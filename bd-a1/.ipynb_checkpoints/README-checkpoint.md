# CSCI461 Big Data Assignment #1

## 1. Project Overview

This project uses the **Twitter US Airline Sentiment Dataset** to perform data analysis, text preprocessing, and machine learning.

The project is implemented in a Dockerized JupyterLab environment to provide a consistent and reproducible execution environment.

## 2. Dataset

- **Dataset Name:** Twitter US Airline Sentiment
- **File:** `data/Tweets.csv`
- **Number of Rows:** 14,640
- **Number of Columns:** 15
- **Format:** CSV

## 3. Project Structure

```text
bd-a1/
├── Dockerfile
├── requirements.txt
├── README.md
├── data/
│   └── Tweets.csv
├── notebook/
│   └── analysis.ipynb
├── output/
│   ├── cleaned_data.csv
│   ├── cleaned_text.csv
│   └── figures/
└── screenshots/
```

Additional output files generated during the modeling and clustering stages will be saved in the `output/` directory.

## 4. Requirements

The following software is required:

- Docker Desktop
- A running Docker Engine
- Git (optional)

Python dependencies are listed in `requirements.txt` and installed automatically during the Docker image build.

## 5. Docker Implementation

### 5.1 Docker Image

A Docker image is a packaged environment containing the required dependencies, project files, and configuration needed to run the application.

The project image is built using:

```powershell
docker build -t bd-a1 .
```

This command builds the image using the Dockerfile in the project root directory and tags it as `bd-a1:latest`.

### 5.2 Docker Container

A Docker container is a running instance of a Docker image.

To create and start the container, run the following command from the project root directory:

```powershell
docker run -d --name bd-a1-container -p 8888:8888 -v "${PWD}:/home/jovyan/bd-a1" bd-a1
```

**Port Mapping:**

The `-p 8888:8888` option maps the host machine's port 8888 to port 8888 inside the container, allowing access to JupyterLab through a web browser.

**Volume Mounting:**

The `-v "${PWD}:/home/jovyan/bd-a1"` option mounts the project directory on the host machine into the container.

This allows changes made to project files inside or outside the container to be reflected in both locations. Generated output files are therefore preserved in the local project directory.

### 5.3 Accessing JupyterLab

After starting the container, open:

http://localhost:8888

To retrieve the JupyterLab access URL and token, run:

```powershell
docker logs bd-a1-container
```

### 5.4 Restarting an Existing Container

If the container has already been created and stopped, start it again using:

```powershell
docker start bd-a1-container
```

Check the container status using:

```powershell
docker ps
```

## 6. Useful Docker Commands

The following commands are used to manage and inspect the Docker environment:

| Command | Description |
|---|---|
| `docker images` | Lists available Docker images. |
| `docker ps` | Lists currently running containers. |
| `docker ps -a` | Lists all containers, including stopped ones. |
| `docker logs bd-a1-container` | Displays container logs. |
| `docker inspect bd-a1-container` | Displays detailed container configuration, including port mappings and mounts. |
| `docker stop bd-a1-container` | Stops the running container. |
| `docker start bd-a1-container` | Starts an existing stopped container. |

To remove the container when it is no longer needed:

```powershell
docker stop bd-a1-container
docker rm bd-a1-container
```

## 7. Output Files

Generated files are stored in the `output/` directory.

Current output files include:

- `cleaned_data.csv` — cleaned dataset.
- `cleaned_text.csv` — preprocessed text data.
- `figures/` — directory for generated visualizations.

Additional files containing model evaluation metrics and clustering summaries will be included after completing the corresponding analysis stages.

## 8. Docker Hub

The Docker image is tagged for Docker Hub using:

```powershell
docker tag bd-a1:latest maysoun1106/bd-a1:latest
```

To upload the image to Docker Hub:

```powershell
docker login
docker push maysoun1106/bd-a1:latest
```

**Docker Hub Repository:** `maysoun1106/bd-a1`

**Image Tag:** `latest`

The image upload must complete successfully before considering the Docker Hub deployment finalized.

## 9. Pulling the Docker Image from Docker Hub

To download the project image from Docker Hub without building it locally, run:

```powershell
docker pull maysoun1106/bd-a1:latest
```

After downloading the image, extract the project ZIP file and navigate to the project root directory.

Create and start a container using:

```powershell
docker run -d --name bd-a1-container -p 8888:8888 -v "${PWD}:/home/jovyan/bd-a1" maysoun1106/bd-a1:latest
```

Open JupyterLab in your browser:

http://localhost:8888

**Note:** Run the command from the project root directory so that the volume mount points to the correct project folder. If a container with the same name already exists, start the existing container or use a different container name.

## 10. Reproducibility

To reproduce the project:

1. Obtain the project files and dataset.
2. Build the Docker image locally or pull it from Docker Hub.
3. Start the Docker container using the provided command.
4. Open JupyterLab through `http://localhost:8888`.
5. Open `notebook/analysis.ipynb`.
6. Select **Kernel → Restart Kernel and Run All Cells**.
7. Verify that all notebook cells execute successfully without errors.
8. Verify that generated files are saved in the mounted `output/` directory.

The project is tested inside the Dockerized JupyterLab environment to ensure consistent execution and reproducibility.

## 11. Screenshots

Screenshots documenting the Docker setup, image creation, container execution, JupyterLab environment, and project configuration are provided in the `screenshots/` directory.