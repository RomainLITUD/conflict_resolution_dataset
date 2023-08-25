# A Comparative Conflict Resolution Dataset Derived from Argoverse-2: Scenarios with vs. without Autonomous Vehicles

Authors: Guopeng Li, Yiru Jiao, and Hans van Lint

- This dataset is derived from the open Argoverse-2 motion forecasting data: [https://www.argoverse.org/av2.html#forecasting-link]
- The dataset, 2 metadata files, and the code to read and visualize the data can be downloaded through the 4.TU data platform:

![Conflict Regimes](https://github.com/RomainLITUD/conflict_resolution_dataset/blob/main/regimes.png)

## Quick to start
Reading and visualizing the dataset requires the following packages"
```
python >=3.8
numpy
scipy>=0.17.0
zarr
h5py
matplotlib
tqdm (optional)
```
- Put the `visual_utils.py` in the root dir.
- Unzip the downloaded `data.zip` file to the root dir.
- The dataset is stored in a compact `zarr` form so even a 'potato' computer can handle this.

## Read and visualize data

- Import the necessary packages:
````python
import zarr
import os
from visual_utils import *

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
        -1: Static background
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
slices, timestep, motion, type, maps = read_scenario(log_ids[2441], root)
````

- Visualize the data
````python
# Visualize the 2441-st scenario
# the parameter other_road_users = True/False controls whether to plot the surrounding agents (True by default)
# The parameter direction = True/False controls whether to plot the start and end directions of the vehicles involved in the conflict (True by default)
fig, ax = visualize(log_ids[2441], root, other_road_users=True, direction=True)
````

## About metadata file

The metadata files is a table, shown as the example below, for AV-involved scenarios (`metafile_av.csv`) and AV-free scenarios (`metafile_hv.csv`), respectively.

| log_id | xi_start | yi_start | xj_start | yj_start | typei | xi_end | yi_end | xj_end | yj_end | typej | direction | PET | ifirst | angle_start | angle_end | start | end |
|     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |     :---:      |
| 2441 | -0.028310 | 0.999599 | -0.278388 | -0.960469 | AV | -0.001506 | 0.999999 | 0.998085 | 0.061850 | HV | R-L | 3.7 | False | 162.213724 | 86.540315 | opposite | cross |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

These tables provide the following information about each scenario:
| column | dtype | information |
|-------|-------|-------|
| `log_id` | str | index of the scenario |
| `[xi_start, yi_start]` | float | direction vector of the first* agent recorded in the scenario at the start** time |
| `[xj_start, yj_start]` | float | direction vector of the second agent recorded in the scenario at the start time |
| `typei` | str | agent type of the first agent recorded in the scenario, being one of {'AV','HV','Pedestrian','Motorcyclist','Cyclist','Bus'} |
| `[xi_end, yi_end]` | float | direction vector of the first* agent recorded in the scenario at the end*** time |
| `[xj_end, yj_end]` | float | direction vector of the second agent recorded in the scenario at the end time |
| `typej` | str | agent type of the second agent recorded in the scenario, being one of {'AV','HV','Pedestrian','Motorcyclist','Cyclist','Bus'} |
| `direction` | str | whether the second-passing vehicle moved from the left ('L-R') or the right ('R-L') of the first-passing agent |
| `PET` | float | post-encroachment-time |
| `ifirst` | bool | whether the first-passing agent is the first agent recorded in the scenario |
| `angle_start` | float | angle between the direction vectors of the two agents at the start time |
| `angle_end` | float | angle between the direction vectors of the two agents at the end time |
| `start` | str | whether the two agents ran parallel (P), crossed (C), or ran opposite (O) to each other before reaching the conflict point |
| `end` | str | whether the two agents ran parallel (P), crossed (C), or ran opposite (O) to each other after reaching the conflict point |
|| **Notes** | * Note that the first agent does not necessarily pass the conflict point first. <br />** We consider the start time as 5 seconds before the first-passing agent passed the conflict point, or the start of the record if the time before passing the conflict point is less than 5 seconds. <br />*** Similarly, the end time is 5 seconds after the second-passing vehicle passed the conflict point, or the end of the record if the time after passing the conflict point is less than 5 seconds. |


One can use the following code to select data in different conditions:
````python
# AV passing the conflict point first
case_av_first = metafile_av[(metafile_av['ifirst'])] # In cases involving AV, the AV is the first recorded agent

# Pedestrian passing the conflict point second
case_ped_first_av_free = metafile_hv[(~metafile_hv['ifirst'])&(metafile_hv['typej']=='Pedestrian')]
````
