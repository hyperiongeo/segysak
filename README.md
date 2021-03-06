[![Actions Status](https://github.com/trhallam/segysak/workflows/python_38/badge.svg)](https://github.com/trhallam/segysak/actions)
[![Actions Status](https://github.com/trhallam/segysak/workflows/python_37/badge.svg)](https://github.com/trhallam/segysak/actions)
[![Actions Status](https://github.com/trhallam/segysak/workflows/python_36/badge.svg)](https://github.com/trhallam/segysak/actions)


# Documentation

Access the documentation for __segysak__ from [readthedocs](https://segysak.readthedocs.io/en/latest/)


# SEGY-SAK <img src="https://github.com/trhallam/segysak/blob/master/docs/figures/logo.png" alt="Logo" title="Logo" width="400" align="right" />

SEGY-SAK aims to be your Python Swiss Army Knife for Seismic Data.

To do this SEGY-SAK offers two things; a commandline interface (CLI) for inspecting and converting SEGY data to a more friendly format called
NETCDF4, and by providing convienience functions for the data using [`xarray`](http://xarray.pydata.org/en/stable/). We try hard to load the
data the same way everytime so your functions will work no-matter which cube/line you load. The `xarray` conventions we use are outlined
in the documentation.

Why NETCDF4? Well, NETCDF4 is a fancy type of enclosed file binary format that allows for faster indexing and data retreival than
SEGY. We try our best to scan in the header information and to make it easy (or easier) to load SEGY in different formats,
different configuration (2D, 2D gathers, 3D, 3D gathers). We do all this with the help of [`segyio`](https://github.com/equinor/segyio)
which is a lower level interface to SEGY. If you stick to our `xarray` format of files we also offer utility functions to
return to SEGY so you can export to other software.

## Current Capabilities

* CLI
  * Convert 2D, 3D and gathers type SEGY to NETCDF4 and back. The NETCDF4 files are one line open with `xarray.open_dataset`.
  * Extract sub-volumes via cropping xline and inline.
  * Read EBCIDC header.
  * Perform a limited header scan.

* Xarray and Python API
  * Load 2D, 3D and gathers type SEGY to Xarray.
  * Access header information and text headers in Python with conveience functions.
  * Select traces by X and Y coordinates.

## Installation
SEGY-SAK can be installed by using pip or python setuptools.

### `pip`
From the command line run the `pip` package manager
```
pip install segysak
```

### Install from source
Clone the SEGY-SAK Github repository and in the top level directory run setuptools via
```
python setup.py install
```

## CLI Quick Start

The command line interface (CLI) provides an easy tool to convert or manipulate SEG-Y data. In your Python command-line
environment it can be accessed by calling `segysak`.

See `segysak --help` for options.

SEG-Y files converted using `segysak convert test.segy` for example can be loaded into Python using `xarray`.
```
test = xarray.open_dataset('test.SEISNC')
```

## `xarray` seismic specification
The `xarray` seismic specification used by segysak to output NETCDF4 files is more performant for Python operations than
standard SEG-Y. Unlike SEG-Y, `xarray` compatable files fit neatly into the Python scientific stack providing operations
like lazy loading, easy slicing, compatability with multi-core and multi-node operations using `dask` as well as
important features such as labelled axes and coordinates.

This specification is not meant to be prescriptive but outlines some basic requirements for SEGYSAK to work.

SEGY-SAK uses the convention `.SEISNC` for the suffix on NETCDF4 files it creates. These files are datasets with specific 1D
and 2D variables and a single 3D variable called `data`. The data variable contains the seismic cube volume and other 
variables are used to describe the coordinate space. Attributes can be used to provide further metadata about the cube.

### 3D and 3D Gathers
SEGY-SAK uses the convention labels of `iline`, `xline` and `offset` to describe the bins of 3D data. Vertical dimensions
are `twt` and `depth`. A typical `xarray` dataset created by SEGY-SAK will return for example
```
>>> seisnc_3d = segysak.segy_loader('test3d.sgy', iline=189, xline=193)
>>> seisnc_3d.dims
Frozen(SortedKeysDict({'iline': 61, 'xline': 202, 'twt': 850}))
```

### 2D and 2D Gathers
For 2D data SEGY-SAK uses the dimensino labels `cdp` and `offset`. This allows the package to distinguish between 2D and 3D
data to allow automation on saving and convience wrappers. The same vertical dimensions apply as for 3D. A typical `xarray`
in 2D format would return
```
>>> seisnc_2d = segysak.segy_loader('test2d.sgy', cdp=21)
>>> seisnc_2d.dims
Frozen(SortedKeysDict({'cdp': 61, 'twt': 850}))
```

### Coordinates
If the `cdpx` and `cdpy` byte locations are specified during loading the SEGY the coordinates will be populated from the
headers with the variable names `cdp_x` and `cdp_y`. These will have dimensions equivalent to the horizontal dimensions
of the data (`iline`, `xline` for 3D and `cdp` for 2D).

### Attributes
Any number of attributes can be added to a SEISNC file. Currently the following attributes are extracted or reserved for
use by SEGY-SAK. 
 * `ns` number of samples per trace
 * `ds` sample interval
 * `text` ebcidc header as ascii text
 * `d3_units` vertical units of the data
 * `d3_domain` vertical domain of the data
 * `epsg` data epsg code
 * `corner_points` corner points of the dataset in grid coordinates
 * `corner_points_xy` corner points of the dataset in xy
 * `source_file` name of the file the dataset was created from
 * `srd` seismic reference datum of the data in vertical units `d3_units` and `d3_domain`
 * `datatype` the data type e.g. amplitude, velocity, attribute
