# A Comparative Conflict Resolution Dataset Derived from Argoverse-2: Scenarios with vs. without Autonomous Vehicles

Authors: Guopeng Li, Yiru Jiao, and Hans van Lint

- This dataset is derived from the open Argoverse-2 motion forecasting data: [https://www.argoverse.org/av2.html#forecasting-link]
- The dataset, a metadata file, and the code to read and visualize the data can be downloaded through the 4.TU data platform:

## Quick to start
Reading and visualizing the dataset requires the following packages"
```
python >=3.8
numpy
zarr
h5py
matplotlib
tqdm (optional)
```
- Put the `math_utils.py` in the root dir.
- Unzip the downloaded `data.zip` file to the root dir.
- The dataset is stored in a compact `zarr` form so even a 'potato' computer can handle this.

## Read and visualize data

- Import the necessary packages:
````python
import zarr
import os
from math_utils import *

folder = 'av' # or 'hv'

root ='./data/'+folder+'/'

log_ids = [name for name in os.listdir(root) if os.path.isdir(os.path.join(root, name))]
print('Number of scenarios: ', len(log_ids))
````

- Read the data
````python
'''
slices: len = nb_objects + 1, slices[n] and slices[n+1] gives the start/end indices of the n-th object
maps: lanes as NumPy array
type: len = nb_objects, contains 7 numbers with the following meanings:
        -1: static background
        0: human-driven vehicles
        1: pedestrians
        2: motorcyclists
        3: cyclists
        4: buses
        10: autonomous vehicles
timestep: timestamps in second, timestep[slices[n]: slices[n+1]] give the timestamps for the n-th object
motion: motion state, with 7 dimensions
    motion[slices[n]: slices[n+1]] gives the motion of the n-th object, the 7 features are the following variables in order:
        [x, y, vx, vy, ax, ay, yaw]
        yaw is to the x-axis, between [-pi, pi]
'''
slices, timestep, motion, type, maps = read_scenario(log_ids[7641], root)
````

- Visualize the data
````python
# visualize the 2441-st scenario, the last parameter True/False means whether plot the surrounding agents
fig, ax = visualize(log_ids[2441], root, other_road_users=False) # other_road_users is True by default
````

## About metadata file

In the metadata file, we provide the following information about each scenario:

| case_id | A | C | D | F | Fa | Fd | S |regime_comb |
|     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |
|48 | 4.3 |15.7 | 0.2 |19.6 | 6.5 | 0.6 | 1.2 | FaCADFSFd |
