# Resting-state Functional MRI pre-processing: Short Tutorial 

Raw resting-state functional MRI images are prone to a series of artifacts and variability sources. For this reason, before performing our statistical analysis, we need to apply a series of procedures that aim at removing the sources of signal that we are not interested in, to clean the ones we want to study. All these procedures together are called *pre-processing* .
This document will guide you through the basic steps that are usually taken in the rs-fMRI pre-processing phase.

## Pre-processing Software 
To date, a large amount of pre-processing software are available and can be freely used to process fMRI data. In this course, we will use FSL to perform pre-processing steps directly from the command line. 

## Data 

In this tutorial we are going to use data from one subject of the OASIS study. 
Data can be found in the folder oasis/fMRI_tutorial_orig 
First, copy this folder to your own directory so that you can mess it up as much as you want!

```
cp -rf oasis/fMRI_tutorial_orig  oasis/fMRI_tutorial 
```

Now go into the subject folder and list the content to have an idea of what data we are going to use. 

```
cd data/fMRI_tutorial/OAS30015_MR_d2004

ls 

```

As you can see we will you T1w, in the /anat folder, and fMRI files, in the /func folder. 

## Overview
Modern fMRI pre-processing pipelines include a variety of processes that can, or cannot, be performed depending on the acquired data quality, and study design. Today, we will have a look at TOT pre-processing steps that are commonly used / OR MAYBE AGREED / in all   BLABLA
This includes: 

- EPI Distortion Correction
- Motion Correction
- Spatial Smoothing
- Temporal Filtering 
- Standard Space Mapping

## Hands-on

### Structural Processing
Given the low spatial resolution and SNR of fMRI images, some registration steps in functional processing involve the use of previously computed transformation during the T1w processing. Importnt steps to have performed are: 
Don't run these commands!

- Brain extraction with BET

```
bet sub-OAS30015_ses-d2004_T1w.nii.gz sub-OAS30015_ses-d2004_T1w_bet -f 0.4

```
- Tissue Segmentation with FAST 

```
fast -n 3 -b -o sub-OAS30015_ses-d2004_T1w_bet_FAST sub-OAS30015_ses-d2004_T1w_bet.nii.gz

```
- Linear and Non linear T1w to MNI Standard Space Mapping

```
flirt -in sub-OAS30015_ses-d2004_T1w_bet.nii.gz  -ref /usr/local/fsl/data/standard/MNI152_T1_2mm_brain -out highres2standard -omat highres2standard.mat -cost corratio -dof 12 -searchrx -90 90 -searchry -90 90 -searchrz -90 90 -interp trilinear

fnirt --iout=highres2standard_head --in=sub-OAS30015_ses-d2004_T1w.nii.gz --aff=highres2standard.mat --cout=highres2standard_warp  --jout=highres2highres_jac --config=T1_2_MNI152_2mm --ref=/usr/local/fsl/data/standard/MNI152_T1_2mm --refmask=/usr/local/fsl/data/standard/MNI152_T1_2mm_brain_mask.nii.gz --warpres=10,10,10
```

In this course, these steps have already been performed for you as they can take quite some time. The results can be found in the /anat folder within the subject directory


### Look at the raw data!

To have an idea of how raw data look like before any pre-processing is performed, is visually check the images. 
To do so, fsleyes is a great toolbox that can be used by typing on the command line: 

``` 
fsleyes <input_image> &
```

The `&` at the end of the command, allow us to keep working on the command line while having a graphical application (such as fsleyes) opened. 


### EPI Distortion Correction

rs-fMRI scans suffers from geometric distortion due to magnetic field inhomogeneity. For this reason, the first step of fMRI pre-processing is usually to corret for such distortions. 
To apply this correction, there are two available methods: 

1. TOPUP
2. FIELDMAP


### Preliminary steps

For some registrations steps we will need only one volume from our fMRI timeseries. 
Run the following code to cut 1 volume in the middle of the functional file: 

```
fslroi sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz sub-OAS30015_ses-d2004_task-rest_run-01_bold_1volume  80 1

```

### Motion Correction

A big issue in raw rs-fMRI scans is the fact that participants usually tend to move during the length of the scanning session, therefore producing artefacts in the images. 
Head motion results in lower quality (more blurry) images, as well as create spurious correlations between voxels in the brain. 
Rs-fMRI pre-processing takes care of the motion during the scan by realligning each volume within a scan to a reference volume. The reference volume is usually the first or the middle vollume of the whole sequence. 

To perform motion correction with fsl we use the mcflirt command: 

```
mcflirt -in  sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz -out mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold

```
To keep track of what we are doing, it is good to add a prefix to the output describing the preprocessing steps run on it. So in this case we add `mc_` to our original functional file.


### Spatial Smoothing

```
fslmaths <inputfile> -s <fwhd> <outputfile>
```


### Temporal Filtering

Rs-fMRI timeseries not only contain signal of interest, but are also charaterized by low-frequency drift due to phyiologial (e.g. respiration) or physical (scanner-related) noise. 
For this reason, we usually apply an high-pass filter that eliminates signal variations due to low-frequency. To do so, a voxel timeseries can be represented in the frequency domain, and low frequency can be deleted *change here*. 



### Standard Space Mapping

Brain shape and size strongly vary across different individuals. 
However, to perform group level analysis, voxels between different brain need to correspond. This can be achieved by "registering" rs-fMRI scans in "native-space" to a standard template. 
This processing step is actually made of three different steps. 

1. Compute the registration of the subject T1w scan to MNI space 
This has been previously run using flirt and fnirt (see above). The output transformation matrix is stored in the /anat folder and called highres2standard_warp.mat

2. Register the fMRI to the T1w file

With this function we compute the transformation matrix needed to bring the fMRI file to the T1w space
```
epi_reg --epi=sub-OAS30015_ses-d2004_task-rest_run-01_bold_1volume.nii.gz --t1=../anat/sub-OAS30015_ses-d2004_T1w.nii.gz --t1brain=../anat/sub-OAS30015_ses-d2004_T1w_bet.nii.gz --out=func2highres
```

3. Combine fMRI2T1w and T12MNI transformation, and apply in one go to our timeseries

```
convert_xfm -omat func2standard  -concat ../anat/highres2standard.mat func2highres.mat  # concatenate T12standard and fMRI2T1w affine transform

convertwarp --ref=$FSLDIR/data/standard/MNI152_T1_2mm_brain --premat=func2highres.mat --warp1=../anat/highres2standard_warp --out=func2standard_warp # concatenate T12standard non-linear and fMRI2T1w affine transform

```
