# Deep learning cardiac motion analysis for human survival prediction (4D*survival*)

[https://github.com/UK-Digital-Heart-Project/4Dsegment](4D*Segment*) is a companion repo for 4D*survival*. It provides a pipeline for processing raw cardiac MRI data into 3D motion meshes that serve as the inputs to the 4D*survival* pipeline. It carries out segmentation (deep learning), non-rigid co-registration, mesh generation and motion tracking using raw grey-scale cardiac MRI data in NIfTI format. 

# Overview
Herein, we show how to run 4D*survival* on the output of 4D*Segment*.
The files in this repository are organized into 3 directories:
* [code](code) : contains base functions for fitting the 2 types of statistical models used in our paper: 4D*survival* (supervised denoising autoencoder for survival outcomes) and a penalized Cox Proportional Hazards regression model.
* [demo](demo) : contains functions for the statistical analyses carried out in our paper:
  * Training of DL model - [demo/demo_hypersearchDL.py](demo/demo_hypersearchDL.py)
  * Generation of Kaplan-Meier plots - [demo/demo_KMplot.py](demo/demo_KMplot.py)
  * statistical comparison of model performance - [demo/demo_modelcomp_pvalue.py](demo/demo_modelcomp_pvalue.py)
  * Bootstrap internal validation - [demo/demo_validate.py](demo/demo_validate.py)
* [data](data) : contains simulated data on which functions from the `demo` directory can be run.

To run the code in the [demo](demo) directory, we provide a [Binder](https://mybinder.org/) interface (for the Jupyter notebooks) and a Docker container (for the corresponding Python scripts). Below are usage instructions:

## 1. Running Jupyter notebooks via Binder

The Jupyter notebooks in the [demo](demo) directory are hosted on Binder, which provides an interactive user interface for executing Jupyter notebooks. Click the link provided below for access:

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/UK-Digital-Heart-Project/4DSurv/master)


## 2. Installation/Usage Guide for Docker Image

A Docker image is available for running the code available in the [demo](demo) directory. This image contains a base Ubuntu linux operating system image set up with all the libraries required to run the code (e.g. *Tensorflow*, *Keras*, *Optunity*, etc.). The image contains all the code, as well as simulated data on which the code can be run. 

### Install Docker
Running our 4D*survival* Docker image requires installation of the Docker software, instructions are available at https://docs.docker.com/install/ 

### Download 4D*survival* Docker image
Once the Docker software has been installed, our 4D*survival* Docker image can be pulled from the Docker hub using the following command:
    
    docker pull ghalibbello/4dsurvival:latest

Once the image download is complete, open up a command-line terminal. On Windows operating systems, this would be the *Command Prompt* (cmd.exe), accessible by opening the [Run Command utility](https://en.wikipedia.org/wiki/Run_command) using the shortcut key `Win`+`R` and then typing `cmd`. On Mac OS, the terminal is accessible via (Finder > Applications > Utilities > Terminal). On Linux systems, any terminal can be used.
Once a terminal is open, running the following command:

    docker images

should show `ghalibbello/4dsurvival` on the list of images on your local system

### Run 4D*survival* Docker image
    
    docker run -it ghalibbello/4dsurvival:latest /bin/bash

launches an interactive linux shell terminal that gives users access to the image's internal file system. This file system contains all the code in this repository, along with simulated data on which the code can be run.
Typing 
```
ls -l
```
will list all the folders in the working directory of the Docker image (/4DSurv). You should see the 3 main folders `code`, `data` and `demo`, which contain the same files as the corresponding folders with the same name in this github repository.

Below we will demonstrate how to perform (within the Docker image) the following analyses:
- [x] Train deep learning network
- [x] Train and validate conventional parameter model

#### Train deep learning network
From the 4dSurv directory, navigate to the `demo` directory by typing:
```
cd demo
ls -l
```
The `demo_hypersearchDL.py` file should be visible. This executes a hyperparameter search (see Methods section in paper) for training of the `4Dsurvival` deep learning network. A demo of this code (which uses simulated input data) can now be run (WARNING: on most machines, this will take several hours to run):
```
python3 demo_hypersearchDL.py
```

Also under the `demo` folder, the `demo_validate.py` file should be visible. This executes the bootstrap-based approach for training and internal validation of the Cox Proportional Hazards model for conventional (volumetric) parameters. A demo of this code (which uses simulated input data) can now be run :
```
python3 demo_validate.py
```

## Citation

Bello GA,  Dawes TJW, Duan J, Biffi C, de Marvao A, Howard LSGE, Gibbs JSR, Wilkins MR, Cook SA, Rueckert D, O'Regan DP. Deep learning cardiac motion analysis for human survival prediction. [*arXiv: 0000.00000*](https://arxiv.org/abs/0000.00000), 2018.

```
@article{Imperial4DSurvival,
  title = {Deep learning cardiac motion analysis for human survival prediction},
  author = {Bello, GA and Dawes, TJW and Duan, Jinming and Biffi, C and de Marvao, A and Howard, LSGE and Gibbs, JSR and Wilkins, MR and Cook, SA and Rueckert, D and O'Regan, DP},
  year = {2018},
  url = {https://arxiv.org/abs/XXXX.XXXXX},
}
```



 
