# CropMOSAIKS Featurization Repository

## Purpose

The featurization repository contains code for proccessing satellite imagery by encoding with random convolutional features. The methodology and workflow requires connection to a [SpatioTemporal Asset Catalog (STAC)](https://stacspec.org/) such as [Microsoft's Planetary Computer (MPC)](https://github.com/microsoft/PlanetaryComputer). The base of the code found in this repostiory was originally created by the team at MPC and is hosted as a [tutorial](https://github.com/microsoft/PlanetaryComputerExamples/blob/main/tutorials/mosaiks.ipynb) on MPC Hub. Our team has adapted and extended this codebase to featurize imagery over time (monthly) and to include a larger portion of the electromagnetic spectrum (i.e., satellite bands). 

## Datasets

Currently this code is adapted to use two satellites, [Landsat 8](https://planetarycomputer.microsoft.com/dataset/landsat-8-c2-l2) and [Sentinel 2](https://planetarycomputer.microsoft.com/dataset/sentinel-2-l2a). 

## Requirements 

- Connection to STAC Collection provider such as MPC
- Computer with Graphical Proccessing Unit (GPU)
- Familiarity with Python code 

## Getting Started

The fastest way to get started is to sign up for a free account with [MPC Hub](https://planetarycomputer.microsoft.com/docs/overview/environment/). With an account, a user is given access to cloud based virtual computing with pre-configured and managed environments. With several options to choose from it is important to select the `GPU - PyTorch` option. This has a longer startup time, but is neccessary for the way our convolutional model is configured and in the end, the GPU will speed up the computation. 

This repo can be cloned into the root directory of the MPC Hub. With the code and the correct environment, several decisions need to be made. These decisions are described in detail below. 

## Notebooks

With access as described above, a user has several options to begin creating their Random Convolutional Features (RCFs). In general the steps are as follows:

- Create a grid of points or load a file containg points that you wish to featurize 
  - One point represents a 0.01 by 0.01 degree grid cell that will be featurized 
    - This is roughly 1 km<sup>2</sup> (exact area varies by geograhpic location)
    - This means a user supplied file should have points with a minimum distance of 0.01 degrees to avoid overlap
  - Grid creation can be done directly in the notebook
    - User selects a country or region and a grid will be created
    - User can supply geometry or a country code can be specified to use the `geopandas` shapefiles
- Select a satellite
  - `landsat-8-c2-l2` or `sentinel-2-l2a`
- Select the desired number of features
  - Defaults to 1000
- Select the relevant bands 
  - Naming conventions are unique to each satellites)
- Select a time period to featurize
  - Constrained by Satellite mission timeline
- Run the notebook in full
  - The notebook is configured to account for all of your desired inputs, but compute power may limit the extent of what is possible based on selected options
    - For example, trying to do too many points in a single run may not only be slow, it may crash the kernel or cause a timeout or disconnect error

All of the above options can be configured in our featurization notebook, `rc_featurization.ipynb`, the primary notebook of this repository. Following these selections, the notebook can be run in full with the resulting workflow of:

1. Find an appropriate STAC item for each point (in parallel, using a spatially partitioned dataset of points)
2. Feed the points and STAC items to a custom Dataset that can read imagery given a point and the URL of a overlapping satellite scene
3. Use `stackstac` to stack the various bands of interest
4. Use a custom Dataloader, which uses our Dataset, to feed our model imagery and save the corresponding features
5. Loop through the year and month combinations selected by the user to output feature files in a compressed feather file format

There is an optional `landcover.ipynb` notebook that can be used to return various land cover land use percentages at a given point (i.e., the same points which you are interested in featurizing) such as cropped area, forrest cover, or built areas. This notebook uses the [10 meter land cover](https://planetarycomputer.microsoft.com/dataset/group/io-land-cover) dataset. This notebook is underdevelopment and has a known bug that will return NULL values around the UTM zone delineations. It is not recommended to use this until this bug can be fixed.

## Constraints

- MPC Hub persistent storagee 
  - Limited to 15 gb (with access to large temporary storage - ~200 GB within a single session)
  - Exceeding storage limits can cause your environment to not load on the next session
  - Download output files and delete from the hub regularly
- MPC compute power
  - A generous amount of compute power is provided free of charge, but processing may still be slow
- MPC Hub RAM
  - Most runs will push the given memory to its limit, and sometimes past
  - We have tried to implement aggressive memory recovery through deletion of objects and garbage collection 

## Future Work

suggested ways to expand this work

## Contributing

Code of conduct, access, API, etc


