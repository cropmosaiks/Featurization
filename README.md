# CropMOSAIKS Featurization Repository

## Purpose

The featurization repository is the first repository a user should use in order to execute the [MOSAIKS approach](https://www.nature.com/articles/s41467-021-24638-z). This repository contains code for proccessing satellite imagery by encoding with random convolutional features. The methodology and workflow requires connection to a [SpatioTemporal Asset Catalog (STAC)](https://stacspec.org/) such as [Microsoft's Planetary Computer (MPC)](https://github.com/microsoft/PlanetaryComputer). The base of the code found in this repostiory was originally created by the team at MPC and is hosted as a [tutorial](https://github.com/microsoft/PlanetaryComputerExamples/blob/main/tutorials/mosaiks.ipynb) on MPC Hub. Our team has adapted and extended this codebase to featurize imagery over time (monthly) and to include a larger portion of the electromagnetic spectrum (i.e., satellite bands). 

## Datasets

Currently this code is adapted to use two satellites, [Landsat 8](https://planetarycomputer.microsoft.com/dataset/landsat-8-c2-l2) and [Sentinel 2](https://planetarycomputer.microsoft.com/dataset/sentinel-2-l2a). These satellites were selected for two primary reasons:
1. Ideal temporal coverage that overlaps with our crop yield data for the country of Zambia, which allows us to geospatially and temporally join satelite feature data with crop yields to execute a supervised machine learning approach (see the Modeling repository for more information)
2. Existing satellite image archives in the MPC STAC, which allows users to import the images straight into the notebook with our existing

These satellites provide options for different band combinations, spectral resolutions, and temporal cycles (meaning the time intervals between passes over the country of Zambia). Additionally, these satellites can be used in combination for the years in which they overlap.

## Requirements 

- Connection to STAC Collection provider such as MPC
- Computer with Graphical Proccessing Unit (GPU)
- Familiarity with Python code 

## Getting Started

The fastest way to get started is to sign up for a free account with [MPC Hub](https://planetarycomputer.microsoft.com/docs/overview/environment/). This process includes a request form, and approval might take 24 hours or more. With an account, a user is given access to cloud-based virtual computing with pre-configured and managed python environments. With several options to choose from, it is important to select the `GPU - PyTorch` option to execute these notebooks. This has a longer startup time than other options, but is neccessary for the way our convolutional model is configured. In the end, the GPU will speed up the computation compared to the CPU options.

This repository can be cloned into the root directory of the MPC Hub. Within these notebooks several decisions need to be made in order to pull in data and process the features for the timeline and specific analyses that fits the user's needs. These decisions are described in detail below.

## Notebooks

With MPC access as described above, a user has several options to begin creating their Random Convolutional Features (RCFs). An overview of the steps are as follows: 

- Create a uniform grid of points over the region of interest, or load a file containg the pre-produced latitude and longitude points to featurize 
  - Grid creation executed directly in the notebook:
    - User selects a country or region and a grid will be created
    - User can supply geometry or a country code can be specified to use the `geopandas` shapefiles
    - For gridding the country of Zambia specifically, a user has two options: `equal angle` cells versus `equal area` grid cells
      - `equal angle` grids are produced using the latitude/longitude geodetic coordinate reference system, EPSG 4326, which is based on Earth's center of mass
        - results in each point representing a 0.01 by 0.01 degree grid cell that will be featurized 
        - This is roughly 1 km<sup>2</sup> (exact area varies by geographic location)
        - This means a pre-processed, user-supplied file should have points with a minimum distance of 0.01 degrees to avoid overlap
      - `equal area` grids are produced using the local coordinate reference system for the region of interest
        - The local EPSG for the country of Zambia is the defualt, but the relevant EPSG for another region of interest may be supplied by the user 
- Select a satellite
  - `landsat-8-c2-l2`
  - `sentinel-2-l2a`
- Select the desired number of features
  - Defaults to 1000, which has resulted in excellent model performance for our project's goals
  - Increasing the number of features increases computational cost and the time it takes to execute the notebook
- Select the relevant bands 
  - Naming conventions are unique to each satellites)
- Select a time period to featurize
  - Constrained by satellite mission timeline:
    - (temporal coverage = February 2013 - present)
    - (temporal coverage = June 2015 - present)
- Run the notebook in full
  - The notebook is configured to account for all of your desired inputs, but compute power may limit the extent of what is possible based on selected options
    - For example, trying to featurize too many points in a single run may not only be slow, it may crash the kernel or cause a timeout or disconnect error

All of the above options can be configured in our featurization notebook, `rc_featurization.ipynb`, the primary notebook of this repository. Following these selections, the notebook can be run in full with the resulting workflow of:

1. Find an appropriate STAC item for each point (in parallel, using a spatially partitioned dataset of points)
2. Feed the points and STAC items to a custom Dataset that can read imagery given a point and the URL of a overlapping satellite scene
3. Use `stackstac` to stack the various bands of interest
4. Use a custom Dataloader, which uses our Dataset, to feed our model imagery and save the corresponding features
5. Loop through the year and month combinations selected by the user to output feature files in a compressed feather file format

There is an optional `landcover.ipynb` notebook that can be used to return various land cover land use percentages at a given point (i.e., the same points which you are interested in featurizing) such as cropped area, forrest cover, or built areas. This notebook uses the [10 meter land cover](https://planetarycomputer.microsoft.com/dataset/group/io-land-cover) dataset. This notebook is underdevelopment and has a known bug that will return NULL values around the UTM zone delineations. It is not recommended to use this until this bug can be fixed.

## Constraints

- MPC Hub persistent storage 
  - Limited to 15 gigbytes (with access to large temporary storage: ~200 gigbytes within a single session)
  - Exceeding storage limits can cause your environment to not load on the next session
  - Download output files and delete from the hub regularly
- MPC compute power
  - A generous amount of compute power is provided free of charge, but processing may still be slow
- MPC Hub RAM
  - Most runs will push the given memory to its limit, and sometimes past
  - We have tried to implement aggressive memory recovery through deletion of objects and garbage collection 

## Future Work

- Stack additional bands to Sentinel 2 in addition to visible spectrum (2,3,4). Examples include short wave infrared (12, 8, and 4), and red edge
- Utilizing notebook for 0.01 degree grid cells (an equal angle grid rather than equal area grid)
- Increasing the cloud cover limit from 10% to 15% or more
- Filtering cloud cover at the level of the resolution you are featurizing (0.01 degree) rather than at the image level. This would maintain more features (because less would be masked by clouds) and likely improve the model performance as executed in the Modeling repository)
- Producing features for regions other than Zambia, such as Tanzania and Nigeria, as those are other countries in sub-Saharan Africa with crop yield data (While the CropMOSAIKS team has access to this crop data, Tanzania and Nigeria were out of the scope. With more time and features for these countries, the CropMOSAIKS team aims to eventually model crop yields for regions beyond Zambia.)

This Featurization repository is designed to guide a user through processing satellite images collected by Landsat 8 and Sentinel 2 on the STAC API. Code is tailored to run on Planetary Computer. Alternatively, a user can download pre-processed feature files from the [MOSAIKS API](https://nadar.gspp.berkeley.edu/home/index/?next=/portal/index/) which hosts features collected from a private satellite. The features available on the MOSAIKS API are global and therefore not limited to the country of Zambia. In order to query features from this website, a user should upload a csv of latitude and longitude points (or create a bounding box) and the features will be processed and sent to the user to download. In order to merge these features with other data of interest and execute future analysis, please see the [Modeling repository](https://github.com/cropmosaiks/Modeling). 

## Contributing

This project was completed on June 9th, 2022, but suggestions for improvements to the code or documentation is welcome and encouraged. Please submit questions, comments, or code via issues or pull requests on either of the repositories. To correspond with the data scientists who produced these materials, please see their personal GitHub accounts at the bottom of the organization's README and feel free to contact them via email.

If you are interested in processing features for a new region other than Zambia and contributing these features to the [MOSAIKS API](https://nadar.gspp.berkeley.edu/home/index/?next=/portal/index/), please see the GitHub repository [here](https://github.com/calebrob6/mosaiks-api) and create a pull request or issue. Additionally, you can contact the authors of the [MOSAIKS paper](https://www.nature.com/articles/s41467-021-24638-z) with questions about the process.

For rules and regulations for this organization, please see the [Code of Conduct](https://github.com/cropmosaiks/.github/blob/main/CODE_OF_CONDUCT.md)


