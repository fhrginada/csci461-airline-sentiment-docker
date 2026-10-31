
FROM quay.io/jupyter/scipy-notebook:latest

WORKDIR /home/jovyan/bd-a1

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /home/jovyan/bd-a1/data \
    /home/jovyan/bd-a1/notebook \
    /home/jovyan/bd-a1/output/figures

COPY data/ /home/jovyan/bd-a1/data/
COPY notebook/ /home/jovyan/bd-a1/notebook/

EXPOSE 8888

CMD ["start-notebook.py", "--ServerApp.root_dir=/home/jovyan/bd-a1"]
