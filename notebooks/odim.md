---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  name: python3
  display_name: Python 3
title: Products ODIM_H5
subtitle: Overview of available RADOLAN products in ODIM_H5 format and how to visualize them.
---

# ODIM_H5

In this notebook an overview over several RADOLAN products in ODIM_H5 format is given.

```{code-cell} python
import io
import os
import glob
import tarfile
import warnings

warnings.filterwarnings("ignore")

import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np
import wradlib as wrl
import xarray as xr
```

## Setup Environment

Define data reading function and plotting function.

```{code-cell} python
def read_radolan(radfile):
    radfile = os.path.join("../data/showcase/odim", radfile)
    return xr.open_dataset(radfile, engine="radolan")
```

```{code-cell} python
def plot_radolan(ds, **kwargs):
    fig = plt.figure(figsize=(10, 8))
    ax = fig.add_subplot(111, aspect="equal")
    var = ds[list(ds.data_vars)[0]]
    pm = var.plot(ax=ax, cmap="viridis", vmin=0, **kwargs)
    ax.set_title(
        "{0} Product\n{1}".format(var.attrs["prodname"], ds.time[0].values.astype("M8[s]"))
    )
    ax.grid(color="r")
```

## Composite

A few products including RW and SF are available free of charge at this [DWD OpenData Server](https://opendata.dwd.de/). A full list of RADOLAN products can be found in the [DWD RADOLAN Produktübersicht](https://www.dwd.de/DE/leistungen/radolan/produktuebersicht/radolan_produktuebersicht_pdf.pdf?__blob=publicationFile&v=6). For specific details on the RADOLAN ODIM_H5 Format see {ref}`odim-format`. 

Currently, most of the RADOLAN composites have a spatial resolution of 1km x 1km, with the [National Composites](#national-composites) (R-, S- and W-series) being 900 x 900 km grids, and the [European Composites](#extended-composites) 1500 x 1400 km grids. The polar-stereographic projection is described in the chapter {doc}`RADOLAN Grid <grid>`.

One difference is the extended National Composite (only WX) with a 1100 x 900 km grid.


### National Composites

 ID  |  INT  | avail | Description
---- | ----: | ----- | -----------
 HX | 5 min | 5 min | Deutschland-Reflektivitäts-Komposit
 RY | 5 min | 5 min | radardata after correction with <br>Quality-composit (QY)
 RL | 1 h | hh:50 | 1 h summation with adjustment by Merging
 RU | 1 h | hh:50 | 1 h summation with standard and <br>merging adjustment "best of three"
 RW | 1 h | hh:50 | 1 h summation with standard <br>adjustment "best of two"
 SF | 24 h | hh:50 | 24 h summation of RW
 SJ | N d | hh:50 | summation of RW from Jan 1st to current day of year
 SM | N d | hh:50 | summation of RW from 1st of current month to current day of month
 W1 | 7 d  | 05:50 | 7 d summation of RW
 W2 | 14 d | 05:50 | 14 d summation of RW
 W3 | 21 d | 05:50 | 21 d summation of RW
 
#### HX Product

Load data from data source

```{code-cell} python
ds = read_radolan("composite_hx_20251023_1050-hd5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### RY Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-ry_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### RL Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-rl_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### RU Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-ru_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### RW Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-rw_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### RW 2 Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-rw.radolan2_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### SF Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-sf_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### SJ Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-sj_10000-2510240550-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### SM Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-sm_10000-2510240550-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### W1 Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-w1_10000-2510240550-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### W2 Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-w2_10000-2510240550-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### W3 Product

```{code-cell} python
ds = read_radolan("raa01-w3_10000-2510240550-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

### Extended Composites

The common central european products with a range of 1500 km by 1400 km are presented in the following table:

 ID |  INT  | avail | Description
--- | ----: | ----- | -----------
 EH |  1 h  | hh:50 | 
 EB |  1 h  | hh:50 | 
 EW |  1 h  | hh:50 | 


#### EH Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-eh_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### EB Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-eb_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### EW Product

Load data from data source

```{code-cell} python
ds = read_radolan("raa01-ew_10000-2510231050-dwd---bin.hdf5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

### VII Composites

These special products mastering vertically integrated ice (VII) come with (1200, 1100), (2440, 2400) and (5300, 6500) grids and 1000m resolution. Composite VII and EuCom VII is in polar stereographic projection and EuComXL VII in lambert azimuthal equal-area projection. 

#### Composite VII Product

Load data from data source

```{code-cell} python
ds = read_radolan("composite_VII_20251023_1050-hd5")
display(ds)
```

```{code-cell} python
plot_radolan(ds)
```

#### EuCom VII Product

Load data from data source

```{code-cell} python
ds = read_radolan("composite_EuCom-VII_20251023_1050-hd5")
display(ds)
```

```{code-cell} python
plot_radolan(ds, vmax=1)
```

#### EuComXL VII Product

Load data from data source

```{code-cell} python
ds = read_radolan("composite_EuComXL-VII_20251023_1050-hd5")
display(ds)
```

```{code-cell} python
plot_radolan(ds, vmax=1)
```

## RADVOR and Nowcast

The following products are based on RADOLAN or ["RADVOR"](https://www.dwd.de/DE/leistungen/radvor/radvor_info/radvor_kurzbeschreibung_pdf.pdf?__blob=publicationFile&v=8) products (Radar-based precipitation forecast in the shortest term). An opendata showcase is given below the following table.

 ID  |  INT  | avail | Description
---- | ----: | ----- | -----------
 WN | 5 min | 5 min | Analyzed and predicted precipitation, 1 analyse, 24 predictions (POLARA)
 RV | 5 min | 5 min | Analyzed and predicted precipitation, 1 analyse, 24 predictions (POLARA)
 RS | 60 min | 5 min | Analyzed and predicted precipitation, 1 analyse, 24 predictions (POLARA)

### WN Product

```{code-cell} python
fname = "../data/radvor/odim/composite_wn_20251023_1050_.tar.gz"
fp = tarfile.open(fname)
fp.extractall()
names = fp.getnames()
buffer = [io.BytesIO(fp.extractfile(name).read()) for name in names]
for buf, name in zip(buffer, names):
    buf.name = name
fp.close()
```

```{code-cell} python
ds = xr.open_mfdataset(buffer, engine="radolan", concat_dim="prediction_time", combine="nested")
display(ds)
```

```{code-cell} python
ds.DBZH.plot(col="prediction_time", col_wrap=5, vmin=0, vmax=60)
```
  
### RV Product

```{code-cell} python
fname = "../data/radvor/odim/composite_rv_20251023_0500_.tar.gz"
fp = tarfile.open(fname)
fp.extractall()
names = fp.getnames()
buffer = [io.BytesIO(fp.extractfile(name).read()) for name in names]
for buf, name in zip(buffer, names):
    buf.name = name
fp.close()
```

```{code-cell} python
ds = xr.open_mfdataset(buffer, engine="radolan", concat_dim="prediction_time", combine="nested")
display(ds)
```

```{code-cell} python
ds.ACRR.plot(col="prediction_time", col_wrap=5, vmin=0, vmax=2)
```

### RS Product

```{code-cell} python
fname = "../data/radvor/odim/composite_rs_20251023_1050_.tar.gz"
fp = tarfile.open(fname)
fp.extractall()
names = fp.getnames()
buffer = [io.BytesIO(fp.extractfile(name).read()) for name in names]
for buf, name in zip(buffer, names):
    buf.name = name
fp.close()
```

```{code-cell} python
ds = xr.open_mfdataset(buffer, engine="radolan", concat_dim="prediction_time", combine="nested")
display(ds)
```

```{code-cell} python
ds.ACRR.plot(col="prediction_time", col_wrap=5, vmin=0, vmax=10)
```
