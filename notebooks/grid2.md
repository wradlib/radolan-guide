---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  name: python3
  display_name: Python 3
title: Grid - WGS84
subtitle: Understanding the spatial reference and projection of the RADOLAN composite grid.
---

# Grid - WGS84

```{note}
The following section is based on the **WGS84 ellipsoidal Earth model**. Projected X/Y coordinates differ slightly from the original sphere-based grids, while lon/lat coordinates remain largely comparable. We demonstrate how to reproject the sphere-based data onto the WGS84 grid to enable direct comparison. Statistical analyses, including differences and correlations, confirm that the physical precipitation fields are effectively unchanged: differences arise primarily from resampling and interpolation during reprojection, especially near steep gradients or convective cores.
```

```{code-cell} ipython3
import wradlib as wrl
import matplotlib.pyplot as pl
import numpy as np
import xarray as xr
from osgeo import osr
import cartopy.crs as ccrs
```

```{code-cell} ipython3
def corners(coords, crs):
    x = coords[0]
    y = coords[1]
    x, y = np.meshgrid(x, y)
    radolan_grid = np.dstack((x, y))
    ll = wrl.georef.reproject(radolan_grid, src_crs=crs, trg_crs=wrl.georef.get_default_projection())
    lon, lat = ll[..., 0], ll[..., 1]
    print(f"SW: {x[0, 0]}, {y[0, 0]} | {lon[0, 0]}, {lat[0, 0]}")
    print(f"SE: {x[0, -1]}, {y[0, 0]} | {lon[0, -1]}, {lat[0, -1]}")
    print(f"NE: {x[0, -1]}, {y[-1, 0]} | {lon[-1, -1]}, {lat[-1, -1]}")
    print(f"NW: {x[0, 0]}, {y[-1, 0]} | {lon[-1, 0]}, {lat[-1, 0]}")
    
```

## CRS and Corner Points

+++

### Original Sphere with km-scale

```{code-cell} ipython3
p_legacy = wrl.georef.create_crs("dwd-radolan")
print(p_legacy.to_wkt(pretty=True))

radolan_coords = wrl.georef.get_radolan_coordinates(nrows=900, ncols=900, mode="edge", crs=p_legacy)
corners(radolan_coords, p_legacy)
```

### Sphere

```{code-cell} ipython3
p_legacy_sphere = wrl.georef.create_crs("dwd-radolan-sphere")
print(p_legacy_sphere.to_wkt(pretty=True))

radolan_coords_sphere = wrl.georef.get_radolan_coordinates(nrows=900, ncols=900, mode="edge", crs=p_legacy_sphere)
corners(radolan_coords_sphere, p_legacy_sphere)
```

### WGS84

```{code-cell} ipython3
p_legacy_wgs84 = wrl.georef.create_crs("dwd-radolan-wgs84")
print(p_legacy_wgs84.to_wkt(pretty=True))

radolan_coords_wgs84 = wrl.georef.get_radolan_coordinates(nrows=900, ncols=900, mode="edge", crs=p_legacy_wgs84)
corners(radolan_coords_wgs84, p_legacy_wgs84)
```

## Special CRS with Coordinate System False Origins

+++

### RX 

900x900

```{code-cell} ipython3
p_rx_sphere = wrl.georef.create_crs("dwd-radolan-sphere-rx")
p_rx_wgs84 = wrl.georef.create_crs("dwd-radolan-wgs84-rx")
print(p_rx_sphere.to_wkt(pretty=True))
print(p_rx_wgs84.to_wkt(pretty=True))

radolan_coords_rx_sphere = wrl.georef.get_radolan_coordinates(nrows=900, ncols=900, mode="edge", crs=p_legacy_sphere)
corners(radolan_coords_rx_sphere, p_legacy_sphere)
print()
radolan_coords_rx_wgs84 = wrl.georef.get_radolan_coordinates(nrows=900, ncols=900, mode="edge", crs=p_legacy_wgs84)
corners(radolan_coords_rx_wgs84, p_legacy_wgs84)
```

### DE1200 

1200x1100

Examplarisch lautet der proj-String für das DE1200-Gitter im WGS84-Erdmodell:

`+proj=stere +lat_0=90 +lat_ts=60 +lon_0=10 +a=6378137 +b=6356752.3142451802 +no_defs +x_0=543196.83521776402 +y_0=3622588.8619310018`

For wradlib radolan xarray backend we keep the projection without any false offsets. The user should be able to directly use the attached coordinates without any projection handling.

```{code-cell} ipython3
p_de1200_sphere = wrl.georef.create_crs("dwd-radolan-sphere-de1200")
p_de1200_wgs84 = wrl.georef.create_crs("dwd-radolan-wgs84-de1200")
print(p_de1200_sphere.to_wkt(pretty=True))
print(p_de1200_wgs84.to_wkt(pretty=True))
p_legacy_sphere = wrl.georef.create_crs("dwd-radolan-sphere")
radolan_coords_de1200_sphere = wrl.georef.get_radolan_coordinates(nrows=1200, ncols=1100, mode="edge", crs=p_legacy_sphere)
corners(radolan_coords_de1200_sphere, p_legacy_sphere)
print()
radolan_coords_de1200_wgs84 = wrl.georef.get_radolan_coordinates(nrows=1200, ncols=1100, mode="edge", crs=p_legacy_wgs84)
corners(radolan_coords_de1200_wgs84, p_legacy_wgs84)
```

### DE4800 
4800x4400

```{code-cell} ipython3
p_de4800_sphere = wrl.georef.create_crs("dwd-radolan-sphere-de4800")
p_de4800_wgs84 = wrl.georef.create_crs("dwd-radolan-wgs84-de4800")
print(p_de4800_sphere.to_wkt(pretty=True))
print(p_de4800_wgs84.to_wkt(pretty=True))

p_legacy_sphere = wrl.georef.create_crs("dwd-radolan-sphere")
radolan_coords_de4800_sphere = wrl.georef.get_radolan_coordinates(nrows=4800, ncols=4400, mode="edge", crs=p_legacy_sphere)
corners(radolan_coords_de4800_sphere, p_legacy_sphere)
print()
radolan_coords_de4800_wgs84 = wrl.georef.get_radolan_coordinates(nrows=4800, ncols=4400, mode="edge", crs=p_legacy_wgs84)
corners(radolan_coords_de4800_wgs84, p_legacy_wgs84)
```

## POLARA - Grids

The POLARA team circulated an internal document `Komposits aus Polara-Produktion` which contains information on projection and grids. In the following section this information is used.

```{code-cell} ipython3
def grid_corner(xy, ll):
    x = xy[..., 0]
    y = xy[..., 1]
    lon = ll[..., 0]
    lat = ll[..., 1]
    print(f"SW: {x[0, 0]}, {y[0, 0]} | {lon[0, 0]}, {lat[0, 0]}")
    print(f"SE: {x[0, -1]}, {y[0, -1]} | {lon[0, -1]}, {lat[0, -1]}")
    print(f"NE: {x[-1, -1]}, {y[-1, -1]} | {lon[-1, -1]}, {lat[-1, -1]}")
    print(f"NW: {x[-1, 0]}, {y[-1, 0]} | {lon[-1, 0]}, {lat[-1, 0]}")
```

### RX 900x900

```{code-cell} ipython3
radolan_grid_rx_sphere_xy = wrl.georef.get_radolan_grid(nrows=900, ncols=900, wgs84=False, mode="edge", crs=p_legacy_sphere)
radolan_grid_rx_sphere_ll = wrl.georef.get_radolan_grid(nrows=900, ncols=900, wgs84=True, mode="edge", crs=p_legacy_sphere)
grid_corner(radolan_grid_rx_sphere_xy, radolan_grid_rx_sphere_ll)
radolan_grid_rx_wgs84_xy = wrl.georef.get_radolan_grid(nrows=900, ncols=900, wgs84=False, mode="edge", crs=p_legacy_wgs84)
radolan_grid_rx_wgs84_ll = wrl.georef.get_radolan_grid(nrows=900, ncols=900, wgs84=True, mode="edge", crs=p_legacy_wgs84)
print()
grid_corner(radolan_grid_rx_wgs84_xy, radolan_grid_rx_wgs84_ll)
```

### DE1200 1200x1100

```{code-cell} ipython3
radolan_grid_de1200_sphere_xy = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=False, mode="edge", crs=p_legacy_sphere)
radolan_grid_de1200_sphere_ll = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=True, mode="edge", crs=p_legacy_sphere)
grid_corner(radolan_grid_de1200_sphere_xy, radolan_grid_de1200_sphere_ll)
radolan_grid_de1200_wgs84_xy = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=False, mode="edge", crs=p_legacy_wgs84)
radolan_grid_de1200_wgs84_ll = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=True, mode="edge", crs=p_legacy_wgs84)
print()
grid_corner(radolan_grid_de1200_wgs84_xy, radolan_grid_de1200_wgs84_ll)
```

### DE4800 4800x4400

```{code-cell} ipython3
radolan_grid_de4800_sphere_xy = wrl.georef.get_radolan_grid(nrows=4800, ncols=4400, wgs84=False, mode="edge", crs=p_legacy_sphere)
radolan_grid_de4800_sphere_ll = wrl.georef.get_radolan_grid(nrows=4800, ncols=4400, wgs84=True, mode="edge", crs=p_legacy_sphere)
grid_corner(radolan_grid_de4800_sphere_xy, radolan_grid_de1200_sphere_ll)
radolan_grid_de4800_wgs84_xy = wrl.georef.get_radolan_grid(nrows=4800, ncols=4400, wgs84=False, mode="edge", crs=p_legacy_wgs84)
radolan_grid_de4800_wgs84_ll = wrl.georef.get_radolan_grid(nrows=4800, ncols=4400, wgs84=True, mode="edge", crs=p_legacy_wgs84)
print()
grid_corner(radolan_grid_de4800_wgs84_xy, radolan_grid_de4800_wgs84_ll)
```

## Example WN DE1200

In the transition period from sphere model to wgs84 ellipsoid model DWD released both versions. In the following example ``WN`` product data is used to showcase this.

```{code-cell} ipython3
SPHERE_FILE = "../data/grid/sphere/WN2204091200_000"
WGS84_FILE = "../data/grid/wgs84/WN2204091200_000"
ds_sphere = xr.open_dataset(SPHERE_FILE, engine="radolan").set_coords("crs")
ds_wgs84 = xr.open_dataset(WGS84_FILE, engine="radolan").set_coords("crs")
```

### Assign Coordinates

We take grid centerpoints as well as edge points.

```{code-cell} ipython3
radolan_grid_de1200_sphere_xy_cp = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=False, mode="center", crs=p_legacy_sphere)
radolan_grid_de1200_sphere_ll_cp = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=True, mode="center", crs=p_legacy_sphere)
radolan_grid_de1200_sphere_xy_sw = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=False, mode="edge", crs=p_legacy_sphere)
radolan_grid_de1200_sphere_ll_sw = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=True, mode="edge", crs=p_legacy_sphere)

radolan_grid_de1200_wgs84_xy_cp = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=False, mode="center", crs=p_legacy_wgs84)
radolan_grid_de1200_wgs84_ll_cp = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=True, mode="center", crs=p_legacy_wgs84)
radolan_grid_de1200_wgs84_xy_sw = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=False, mode="edge", crs=p_legacy_wgs84)
radolan_grid_de1200_wgs84_ll_sw = wrl.georef.get_radolan_grid(nrows=1200, ncols=1100, wgs84=True, mode="edge", crs=p_legacy_wgs84)
```

```{code-cell} ipython3
ds_sphere = ds_sphere.assign_coords(
    {
        "x_cp": (["y", "x"], radolan_grid_de1200_sphere_xy_cp[..., 0]),
        "y_cp": (["y", "x"], radolan_grid_de1200_sphere_xy_cp[..., 1]),
        "x_sw": (["y1", "x1"], radolan_grid_de1200_sphere_xy_sw[..., 0]),
        "y_sw": (["y1", "x1"], radolan_grid_de1200_sphere_xy_sw[..., 1]),
        "lon_cp": (["y", "x"], radolan_grid_de1200_sphere_ll_cp[..., 0]),
        "lat_cp": (["y", "x"], radolan_grid_de1200_sphere_ll_cp[..., 1]),
        "lon_sw": (["y1", "x1"], radolan_grid_de1200_sphere_ll_sw[..., 0]),
        "lat_sw": (["y1", "x1"], radolan_grid_de1200_sphere_ll_sw[..., 1]),

    }
    
)
ds_wgs84 = ds_wgs84.assign_coords(
    {
        "x_cp": (["y", "x"], radolan_grid_de1200_wgs84_xy_cp[..., 0]),
        "y_cp": (["y", "x"], radolan_grid_de1200_wgs84_xy_cp[..., 1]),
        "x_sw": (["y1", "x1"], radolan_grid_de1200_wgs84_xy_sw[..., 0]),
        "y_sw": (["y1", "x1"], radolan_grid_de1200_wgs84_xy_sw[..., 1]),
        "lon_cp": (["y", "x"], radolan_grid_de1200_wgs84_ll_cp[..., 0]),
        "lat_cp": (["y", "x"], radolan_grid_de1200_wgs84_ll_cp[..., 1]),
        "lon_sw": (["y1", "x1"], radolan_grid_de1200_wgs84_ll_sw[..., 0]),
        "lat_sw": (["y1", "x1"], radolan_grid_de1200_wgs84_ll_sw[..., 1]),

    }
)
```

### Visualize Selection

```{code-cell} ipython3
off = 4
sel_sphere = ds_sphere.isel(
    x=slice(470 - off, 470 + off), 
    y=slice(600 - off, 600 + off),
    x1=slice(470 - off, 470 + off + 1), 
    y1=slice(600 - off, 600 + off + 1),
)
sel_wgs84 = ds_wgs84.isel(
    x=slice(470 - off, 470 + off), 
    y=slice(600 - off, 600 + off),
    x1=slice(470 - off, 470 + off + 1), 
    y1=slice(600 - off, 600 + off + 1),
)
```

### Visualize Grid Corners

```{code-cell} ipython3
def plot_grid(ds, coords, corner, ax, **kwargs):
    if coords == "longlat":
        x1 = "lon_sw"
        y1 = "lat_sw"
        x2 = "lon_cp"
        y2 = "lat_cp"
    else:
        x1 = "x_sw"
        y1 = "y_sw"
        x2 = "x_cp"
        y2 = "y_cp"
    
    if corner == "SW":
        dsp1 = ds.isel(x1=slice(0, 10), y1=slice(0, 10))
        dsp2 = ds.isel(x=slice(0, 9), y=slice(0, 9))
        dsp = ds.isel(x1=slice(0, 1), y1=slice(0, 1))
    elif corner == "SE":
        dsp1 = ds.isel(x1=slice(-10, None), y1=slice(0, 10))
        dsp2 = ds.isel(x=slice(-9, None), y=slice(0, 9))
        dsp = ds.isel(x1=slice(-1, None), y1=slice(0, 1))
    elif corner == "NE":
        dsp1 = ds.isel(x1=slice(-10, None), y1=slice(-10, None))
        dsp2 = ds.isel(x=slice(-9, None), y=slice(-9, None))
        dsp = ds.isel(x1=slice(-1, None), y1=slice(-1, None))
    elif corner == "NW":
        dsp1 = ds.isel(x1=slice(0, 10), y1=slice(-10, None))
        dsp2 = ds.isel(x=slice(0, 9), y=slice(-9, None))
        dsp = ds.isel(x1=slice(0, 1), y1=slice(-1, None))
    else:
        corner = "CN"
        dsp1 = ds.isel(x1=slice(466, 475), y1=slice(596, 605))
        dsp2 = ds.isel(x=slice(466, 474), y=slice(596, 604))
        dsp = ds.isel(x1=slice(470, 471), y1=slice(600, 601))
        
    dsp1.plot.scatter(x=x1, y=y1, marker="x", ax=ax, **kwargs)
    dsp2.plot.scatter(x=x2, y=y2, marker=".", ax=ax, **kwargs)
    dsp.plot.scatter(x=x1, y=y1, marker="X", s=120, ax=ax, 
                     label=f"{dsp[x1].values.item():.10g}, {dsp[y1].values.item():.10g}", 
                     **kwargs)
    
    lon = dsp1[x1].values
    lat = dsp1[y1].values
    ax.plot(lon, lat, **kwargs)
    ax.plot(lon.T, lat.T, **kwargs)
    ax.grid(
        True,
        which="both",
        linestyle="--",
        linewidth=0.5,
        alpha=0.7
    )

    ax.set_title(f"{corner}")
    ax.legend()
```

```{code-cell} ipython3
fig, ((ax0, ax1), (ax2, ax3)) = pl.subplots(2, 2, figsize=(16, 16))

plot_grid(ds_sphere, "longlat", "NW", ax0, color="lightblue")
plot_grid(ds_wgs84, "longlat", "NW", ax0, color="orange")

plot_grid(ds_sphere, "longlat", "NE", ax1, color="lightblue")
plot_grid(ds_wgs84, "longlat", "NE", ax1, color="orange")

plot_grid(ds_sphere, "longlat", "SW", ax2, color="lightblue")
plot_grid(ds_wgs84, "longlat", "SW", ax2, color="orange")

plot_grid(ds_sphere, "longlat", "SE", ax3, color="lightblue")
plot_grid(ds_wgs84, "longlat", "SE", ax3, color="orange")

fig.tight_layout()
```

### Visualize Grid Center

```{code-cell} ipython3
fig = pl.figure(figsize=(8, 8))
ax = fig.add_subplot(111)
plot_grid(ds_sphere, "longlat", "CN", ax, color="lightblue")
plot_grid(ds_wgs84, "longlat", "CN", ax, color="orange")
fig.tight_layout()
```

### Visualize Map projection

```{code-cell} ipython3
map_proj_wgs84 = ccrs.Stereographic(true_scale_latitude=60., central_latitude=90., central_longitude=10.,
                                    false_easting=0., false_northing=0.)

sphere = ccrs.Globe(ellipse="sphere", semimajor_axis="6370040", semiminor_axis="6370040")
map_proj_sphere = ccrs.Stereographic(true_scale_latitude=60., central_latitude=90., central_longitude=10., 
                                     globe=sphere,
                                     false_easting=0, false_northing=0)
cbar_kwargs={
        "pad": 0.1,
    }
```

#### 1D XY Coordinates

```{code-cell} ipython3
fig, (ax0, ax1) = pl.subplots(2, 1, figsize=(10, 10), subplot_kw=dict(projection=ccrs.PlateCarree()))

sel_sphere.WN.plot(x="x", y="y", ax=ax0, transform=map_proj_sphere, cmap="viridis", cbar_kwargs=cbar_kwargs)
ax0.scatter(sel_sphere.x, sel_sphere.y, c="r", marker="o", transform=map_proj_sphere, label="sphere - cp")
ax0.gridlines(draw_labels=True)

sel_wgs84.WN.plot(x="x", y="y", ax=ax1, transform=map_proj_wgs84, cmap="viridis", cbar_kwargs=cbar_kwargs)
ax1.scatter(sel_wgs84.x, sel_wgs84.y, c="b", marker="x", transform=map_proj_wgs84, label="wgs84")
ax1.gridlines(draw_labels=True)

fig.tight_layout()
```

#### 2D Longlat Coordinates

```{code-cell} ipython3
fig, (ax0, ax1) = pl.subplots(2, 1, figsize=(10, 10), subplot_kw=dict(projection=ccrs.PlateCarree()))

# plot for wgs84 grid
sel_sphere.WN.plot(x="lon_cp", y="lat_cp", ax=ax0, infer_intervals=True, cmap="viridis", cbar_kwargs=cbar_kwargs)
ax0.scatter(sel_sphere.lon_cp, sel_sphere.lat_cp, c="r", marker="o", label="sphere - cp")
ax0.scatter(sel_sphere.lon_sw, sel_wgs84.lat_sw, c="b", marker="x", label="sphere - sw")
ax0.set_title(f"Sphere")
ax0.gridlines(draw_labels=True)
ax0.legend()

# plot for sphere grid
sel_wgs84.WN.plot(x="lon_cp", y="lat_cp", ax=ax1, infer_intervals=True, cmap="viridis", cbar_kwargs=cbar_kwargs)
ax1.scatter(sel_wgs84.lon_cp, sel_wgs84.lat_cp, c="y", marker="o", label="wgs84 - cp")
ax1.scatter(sel_wgs84.lon_sw, sel_wgs84.lat_sw, c="c", marker="x", label="wgs84 - sw")
ax1.set_title(f"WGS84")
ax1.gridlines(draw_labels=True)
ax1.legend()

fig.suptitle("Plotting via longlat coordinates")
fig.tight_layout()
```

#### 2D XY Coordinates

```{code-cell} ipython3
fig, (ax0, ax1) = pl.subplots(2, 1, figsize=(10, 10), subplot_kw=dict(projection=ccrs.PlateCarree()))

# plot for wgs84 grid
sel_sphere.WN.plot(x="x_cp", y="y_cp", ax=ax0, transform=map_proj_sphere, infer_intervals=True, cmap="viridis", cbar_kwargs=cbar_kwargs)
ax0.scatter(sel_sphere.x_cp, sel_sphere.y_cp, c="r", marker="o", transform=map_proj_sphere, label="sphere - cp")
ax0.scatter(sel_sphere.x_sw, sel_sphere.y_sw, c="b", marker="x", transform=map_proj_sphere, label="sphere - sw")
ax0.set_title(f"Sphere")
ax0.gridlines(draw_labels=True)
ax0.legend()

# plot for sphere grid
sel_wgs84.WN.plot(x="x_cp", y="y_cp", ax=ax1,  transform=map_proj_wgs84, infer_intervals=True, cmap="viridis", cbar_kwargs=cbar_kwargs)
ax1.scatter(sel_wgs84.x_cp, sel_wgs84.y_cp, c="y", marker="o",  transform=map_proj_wgs84, label="wgs84 - cp")
ax1.scatter(sel_wgs84.x_sw, sel_wgs84.y_sw, c="c", marker="x",  transform=map_proj_wgs84, label="wgs84 - sw")
ax1.set_title(f"WGS84")
ax1.gridlines(draw_labels=True)
ax1.legend()

fig.suptitle("Plotting via xy coordinates")
fig.tight_layout()
```

## Project back and forth

using rioxarray

+++

### Fix NaN and CRS

```{code-cell} ipython3
import rioxarray
import numpy as np

ds_sphere_x = ds_sphere.rio.write_crs(p_legacy_sphere, "spatial_ref")

da = ds_sphere_x["WN"]
da.attrs.pop("_FillValue", None)
da.encoding.pop("_FillValue", None)
da = da.rio.write_nodata(-9999)
ds_sphere_x["WN"] = da

ds_wgs84_x  = ds_wgs84.rio.write_crs(p_legacy_wgs84, "spatial_ref")

da = ds_wgs84_x["WN"]
da.attrs.pop("_FillValue", None)
da.encoding.pop("_FillValue", None)
da = da.rio.write_nodata(-9999)
ds_wgs84_x["WN"] = da
```

### Reproject

```{code-cell} ipython3
sphere_on_wgs = ds_sphere_x["WN"].rio.reproject_match(
    ds_wgs84_x["WN"],
    resampling="bilinear"
)
ds_wgs84_x = ds_wgs84_x.assign(WNX=sphere_on_wgs)

wgs_on_sphere = ds_wgs84_x["WN"].rio.reproject_match(
    ds_sphere_x["WN"],
    resampling="bilinear"
)
ds_sphere_x = ds_sphere_x.assign(WNX=wgs_on_sphere)
```

### Calculate Differences

```{code-cell} ipython3
diff_wgs84 = ds_wgs84_x["WN"] - ds_wgs84_x["WNX"]
ds_wgs84_x = ds_wgs84_x.assign(DIFF=diff_wgs84)
abs_diff_wgs84 = np.abs(diff_wgs84)
ds_wgs84_x = ds_wgs84_x.assign(ABSDIFF=abs_diff_wgs84)
print("Mean difference:", float(diff_wgs84.mean()))
print("Mean absolute difference:", float(abs_diff_wgs84.mean()))
print("Max absolute difference:", float(abs_diff_wgs84.max()))
print("Std difference:", float(diff_wgs84.std()))
```

```{code-cell} ipython3
diff_sphere = ds_sphere_x["WN"] - ds_sphere_x["WNX"]
ds_sphere_x = ds_sphere_x.assign(DIFF=diff_sphere)
abs_diff_sphere = np.abs(diff_sphere)
ds_sphere_x = ds_sphere_x.assign(ABSDIFF=abs_diff_sphere)
print("Mean difference:", float(diff_sphere.mean()))
print("Mean absolute difference:", float(abs_diff_sphere.mean()))
print("Max absolute difference:", float(abs_diff_sphere.max()))
print("Std difference:", float(diff_sphere.std()))
```

### Visualize at Grid Center

```{code-cell} ipython3
fig, ((ax0, ax1), (ax2, ax3), (ax4, ax5), (ax6, ax7) ) = pl.subplots(4, 2, figsize=(16, 24), sharex=True, sharey=True)

ds_sphere_x["WN"].isel(x=slice(466, 474), y=slice(596, 604)).plot(x="lon_cp", y="lat_cp", ax=ax0, vmin=-32, vmax=40, cmap="viridis")
ax0.set_title("Sphere WN")
ds_wgs84_x["WN"].isel(x=slice(466, 474), y=slice(596, 604)).plot(x="lon_cp", y="lat_cp", ax=ax1, vmin=-32, vmax=40, cmap="viridis")
ax1.set_title("WGS84 WN")
ds_sphere_x["WNX"].isel(x=slice(466, 474), y=slice(596, 604)).plot(x="lon_cp", y="lat_cp", ax=ax2, vmin=-32, vmax=40, cmap="viridis")
ax2.set_title("WGS84 WN reprojected to Sphere")
ds_wgs84_x["WNX"].isel(x=slice(466, 474), y=slice(596, 604)).plot(x="lon_cp", y="lat_cp", ax=ax3, vmin=-32, vmax=40, cmap="viridis")
ax3.set_title("Sphere WN reprojected to WGS84")
ds_sphere_x["DIFF"].isel(x=slice(466, 474), y=slice(596, 604)).plot(x="lon_cp", y="lat_cp", ax=ax4, vmin=-15, vmax=15, cmap="seismic")
ax4.set_title("DIFF of the above")
ds_wgs84_x["DIFF"].isel(x=slice(466, 474), y=slice(596, 604)).plot(x="lon_cp", y="lat_cp", ax=ax5, vmin=-15, vmax=15, cmap="seismic")
ax5.set_title("DIFF of the above")
ds_sphere_x["ABSDIFF"].isel(x=slice(466, 474), y=slice(596, 604)).plot(x="lon_cp", y="lat_cp", ax=ax6, vmin=0, vmax=15, cmap="viridis")
ax6.set_title("ABSDIFF of the above")
ds_wgs84_x["ABSDIFF"].isel(x=slice(466, 474), y=slice(596, 604)).plot(x="lon_cp", y="lat_cp", ax=ax7, vmin=0, vmax=15, cmap="viridis")
ax7.set_title("ABSDIFF of the above")

fig.tight_layout()
```

### Visualize Complete Grid

```{code-cell} ipython3
fig, ((ax0, ax1), (ax2, ax3), (ax4, ax5), (ax6, ax7)) = pl.subplots(4, 2, figsize=(16, 24), sharex=True, sharey=True)

ds_sphere_x["WN"].plot(x="lon_cp", y="lat_cp", ax=ax0, vmin=-32, vmax=50)
ax0.set_title("Sphere WN")
ds_wgs84_x["WN"].plot(x="lon_cp", y="lat_cp", ax=ax1, vmin=-32, vmax=50)
ax1.set_title("WGS84 WN")
ds_sphere_x["WNX"].plot(x="lon_cp", y="lat_cp", ax=ax2, vmin=-32, vmax=50)
ax2.set_title("WGS84 WN reprojected to Sphere")
ds_wgs84_x["WNX"].plot(x="lon_cp", y="lat_cp", ax=ax3, vmin=-32, vmax=50)
ax3.set_title("Sphere WN reprojected to WGS84")
ds_sphere_x["DIFF"].plot(x="lon_cp", y="lat_cp", ax=ax4, vmin=-15, vmax=15, cmap="seismic")
ax4.set_title("DIFF of the above")
ds_wgs84_x["DIFF"].plot(x="lon_cp", y="lat_cp", ax=ax5, vmin=-15, vmax=15, cmap="seismic")
ax5.set_title("DIFF of the above")
ds_sphere_x["ABSDIFF"].plot(x="lon_cp", y="lat_cp", ax=ax6, vmin=0, vmax=15, cmap="viridis")
ax6.set_title("ABSDIFF of the above")
ds_wgs84_x["ABSDIFF"].plot(x="lon_cp", y="lat_cp", ax=ax7, vmin=0, vmax=15, cmap="viridis")
ax7.set_title("ABSDIFF of the above")

fig.tight_layout()
```
