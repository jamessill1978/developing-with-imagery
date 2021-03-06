# How did they do that?
How did Microsoft get those NAIP MRFs into blob storage? The theme of this lecture is working towards understanding how to create image services that use imagery in S3 by first understanding how the imagery gets into S3.

# Preparation
1. Watch:
    - [Workflows for Optimizing Imagery Management in the Cloud](https://www.esri.com/videos/watch?videoid=gQv3IHKN9j8&title=arcgis-workflows-for-optimizing-image-management-services-in-the-cloud)
    - [Managing Imagery Using Mosaic Datasets and Image Services](https://www.youtube.com/watch?v=4UlVlN0G9qk)
2. Recommended Reading:
    - [Managing Imagery in the Cloud](http://proceedings.esri.com/library/userconf/proc17/tech-workshops/tw_630-625.pdf)
    - [MRF](https://community.esri.com/thread/212729-mrf-s3-mosaics-caches-and-optimization)
    - [Optimize Rasters](https://github.com/Esri/OptimizeRasters)
    - [MDCS](https://github.com/Esri/mdcs-py)


# Lecture - Interfacing with imagery in Azure, AWS, and Google Cloud

## Part 1 - Private Storage
*For this class, all students will need azure storage accounts.*

## Private Buckets

### Azure Blob Storage
In this demo, I will create a blob storage container in my Azure storage accound, *jhuimagery*.

1. Create a storage account, i.e. *jhuimagery*.
    - Resource Group: greg's courses
    - status: Available
    - Location: Central US, East US 2
    - Performance/Access tier: Standard/Hot
    - Replication: Read-access geo-redundant storage (RA-GRS)
    - Account kind: BlobStorage

2. Create a container
    - Select the **+ Container** button.
    - Give the **Container** a name.
    - Select a private **Container**.

## Part 2 - Geting Data into the Azure Storage using [OptimizeRasters](https://github.com/Esri/OptimizeRasters)
OptimizeRasters is a set of tools for converting raster data to optimized Tiled TIF or MRF files, moving data to cloud storage, and creating Raster Proxies. It can do a lot of the work for us!

In this demo, I will upload some astronomy images into a blob storage container and then visualize them in ArcGIS Pro and Jupyter.

1. Create a profile using the **Profile Editor** tool in **Optimize Rasters**
    - Profile Type: Microsoft Azure
    - Profile Name: *User Defined*
    - Access/Account Key ID: *name of container*
    - Secret Access Account Key: *Key 1*

2. Use Optimize Rasters to Convert to MRF
    ![](https://github.com/gbrunner/developing-with-imagery/blob/master/Week%202/sings_to_local.png?raw=true)
3. Use Optimize Rasters to convert to MRF add rasters to the **Container**
    ![](https://github.com/gbrunner/developing-with-imagery/blob/master/Week%202/sings_to_azure.png?raw=true)
    
4. View results in ArcGIS Pro
    - Create a **Cloud Storage Connection** file to your private container.
    - Service Provider: Azure
    - Container Name: *User Defined*
    - Access/Account Key ID: *name of container*
    - Secret Access Account Key: *Key 1*
    
![](https://raw.githubusercontent.com/gbrunner/developing-with-imagery/master/Week%202/azure_private_connection.png)

## Part 3 - ArcGIS Mosaic Datasets
Mosaic datasets can be built from imagery stored in Azure or AWS. Landsat path row 13-32 overlaps New York City. Let;s build some Landsat 8 (OLI) mosaics from this path row.


1. Create two mosaic datasets in the GDB in ArcGIS Pro. Call one **Master_OLI** and the other **OLI_MS_QA**

![](https://github.com/gbrunner/developing-with-imagery/blob/master/Week%202/create_mosaic.png?raw=true)

2. Add Rastes to **Master_OLI**. Specify the **Landsat8_OLI_allBands.art** raster function template so that every OLI band gets into the mosaic.

![](https://github.com/gbrunner/developing-with-imagery/blob/master/Week%202/add_rasters_master_oli.png?raw=true)

3. Add Rasters to **OLI_MS_QA**. This uses **Master_OLI** as the input data (table). Use the **Landsat8_OLI_MS_Table_AR_QA.art** raster function template. This will composite the MS and QA bands into a single raster.

![](https://github.com/gbrunner/developing-with-imagery/blob/master/Week%202/add_rasters_oli_ms_qa.png?raw=true)

4. Set time parameters on **OLI_MS_QA** to work with time.

![](https://github.com/gbrunner/developing-with-imagery/blob/master/Week%202/set_time.png?raw=true)

5. Voila! Make sure to show:
    - Mosaic dataset tables.
    - Make sure to talk about DRA, stretch, histograms, and why we didn't build overviews.
    - Make sure to talk about performance.
    - Talk about mosaic rules
    - Talk about interpolation
    
## Part 4 - [MDCS](https://github.com/Esri/mdcs-py)
```
ECHO Build OLI Mosaics
```

```
ECHO Building Master_OLI Mosaic Dataset
c:\PYTHON27\ArcGIS10.5\python.exe C:\Image_Mgmt_Workflows\MDCS\scripts\MDCS.py -
```

```
ECHO Building LandsatArchive_OLI_MS Mosaic Dataset
C:\Python27\ArcGIS10.5\python.exe C:\Image_Mgmt_Workflows\MDCS\scripts\MDCS.py 
```

## Supplemental - Rasterio with private buckts and request pays buckets
Create an AWS Session and if necessary, specify the requester pays option.
```
import rasterio
import boto3

 with rasterio.Env(rasterio.session.AWSSession(boto3.Session(), requester_pays=True)) as env:
  with rasterio.open('s3://sentinel-s2-l2a/tiles/14/S/PB/2019/12/23/0/R10m/B04.jp2') as src:
    print(src.profile)
```

# Exercises and Homework
1. Create a private container in your Azure Blob Storage account named **dem**
2. Using Optimize Rasters, convert the elevation models in *C:\Users\greg6750\Documents\IPython Notebooks\intro-prog-for-gis-rs\Week 4\data\data\dems* to COG or MRF and upload them into **dem**
3. Using ArcGIS Pro, create a mosaic dataset from the rasters in **dem**. Be sure to build pyramids and overviews. Where did you store your overviews?
4. Create an [NDVI](https://www.usgs.gov/land-resources/nli/landsat/landsat-normalized-difference-vegetation-index?qt-science_support_page_related_con=0#qt-science_support_page_related_con) mosaic from the Master_OLI mosaic. Follow the same proceedure that I used when creating the **OLI_MS_QA** mosaic, but use the **Landsat8_OLI_NDVI_Table_AR_QA.art** raster function template. Enable time. How many bands does the NDVI mosaic have. WHat do they correspond to? 
5. Create a [Tasseled Cap](https://community.esri.com/docs/DOC-1868) mosaic from the Master_OLI mosaic. Follow the same proceedure that I used when creating the OLI_MS_QA mosaic, but use the **Landsat8_OLI_MS_Table_TC_AR_QA.art** raster function template. Enable time. How many bands does the tasseled cap mosaic have. WHat do they correspond to?
6. **CHALLENGE!** Use MDCS to create the NDVI or tassled cap mosaic. 

# Github Links
  - [Optimize Rasters](https://github.com/Esri/OptimizeRasters)
  - [MDCS](https://github.com/Esri/mdcs-py)
