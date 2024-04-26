# Quick Start

## Acknowledgements

We refer to the code implementation of [lib-city](https://bigscity-libcity-docs.readthedocs.io/en/latest/get_started/quick_start.html)

```latex
@inproceedings{libcity,
  author = {Wang, Jingyuan and Jiang, Jiawei and Jiang, Wenjun and Li, Chao and Zhao, Wayne Xin},
  title = {LibCity: An Open Library for Traffic Prediction},
  year = {2021},
  isbn = {9781450386647},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3474717.3483923},
  doi = {10.1145/3474717.3483923},
  booktitle = {Proceedings of the 29th International Conference on Advances in Geographic Information Systems},
  pages = {145–148},
  numpages = {4},
  keywords = {Spatial-temporal System, Reproducibility, Traffic Prediction},
  location = {Beijing, China},
  series = {SIGSPATIAL '21}
}
```


## 1. Download dataset

You can create a new folder "raw_data" under the root path and download a dataset from the collection [libcity](https://bigscity-libcity-docs.readthedocs.io/en/latest/tutorial/install_quick_start.html#download-one-dataset) under the new path.

Then simply add the mapping matrix "XXX.mor.py" into the folder of a dataset e.g. ,  $ROOT_PATH/raw_data/METR_LA/METR_LA.mor.py. 



## 2. Execution

Under the root path with the run_model.py, the program should be executed properly.

```bash
python3 ./run_model.py --task traffic_state_pred --model HIEST --config configs --dataset METR_LA
```



## 3. Model

Our model is under the path of ./code/HIEST.py.
We also provide an implementation of a Traffic-Transformer under the [guide of the lib-city](https://bigscity-libcity-docs.readthedocs.io/en/latest/developer_guide/implemented_models.html) 

These two models are for 'traffic-state-prediction', you can add them into the pipeline under the [instructions]((https://bigscity-libcity-docs.readthedocs.io/en/latest/developer_guide/implemented_models.html) ) provided by lib-city.

## 4. Processed Data

For the attributes self.adj_mx and self.Mor, they will be initialized with the processed adjacency matrix and mapping matrix.

## 5. The utils for solving BCC

The utils for solving BCC are under the path of ./utils .

## 6. The visualization code

Our visualization result is implemented by the [QGIS](https://qgis.org/en/site/).

The visualization code is used to generate the Geo_JSON file to be imported into the QGIS.

We generate a .json file for each regional/global node.

Then you can import them as follows:

![image-2](./README.assets/2.png)

You can also search and install the QuickMap services to add the base map.

![image-3](./README.assets/3.png)

## 7. Running environment


We implement the customized environment with [singularity](https://docs.sylabs.io/guides/3.7/user-guide/index.html) image for better execution.

If you are using Docker, the key idea should be similar with our implementation.

The singularity official documentation will provide the quick start-up with installation steps.

*All of the following scripts are executed on the **root path** of lib-city!*

### 7.1 Base image

As we refer to the implementation of the lib-city, we follow their basic pytorch major version of 1.7.1 with cuda11.0.

A good practice is to use a dev version of the PyTorch base image from the official docker registry

```sh
# https://hub.docker.com/layers/pytorch/pytorch/1.7.1-cuda11.0-cudnn8-devel/images/sha256-f0d0c1b5d4e170b4d2548d64026755421f8c0df185af2c4679085a7edc34d150?context=explore
singularity pull docker://pytorch/pytorch:1.7.1-cuda11.0-cudnn8-devel
```

If everything goes well, you will see the following INFO when pulling the base image

![image-0](./README.assets/0.png)

Once the downloading is done, you will get a **SIF image** with the suffix **.sif(like pytorch_1.7.1-cuda11.0-cudnn8-devel.sif)** on your local machine. This will be used as a base image in the following steps.

### 7.2 Install Requirements

1. Create a definition file(named HIEST.def) as follows,

```sh
#Bootstrap is used to specify the agent,where the base image from,here localimage means to build from a local image
Bootstrap: localimage
## This is something like 'From' in DOCKERFILE to indicate the base image
From: ./pytorch_1.7.1-cuda11.0-cudnn8-devel.sif

# %files can be used to copy files from host into the image
# like 'COPY' in DOCKERFILE
# Here we copy the requirements.txt into the image, then we can use it to install the required dependencies.
%files
    requirements.txt /opt

# %post is used to build the new image
# Usage is same to shell.Here we used pip to install dependencies.
%post
    pip install -r /opt/requirements.txt
    pip install protobuf==3.20.0 #to solve some warning we met
 
#% environment is used to set env_variables once the image starts
# These lines are necessary to load cuda
%environment
    export PATH=$PATH:/usr/local/cuda-11.0/bin
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-11.0/lib64:/usr/lib/x86_64-linux-gnu
```

2. Now execute the following command to build the image

```sh
## still on the root path
singularity build HIEST.sif HIEST.def
```

You will see the following INFO when building the new image

![image-1](./README.assets/1.png)

If nothing is wrong after creating SIF file, then you will get the image file **HIEST.sif** on the root path.

### 7.3 Execution

Now the environment is ready, and all the code should be able to execute properly now.

Here are the slurm script and command for reference

```shell
# pwd
# this is the key command,remember to add the '--nv' option
singularity exec --nv HIEST.sif python3 ./run_model.py --task traffic_state_pred
--model HIEST --dataset METR_LA
```

