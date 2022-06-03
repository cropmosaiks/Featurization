# CropMOSAIKS Featurization Repository

## Purpose

The featurization repository is the first repository a user should use in order to execute the [MOSAIKS approach](https://www.nature.com/articles/s41467-021-24638-z). It is designed to guide a user through processing satellite images collected by Landsat 8 and Sentinel 2 on the STAC API. Code is tailored to run on Planetary Computer. This repository contains code for proccessing satellite imagery by encoding with random convolutional features. The methodology and workflow requires connection to a [SpatioTemporal Asset Catalog (STAC)](https://stacspec.org/) such as [Microsoft's Planetary Computer (MPC)](https://github.com/microsoft/PlanetaryComputer). The base of the code found in this repostiory was originally created by the team at MPC and is hosted as a [tutorial](https://github.com/microsoft/PlanetaryComputerExamples/blob/main/tutorials/mosaiks.ipynb) on MPC Hub. Our team has adapted and extended this codebase to featurize imagery over time (monthly) and to include a larger portion of the electromagnetic spectrum (i.e., satellite bands).

Alternatively, instead of processing satellite images, a user can download pre-processed feature files from the [MOSAIKS API](https://nadar.gspp.berkeley.edu/home/index/?next=/portal/index/) which hosts features collected from a private satellite. The features available on the MOSAIKS API are global and therefore not limited to the country of Zambia. In order to query features from this website, a user should upload a csv of latitude and longitude points (or create a bounding box) and the features will be processed and sent to the user to download. In order to merge these features with other data of interest and execute future analysis, please see the [Modeling repository](https://github.com/cropmosaiks/Modeling).

## Datasets

### Input 

#### Satellites

Currently this code is adapted to use two satellites, [Landsat 8](https://planetarycomputer.microsoft.com/dataset/landsat-8-c2-l2) and [Sentinel 2](https://planetarycomputer.microsoft.com/dataset/sentinel-2-l2a). These satellites were selected for two primary reasons:
1. Ideal temporal coverage that overlaps with our crop yield data for the country of Zambia, which allows us to geospatially and temporally join satelite feature data with crop yields to execute a supervised machine learning approach (see the Modeling repository for more information)
2. They are public satellite image archives readily available on the MPC [data catalog](https://planetarycomputer.microsoft.com/catalog). The existing MOSAIKS pipeline uses private satellite imagery

These satellites provide options for different band combinations, spectral resolutions, and temporal cycles (meaning the time intervals between passes over the country of Zambia). Additionally, these satellites can be used in combination for the years in which they overlap.

#### Geo-data

Aditionally a user will be required to supply a gesopatial polygon in order to make a grid of points at 0.01 degree intervals in WGS84. In place of a polygon, a simple bounding box can also be used. In general, we have been conducting analysis within a single country and use the outline of that country to first create our grid of points. 

More recently, our team has begun by first creating a grid of points that is premasked for cropland area in order to only retain the top 10% of points per district with crop land. These points are then uploaded to the `data` folder and used to match imagery in our featurization pipeline. 

### Output

The technique of Random Convolutional Features (RCFs), a subset of [Random Kitchen Sinks](https://people.eecs.berkeley.edu/~brecht/kitchensinks.html#:~:text=Features%20are%20those%20pesky%20data,machine%20learning%20algorithms%20are%20trained.&text=Random%20features%20thus%20provide%20a,are%20very%20easy%20to%20implement.), is a way to encode geospatial locations with information based on the satellite image of that location. These features reflect information such as image colors and image textures. This information could be the delimination between colors (like the edge of a field, forest, or building that appears as a line from space), and combinations of colors such as blue next to green. In practice, the specific nature of what information a feature holds is not necessary for it to be useful. In fact, we generally do not investigate possiblities for what a feature might describe, but rather we use them, and the relationship between them to build a model which is capable of predicting what we are interested in. 

With the feature data frame that is made from these notebooks, each row represents an image, and each feature represents a column. Each cell contains a numerical value for that feature at that location, which is statistically coorelated with the numerical value of crop yield data for that location during the modeling step (or other data provided by the user). Random Convolutional Features can either be created from the featurization repository in this organization, or downloaded from the [MOSAIKS API](https://nadar.gspp.berkeley.edu/home/index/?next=/portal/index/). For more information about featurization and the MOSAIKS pipeline, please see this paper by [Rolf et al. 2021](https://www.nature.com/articles/s41467-021-24638-z).

## Requirements 

- Connection to STAC Collection provider such as Microsoft Planetary Computer (MPC)
  - If using MPC see [environment docs](https://github.com/microsoft/PlanetaryComputerDataCatalog/blob/develop/docs/overview/environment.md) for further direction
- Computer with Graphical Proccessing Unit (GPU)
  - NVIDIA GPU with [CUDA](https://developer.nvidia.com/cuda-toolkit)
- Familiarity with Python code 
  - `PyTorch` in particular 

## Getting Started

The fastest way to get started is to sign up for a free account with [MPC Hub](https://planetarycomputer.microsoft.com/docs/overview/environment/). This process includes a request form, and approval might take 24 hours or more. With an account, a user is given access to cloud-based virtual computing with pre-configured and managed python environments. With several options to choose from, it is important to select the `GPU - PyTorch` option to execute these notebooks. This has a longer startup time than other options, but is neccessary for the way our convolutional model is configured. In the end, the GPU will speed up the computation compared to the CPU options.

This repository can be cloned into the root directory of the MPC Hub. Within these notebooks several decisions need to be made in order to pull in data and process the features for the timeline and specific analyses that fits the user's needs. These decisions are described in detail below.

## Notebooks

<details>
  <summary> 
    <code class="notranslate">dense_grid.ipynb</code>
  </summary>
  <br>
  <a href="https://github.com/cropmosaiks/Featurization/blob/main/dense_grid.ipynb">dense_grid.ipynb</a>
  
</details>

<details>
  <summary> 
    <code class="notranslate">
      rcf_multiband.ipynb
    </code>
  </summary>
  <br>
  <a href="https://github.com/cropmosaiks/Featurization/blob/main/rcf_multiband.ipynb">rcf_multiband.ipynb</a>
  
</details>

<details>
  <summary> 
    <code class="notranslate">
      s2_l8_multiband.ipynb
    </code>
  </summary>
  <br>
  <a href="https://github.com/cropmosaiks/Featurization/blob/main/s2_l8_multiband.ipynb">s2_l8_multiband.ipynb</a>
  
</details>

<details>
  <summary> 
    <code class="notranslate">
      land_cover_9_class.ipynb
    </code>
  </summary>
  <br><p>
  <a href="https://github.com/cropmosaiks/Featurization/blob/main/land_cover_9_class.ipynb">land_cover_9_class.ipynb</a>
  
  There is an optional `landcover.ipynb` notebook that can be used to return various land cover land use percentages at a given point (i.e., the same points which you are interested in featurizing) such as cropped area, forrest cover, or built areas. This notebook uses the [10 meter land cover](https://planetarycomputer.microsoft.com/dataset/group/io-land-cover) dataset. This notebook is under development and has a known bug that will return NULL values around the UTM zone delineations. It is not recommended to use this until this bug can be fixed.
  </p>
</details>

<details>
  <summary> 
    <code class="notranslate">
      Sentinel_2_RGB.ipynb
    </code>
  </summary>
  <br>
  <a href="https://github.com/cropmosaiks/Featurization/blob/main/Sentinel_2_RGB.ipynb">Sentinel_2_RGB.ipynb</a>
  
</details>

## Generalized Steps  

With MPC access as described above, a user has several options to begin creating their Random Convolutional Features (RCFs). An overview of the steps are as follows: 

### Step 1 - Create a grid of points

Use the `dense_grid.ipynb` to execute the following steps:

- Create a uniform grid of points over the region of interest, or load a file containg the pre-produced latitude and longitude points to featurize 
  - User selects a country or region and a grid will be created
  - User can supply geometry or a country code can be specified to use the `geopandas` shapefiles
  - For gridding the country of Zambia specifically, a user has two options: `equal angle` cells versus `equal area` grid cells
    - `equal angle` grids are produced using the latitude/longitude geodetic coordinate reference system, EPSG 4326, which is based on Earth's center of mass
      - results in each point representing a 0.01 by 0.01 degree grid cell that will be featurized 
      - This is roughly 1 km<sup>2</sup> (exact area varies by geographic location)
      - This means a pre-processed, user-supplied file should have points with a minimum distance of 0.01 degrees to avoid overlap
    - `equal area` grids are produced using the local coordinate reference system for the region of interest
      - The local EPSG for the country of Zambia is the defualt, but the relevant EPSG for another region of interest may be supplied by the user 

### Step 2 - Select a featurization notebook

The featurization notebooks are:

- `rcf_multiband.ipynb` 
  - For use with the `landsat-c2-l2` satellite collection or `sentinel-2-l2a`  
- `Sentinel_2_RGB.ipynb`
  - For use with `sentinel-2-l2a` in only the visible spectrum.
  - MUCH faster than other options 
- `s2_l8_multiband.ipynb` 
  -  For use with the `landsat-8-c2-l2` satellite collection or `sentinel-2-l2a`  
  -  `landsat-8-c2-l2` is now deprecated in favor of the `landsat-c2-l2` collection

### Step 3 - Select Options

Options include selecting a satellite collection, the number of features to produce, the spectral bands, select the time period. These options are selected in the `rcf_multiband.ipynb`, `Sentinel_2_RGB.ipynb`, or `s2_l8_multiband.ipynb` notebooks. 

- Select a satellite
  - `landsat-8-c2-l2`
  - `sentinel-2-l2a`
- Select the desired number of features
  - Defaults to 1000, which has resulted in excellent model performance for our project's goals
  - Increasing the number of features increases computational cost and the time it takes to execute the notebook
- Select the relevant bands 
  - Naming conventions are unique to each satellites
- Select a time period to featurize
  - Constrained by satellite mission timeline:
    - Landsat 8: temporal coverage = February 2013 - present
    - Sentinel 2: temporal coverage = June 2015 - present

### Step 3 - Run the notebook in full

The notebook is configured to account for all of your desired inputs, but compute power may limit the extent of what is possible based on selected options

- For example, trying to featurize too many points in a single run may not only be slow, it may crash the kernel or cause a timeout or disconnect error

### How the notebooks work

The general notebook workflow:

1. Find an appropriate STAC item for each point (in parallel, using a spatially partitioned dataset of points)
2. Feed the points and STAC items to a custom Dataset that can read imagery given a point and the URL of a overlapping satellite scene
3. Use `stackstac` to stack the various bands of interest
4. Use a custom Dataloader, which uses our Dataset, to feed our model imagery and save the corresponding features
5. Loop through the year and month combinations selected by the user to output feature files in a compressed feather file format

## Constraints

- MPC Hub persistent storage 
  - Limited to 15 gigabytes (with access to large temporary storage: ~200 gigabytes within a single session)
  - Exceeding storage limits can cause your environment to not load on the next session
  - Download output files and delete from the hub regularly
- MPC compute power
  - A generous amount of compute power is provided free of charge, but processing may still be slow
- MPC Hub RAM
  - Most runs will push the given memory to its limit, and sometimes past
  - We have tried to implement aggressive memory recovery through deletion of objects and garbage collection 
- MPC GPU node limit. The MPC Hub provides free GPU access but access is limited. The GPU nodes are first come, first serve. It can be frustrating to access The MPC Hub when all nodes are being consumed by other users. 
  - There are many ways to do [compute with MPC data](https://planetarycomputer.microsoft.com/docs/concepts/computing/) and it may be neccessary to use alternate options 

## Future Work

There are many expansions and future research that could be contributed to this project of extending the MOSAIKS approach. A few of these ideas include:

- Testing how the cloud cover limit effects results. Currently set to 10% and would recommend testing 15%, 20%, or more
  - Alternatively, determining best method for using every least cloudy image for any given month and only throwing away 1 km points that do not meet a cloud thresshold. In this way, more whole images may be retained, and fewer points would be lost to cloud cover.  
- Producing features for regions other than Zambia, such as Tanzania and Nigeria, as those are other countries in sub-Saharan Africa with crop yield data (While the CropMOSAIKS team has access to this crop data, Tanzania and Nigeria were out of the scope. With more time and features for these countries, the CropMOSAIKS team aims to eventually model crop yields for regions beyond Zambia.)

 

## Contributing

The capstone project was completed on June 9th, 2022. Some team members will be continuing work in this field and the repositories are likely to stay active for some time to come. Suggestions for improvements to the code or documentation is welcome and encouraged. Please submit questions, comments, or code via issues or pull requests on either of the repositories. To correspond with the data scientists who produced these materials that extend the MOSAIKS approach, please see their personal GitHub accounts at the bottom of the organization's README and feel free to contact them via email.

If you are interested in processing features for a new region other than Zambia and contributing these features to the [MOSAIKS API](https://nadar.gspp.berkeley.edu/home/index/?next=/portal/index/), please see the GitHub repository [here](https://github.com/calebrob6/mosaiks-api) and create a pull request or issue. Additionally, you can contact the authors of the [MOSAIKS paper](https://www.nature.com/articles/s41467-021-24638-z) with questions about the process.

For rules and regulations for this organization, please see the [Code of Conduct](https://github.com/cropmosaiks/.github/blob/main/CODE_OF_CONDUCT.md)


