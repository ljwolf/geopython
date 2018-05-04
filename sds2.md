# Exploratory Spatial Data Analysis


## Imports

We also need esda and mapclassify for this notebook.

```python
import pandas as pd
import geopandas as gpd
import libpysal.api as lp
import matplotlib.pyplot as plt
import rasterio as rio
import numpy as np
import contextily as ctx
import shapely.geometry as geom
%matplotlib inline
```

```python
df = gpd.read_file('data/neighborhoods.shp')
# was created in previous notebook with df.to_file('data/neighborhoods.shp')
```

```python
df.head()
```

We have an `nan` to first deal with:

```python
pd.isnull(df['median_pri']).sum()
```

```python
df = df
df['median_pri'].fillna((df['median_pri'].mean()), inplace=True)

```

## Attribute Distribution (a-spatial)


```python
import seaborn as sbn
```

```python

sbn.distplot(df['median_pri'])
```

## Spatial Distribution - Geovisualization Revisited

```python
df.plot(column='median_pri')
```


```python
fig, ax = plt.subplots(figsize=(12,10), subplot_kw={'aspect':'equal'})
df.plot(column='median_pri', scheme='Quantiles', legend=True, ax=ax)
#ax.set_xlim(150000, 160000)
#ax.set_ylim(208000, 215000)
```

```python
fig, ax = plt.subplots(figsize=(12,10), subplot_kw={'aspect':'equal'})
df.plot(column='median_pri', scheme='Quantiles', k=4, legend=True, ax=ax)
#ax.set_xlim(150000, 160000)
#ax.set_ylim(208000, 215000)
```


```python
import mapclassify.api as mc
```

```python
y = df['median_pri']
mb4 = mc.Maximum_Breaks(y, k=4)
```

```python
mb4
```

```python
mb4.yb
```

```python
df['mb4'] = mb4.yb
```

```python
fig, ax = plt.subplots(figsize=(12,10), subplot_kw={'aspect':'equal'})
df.plot(column='mb4',  legend=True, ax=ax)
#ax.set_xlim(150000, 160000)
#ax.set_ylim(208000, 215000)
```


```python
f, ax = plt.subplots(1, figsize=(9, 9))
df.assign(cl=mb4.yb).plot(column='cl', categorical=True, \
                                      k=4, cmap='OrRd', linewidth=0.1, ax=ax,\
                                      edgecolor='grey', legend=True)
ax.set_axis_off()
plt.show()
```

```python
mb = mc.Maximum_Breaks
k = 7
f, ax = plt.subplots(1, figsize=(9, 9))
df.assign(cl=mb(y, k=k).yb).plot(column='cl', categorical=True, \
                                      k=k, cmap='OrRd', linewidth=0.1, ax=ax,\
                                      edgecolor='grey', legend=True)
ax.set_axis_off()
plt.show()
```

```python
for k in range(4, 9):
    
    f, ax = plt.subplots(1, figsize=(12, 10))
    df.assign(cl=mb(y, k=k).yb).plot(column='cl', categorical=True, \
                                      k=k, cmap='OrRd', linewidth=0.1, ax=ax,\
                                      edgecolor='grey', legend=True)
    ax.set_axis_off()
    plt.show()
```

```python
Q = mc.Quantiles
for k in range(4, 9):
    
    f, ax = plt.subplots(1, figsize=(12, 10))
    df.assign(cl=Q(y, k=k).yb).plot(column='cl', categorical=True, \
                                      k=k, cmap='OrRd', linewidth=0.1, ax=ax,\
                                      edgecolor='grey', legend=True)
    ax.set_axis_off()
    plt.show()
```


## Spatial Autocorrelation

### Binary Case

```python
y.median()
```

```python
yb = y > y.median()
df['yb'] = yb
```

```python
fig, ax = plt.subplots(figsize=(12,10), subplot_kw={'aspect':'equal'})
df.plot(column='yb', scheme='Quantiles', k=2, legend=True, ax=ax)
#ax.set_xlim(150000, 160000)
#ax.set_ylim(208000, 215000)
```

```python
import esda 
wq =  lp.Queen.from_dataframe(df)
wq.transform = 'b'
jc = esda.join_counts.Join_Counts(yb, wq)
```

```python
jc.bb
```

```python
jc.ww
```

```python
jc.bw
```

```python
jc.p_sim_bb
```

```python
jc.mean_bb
```


### Continuous Case


```python
wq.transform = 'r'
```

```python
y = df['median_pri']
```

```python
mi = esda.moran.Moran(y, wq)
```

```python
mi.EI
```


```python
mi.p_sim
```

```python
mi.I
```

```python
esda.moran.Moran??
```

```python
wq.s0
```

```python
miy = esda.moran.Moran(y, wq)
```

```python
miy.I
```

```python
miy.p_sim
```

```python
miy.EI
```


### Local Autocorrelation: Hot and Cold Spots

```python
np.random.seed(12345)
import esda
```

```python
wq.transform = 'r'
lag_price = lp.lag_spatial(wq, df['median_pri'])

```

```python
price = df['median_pri']
b, a = np.polyfit(price, lag_price, 1)
f, ax = plt.subplots(1, figsize=(9, 9))

plt.plot(price, lag_price, '.', color='firebrick')

 # dashed vert at mean of the price
plt.vlines(price.mean(), lag_price.min(), lag_price.max(), linestyle='--')
 # dashed horizontal at mean of lagged price 
plt.hlines(lag_price.mean(), price.min(), price.max(), linestyle='--')

# red line of best fit using global I as slope
plt.plot(price, a + b*price, 'r')
plt.title('Moran Scatterplot')
plt.ylabel('Spatial Lag of Price')
plt.xlabel('Price')
plt.show()

```

```python
li = esda.moran.Moran_Local(y, wq)
```

```python
pd.isnull(y).sum()
```

```python
li.geoda_quads
```

```python
li.quads
```

```python
li.q
```

```python
(li.p_sim < 0.06).sum()
```

```python
df['median_pri'] = y
```

```python
df.plot(column='median_pri', scheme='quantiles')
```

```python
df.plot(column='median_pri', scheme='quantiles', k=6)
```

```python
sig = li.p_sim < 0.01
hotspot = sig * li.q==1
coldspot = sig * li.q==3
```

```python
df = df
from matplotlib import colors
hmap = colors.ListedColormap(['grey', 'red'])
f, ax = plt.subplots(1, figsize=(9, 9))
df.assign(cl=hotspot*1).plot(column='cl', categorical=True, \
        k=2, cmap=hmap, linewidth=0.1, ax=ax, \
        edgecolor='grey', legend=True)
ax.set_axis_off()
plt.show()
```

```python
df = df
from matplotlib import colors
hmap = colors.ListedColormap(['grey', 'blue'])
f, ax = plt.subplots(1, figsize=(9, 9))
df.assign(cl=coldspot*1).plot(column='cl', categorical=True, \
        k=2, cmap=hmap, linewidth=0.1, ax=ax, \
        edgecolor='grey', legend=True)
ax.set_axis_off()
plt.show()
```

```python
coldspot.sum()
```
