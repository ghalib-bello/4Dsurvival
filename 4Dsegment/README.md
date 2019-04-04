# Deep learning cardiac motion analysis for human survival prediction (4D*survival*)

[4D*Segment*](https://github.com/UK-Digital-Heart-Project/4Dsegment) is the companion repo for 4D*survival*. It provides a pipeline for processing raw cardiac MRI data into 3D motion meshes that serve as the inputs to the 4D*survival* pipeline. It carries out segmentation (deep learning), non-rigid co-registration, mesh generation and motion tracking using raw grey-scale cardiac MRI data in NIfTI format. 

# Overview
Herein, we show how to run 4D*survival* on the output of 4D*Segment*. If 4D*Segment* runs successfully, there should be a `data` folder containing:
* `subjnames.txt` : a file containing IDs of all subjects whose raw MRI data was successfully processed by the 4D*Segment* pipeline
* subfolders labelled with subject IDs: these subfolders contain, among other things, mesh (point-wise) data for each subject
* `matchedpointsnew.txt` : contains mapping required for mesh-downsampling (4D*survival* downsamples meshes before feeding them into the survival prediction algorithm)

A CSV (comma-delimited) file containing survival outcomes should be copied into the `data` directory. This file should be named `surv_outcomes.csv` and should contain 3 labeled columns: 
* `ID`: subject ID (should match corresponding folder names in the `data` directory)
* `status` : dead/alive status of subject
* `time` : observation time for subject

Next, a Docker image should be downloaded to run the prediction pipeline. This is discussed below

## Installation/Usage Guide for Docker Image

### Install Docker
Running our 4D*survival* Docker image requires installation of the Docker software, instructions are available at https://docs.docker.com/install/ 

### Download 4D*survival* Docker image
Once the Docker software has been installed, our 4D*survival* Docker image can be pulled from the Docker hub using the following command:
    
    docker pull ghalibbello/4dsurvival_new:latest

Once the image download is complete, open up a command-line terminal. On Windows operating systems, this would be the *Command Prompt* (cmd.exe), accessible by opening the [Run Command utility](https://en.wikipedia.org/wiki/Run_command) using the shortcut key `Win`+`R` and then typing `cmd`. On Mac OS, the terminal is accessible via (Finder > Applications > Utilities > Terminal). On Linux systems, any terminal can be used.
Once a terminal is open, running the following command:

    docker images

should show `ghalibbello/4dsurvival_new` on the list of images on your local system

### Run 4D*survival* Docker image
We will run the docker image and mount the `data` folder produced after running 4D*Segment* :
    
    docker run -it --rm -v <4DSEgment-folder-path>/data/:/4Dsegment ghalibbello/4dsurvival_new:latest /bin/bash

The launches an interactive linux shell terminal that gives users access to the image's internal file system, and mounts the aforementioned `data` directory from the local host to the `/4Dsegment` directory within the 4D*survival* docker image. 

Typing 
```
ls -l
```
will list the contents of the `/4Dsegment`, showing the mount was successful. 
Next, type the following commands:

```
cd /4DSurv
ls -l
```

This will list all the folders in the working directory of the Docker image (/4DSurv). You should see the 4 main folders `code`, `data`, `demo` an `setup`.

The first step would be to convert the output of 4D*Segment* into a format that can be fed into 4D*survival*. To do this, navigate to the `setup` directory by typing:
```
cd setup
ls -l
```
This should list one file: `inputdata_setup.py`. Now, run this file:
```
python3 inputdata_setup.py /4Dsegment
```

If all goes well, the 4D*segment* output will be transformed into a format that is ready to be fed into the 4D*Survival* prediction pipeline. 


Now we will demonstrate how to perform the following analyses:
- [x] Train and validate deep learning network

#### Train & validate deep learning network
From the 4dSurv directory, navigate to the `demo` directory by typing:
```
cd /4DSurv/demo
ls -l
```
The `demo_validateDL.py` file should be visible. This code (which uses as input the mesh data we just processed) can now be run (WARNING: on most machines, this will take several hours to run):
```
python3 demo_validateDL.py
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



 
