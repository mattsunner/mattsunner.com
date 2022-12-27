+++ 
draft = false
date = 2022-12-27T08:54:46-06:00
title = "Creating a Data Science & EDA Template with Docker"
description = "Overview of the Docker EDA template used for data science projects"
slug = "docker-eda-template"
authors = ["Matthew Sunner"]
tags = ["Docker", "Python", "Data Science"]
categories = []
externalLink = ""
series = []
+++

### Introduction and Overview of Docker

Data science and analytics related projects all have one thing in common: The need for reproducibility of analysis and results. This is often associated to the environment in which analysis is done. I have built a simple Docker based template for reproducing analytics environments in an easy to do manner. The template and source code can be found: [here](https://github.com/mattsunner/ds-container)).

Managing Python versions is perhaps the most integral step in reproducibility and resolving the “it works on my machine” issue. The Python community has a variety of tools available specifically for dealing with Python versions, including manual installs, Anaconda and pyenv. The latter is a command line tool that specializes in global and local version management, allowing for local development using any version of Python. Anaconda, which is the industry standard for data science in the Python community is both a distribution of Python, as well as an integrated package management system with a focus on scientific computing. Docker is also a popular option for software development and excels at version management, since the configuration information is all stored within project files.

Docker is a containerization software as a service that allows for development and deployment of software in explicitly defined containers. These can be thought of as stripped down VMs, but for our instance, this container will be the analysis environment. It will be completely isolated from other analysis projects, as well as from the machine it is developed on. This allows for the container to be run on any computer or server with the same results and same pinned dependencies working properly. 

### Building the Environment

The basic building blocks for this template’s development environment consist of two documents: a Dockerfile and a docker-compose.yml file. These two files are read by the Docker daemon and provide the architecture and install steps for the environment. In order to follow along with this post, be sure to install Docker on your local machine. Docker provides excellent documentation on the install process that can be found on Docker’s [website](https://www.docker.com/). Below is the Dockerfile used to build the development environment:

```docker
FROM continuumio/miniconda3:4.10.3p1

WORKDIR /app

RUN conda install \
    xarray \
    netCDF4 \
    bottleneck \
    numpy \
    pandas \
    matplotlib \
    jupyterlab \
    scikit-learn \
    seaborn  

COPY . /app/

CMD ["jupyter-lab","--ip=0.0.0.0","--no-browser","--allow-root"]
```

The first line in this file sets up the base image to use to create the environment. Docker has a variety of signed images that can be selected from in nearly all programming languages and use cases. Using a pre-set image allows for less boilerplate and manual install. For instance, the continuumio/miniconda3 image in the above Dockerfile already has an Anaconda environment installed, removing the need to manually install one. 

The second line sets the working directory in the container (which is separate from the local machine’s filesystem). Next, the `RUN` command executes a command line command. In this instance, it loads in the listed dependencies into the Anaconda environment. These will be uniform across runs. If there are additional libraries needed for analysis, for instance, PyTorch for deep learning, they are added to the list in the `RUN` step. 

Next, the `COPY` command copies the contents of the local machine folder into the `/app` folder in the container. This is what gives access to the data files and notebook files that are created. Lastly, the `CMD` sets the command to run Jupyter and launch the environment to create notebooks. This Dockerfile is as minimal as possible to get the development environment up and running. 

The next required file for the setup of this system is the `docker-compose.yml` file. 

```docker
version: "3"
services:
  jupyter:
    build: .
    ports:
      - 8888:8888
    volumes:
      - base_volume:/project
volumes:
  base_volume:
    external: true
```

This file works with the `docker compose` command that comes with Docker. The purpose of using Docker Compose is to allow a level of orchestration to occur between docker containers. In the current version of this template, there is only one container, “jupyter”, being used. However, I wanted to make sure the infrastructure was in place if I ever need to add in a database container or additional worker containers. Without going into too much detail above, the Docker Compose YAML file above sets the version, lists out the services, just “jupyter” in this case, and assigns the volumes needed for data persistency and connection to the local file system within the cloned repository. This allows for the user to simply run the command `docker compose up` and Docker will handle the build and launch steps to access the container. This file also establishes the persistent volume to be used, which will allow for the container to have persistent storage and connection to the local file system.

### Using the System

To use this template system, I have included detailed instructions in the [Repository Documentation](https://github.com/mattsunner/ds-container) on GitHub. At a high level, this template repository just needs to be activated, then cloned down to a local machine. Once there, ensure all dependencies are added to the list in the `Dockerfile`, then run `docker compose up`. This will pull the container from Docker Hub, install the dependencies, and provide a link to copy into your web browser. Once there, the environment operates like a normal Jupyter Notebook. 

This provides a simple to get started with environment that does not require the installation of Anaconda or any other libraries on your local machine. All that is required is Docker and a brief understanding of how to download and use templates on GitHub. This has been my go-to starting point for any EDA or notebook based projects for a few weeks now. There are a few updates tot the system that I plan to make in the future. These include additional standard libraries to include, as well as adding in additional containers to the `docker-compose.yml` file to handle database connections for testing and EDA.
