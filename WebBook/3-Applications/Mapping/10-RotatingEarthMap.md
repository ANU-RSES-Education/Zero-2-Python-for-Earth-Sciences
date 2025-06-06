---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.6
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Global Plate Motions on a spinning globe

+++ {"solution": "hidden", "solution_first": true}



_The surface of the Earth is continually moving in response to convection currents in the underlying mantle. A map of the surface velocities shows the motions are organised into a strikingly simple pattern of surface "plates" that are almost rigid bodies. The surface strain rate - coloured contours - is concentrated at the plate boundaries. On closer inspection, there are many regions within the continental crust where the motions are diffuse, not at all plate-like. The arrows are colored by their angle to North as a means of distinguishing the different plates. The scale is such that the longest vector represents 10cm/yr._

The image is composed of a background grayscale topography / bathymetry map, plate motion vectors (in a Europe-fixed frame) and contours of the observed strain rate second invariant. These data are available after running [Notebook 0](/notebooks/Mapping/0%20-%20Preliminaries.ipynb#) to download / install everything.

```{code-cell} ipython3
%pylab inline

from osgeo import gdal
from osgeo import gdal_array

import cartopy.crs as ccrs
import matplotlib.pyplot as plt

import cartopy.feature as cfeature

from scipy.io import netcdf
```

```{code-cell} ipython3
# from cloudstor import cloudstor
# teaching_data = cloudstor(url="L93TxcmtLQzcfbk", password='')

# teaching_data.download_file_if_distinct("BlueMarbleNG-TB_2004-12-01_rgb_3600x1800.TIFF", "Resources/BlueMarbleNG-TB_2004-12-01_rgb_3600x1800.TIFF")
# teaching_data.download_file_if_distinct("color_etopo1_ice_low.tif", "Resources/color_etopo1_ice_low.tif")
# teaching_data.download_file_if_distinct("EMAG2_image_V2_no_compr.tif", "Resources/EMAG2_image_V2_no_compr.tif")
# teaching_data.download_file_if_distinct("global_age_data.3.6.z.npz", "Resources/global_age_data.3.6.z.npz")
# teaching_data.download_file_if_distinct("etopo1_grayscale_hillshade.tif", "Resources/etopo1_grayscale_hillshade.tif")
# teaching_data.download_file_if_distinct("ETOPO1_Ice_c_geotiff.tif", "Resources/ETOPO1_Ice_c_geotiff.tif")

# teaching_data.download_file_if_distinct("HYP_50M_SR_W/HYP_50M_SR_W.tif", "Resources/HYP_50M_SR_W/HYP_50M_SR_W.tif")
# teaching_data.download_file_if_distinct("OB_50M/OB_50M.tif", "Resources/OB_50M/OB_50M.tif")
# teaching_data.download_file_if_distinct("velocity_NNR.nc", "Resources/velocity_NNR.nc")
```

```{code-cell} ipython3
# The colormap routine creates enormous arrays in intermediary calculations. This is
# a way to avoid memory errors: process to RGB (int8) in advance

def apply_colormap_to_image(rawimage, colormap, norm):

    valid = rawimage > -20000
    greyimage = norm(rawimage)
    rgbimage = np.empty((greyimage.shape[0], greyimage.shape[1] , 4), dtype=uint8)

    for i in range(0, greyimage.shape[0]):
        rgbimage[i,:,:] = colormap(greyimage[i,:], bytes=True) 

    rgbimage2 = rgbimage[:,:,0:3]
    rgbimage2[~valid, 0] = 255
    rgbimage2[~valid,1] = 255
    rgbimage2[~valid,2] = 255
        
    return rgbimage2
    
```

```{code-cell} ipython3
base_projection     = ccrs.PlateCarree() 
global_extent     = [-180.0, 180.0, -90.0, 90.0]

# strainrate_extent=[-180,180,-68,80]
# strainrate = numpy.loadtxt("Resources/sec_invariant_strain_0.2.dat")
# strainrate_data = strainrate.reshape(741,1800,3)  # I had to look at the data to work this out !
# # strainrate_img  = strainrate_data[:,:,2] # Not actually used here !
```

```{code-cell} ipython3
# Note: we need to manage the memory here
# memory constraints than the native installation. Be sure to del() unused arrays/images

# Etopo Height field as geotiff

etopoH = gdal.Open("Resources/ETOPO1_Ice_c_geotiff.tif") 
etopoH_img = etopoH.ReadAsArray()[::2,::2].astype(numpy.float16)
del(etopoH)

colormap = plt.get_cmap('terrain')

norm = matplotlib.colors.Normalize(vmin=-6000, vmax=5000)
etopoH_img_grey = apply_colormap_to_image(etopoH_img, colormap, norm)


# The remaining values have been processed into elevations with a data range of -2951 to 11687 meters.
venusH = gdal.Open("Resources/Venus_Magellan_Topography_Global_4641m_v02.tif")
venusH_img = venusH.ReadAsArray().astype(numpy.float16)
del(venusH)

# Mean venus level is -2000
norm = matplotlib.colors.Normalize(vmin=-2000, vmax=4000)
venusH_img_grey = apply_colormap_to_image(venusH_img, colormap, norm)

# These are alternative images 

# etopo1       = gdal.Open("Resources/color_etopo1_ice_low.tif")
# etopo_img    = etopo1.ReadAsArray().transpose(1,2,0)
# del(etopo1)

# global_shrelief = gdal.Open("Resources/etopo1_grayscale_hillshade.tif")
# global_shrelief_img = global_shrelief.ReadAsArray()[::2,::2].astype(numpy.float16)
# del(global_shrelief)
# print "global_shrelief_img - ", global_shrelief_img.shape
```

```{code-cell} ipython3
# How does that look in the Native Projection ?

fig = plt.figure(figsize=(12, 6), facecolor="none")
# ax = plt.axes(projection=base_projection)
# ax.imshow(etopoH_img_grey[::1,::1], transform=ccrs.PlateCarree(), origin="upper", 
#           alpha=1.0, extent=global_extent,  zorder=0)

ax = plt.axes(projection=base_projection)
ax.imshow(venusH_img_grey[::1,::1], transform=ccrs.PlateCarree(), origin="upper", 
          alpha=1.0, extent=global_extent,  zorder=0)
```

```{code-cell} ipython3
# Some features we may need for the map 


coastline = cfeature.NaturalEarthFeature('physical', 'coastline', '50m',
                           edgecolor=(0.0,0.0,0.0),
                           facecolor="none")

rivers = cfeature.NaturalEarthFeature('physical', 'rivers_lake_centerlines', '50m',
                                        edgecolor='Blue', facecolor="none")

lakes = cfeature.NaturalEarthFeature('physical', 'lakes', '50m',
                                        edgecolor="blue", facecolor="blue")

ocean = cfeature.NaturalEarthFeature('physical', 'ocean', '50m',
                           edgecolor="green",
                           facecolor="blue")

graticules_5 = cfeature.NaturalEarthFeature('physical', 'graticules_5', '10m',
                           edgecolor="black", facecolor=None)
```

```{code-cell} ipython3
## Reading the velocity vector data from the EU-fixed dataset

rootgrp = netcdf.netcdf_file(filename="Resources/velocity_NNR.nc", version=2)

skip = 1

ve = rootgrp.variables["ve"]
vn = rootgrp.variables["vn"]

lonv = rootgrp.variables["lon"]
latv = rootgrp.variables["lat"]

lons = lonv[::skip]
lats = latv[::skip]
llX, llY = np.meshgrid(lons,lats)

#llX = llX.reshape(-1)
#llY = llY.reshape(-1)

Veast = (np.array(ve[::skip,::skip]).T)
Vnorth = (np.array(vn[::skip,::skip]).T)

Vorientation = np.arctan2(Vnorth,Veast)
```

```{code-cell} ipython3
# I used Robinson in the article but it keeps crashing for me now !

fig = plt.figure(figsize=(12, 6), facecolor="white", edgecolor="white")

for central_long in range(0,360,1):

    projection = ccrs.Orthographic(central_longitude=360-central_long, central_latitude=15.0, globe=None)
    ax1 = plt.subplot(1,2,1, projection=projection, frameon=False)
    
    ax1.imshow(etopoH_img_grey[::4,::4,:], transform=ccrs.PlateCarree(), origin="upper", 
              alpha=1, extent=global_extent, interpolation="spline16", zorder=0)

    ax2 = plt.subplot(1,2,2, projection=projection, frameon=False)
    
    ax2.imshow(venusH_img_grey, transform=ccrs.PlateCarree(), origin="upper", 
              alpha=1, extent=global_extent, interpolation="spline16", zorder=0)


    # skip=3
    # ax1.quiver(llX[::skip,::skip], llY[::skip,::skip], Veast[::skip,::skip], Vnorth[::skip,::skip], Vorientation[::skip,::skip], scale=500, transform=ccrs.PlateCarree(),
    #         cmap=cm.Blues, alpha=0.2, zorder=13, pivot="mid")

    # skip = 13
    # ax1.quiver(llX[::skip,::skip], llY[::skip,::skip], Veast[::skip,::skip], Vnorth[::skip,::skip], Vorientation[::skip,::skip], scale=1000, transform=ccrs.PlateCarree(),
    #         cmap=cm.Reds, alpha=0.6, zorder=13, pivot="mid")
    
    
    # ax1.add_feature(coastline, linewidth=1.0,  edgecolor="#000000", zorder=2, alpha=0.75)
    
    fig.savefig(f"PlanetTopography{central_long:03d}.png", dpi=300)

    plt.clf()

```

```{code-cell} ipython3
0/0
```

```{code-cell} ipython3
## This was an alternative to the background image which I thought might look good:

globalrelief      = gdal.Open("Resources/HYP_50M_SR_W/HYP_50M_SR_W.tif")
globalrelief_img  = globalrelief.ReadAsArray().transpose(1,2,0)
del(globalrelief)

globalbathym      = gdal.Open("Resources/OB_50M/OB_50M.tif")
globalbathym_img  = globalbathym.ReadAsArray().transpose(1,2,0)
del(globalbathym)

print ("etopoH_img - ", etopoH_img.shape)
print ("globalrelief_img - ", globalrelief_img.shape)

blended_img = np.empty_like(globalrelief_img)
blended_img[...,0] = np.where( etopoH_img < 0.0, globalbathym_img[...,0], globalrelief_img[...,0] )
blended_img[...,1] = np.where( etopoH_img < 0.0, globalbathym_img[...,1], globalrelief_img[...,1] )
blended_img[...,2] = np.where( etopoH_img < 0.0, globalbathym_img[...,2], globalrelief_img[...,2] )

del(globalbathym_img)
del(globalrelief_img)

fig = plt.figure(figsize=(12, 6), facecolor="none")
ax = plt.axes(projection=base_projection)
ax.imshow(blended_img[::1,::1], transform=ccrs.PlateCarree(), origin="upper", 
          alpha=1.0, extent=global_extent,  zorder=0)
```

```{code-cell} ipython3
# Do this if the relief / bathym sizes don't match the etopo data (to make the blended image)
# The datasets we downloaded can be manipulated trivially without the need for this and I have
# commented it all out so you can run all cells without reprocessing the data files. 


"""
import scipy.ndimage
import scipy.misc


etopoH = gdal.Open("Resources/ETOPO1_Ice_g_geotiff.tif")
etopoH_img = etopoH.ReadAsArray()

print 

etopoH_transform = etopoH.GetGeoTransform()
globalrelief_transform = globalrelief.GetGeoTransform()

# Resize to match globalrelief ... this resize is int only ??

globaletopoH = scipy.misc.imresize(etopoH_img, globalrelief_img.shape, mode='F')

## How to turn this array back into the appropriate geotiff

from osgeo import gdal
from osgeo import osr

# data exists in 'ary' with values range 0 - 255
# Uncomment the next line if ary[0][0] is upper-left corner
#ary = numpy.flipup(ary)

Ny, Nx = globaletopoH.shape
driver = gdal.GetDriverByName("GTiff")
# Final argument is optional but will produce much smaller output file
ds = driver.Create('output.tif', Nx, Ny, 1, gdal.GDT_Float64, ['COMPRESS=LZW'])

# this assumes the projection is Geographic lat/lon WGS 84
srs = osr.SpatialReference()
srs.ImportFromEPSG(4326)
ds.SetProjection(srs.ExportToWkt())

ds.SetGeoTransform( globalrelief_transform  ) # define GeoTransform tuple
ds.GetRasterBand(1).WriteArray(globaletopoH)
ds = None
"""
```
