# Resting-state Functional MRI pre-processing: Short Tutorial 

Raw resting-state functional MRI images are prone to a series of artifacts and variability sources. For this reason, before performing our statistical analysis, we need to apply a series of procedures that aim at removing the sources of signal that we are not interested in, to clean the ones we want to study. All these procedures together are called *pre-processing* .
This document will guide you through the basic steps that are usually taken in the rs-fMRI pre-processing phase.

## Pre-processing Software 
To date, a large amount of pre-processing software are available and can be freely used to process fMRI data. In this course, we will use FSL to perform pre-processing steps directly from the command line. 

## Data 

tbd

## Overview
Modern fMRI pre-processing pipelines include a variety of process that can, or cannot, be performed depending on the acquired data quality, ** , and study design. Today, we will have a look at TOT pre-processing steps that are commonly used / OR MAYBE AGREED / in all   BLABLA
This includes: 

- EPI Distortion Correction
- Motion Correction
- Spatial Smoothing
- Temporal Filtering 
- Standard Space Mapping

## Hands-on

### Structural Processing
Given the low quality and SNR of fMRI images, some registration steps in functional processing involve the use of previously computed transformation during the T1w processing. Importnt steps to have performed are: 
- BET
- Standard Space Mapping

In this course, such steps hae already been performed and can be found in BLBALABLA

### Look at the raw data!

``` 
fsleyes <input_image> &
```
### EPI Distortion Correction

### Motion Correction

### Spatial Smoothing

### Temporal Filtering

### Standard Space Mapping



