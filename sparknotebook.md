## Creating a Spark/Notebook Environment

The purpose of this guide is to be able to run Spark Locally in a docker container and be able to write notebooks that use Spark from that container. Having docker installed is required.

## Starting the Container

Run the command

```
docker run -it --name spark-notebook -p 8888:8888 alexmerced/spark33playground
```

This command does the following
- `-it` starts the container in interactive mode which you can exit with the command `exit`
- `--name` this gives the container a name so you can easily turn it on and off with `docker start spark-notebook && docker attach spark-notebook` and `docker stop spark-notebook`
- `-p 8080:8080` maps port 8080 in the container to port 8080 in the host machine
- `alexmerced/spark33playground` a docker image that has Spark 3.3 running [Dockerfile used to create image](https://github.com/AlexMercedCoder/apache-iceberg-docker-starter-image/blob/main/SPARK33ICEBERGNESSIE.DOCKERFILE)

## Get the notebook server running

Once you are in the Docker containers shell we need to install jupyter notebook.

```
pip install jupyterlab pyspark
```

The normal command `jupyter-lab` won't work so we'll have to use the binary directly and pass it a flag to host the server on `0.0.0.0` so it is accessible outside of the container.

```
~/.local/bin/jupyter-notebook --ip 0.0.0.0
```

Now you have a notebook environment that should work
