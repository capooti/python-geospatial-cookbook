# Switch cell values in a raster

I have [this single band raster](../data/raster_to_update.tif) . Its datatype is uint8.
Its cell values can be 0,1,2,3,4.
The raster came with 1 and 2 inverted, so I want to change all cells with 1s to 2s and viceversa.

Here is a possible workflow using Rasterio and Numpy.

Here is my raster and its main metadata:

```python
import rasterio

ds_path = '../data/raster_to_update.tif'
ds = rasterio.open(ds_path)
arr = ds.read(1)
ds.shape, ds.dtypes
```

```
((1439, 1560), ('uint8',))
```

Let's check the unique values for the cells and their counts:

```python
import numpy as np
np.unique(arr, return_counts=True)
```

```
(array([0, 1, 2, 3, 4], dtype=uint8),
 array([1780986,  153547,  196223,   86853,   27231]))
```

Now, using numpy, I am flipping all cell with 1's to 2's and viceversa:

```python
arr[arr==1] = 201
arr[arr==2] = 202
arr[arr==201] = 2
arr[arr==202] = 1
```

It is easy to see that the frequency of 1's and 2's has correctly changed:

```python
np.unique(arr, return_counts=True)
```

```
(array([0, 1, 2, 3, 4], dtype=uint8),
 array([1780986,  196223,  153547,   86853,   27231]))
```

Finally, I can write results to an output tif:

```python
out_path = '/tmp/out.tiff'
with rasterio.open(
        out_path,
        'w',
        driver='GTiff',
        height=arr.shape[0],
        width=arr.shape[1],
        count=1,
        dtype=rasterio.uint8,
        crs=rasterio.crs.CRS.from_epsg(4326),
        transform=ds.transform,
    ) as dst:
    dst.write(arr, 1)
dst.close()
```