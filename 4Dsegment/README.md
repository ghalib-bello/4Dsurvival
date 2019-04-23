# Deep learning cardiac motion analysis for human survival prediction (4D*survival*)

[4D*Segment*](https://github.com/UK-Digital-Heart-Project/4Dsegment) is the companion repo for 4D*survival*. It provides a pipeline for processing raw cardiac MRI data into 3D motion meshes that serve as the inputs to the 4D*survival* pipeline. 
If you have your raw cardiac MRI scan data in the form of grey-scale images (in NIfTI format), 4D*Segment* can carry out segmentation (deep learning -based) of these images, non-rigid co-registration, mesh generation and motion tracking. 

# Overview
Herein, we show how to run 4D*survival* on the output of 4D*Segment*. If 4D*Segment* runs successfully, there should be a `data` folder containing:
* `subjnames.txt` : a file containing IDs of all subjects whose raw MRI data was successfully processed by the 4D*Segment* pipeline
* subfolders labelled with subject IDs: these subfolders contain, among other things, mesh (point-wise) data for each subject. As the [4D*Segment*](https://github.com/UK-Digital-Heart-Project/4Dsegment) instructions indicate, the key output of the pipeline is point-wise 3D mesh data representing positions of points on the heart throughout the cardiac cycle. For each subject, this data is stored within the `motion` subfolder. This data summarizes 20 computational meshes (both vtk and txt files) for a complete cardiac cycle (20 frames). In each of 20 meshes, only spatial locations of vertices are recorded. Vertex spatial position (*x*, *y* and *z*) on the same row in the txt files corresponds to the same anatomical location across the cardiac cycle. **This vertex spatial position data (across 20 frames) is the input to the 4D*Survival* pipeline** (note that we down-sample this data [see below] before feeding it into 4D*Survival*).
* `matchedpointsnew.txt` : contains mapping required for mesh-downsampling (4D*survival* downsamples meshes before feeding them into the survival prediction algorithm). This file should contain 2 columns of text, the first listing indices of vertices in the down-sampled mesh, and the second column listing the indices of corresponding closest vertices in the full mesh (see [a sample file](sample_files/matchedpointsnew.txt)).

If the components listed above are all present in the 4D*Segment* output `data` directory, then the next step is to add survival outcome data to this directory. A CSV (comma-delimited) file (which should be named `surv_outcomes.csv`) containing survival outcomes should be copied into the `data` directory. This file **must** contain 3 labeled columns in ***exactly*** the following order (see [a sample file](sample_files/surv_outcomes.csv)): 
* `ID`: subject ID (should match corresponding folder names in the `data` directory)
* `status` : dead/alive status of subject at the end of observation period (dead/alive status should be coded as integers 1/0 respectively, i.e. 1: dead, 0: alive)
* `time` : length of observation period for subject (in days)

**NOTE**: The columns must be in the order `ID`, `status`, `time`

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

This will list all the folders in the working directory of the Docker image (`/4DSurv`). You should see the 4 main folders `code`, `data`, `demo` an `setup`.

The first step would be to convert the output of 4D*Segment* into a format that can be fed into 4D*survival*. To do this, navigate to the `setup` directory by typing:
```
cd setup
ls -l
```
This should list one file: `inputdata_setup.py`. Now, run this file:
```
python3 inputdata_setup.py /4Dsegment_output
```

If all goes well, the 4D*segment* output will be transformed into a format that is ready to be fed into the 4D*Survival* prediction pipeline - this format takes the form of an input file written to `/4DSurv/data/inputdata_DL.pkl` (check the `/4DSurv/data` directory to make sure the file is there). Now that we have this file, we can run the DL prediction pipeline (training/validation, KM plot generation, etc.).

________

Now we will demonstrate how to perform the following analyses:
- [x] Train and validate deep learning network
- [x] Generate KM plot for deep learning network predictions
- [x] Generate predictions with saved prediction models


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
This code will run a bootstrap-based internal validation of the DL model, and save the final model (run on the full training sample). The model will be saved in the `/4DSurvival_results` shared/mounted directory, as an [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) file named `saved_model__DL.h5` Results of the internal validation (predictive performance, etc.) are saved in the `/4DSurvival_results` shared/mounted directory under filename `RESULTS_demo_validateDL.txt`.

 
 
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


#### Generate predictions with saved prediction models
As described in the previous section ('**Train & validate deep learning network**'), after training and validation of the 4D*survival* deep learning model, the final trained model is saved as an HDF5 file under the `/4DSurvival_results` mounted directory. This saved 4D*survival* model can be used to generate predictions/risk scores for new patients for whom *raw* mesh motion data is available (i.e. output of 4D*Segment*). Below, we outline this process:

For the new batch of patients, their raw cardiac MRI scan data (in the form of grey-scale images in NIfTI format), should have already been run through 4D*Segment*, which will carry out segmentation of these images, non-rigid co-registration, mesh generation and motion tracking. If 4D*Segment* completes successfully, it should output a `data` folder with similar contents as described above under the '**Overview**' section, i.e. (1) `subjnames.txt` (file containing IDs of all subjects whose raw MRI data was successfully processed by the 4D*Segment* pipeline); (2) subfolders labelled with subject IDs; (3) `matchedpointsnew.txt` (file containing mapping required for mesh-downsampling). **NOTE**: since new patients are not expected to have survival outcome data, `surv_outcomes.csv` file (described above in the `Overview` section) should be absent from the `data` folder. Once we confirm all the aforementioned criteria are met, the next step would be to convert the output of 4D*Segment* into a format that can be fed into the saved 4D*survival* model to generate predictions. Assuming the 4D*Segment* output `data` folder was mounted on `/4Dsegment_output` and the saved model was saved in `/4DSurvival_results`, the following commands can be used:
```
cd /4DSurv/setup
python3 inputdata_setup.py /4Dsegment_output
```

If all goes well, `inputdata_setup.py` will output a file `inputdata_DL_nooutcome.pkl` saved to `/4DSurv/data/` (check the `/4DSurv/data` directory to make sure the file is there). This file contains 2 components: a mesh motion data matrix (containing vectorized motion data for each subject) and a list of subject IDs. The mesh motion data matrix is fed into the saved prediction model to generate predictions. Assuming the saved prediction model is stored in `/4DSurvival_results` as `saved_model__DL.h5`, we can accomplish this via the following commands:
```
cd /4DSurv/demo
```

```
python3 deploy_modelDL.py /4DSurvival_results/saved_model__DL.h5 /4DSurv/data/inputdata_DL_nooutcome.pkl
```
The last command runs the `deploy_modelDL.py` script. The script takes two arguments, (1) the location of the saved model and (2) the location of the mesh motion data for the new subjects. 


This script outputs a file called `predictions_DLnetwork.csv` stored under `/4DSurvival_results`. This file is a CSV file with 2 columns, the first containing subject IDs and the second containing predictions/risk scores for each of the subjects.

________

#### Covariate data
Covariate data must be structured in the form of a CSV file satisfying the following requirements (see a [sample file](sample_files/covariates_sample.csv) for an example):

1. First column must be named 'ID' and contain subject IDs
2. All covariates should be numeric. No characters should be used. Categorical variables (e.g. sex, race/ethnicity) must be coded as numerals
3. The data should not contain missing or infinite values

#### Train & validate deep learning network (mesh motion + covariates) and generate Kaplan-Meier plots
We recommend saving the covariate data CSV under the `data` folder. To use the covariate data in our training model, it needs to be combined with mesh motion data into a [.pkl](https://docs.python.org/3/library/pickle.html) file and saved. To do this, we run the following commands:

```
cd /4DSurv/setup
```

```
python3 inputdata_setup.py /4Dsegment_output /4Dsegment_output/covariates.csv
```

The last command runs the `inputdata_setup.py` script. The script takes two arguments, (1) the directory into which the `data` folder (produced after running 4D*Segment*) was mounted (2) the location of the covariate data file. If the covariate data is formatted correctly (as outlined above), this script will produce a file `inputdata_DL_wcovariates.pkl` saved under the directory `/4DSurv/data`. 

To train and validate the model, we then run the following commands:

```
cd /4DSurv/demo
```

```
python3 demo_validateDL_wcovs.py
```

This code will run an internal validation of the DL model using nested cross-validation. It will save the final model (run on the full training sample) in the `/4DSurvival_results` shared/mounted directory, as an [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) file named `saved_model__DL_wcovariates.h5`. Results of the internal validation (predictive performance, etc.) are saved in the `/4DSurvival_results` shared/mounted directory under filename `RESULTS_demo_validateDL_wcovs.txt`.


Kaplan-Meier plots can be generated from the nested cross-validated results:
```
cd /4DSurv/demo
```

```
python3 demo_KMplotDL_wcovs.py
```
This code will generate a KM plot saved in the `/4DSurvival_results` shared/mounted directory, as a PNG file named `RESULTS_demo_KMplot_DL_wcovariates.png`



### Features to be introduced soon..
- [x] DL training on GPU
- [x] Generating predictions with saved models
- [x] Incorporation of covariate data
