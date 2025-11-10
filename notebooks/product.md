---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  name: python3
  display_name: Python 3
title: Products
subtitle: Overview of available RADOLAN products and how to visualize them.
---

```{include} ../_includes/license_block.md
```

# Product

In this notebook an overview over several RADOLAN products is given.

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
```

## Setup Environment

Get RADOLAN Grid and RADOLAN Extended Grid.

```{code-cell} python
# Get coordinates
radolan_grid_xy = wrl.georef.get_radolan_grid(900, 900)
radolan_egrid_xy = wrl.georef.get_radolan_grid(1500, 1400)
radolan_wgrid_xy = wrl.georef.get_radolan_grid(1100, 900)
x = radolan_grid_xy[:, :, 0]
y = radolan_grid_xy[:, :, 1]

xe = radolan_egrid_xy[:, :, 0]
ye = radolan_egrid_xy[:, :, 1]

xw = radolan_wgrid_xy[:, :, 0]
yw = radolan_wgrid_xy[:, :, 1]
```

Define data reading function and plotting function.

```{code-cell} python
def read_radolan(radfile):
    radfile = os.path.join("../data/showcase", radfile)
    return wrl.io.read_radolan_composite(radfile)
```

```{code-cell} python
def plot_radolan(data, attrs, grid, clabel=None):
    fig = plt.figure(figsize=(10, 8))
    ax = fig.add_subplot(111, aspect="equal")
    x = grid[:, :, 0]
    y = grid[:, :, 1]
    pm = ax.pcolormesh(x, y, data, cmap="viridis")
    cb = fig.colorbar(pm, shrink=0.75)
    cb.set_label(clabel)
    plt.xlabel("x [km]")
    plt.ylabel("y [km]")
    plt.title(
        "{0} Product\n{1}".format(attrs["producttype"], attrs["datetime"].isoformat())
    )
    plt.xlim((x[0, 0], x[-1, -1]))
    plt.ylim((y[0, 0], y[-1, -1]))
    plt.grid(color="r")
```

## Composite

A few products including RW and SF are available free of charge at this [DWD OpenData Server](https://opendata.dwd.de/). A full list of RADOLAN products can be found in the [DWD RADOLAN Produktübersicht](https://www.dwd.de/DE/leistungen/radolan/produktuebersicht/radolan_produktuebersicht_pdf.pdf?__blob=publicationFile&v=6). Specific details on the RADOLAN Format can be retrieved from the [DWD RADOLAN/RADVOR-OP Kompositformat](https://www.dwd.de/DE/leistungen/radolan/radolan_info/radolan_radvor_op_komposit_format_pdf.pdf?__blob=publicationFile).

Currently, most of the RADOLAN composites have a spatial resolution of 1km x 1km, with the [National Composites](#national-composites) (R-, S- and W-series) being 900 x 900 km grids, and the [European Composites](#extended-composites) 1500 x 1400 km grids. The polar-stereographic projection is described in the chapter {doc}`RADOLAN Grid <grid>`.

One difference is the extended National Composite (only WX) with a 1100 x 900 km grid.

Also the [PG/PC Product](#pg-and-pc-product) with 460 x 460 km grid and runlength-coding is shortly described.


### National Composites

 ID  |  INT  | avail | Description
---- | ----: | ----- | -----------
 RX/WX | 5 min | 5 min | original radardata in qualitative RVP6-units (1 byte coded)
 RZ | 5 min | 5 min | radardata after correction of PBB converted to rainrate <br>with improved Z-R-relation
 RY | 5 min | 5 min | radardata after correction with <br>Quality-composit (QY)
 RH | 1 h | 5 min | 1 h summation of RZ-composit
 RB | 1 h | hh:50 | 1 h summation with preadjustment
 RW | 1 h | hh:50 | 1 h summation with standard <br>adjustment "best of two"
 RL | 1 h | hh:50 | 1 h summation with adjustment by Merging
 RU | 1 h | hh:50 | 1 h summation with standard and <br>merging adjustment "best of three"
 SQ | 6 h | hh:50 | 6 h summation of RW
 SH | 12 h | hh:50 | 12 h summation of RW
 SF | 24 h | hh:50 | 24 h summation of RW
 W1 | 7 d  | 05:50 | 7 d summation of RW
 W2 | 14 d | 05:50 | 14 d summation of RW
 W3 | 21 d | 05:50 | 21 d summation of RW
 W4 | 30 d | 05:50 | 30 d summation of RW

#### RX Product

Load data from data source.

```{code-cell} python
data, attrs = read_radolan("raa01-rx_10000-1408102050-dwd---bin.gz")
```


Mask data and apply scale and offset

```{code-cell} python
data = np.ma.masked_equal(data, -9999) / 2 - 32.5
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="dBZ")
```

#### RZ Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-rz_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * 5min-1")
```

#### RY Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-ry_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * 5min-1")
```

#### RH Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-rh_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * h-1")
```

#### RB Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-rb_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * h-1")
```

#### RL Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-rl_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * h-1")
```

#### RW Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-rw_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * h-1")
```

#### RU Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-ru_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * h-1")
```

#### SQ Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-sq_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * 6h-1")
```

#### SH Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-sh_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * 12h-1")
```

#### SF Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-sf_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * 24h-1")
```

#### W1 Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-w1_10000-1408110550-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * 7d-1")
```

#### W2 Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-w2_10000-1408110550-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_grid_xy, clabel="mm * 14d-1")
```

#### WX Product

```{code-cell} python
data, attrs = read_radolan("raa01-wx_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999) / 2 - 32.5
```

```{code-cell} python
plot_radolan(data, attrs, radolan_wgrid_xy, clabel="dBZ")
```

### Extended Composites

The common central european products with a range of 1500 km by 1400 km are presented in the following table:

 ID |  INT  | avail | Description
--- | ----: | ----- | -----------
 EX | 5 min | 5 min | analogue RX
 EZ | 5 min | 5 min | analogue RZ
 EY | 5 min | 5 min | analogue EY after correction <br>with Quality-composit
 EH |  1 h  | hh:50 | analogue RH (no preadjustment) <br>1 h summation of EY-composite
 EB |  1 h  | hh:50 | analogue RB (with preadjustment) <br>1 h summation
 EW |  1 h  | hh:50 | analogue RW (full adjustment) <br>1 h summation


#### EX Product

Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-ex_10000-1408102050-dwd---bin.gz")
```

Mask data and apply scale and offset

```{code-cell} python
data = np.ma.masked_equal(data, -9999) / 2 - 32.5
```

```{code-cell} python
plot_radolan(data, attrs, radolan_egrid_xy, clabel="dBZ")
```

#### EZ Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-ez_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_egrid_xy, clabel="mm * 5min-1")
```

#### EY Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-ey_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_egrid_xy, clabel="mm * 5min-1")
```

#### EH Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-eh_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_egrid_xy, clabel="mm * h-1")
```

#### EB Product


Load data from data source

```{code-cell} python
data, attrs = read_radolan("raa01-eb_10000-1408102050-dwd---bin.gz")
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, -9999)
```

```{code-cell} python
plot_radolan(data, attrs, radolan_egrid_xy, clabel="mm * h-1")
```

### PG and PC Product

The PG/PC product is a bit different from the normal RADOLAN formats. The header is actually the same, but the data is runlength encoded. Also, the RADOLAN grid cells have 2km edge length (460x460 cells).

Load data from data source

```{code-cell} python
radfile = "../data/misc/raa00-pc_10015-1408030905-dwd---bin.gz"
data, attrs = wrl.io.read_radolan_composite(radfile, missing=255)
radolan_grid_pc = wrl.georef.get_radolan_grid(460, 460)
```

Mask data

```{code-cell} python
data = np.ma.masked_equal(data, 255)
print(data.shape)
```

```{code-cell} python
# plot the images side by side
plt.figure(figsize=(10, 8))
plt.subplot(111, aspect="equal")
X = radolan_grid_pc[:, :, 0]
Y = radolan_grid_pc[:, :, 1]
# color-scheme taken from DWD "legend_radar_products_pc.pdf"
colors = [
    "lightgrey",
    "yellow",
    "lightblue",
    "magenta",
    "green",
    "red",
    "darkblue",
    "darkred",
]
cmap = mpl.colors.ListedColormap(colors, name="DWD-pc-scheme")
bounds = np.arange(len(colors) + 1)
norm = mpl.colors.BoundaryNorm(bounds, cmap.N)
plt.pcolormesh(X, Y, data, cmap=cmap, norm=norm)
plt.xlim((X[0, 0], X[-1, -1]))
plt.ylim((Y[0, 0], Y[-1, -1]))

# add colorbar and do some magic for proper visualisation
cb = plt.colorbar(shrink=0.75, norm=norm, boundaries=bounds)
loc = bounds + 0.5
cb.set_ticks(loc[:-1])
labels = bounds[:-1]
cb.set_ticklabels(labels)
cl = cb.ax.get_yticklabels()
cl[-1].set_text("9")
cb.ax.set_yticklabels([elem.get_text() for elem in cl])
plt.title("RADOLAN PG Product \n" + attrs["datetime"].isoformat())
plt.grid(color="r")
```

## RADVOR

Further new developments are the ["RADVOR"](https://www.dwd.de/DE/leistungen/radvor/radvor_info/radvor_kurzbeschreibung_pdf.pdf?__blob=publicationFile&v=8) products (Radar-based precipitation forecast in the shortest term). An opendata showcase is given below the following table.



 ID  |  INT  | avail | Description
---- | ----: | ----- | -----------
 RV | 5 min | 5 min | Analyzed and predicted precipitation, 1 analyse, 24 predictions (POLARA)
 RS | 60 min | 5 min | Analyzed and predicted precipitation, 1 analyse, 24 predictions (POLARA)
 RQ | 60 min | 15 min | Quantified precipitation analysis and prediction, 1 analyse, 2 predictions
 RE | 60 min | 5 min | Analysis and prediction of the aggregate state and hail, 1 analyse, 24 predictions (POLARA)
 FS | 60 min | 15 min | snow depth analysis and forecast, 1 analyse, 2 predictions
 FQ | 360 min | 15 min | snow depth analysis and forecast, 1 analyse, 2 predictions



### RV Product

```{code-cell} python
fname = "../data/radvor/DE1200_RV2210180700.tar.bz2"
fp = tarfile.open(fname)
fp.extractall()
names = fp.getnames()
buffer = [io.BytesIO(fp.extractfile(name).read()) for name in names]
for buf, name in zip(buffer, names):
    buf.name = name
fp.close()
```

```{code-cell} python
ds = wrl.io.open_radolan_mfdataset(buffer)
display(ds)
```

```{code-cell} python
ds.RV.plot(col="prediction_time", col_wrap=5, vmax=20)
```

### RE Product

This product isn't implemented with all features, yet. Use with care!

```{code-cell} python
files = [
    "../data/radvor/RE2210180700_120.gz",
    "../data/radvor/RE2210180700_115.gz",
    "../data/radvor/RE2210180700_110.gz",
    "../data/radvor/RE2210180700_105.gz",
    "../data/radvor/RE2210180700_100.gz",
    "../data/radvor/RE2210180700_095.gz",
    "../data/radvor/RE2210180700_090.gz",
    "../data/radvor/RE2210180700_085.gz",
    "../data/radvor/RE2210180700_080.gz",
    "../data/radvor/RE2210180700_075.gz",
    "../data/radvor/RE2210180700_070.gz",
    "../data/radvor/RE2210180700_065.gz",
    "../data/radvor/RE2210180700_060.gz",
    "../data/radvor/RE2210180700_055.gz",
    "../data/radvor/RE2210180700_050.gz",
    "../data/radvor/RE2210180700_045.gz",
    "../data/radvor/RE2210180700_040.gz",
    "../data/radvor/RE2210180700_035.gz",
    "../data/radvor/RE2210180700_030.gz",
    "../data/radvor/RE2210180700_025.gz",
    "../data/radvor/RE2210180700_020.gz",
    "../data/radvor/RE2210180700_015.gz",
    "../data/radvor/RE2210180700_010.gz",
    "../data/radvor/RE2210180700_005.gz",
    "../data/radvor/RE2210180700_000.gz",
]
```

```{code-cell} python
ds = wrl.io.open_radolan_mfdataset(files)
display(ds)
```

```{code-cell} python
ds.RE.plot(col="prediction_time", col_wrap=5, vmax=2)
```

### RQ Product

```{code-cell} python
files = [
    "../data/radvor/RQ2210180700_120.gz",
    "../data/radvor/RQ2210180700_060.gz",
    "../data/radvor/RQ2210180700_000.gz",
]
```

```{code-cell} python
ds = wrl.io.open_radolan_mfdataset(files)
display(ds)
```

```{code-cell} python
ds.RQ.plot(col="prediction_time")
```
