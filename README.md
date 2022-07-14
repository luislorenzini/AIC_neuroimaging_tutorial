# Introduction to Resting-state Functional MRI: A Short Pre-Processing Tutorial 

Raw resting-state functional MRI images are prone to a series of artifacts and variability sources. For this reason, before performing our statistical analysis, we need to apply a series of procedures that aim at removing the sources of signal that we are not interested in, to clean the ones we want to study. All these procedures together are called *pre-processing* .
This document will guide you through the basic steps that are usually taken in the rs-fMRI pre-processing phase.

## Pre-processing Software 
To date, a large amount of pre-processing software are available and can be freely used to process fMRI data. In this course, we will use FSL to perform pre-processing steps directly from the command line. 

## Data 

In this tutorial we are going to use data from one subject of the OASIS study. 
Data can be found in the folder oasis/fMRI_tutorial_orig 
First, copy this folder to your own directory so that you can mess it up as much as you want!

```
cp -rf oasis/fMRI_tutorial_orig  data/fMRI_tutorial 
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
fsleyes func/sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz &
```

The `&` at the end of the command, allow us to keep working on the command line while having a graphical application (such as fsleyes) opened. 


### EPI Distortion Correction

Some parts of the brain can appear distorted depending on their magnetic properties. One common way to correct the distortions with fMRI data is by acquiring one volume with an opposite phase-encoding direction, and merging the two types of images running [TOPUP](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/topup). In this dataset we don’t have the data required for TOPUP so we will skip this step.
*Note however that you should run it if your data allows.*

### Preliminary steps

For some registrations steps we will need only one volume from our fMRI timeseries. 
Run the following code to cut 1 volume in the middle of the functional file: 

```
fslroi func/sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz func/sub-OAS30015_ses-d2004_task-rest_run-01_bold_1volume  80 1

```

### Motion Correction

A big issue in raw rs-fMRI scans is the fact that participants usually tend to move during the length of the scanning session, therefore producing artefacts in the images. 
Head motion results in lower quality (more blurry) images, as well as create spurious correlations between voxels in the brain. 
Rs-fMRI pre-processing takes care of the motion during the scan by realligning each volume within a scan to a reference volume. The reference volume is usually the first or the middle vollume of the whole sequence. 

To perform motion correction with fsl we use the mcflirt command: 

```
mcflirt -in  func/sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz -out func/mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold

```
To keep track of what we are doing, it is good to add a prefix to the output describing the preprocessing steps run on it. So in this case we add `mc_` to our original functional file.

we can now have a look at original and motion corrected image by typing 

```
fsleyes func/sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz func/mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold &
```

In fsleyes we can use the options in the lower left panel to hide or move images up. Moreover, by clicking on -> View -> Timeseries we can compare the timeseries between different images. 


### Standard Space Mapping

Brain shape and size strongly vary across different individuals. 
However, to perform group level analysis, voxels between different brain need to correspond. This can be achieved by "registering" rs-fMRI scans in "native-space" to a standard template. 
This processing step is actually made of three different steps. 

1. Compute the registration of the subject T1w scan to MNI space 
This has been previously run using flirt and fnirt (see above). The output transformation matrix is stored in the /anat folder and called highres2standard_warp.mat

2. Register the fMRI to the T1w file

With the following function we compute the transformation matrix needed to bring the fMRI file to the T1w space

```
epi_reg --epi=func/sub-OAS30015_ses-d2004_task-rest_run-01_bold_1volume.nii.gz --t1=anat/sub-OAS30015_ses-d2004_T1w.nii.gz --t1brain=anat/sub-OAS30015_ses-d2004_T1w_bet.nii.gz --out=func/func2highres

```

3. Combine fMRI2T1w and T12MNI transformation, and apply in one go to our timeseries

```
convert_xfm -omat func/func2standard  -concat anat/highres2standard.mat func/func2highres.mat  # concatenate T12standard and fMRI2T1w affine transform

convertwarp --ref=$FSLDIR/data/standard/MNI152_T1_2mm_brain --premat=func/func2highres.mat --warp1=anat/highres2standard_warp --out=func/func2standard_warp # concatenate T12standard non-linear and fMRI2T1w affine transform

applywarp --ref=$FSLDIR/data/standard/MNI152_T1_2mm_brain --in=func/mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz  --out=func/MNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz  --warp=func/func2standard_warp

```

The output of these functions is stored in func/MNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz . This is our fMRI scan in MNI space. You can check this by typing 

```
fslinfo func/MNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz  # check characteristics (dimensions) of the MNI functional file 

fslinfo func/sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz # check characteristics (dimensions) of the native-space functional file 

```

Lets also have a look at the MNI file!


```
fsleyes func/MNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz &

```



### Spatial Smoothing

With spatial smoothing we refer to the process of averaging the data points (voxels) with their neighbours. The downside of smoothing is that we loose spatial specificity (resolution). However, with this process has the effect of a low pass filter, removing high frequencing and enhancing low frequencing. Morevoer, spatial correlations within the data are more pronounced and activation can be more easily detected. 
In other words: Smoothing fMRI data increase signal to noise ratio. 
The standard procedure for spatial smoothing is applying a gaussisan function of a specific width, called gaussian kernel. The size of the gaussian kernel determines how much the data is smoothed and is expressed with the Full Width at Half Maximum (FWHM). 

```
fslmaths func/MNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz -s 4 func/sMNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz
```

By changing the "-s" option we can change the FWHM, increasing or decreasing the the smoothing. Try it out and check the results with fsleyes!



### Temporal Filtering

Rs-fMRI timeseries not only contain signal of interest, but are also charaterized by low-frequency drift due to phyiologial (e.g. respiration) or physical (scanner-related) noise. 
For this reason, we usually apply an high-pass filter that eliminates signal variations due to low-frequency. To do so, a voxel timeseries can be represented in the frequency domain, and low frequency can be deleted set to 0. 

![alt text](https://github.com/luislorenzini/AIC_neuroimaging_tutorial/blob/main/figs/FWHM.png)


```

fslmaths func/sMNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz -bptf 45.45 1 func/hpsMNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz

```


### Resting state Networks and Noise Components

Once the data is processed we can try to run an independent component analyis (ICA) on the fMRI timeseries. ICA is usually performed for two reasons:
1. Identifyy resting state networks (i.e. groups of areas that covary [work] together). This step is often done at the group level. 
2. Identify further sources of noice from the data 

ICA can be run using the melodic command from FSL. 

```
melodic -i func/sMNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz -o func/melodic 
```
This will create a directory called "melodic" in our func folder

Open the melodic_IC.nii.gz file in the output folder using fsleyes 

Can you recognize some of the canonical resting state networks?

Do you seem some components that you think might be linked to artefacts? You can clean the original signal by writing them down and running: 

```
fslregfilt -i sMNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz -o denosised_sMNI_mc_sub-OAS30015_ses-d2004_task-rest_run-01_bold.nii.gz -d func/melodic/melodic_mix -f " 1,2,3"
```