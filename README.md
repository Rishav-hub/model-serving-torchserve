<div align="center">

# Model Serving Torchserve

<a href="https://pytorch.org/get-started/locally/"><img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-ee4c2c?logo=pytorch&logoColor=white"></a>
<a href="https://pytorchlightning.ai/"><img alt="Lightning" src="https://img.shields.io/badge/-Lightning-792ee5?logo=pytorchlightning&logoColor=white"></a>
<a href="https://hydra.cc/"><img alt="Config: Hydra" src="https://img.shields.io/badge/Config-Hydra-89b8cd"></a>
<a href="https://github.com/ashleve/lightning-hydra-template"><img alt="Template" src="https://img.shields.io/badge/-Lightning--Hydra--Template-017F2F?style=flat&logo=github&labelColor=gray"></a><br>
[![Paper](http://img.shields.io/badge/paper-arxiv.1001.2234-B31B1B.svg)](https://www.nature.com/articles/nature14539)
[![Conference](http://img.shields.io/badge/AnyConference-year-4b44ce.svg)](https://papers.nips.cc/paper/2020)

</div>

## Description

This is a basic computer vision template which has model serving using Torchserve

## How to run

Install dependencies

```bash
# clone project
git clone https://github.com/Rishav-hub/model-serving-torchserve
cd your-repo-name

# [OPTIONAL] create conda environment
conda create -p ./env python=3.9 -y
conda activate ./env

# install pytorch according to instructions
# https://pytorch.org/get-started/

# install requirements
pip install -r requirements.txt
```

Train model with default configuration

```bash
# train on CPU
python src/train.py trainer=cpu

# train on GPU
python src/train.py trainer=gpu
```

## Model Serving

### Windows 

- You need to have `openjdk17`
- Refer [here](https://pytorch.org/serve/torchserve_on_win_native.html)

```bash

# Using pip
pip install torchserve torch-model-archiver torch-workflow-archiver

# Using conda
conda install -c pytorch torchserve torch-model-archiver torch-workflow-archiver

```

### Using Docker

- Refer [here](https://hub.docker.com/r/pytorch/torchserve)
- All the steps needs to be done once the docker container is running.

```bash
docker run -it --rm --net=host -v `pwd`:/opt/src pytorch/torchserve:latest bash
```

### Steps to serve model

1. First train and save the jit scripted model

```bash
python src/train.py experiment=example
```

2. Inside `src/torch_handlers/` you have to add model handler. There are some default handlers [here](https://pytorch.org/serve/default_handlers.html). Handler is just a python class that has some operations like model initialization and data preprocessing steps. To build custom handler refer [here](https://github.com/pytorch/serve/blob/master/ts/torch_handler/base_handler.py)

3. You can add some extra file like model configurations and label mapping inside `src/torch_handlers/mnist_classes`. 

4. Create `.mar` file or model archival file which contains your code, models and some extra files that you have to add.

The paths needs to be changed according to individuals system
```bash
torch-model-archiver --model-name mnist_basic --version 1.0 --serialized-file /home/ubuntu/lightning-hydra-template/logs/train/runs/2022-10-24_17-36-21/model.script.pt --handler /home/ubuntu/lightning-hydra-template/src/torch_handlers/mnist_handler.py --extra-files /home/ubuntu/lightning-hydra-template/src/torch_handlers/mnist_classes/index_to_name.json
```

5. Star torchserve server. If you are using torchserve docker then first run the container in executable mode, attach voulmes and then start the server.

```bash
docker run -it --rm --net=host -v `pwd`:/opt/src pytorch/torchserve:latest bash

cd /opt/src

torchserve --start --model-store model_store --models mnist=mnist_basic.mar

torchserve --stop
```
6. Inference API

Health check
```bash
curl http://localhost:8080/ping
```

Take test images from: https://drive.google.com/drive/folders/15wU9RJ05uX9ft8WRhNLvQXNTt6inoVDS?usp=sharing

```bash
curl http://127.0.0.1:8080/predictions/densenet161 -T 0.png
```
7. Management API

List all the models
```bash
curl "http://localhost:8081/models"
```
Dynamic Hot Swapping Model + Code

Since the model + code is stored in `.mar` file, you can register a different model version with the new code and then set the default model serving to the newer version !

```bash
curl -v -X PUT http://localhost:8081/models/mnist/2.0/set-default
```

8. Model Metrics
```bash
curl http://127.0.0.1:8082/metrics
```














