# Deep learning cardiac motion analysis for human survival prediction (4D*survival*)

[4D*Segment*](https://github.com/UK-Digital-Heart-Project/4Dsegment) is the companion repo for 4D*survival*. It provides a pipeline for processing raw cardiac MRI data into 3D motion meshes that serve as the inputs to the 4D*survival* pipeline. 
If you have your raw cardiac MRI scan data in the form of grey-scale images (in NIfTI format), 4D*Segment* can carry out segmentation (deep learning -based) of these images, non-rigid co-registration, mesh generation and motion tracking. 

# Overview
Herein, we show how to run 4D*survival* on the output of 4D*Segment*. If 4D*Segment* runs successfully, there should be a `data` folder containing:
* `subjnames.txt` : a file containing IDs of all subjects whose raw MRI data was successfully processed by the 4D*Segment* pipeline
* subfolders labelled with subject IDs: these subfolders contain, among other things, mesh (point-wise) data for each subject. As the [4D*Segment*](https://github.com/UK-Digital-Heart-Project/4Dsegment) instructions indicate, the key output of the pipeline is point-wise 3D mesh data representing positions of points on the heart throughout the cardiac cycle. For each subject, this data is stored within the `motion` subfolder. This data summarizes 20 computational meshes (both vtk and txt files) for a complete cardiac cycle (20 frames). In each of 20 meshes, only spatial locations of vertices are recorded. Vertex spatial position (*x*, *y* and *z*) on the same row in the txt files corresponds to the same anatomical location across the cardiac cycle. **This vertex spatial position data (across 20 frames) is the input to the 4D*Survival* pipeline** (note that we down-sample this data [see below] before feeding it into 4D*Survival*).
* `matchedpointsnew.txt` : contains mapping required for mesh-downsampling (4D*survival* downsamples meshes before feeding them into the survival prediction algorithm)

If the components listed above are all present in the 4D*Segment* output `data` directory, then the next step is to add survival outcome data to this directory. A CSV (comma-delimited) file (which should be named `surv_outcomes.csv`) containing survival outcomes should be copied into the `data` directory. This file **must** contain 3 labeled columns (see [a sample file](sample_files/surv_outcomes.csv)): 
* `ID`: subject ID (should match corresponding folder names in the `data` directory)
* `status` : dead/alive status of subject at the end of observation period (dead/alive status should be coded as integers 1/0 respectively, i.e. 1: dead, 0: alive)
* `time` : length of observation period for subject (in days)

Next, a Docker image should be downloaded to run the prediction pipeline. This is discussed below

## Installation/Usage Guide for Docker Image

### Install Docker
Running our 4D*survival* Docker image requires installation of the Docker software, instructions are available at https://docs.docker.com/install/ 

### Download 4D*survival* Docker image
Once the Docker software has been installed, our 4D*survival* Docker image can be pulled from the Docker hub using the following command:
    
    docker pull ghalibbello/4dsurvival_ext:latest

Once the image download is complete, open up a command-line terminal. On Windows operating systems, this would be the *Command Prompt* (cmd.exe), accessible by opening the [Run Command utility](https://en.wikipedia.org/wiki/Run_command) using the shortcut key `Win`+`R` and then typing `cmd`. On Mac OS, the terminal is accessible via (Finder > Applications > Utilities > Terminal). On Linux systems, any terminal can be used.
Once a terminal is open, running the following command:

    docker images

should show `ghalibbello/4dsurvival_ext` on the list of images on your local system

### Run 4D*survival* Docker image
We will run the docker image and mount the `data` folder produced after running 4D*Segment* :
    
    docker run -it --rm -v <4DSegment-folder-path>/:/4Dsegment_output -v <empty-results-folder>/:/4DSurvival_results ghalibbello/4dsurvival_ext:latest /bin/bash

In the above command, `<4DSegment-folder-path>` is simply a placeholder for the directory on your local machine where the 4D*Segment* `data` folder resides. And `<empty-results-folder>` is a placeholder for an empty directory on your local machine where you would like the results of 4D*Survival* (risk score predictions, saved DL models, Kaplan-Meier plot images, text files summarizing validation results) to be saved.
Running the above command launches an interactive linux shell terminal that gives users access to the Docker image's internal file system, and also mounts the local folders `<4DSEgment-folder-path>` and `<empty-results-folder>` onto the `/4Dsegment_output` and `/4DSurvival_results` directories within the 4D*survival* docker image. 

Typing 
```
ls -l /4Dsegment_output
```
should list the contents of the `/4Dsegment_output`, showing the mount was successful. 

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
python3 inputdata_setup.py /4Dsegment_output
```

If all goes well, the 4D*segment* output will be transformed into a format that is ready to be fed into the 4D*Survival* prediction pipeline. This format takes the form of an input file written to `/4DSurv/data/inputdata_DL.pkl`. Now that we have this file, we can run the DL prediction pipeline (training/validation, KM plot generation, etc.).


Now we will demonstrate how to perform the following analyses:
- [x] Train and validate deep learning network
- [x] Generate KM plot for deep learning network predictions

#### Train & validate deep learning network
Navigate to the `demo` directory by typing:
```
cd /4DSurv/demo
ls -l
```
The `demo_validateDL.py` file should be visible. This code (which uses as input the mesh data we just processed) can now be run (WARNING: on most machines, this will take several hours to run):
```
python3 demo_validateDL.py
```
This code will run a bootstrap validation of the DL model, and save the final model (run on the full training sample). The model will be saved in the `/4DSurvival_results` shared/mounted directory, as an [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) file named `saved_model__DL.h5` 

 
 
#### Generate KM plot for deep learning network predictions
To generate a KM Plot for the DL network, the validation step above (running `demo_validateDL.py`) must have completed successfully. This is because the output of `demo_validateDL.py` is required for KM plot generation. 
Navigate to the `demo` directory by typing:
```
cd /4DSurv/demo
ls -l
```
The `demo_KMplotDL.py` file should be visible. Generate KM plots by typing:
```
python3 demo_KMplotDL.py
```
This code will generate a KM plot saved in the `/4DSurvival_results` shared/mounted directory, as a PNG file named `RESULTS_demo_KMplot_DL.png` 



### Features to be introduced soon..
- [x] DL training on GPU
- [x] Generating predictions with saved models
- [x] Incorporation of covariate data
