---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  name: python3
  display_name: Python 3
title: Introduction
subtitle: An overview of the RADOLAN system, its data products, and how to use this guide.
---

```{include} ../_includes/license_block.md
```

# Intro

This chapter shows simple loading and visualization of RADOLAN data.

```{code-cell} python
import warnings

warnings.filterwarnings("ignore")

import matplotlib.pyplot as plt
import numpy as np
import wradlib as wrl
import xarray as xr
```

## wradlib reader

All RADOLAN composite products can be read by {func}`wradlib.io.read_radolan_composite`:

```
data, metadata = wradlib.io.read_radolan_composite("mydrive:/path/to/my/file/filename")
```

Here, ``data`` is a two dimensional integer or float array of shape (number of rows, number of columns). ``metadata`` is a dictionary which provides metadata from the files header section, e.g. using the keys `producttype`, `datetime`, `intervalseconds`, `nodataflag`.

### Import data

```{code-cell} python
# load radolan files
rw_filename = "../data/misc/raa01-rw_10000-1408102050-dwd---bin.gz"
rwdata, rwattrs = wrl.io.read_radolan_composite(rw_filename)
# print the available attributes
print("RW Attributes:", rwattrs)
```

### Mask and get coordinates

The {doc}`grid` coordinates can be calculated with {func}`wradlib.georef.get_radolan_grid`.

```{code-cell} python
# do some masking
sec = rwattrs["secondary"]
rwdata.flat[sec] = -9999
rwdata = np.ma.masked_equal(rwdata, -9999)
```

```{code-cell} python
# Get coordinates
radolan_grid_xy = wrl.georef.get_radolan_grid(900, 900)
x = radolan_grid_xy[:, :, 0]
y = radolan_grid_xy[:, :, 1]
```

### Simple wradlib plot

With the following code snippet the RW-product is visualized in the {ref}`Polar Stereographic Projection <grid-projection>`. For more details, refer to {func}`matplotlib.pyplot.pcolormesh`.

```{code-cell} python
# plot function
plt.pcolormesh(x, y, rwdata, cmap="viridis")
cb = plt.colorbar(shrink=0.75)
cb.set_label("mm * h-1")
plt.title("RADOLAN RW Product Polar Stereo \n" + rwattrs["datetime"].isoformat())
plt.grid(color="r")
```

A much more comprehensive section using several RADOLAN composites is shown in chapter {doc}`product`.

## Xarray backend

RADOLAN binary format data will be imported into an {class}`xarray.Dataset` with attached coordinates.

```{code-cell} python
# load radolan files
rw_filename = "../data/misc/raa01-rw_10000-1408102050-dwd---bin.gz"
ds = wrl.io.open_radolan_dataset(rw_filename)
# print the xarray dataset
display(ds)
```

### Simple xarray plot

```{code-cell} python
ds.RW.plot()
```

### Simple selection

```{code-cell} python
ds.RW.sel(x=slice(-100000, 100000), y=slice(-4400000, -4200000)).plot()
```

### Map plot using cartopy

We will use {class}`cartopy.crs.Stereographic` for map-making.

```{code-cell} python
import cartopy.crs as ccrs

map_proj = ccrs.Stereographic(
    true_scale_latitude=60.0, central_latitude=90.0, central_longitude=10.0
)
```

```{code-cell} python
fig = plt.figure(figsize=(10, 8))
ds.RW.plot(subplot_kws=dict(projection=map_proj))
ax = plt.gca()
ax.gridlines(draw_labels=True, y_inline=False)
```

### Open multiple files

```{code-cell} python
# load radolan files
sf_filenames = [
    "../data/misc/raa01-sf_10000-1305270050-dwd---bin.gz",
    "../data/misc/raa01-sf_10000-1305280050-dwd---bin.gz",
]
ds = wrl.io.open_radolan_mfdataset(sf_filenames)
# print the xarray dataset
display(ds)
```

```{code-cell} python
fig = plt.figure(figsize=(10, 5))
ds.SF.plot(col="time")
```

We can also use {func}`xarray.open_dataset` and {func}`xarray.open_mfdataset` with ``radolan``-engine to acquire RADOLAN data.

```{code-cell} python
rw_filename = "../data/misc/raa01-rw_10000-1408102050-dwd---bin.gz"
ds = xr.open_dataset(rw_filename, engine="radolan")
display(ds)
```

```{code-cell} python
sf_filename = "../data/misc/raa01-sf_10000-1305*"
ds = xr.open_mfdataset(sf_filename, engine="radolan")
display(ds)
```
