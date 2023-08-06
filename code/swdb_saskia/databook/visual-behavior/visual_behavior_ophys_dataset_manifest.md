# Identifying experiments and sessions of interest using the data manifest

This Jupyter notebook illustrates what data is available as part of the <b>Visual Behavior - 2P dataset</b>, and helps you to understand the experimental design and dimensions of the dataset. The notebook will demonstrate how to identify experiments and sessions that you may be interested in analyzing using the data manifests provided by the `VisualBehaviorOphysProjectCache`, and exploring the metadata columns that describe the experimental conditions including transgenic lines, targeted areas, imaging depths, microscopes that were used, session types, and dataset variants. 

We will first install allensdk into your environment by running the appropriate commands below. 

## Install AllenSDK into your local environment

You can install AllenSDK locally with:


```python
!pip install allensdk
```



## Install AllenSDK into your notebook environment (good for Google Colab)

You can install AllenSDK into your notebook environment by executing the cell below.

If using Google Colab, click on the RESTART RUNTIME button that appears at the end of the output when this cell is complete,. Note that running this cell will produce a long list of outputs and some error messages. Clicking RESTART RUNTIME at the end will resolve these issues.
You can minimize the cell after you are done to hide the output.


```python
!pip install --upgrade pip
!pip install allensdk
```

## Import necessary packages


```python
import numpy as np

from allensdk.brain_observatory.behavior.behavior_project_cache import VisualBehaviorOphysProjectCache

## First, load the project cache - your access point for all tables and data


```python
# Update this to a valid directory in your filesystem
output_dir = r"\Data\visual_behavior_ophys_cache_dir"
```


```python
cache = VisualBehaviorOphysProjectCache.from_s3_cache(cache_dir=output_dir)
```

The data manifest is comprised of three types of tables: 

1. `behavior_session_table` 
2. `ophys_session_table` 
3. `ophys_experiment_table` 

The `behavior_session_table` contains metadata for every <b>behavior session</b> in the dataset. Some behavior sessions have 2-photon data associated with them, while others took place during training in the behavior facility. The different training stages that mice are progressed through are described by the `session_type`. 

The `ophys_session_table` contains metadata for every 2-photon imaging (aka optical physiology, or ophys) session in the dataset, associated with a unique `ophys_session_id`. An <b>ophys session</b> is one continuous recording session under the microscope, and can contain different numbers of imaging planes (aka experiments) depending on which microscope was used. For Scientifica sessions, there will only be one experiment (aka imaging plane) per session. For Multiscope sessions, there can be up to eight imaging planes per session. Quality Control (QC) is performed on each individual imaging plane within a session, so each can fail QC independent of the others. This means that a Multiscope session may not have exactly eight experiments (imaging planes). 

The `ophys_experiment_table` contains metadata for every <b>ophys experiment</b> in the dataset, which corresponds to a single imaging plane recorded in a single session, and associated with a unique `ophys_experiment_id`. A key part of our experimental design is targeting a given population of neurons, contained in one imaging plane, across multiple `session_types` (further described below) to examine the impact of varying sensory and behavioral conditions on single cell responses. The collection of all imaging sessions for a given imaging plane is referred to as an <b>ophys container</b>, associated with a unique `ophys_container_id`. Each ophys container may contain different numbers of sessions, depending on which experiments passed QC, and how many retakes occured (when a given `session_type` fails QC on the first try, an attempt is made to re-acquire the session_type on a different recording day - this is called a retake, also described further below). 

### To understand the difference between an `ophys_experiment`, an `ophys_session`, and an `ophys_container`, the following schematic can be helpful

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/data_structure.png", width="900"/>
</div>

Note that this represents a multi-plane imaging dataset. For single-plane imaging, there will only be one plane, corresponding to one row of this diagram.

## Lets go through each table and examine what metadata columns are available

# Behavior Sessions Table

In this dataset, mice are trained on a visual change detection task. This task involves a continuous stream of stimuli, and mice learn to lick in response to a change in the stimulus identity to earn a water reward. There are different stages of training in this task, described below. The metadata for each behavior session in the dataset can be found in the `behavior_sessions_table` and can be used to identify behavior sessions you may want to analyze. 

### Load the `behavior_sessions_table` from the cache


```python
behavior_sessions = cache.get_behavior_session_table()

print(f"Total number of behavior sessions: {len(behavior_sessions)}")

behavior_sessions.head()
```


### What columns does the behavior_session table have and what values can they take?


```python
behavior_sessions.columns
```

### behavior sessions can take place on different experimental systems


```python
print('behavior data could be recorded on these experimental systems:\n')
print(np.sort(behavior_sessions.equipment_name.unique()))
```

`equipment_name` values starting with 'BEH' indicate behavioral training in the behavior facility, while values starting with 'CAM2P' or 'MESO' indicate behavior sessions that took place under a 2-photon microscope - either a Scientifica single plane imaging system ('CAMP2P.4', 'CAM2P.4', or 'CAM2P.5') or a modified Mesoscope system, also called Multiscope, for multi-plane imaging ('MESO.1').

## Mouse specific metadata

The `mouse_id` is a 6-digit unique identifier for each experimental animal in the dataset


```python
print('there are ', len(behavior_sessions.mouse_id.unique()), 'mice in the dataset')
```

#### The transgenic line determines which neurons are labeled in a given mouse, and what they are labeled with


```python
print('the different transgenic lines included in this dataset are:\n')
print(np.sort(behavior_sessions.full_genotype.unique()))
```

`full_genotype` refers to the full name of the transgenic mouse line, including all driver and reporter lines in the cross. `driver_line` and `reporter_line` have their own unique columns in the table. The first element of the `full_genotype` is the `cre_line` (which also has its own column in the table, and is a subset of `driver_line`). The `cre_line` determines which genetically identified neuron type will be labeled by the `reporter_line`. 


```python
print('the different cre lines used in this dataset are:\n')
print(np.sort(behavior_sessions.cre_line.unique()))
```

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/cre_lines2.png" width="900"/>
</div>

In this dataset, we have 3 `cre_lines`, 'Slc17a7-IRES2-Cre', which labels excitatory neurons across all cortical layers, 'Sst-IRES-Cre' which labels somatostatin expressing inhibitory interneurons, and 'Vip-IRES-Cre', which labels vasoactive intestinal peptide expressing inhibitory interneurons. There are also 3 `reporter_lines`, 'Ai93(TITL-GCaMP6f)' which expresses the genetically encoded calcium indicator GCaMP6f (f is for 'fast', this reporter has fast offset kinetics, but is only moderately sensitive to calcium relative to other sensors) in cre labeled neurons, 'Ai94(TITL-GCaMP6s)' which expresses the indicator GCaMP6s (s is for 'slow', this reporter is very sensitive to calcium but has slow offset kinetics), and 'Ai148(TIT2L-GC6f-ICL-tTA2', which  expresses GCaMP6f using a self-enhancing system to achieve higher expression than other reporter lines (which proved necessary to label inhibitory neurons specifically). The specific `indicator` expressed by each `reporter_line` also has its own column in the table.


```python
print('the different reporter lines used in this dataset are:\n')
print(np.sort(behavior_sessions.reporter_line.unique()))
```

```python
print('the different indicators used in this dataset are:\n')
print(np.sort(behavior_sessions.indicator.unique()))
```

* For more information about transgenic lines, see characterization data here: https://observatory.brain-map.org/visualcoding/transgenic
* for more information on GCaMP6, see this paper: https://www.nature.com/articles/nature12354
* For more information on reporter lines, see these papers: https://doi.org/10.1016/j.neuron.2015.02.022, https://www.sciencedirect.com/science/article/pii/S0092867418308031

#### how many mice per transgenic line?


```python
behavior_sessions.groupby(['full_genotype', 'mouse_id']).count().reset_index().groupby('full_genotype').count()[['mouse_id']]
```

Other mouse specific metadata includes `sex` and `age_in_days`

## Session Type - a very important piece of information

The `session_type` for each behavior session indicates the behavioral training stage or 2-photon imaging conditions for that particular session. This determines what stimuli were shown and what task parameters were used.  


```python
print('the session_types available in this dataset are:\n')
print(np.sort(behavior_sessions.session_type[
                  ~behavior_sessions.session_type.isna()].unique()))
```

Mice are progressed through a series of training stages to shape their behavior prior to 2-photon imaging. Mice are automatically advanced between stages depending on their behavioral performance. For a detailed description of the change detection task and advancement criteria, please see the technical whitepaper: LINK

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/automated_training.png" width="900"/>
</div>

Training with the change detection task begins with simple static grating stimuli, changing between 0 and 90 degrees in orientation. On the very first day, mice are automatically given a water reward when the orientation of the stimulus changes (`TRAINING_0_gratings_autorewards_15min`). On subsequent days, mice must lick following the change in order to receive a water reward (`TRAINING_1_gratings`). In the next stage, stimuli are flashed, with a 500ms inter stimulus interal of mean luminance gray screen (`TRAINING_2_gratings_flashed`). 

Once mice perform the task well with gratings, they are transitioned to natural image stimuli. Different groups of mice are trained with different sets of images, image set A or image set B (described further below). In the following description, we use `X` as a placeholder for image set `A` or `B` in the `session_type` name. Training with images begins with a 10ul water reward volume (`TRAINING_3_images_X_10uL_reward`), which is then decreased to 7ul once mice perform the task consistently with images (`TRAINING_4_images_X_training`). When mice have reached criterion to be transferred to the 2-photon imaging portion of the experiment, they are labeled as 'handoff_ready' (`TRAINING_4_images_X_handoff_ready`.) If behavior performance returns to below criterion level, they are labeled as 'handoff_lapsed'(`TRAINING_4_images_X_handoff_lapsed`). 


```python
# reminder about possible session types 
print('the different session_types available in this dataset are:\n')
print(np.sort(behavior_sessions.session_type[
                  ~behavior_sessions.session_type.isna()].unique()))
```

 You will notice that some mice only go up to `TRAINING_4`, while others have the final training stage labeled `TRAINING_5`. This is due to a minor change made partway through data collection, where an `epilogue` stimulus was introduced during the final training stage prior to 2-photon imaging in order to habituate the mice to this stimulus, which is used during 2-photon imaging to aid in session to session registration. The `epilogue` is a 30 minute movie clip repeated 10 times, for a total of 5 minutes, and occurs at the end of the 60 minute behavioral session, followed by 5 minutes of blank gray screen. Training sessions with an epilogue movie include `TRAINING_5_images_X_epilogue`, `TRAINING_5_images_X_handoff_ready` , `TRAINING_5_images_X_handoff_lapsed`. 

### `session_types` during 2-photon imaging

When mice are transferred to the 2-photon rig for the imaging portion of the experiment, they first undergo 1-3 habituation sessions to get accustomed to the new experimental environment (`OPHYS_0_images_X_habituation`). During these sessions, mice perform the task under the microscope, but no experimental data is recorded. 

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/experiment_design.png" width="900"/>
</div>

During the 2-photon imaging portion of the experiment, mice perform the task with the same set of images they saw during training (either image set `A` or `B`), as well as an additional novel set of images (whichever of `A` or `B` that they did not see during training). This allows evaluation of the impact of different sensory contexts on neural activity - familiarity versus novelty. Sessions with <b>familiar images</b> include those starting with `OPHYS_0`, `OPHYS_1`, `OPHYS_2`, and `OPHYS_3`. Sessions with <b>novel images</b> include those starting with `OPHYS_4`, `OPHYS_5`, and `OPHYS_6`. 

Interleaved between active behavior sessions are <b>passive viewing</b> sessions where mice are given their daily water ahead of the sesssion (and are thus satiated) and view the stimulus with the lick spout retracted so they are unable to earn water rewards. This allows comparison of neural activity in response to stimuli under different behavioral context - active task engagement and passive viewing without reward. Passive sessions include `OPHYS_2_images_A_passive` (passive session with familiar images), and `OPHYS_5_images_A_passive` (passive session with novel images).

The final session during the 2-photon imaging phase is `OPHYS_7_receptive_field_mapping`, however 2-photon data is not available for these sessions in this data release (but will be made available in a subsequent release).

## Dataset variants - different mice were subject to different experimental conditions

As hinted to above, some mice were trained with image set A, and others with image set B. Including these two groups of mice, with swapped stimulus conditions, was included in the dataset as a control for the effects of novelty, to ensure that any observed changes were truly due to lack of familiarity with the novel image set, rather than a result of specific features of the image set that was used. In addition, some mice were imaged on the Scientifica single plane imaging systems, and other mice were imaged on Multiscope for multi-plane imaging. These distinct groups of mice are referred to as <b>dataset variants</b> and can be identified using the `project_code` column

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/datasets.png" width="900"/>
</div>

Project_code is only defined for ophys sessions, for technical reasons, so let's fill in the gaps so that all mice have a project_code


```python
# get a table of the project code for each mouse
project_code_lookup = behavior_sessions[behavior_sessions.project_code.isnull()==False].reset_index().drop_duplicates('mouse_id')[['mouse_id','project_code']]
project_code_lookup
```


```python
behavior_sessions = behavior_sessions.merge(project_code_lookup, on='mouse_id',
                                            how='left', suffixes=('_session', '_mouse'))
behavior_sessions = behavior_sessions.drop(columns='project_code_session')
behavior_sessions = behavior_sessions.rename(columns={'project_code_mouse': 'project_code'})
```

#### What `project_codes` are available? What  `session_types` belong to each? 


```python
behavior_sessions.project_code.unique()
```

```python
for project_code in behavior_sessions.project_code.unique(): 
    project_sessions = behavior_sessions[behavior_sessions.project_code==project_code]
    print('\n project_code:', project_code)
    print('\n has these session types:\n', np.sort(
        project_sessions.session_type[~project_sessions.session_type.isna()].unique()))
    print('\n')
```

Notice that for `project_codes` `VisualBehavior` and `VisualBehaviorMultiscope`, mice are trained on image set A, while for `VisualBehaviorTask1B`, mice are trained on image set B


```python

```

## Ophys Sessions Table

The `ophys_session_table` includes all of the metadata columns available in the `behavior_session_table`, as well as additional information specific to 2-photon imaging, namely the list of `ophys_experiment_ids` and `ophys_container_ids` associated with each `ophys_session_id`. 


```python
ophys_sessions = cache.get_ophys_session_table()

print(f"Total number of ophys sessions: {len(ophys_sessions)}\n")

print(ophys_sessions.columns)

ophys_sessions.head()
```


```python
# what do the ophys_experiment_id and ophys_container_id columns look like? 
# are there always the same number of experiments and containers in different sessions? 
# does the number of experiments and containers depend on the microscope used? 
ophys_sessions[['ophys_experiment_id', 'ophys_container_id', 'equipment_name']][:15]
```

## Session order 

###  The `ophys_session_table` only includes sessions that pass ophys QC 

#### (but the `behavior_session_table` includes all the sessions)

The `ophys_session_table` only includes sessions with 2-photon imaging data that passed our QC criteria. Importantly, sessions that took place during 2-photon imaging, but did NOT pass QC, can be found in the `behavior_session_table`, as it includes the full training history for every mouse. In the `behavior_session_table`, only sessions with passing ophys data will have an `ophys_session_id`. We can use this to identify ophys sessions that didnt pass QC, but still have behavior data. 

#### Let's look at all the behavior sessions that took place on a 2-photon rig for one mouse, in order of acquisition date


```python
# pick a mouse
mouse_id = 445002 
# get behavior sessions that took place on the microscope
mouse_ophys_sessions = behavior_sessions[(behavior_sessions.mouse_id==mouse_id)&
                                         (behavior_sessions.equipment_name=='CAM2P.3')]
# only look at the relevant columns
mouse_ophys_sessions.sort_values(by='date_of_acquisition')[['session_type', 'date_of_acquisition', 'ophys_session_id']]
```

Notice that only a subset of all OPHYS sessions have an `ophys_session_id` - these are the sessions that passed QC. Sessions with NaN as the `ophys_session_id` either do not have 2P data recorded (as in habituation sessions), or failed QC and were retaken on a subsequent day, such as `OPHYS_5_images_B_passive` in this case


```python
print('there are', len(mouse_ophys_sessions), 'ophys sessions in the behavior_session_table for this mouse')
print('this includes ophys sessions that failed QC for ophys, but still have behavior data')
```

#### What is available in the `ophys_session_table` for this mouse? 


```python
print('there are', len(ophys_sessions[ophys_sessions.mouse_id==mouse_id]), 'sessions in the ophys_session_table for this mouse')
print('these are the sessions with valid ophys data')
```

```python
ophys_sessions[ophys_sessions.mouse_id==mouse_id][['date_of_acquisition', 'session_type']]
```


### Due to QC and retakes, session types in the `ophys_session_table` do not always occur in sequential order

The schematic above depicts ophys sessions OPHYS1-6 in a specific order, however this order is rarely perfectly maintained due to QC failures. The example above shows OPHYS_1-4 in the correct order, but then OPHYS_5 comes after OPHYS_6 because the first attempt at OPHYS_5 failed (as we can see from the behavior_sessions for this mouse), and had to be retaken after OPHYS_6. 

#### Let's look at the session order for a different mouse, imaged on the Multiscope

```python
# pick a mouse
mouse_id = 453911
# get behavior sessions that took place on the microscope
mouse_ophys_sessions = behavior_sessions[(behavior_sessions.mouse_id==mouse_id)&
                                         (behavior_sessions.equipment_name=='MESO.1')]
# only look at the relevant columns
mouse_ophys_sessions.sort_values(by='date_of_acquisition')[['date_of_acquisition', 'session_type', 'ophys_session_id']]
```

Looks like lots of retakes for this one (where `ophys_session_id` = NaN). Also note that there are multiple retakes for some `session_types`. This can happen for mice imaged on Multiscope, because retakes can be triggered by QC failure of any one of the 8 imaging planes in the session. 

#### Let's look at how failures and retakes affects the session order in the `ophys_sessions` table for this mouse


```python
ophys_sessions[ophys_sessions.mouse_id==mouse_id][['date_of_acquisition', 'session_type']]
```

It looks like the first set of sessions are taken in sequential order, but after that there are a few retakes of some of the `session_types`. This suggests that some of the imaging planes for this Multiscope mouse passed QC on the first time around, but retakes were needed to get passing ophys data for other imaging planes. 

#### But they're not always out of order, sometimes things go perfectly! 


```python
# pick a mouse
mouse_id = 438912
# get behavior sessions that took place on the microscope
mouse_ophys_sessions = behavior_sessions[(behavior_sessions.mouse_id==mouse_id)&
                                         (behavior_sessions.equipment_name=='MESO.1')]
# only look at the relevant columns
mouse_ophys_sessions.sort_values(by='date_of_acquisition')[['date_of_acquisition', 'session_type', 'ophys_session_id']]
```


Well, nearly perfectly, OPHYS_5 came after OPHYS_6

### Prior Exposures

Because the session types can be out of order due to retakes, and because of some of our other experimental design decisions, it is helpful to know some information about the history of the mouse relative to a given session. To serve this purpose, we have included metadata describing the `prior_exposures_to_image_set`, `prior_exposures_to_session_type`, and `prior_exposures_to_omissions` as columns in all the manifest data tables. 

### `prior_exposures_to_image_set`

A key aspect of our experimental design is the inclusion of novel stimuli during the imaging phase of the experiment. However, after the very first session with these novel images, they actually start to become more and more familiar. So, it is important to know whether a given session is truly the first exposure to that image set. In addition, it is useful to know whether subsequent sessions are the second, third, fourth, etc. exposure to that image set, for analysis of changes in activity with experience following novelty. The `prior_exposures_to_image_set` column describes the number of sessions that a given mouse has observed the stimulus set that was shown in that session, prior to the start of that session. For the very first exposure to a novel image set, the value of `prior_exposures_to_image_set` will be 0. 

#### Let's look at the `prior_exposures_to_image_set` column for one of the mice we looked at above, first in the `behavior_session_table`, which contains all sessions the mouse experienced, then in the `ophys_session_table`, which only includes sessions that passed ophys QC


```python
mouse_id = 445002 
# get behavior sessions that took place on the microscope
mouse_ophys_sessions = behavior_sessions[(behavior_sessions.mouse_id==mouse_id)&
                                         (behavior_sessions.equipment_name=='CAM2P.3')]
# only look at the relevant columns
mouse_ophys_sessions.sort_values(by='date_of_acquisition')[['session_type', 'date_of_acquisition', 'ophys_session_id', 'prior_exposures_to_image_set']]
```

Note that `prior_exposures_to_image_set` is a high number for `OPHYS_0-3`, because that is the image set the mouse was trained on, and that it re-sets to zero for the first exposure to the novel image set in `OPHYS_4`

#### Let's double check the full training history for this mouse


```python
mouse_id = 445002 
# get behavior sessions that took place on the microscope
mouse_ophys_sessions = behavior_sessions[(behavior_sessions.mouse_id==mouse_id)]
# only look at the relevant columns
mouse_ophys_sessions.sort_values(by='date_of_acquisition')[['session_type', 'date_of_acquisition', 'ophys_session_id', 'prior_exposures_to_image_set']]
```

Knowing the prior exposures number is especially important for the `ophys_session_table`, because the sessions that failed ophys QC are not visible there, so it is difficult to know whether a given session was the first of its type or a retake. 

```python
ophys_sessions[ophys_sessions.mouse_id==mouse_id][['date_of_acquisition', 'session_type', 'prior_exposures_to_image_set']]
```

### `prior_exposures_to_session_type`

In some cases, you may want to know how many times a given `session_type` was seen by the mouse. For example, to know whether a passive viewing session was the very first time the mouse experienced a passive session with no lick spout, as there may be a difference in expectation of reward between the first passive session and a later one where the mouse has become accustomed to sometimes having the lick spout removed. 

#### compare `prior_exposures_to_session_type` in the `behavior_session_table` with the `ophys_session_table` for a given mouse


```python
# pick a mouse
mouse_id = 456915
# get behavior sessions that took place on the microscope
mouse_ophys_sessions = behavior_sessions[(behavior_sessions.mouse_id==mouse_id)&
                                         (behavior_sessions.equipment_name=='MESO.1')]
# only look at the relevant columns
mouse_ophys_sessions.sort_values(by='date_of_acquisition')[['date_of_acquisition', 'session_type', 'ophys_session_id', 'prior_exposures_to_session_type']]
```


```python
ophys_sessions[ophys_sessions.mouse_id==mouse_id][['date_of_acquisition', 'session_type', 'prior_exposures_to_session_type']]
```


Without the `prior_exposures_to_session_type` column in the `ophys_session_table`, it would be difficult to know that `OPHYS_2_images_A_passive` was actually the second time (1 prior exposure) that the mouse had experienced a passive vieweing session

### `prior_exposures_to_omissions`

Another unique aspect of the experimental design of this dataset is the inclusion of stimulus omissions in the 2-photon portion of the experiment. During behavioral training, mice experience a highly regular cadence of himage presentations, with 250ms per stimulus presentation, with a 500ms gray screen in between. During imaging sessions, stimulus presentations (other than the change and pre-change images) are omitted with a 5% probability, resulting in some inter stimlus intervals appearing as an extended gray screen period. This allows exploration of potential effects of temporal expectation on neural activity. 

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/omissions.png" width="900"/>
</div>

#### Let's look at `prior_exposures_to_omissions` in a few mice


```python
np.sort(behavior_sessions[behavior_sessions.equipment_name=='CAM2P.4'].mouse_id.unique())
```

```python
# pick a mouse
mouse_id = 436662
# get behavior sessions that took place on the microscope
mouse_ophys_sessions = behavior_sessions[(behavior_sessions.mouse_id==mouse_id)&
                                        (behavior_sessions.equipment_name=='CAM2P.4')]
# only look at the relevant columns
mouse_ophys_sessions.sort_values(by='date_of_acquisition')[['date_of_acquisition', 'session_type', 'ophys_session_id', 'equipment_name',  'prior_exposures_to_omissions']]
```


In this case (and in most cases), omissions do not occur until the first true imaging session on the 2-photon rig, `OPHYS_1`, i.e. they are not included in habituation sessions. However, in a small number of mice from the beginning of our data collection process, omissions did occur in habituation sessions (but never during training). This is something to be careful of if you are looking at something like the change in omission related activity with experience. 

Here is a mouse that saw omissions during habituation sessions. Also note that the first two habituation sessions took place on different microscopes (this is extremely rare, every effort is made to image a given mouse on the same 2-photon rig during its entire lifetime). 


```python
# pick a mouse
mouse_id = 423606
# get behavior sessions - include training as well
mouse_ophys_sessions = behavior_sessions[(behavior_sessions.mouse_id==mouse_id)]
# only look at the relevant columns
mouse_ophys_sessions.sort_values(by='date_of_acquisition')[['date_of_acquisition', 'session_type', 'ophys_session_id', 'equipment_name',  'prior_exposures_to_omissions']]
```


### Here is how to identify all mice that saw omissions during habituation sessions


```python
# get all behavior sessions that were habituation sessions (image set A or B) 
# where the prior exposures to omissions was not zero
habituation_with_omission = behavior_sessions[((behavior_sessions.session_type=='OPHYS_0_images_A_habituation')|
                              (behavior_sessions.session_type=='OPHYS_0_images_B_habituation'))&
                              (behavior_sessions.prior_exposures_to_omissions>0)]

mice_with_omission_during_habituation = habituation_with_omission.mouse_id.unique()

print(len(mice_with_omission_during_habituation), ' mice had omissions during habituation')
```

```python

```

## Ophys Experiment Table

The `ophys_experiment_table` contains all ophys data that passes QC, organized according to individual imaging planes in individual sessions, each associated with an `ophys_experiment_id`. The `ophys_experiment_table` contains all the columns in `ophys_session_table`, plus a few additional ones specific to individual imaging planes, namely `imaging_depth` and `targeted_structure`.


```python
ophys_experiments = cache.get_ophys_experiment_table()

print(f"Total number of ophys experiments: {len(ophys_experiments)}\n")

ophys_experiments.head()
```



#### Compare the columns of `ophys_sessions_table` with `ophys_experiments_table`


```python
ophys_sessions.columns
```

```python
ophys_experiments.columns
```

#### What `imaging_depths` and `targeted_structures` are available? Are they different depending on `project_code`?


```python
# loop through project codes and print the available imaging_depths and targeted_structures
for project_code in ophys_experiments.project_code.unique():
    
    project_experiments = ophys_experiments[ophys_experiments.project_code==project_code]
    print('\nimaging_depths available for', project_code, 'include: ', project_experiments.imaging_depth.unique())
    print('\ntargeted_structures available for', project_code, 'include: ', project_experiments.targeted_structure.unique())
    print('\n')
```

### `ophys_experiment_table` is useful for identifying `ophys_containers` to analyze

Compare the `ophys_container_id` column of the `ophys_experiment_table` with the `ophys_session_table`. In `ophys_session_table`, each `ophys_session_id` is associated with one or more imaging planes (`ophys_experiment_ids`), while in the `ophys_experiment_table`, you can evaluate each of those imaging planes indepdently. This is particularly helpful for identifying `ophys_containers` that you want to analyze - the set of all imaging sessions for a given imaging plane. 

The `ophys_experient_table` has all the same columns as `ophys_session_table`, just reorgnized by `ophys_experiment_id`


```python
print(ophys_experiments.columns)
```

This means that each `ophys_experiment_id` has a single `ophys_container_id`.

#### Let's pick an `ophys_container_id` and see what `ophys_experiments` it contains? 


```python
ophys_container_id = ophys_experiments.ophys_container_id.unique()[50]
```


```python
container_experiments = ophys_experiments[ophys_experiments.ophys_container_id==ophys_container_id]
container_experiments
```

Thats 7 different recording sessions for this single imaging plane. Remember that one `ophys_container_id` is linked to one imaging plane, recorded in multiple sessions

#### What are the session types for this container? 


```python
container_experiments.session_type.unique()
```

### Reminder about structure & terminology of the dataset

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/data_structure.png" width="900"/>
</div>

### Reminder about cre lines

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/cre_lines2.png" width="900"/>
</div>

### Reminder about dataset variants aka project_codes

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/datasets.png" width="900"/>
</div>

### Reminder about session types

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/automated_training.png" width="900"/>
</div>

<div>
<img src="https://allensdk.readthedocs.io/en/latest/_static/visual_behavior_2p/experiment_design.png" width="900"/>
</div>


```python

```

## Identifying experiments and sessions of interest

### Get all experiments for one container from an Sst-IRES-Cre mouse in the VisualBehaviorTask1B project code 


```python
# get all Sst experiments in the relevant project code
sst_experiments = ophys_experiments[(ophys_experiments.cre_line=='Sst-IRES-Cre')&
                 (ophys_experiments.project_code=='VisualBehaviorTask1B')]

# pick some container from this set
ophys_container_id = sst_experiments.ophys_container_id.unique()[1]
print(ophys_container_id)
```

    941373529



```python
# what experiments are there for this container? 
sst_container_experiments = sst_experiments[sst_experiments.ophys_container_id==ophys_container_id]
sst_container_experiments
```




### Load the BehaviorOphysExperiment dataset for each ophys_experiment_id in the container and plot the max intensity projection - are they well aligned across sessions? can you identify the same neurons?  


```python
import matplotlib.pyplot as plt
```


```python
# ophys_experiment_ids are the index of the ophys_experiment_table
ophys_experiment_ids = sst_container_experiments.index.values

# create figure axis
fig, ax = plt.subplots(1, len(ophys_experiment_ids), figsize=(20,5))
# enumerate over experiments in this container
for i, ophys_experiment_id in enumerate(ophys_experiment_ids): 
    # get the dataset object
    dataset = cache.get_behavior_ophys_experiment(ophys_experiment_id=ophys_experiment_id)
    # get the max intensity projection and plot on the appropriate axis
    ax[i].imshow(dataset.max_projection.data, cmap='gray')
    ax[i].set_title(ophys_experiment_id)
```

```python

```

### Get all imaging planes recorded during one session with novel images in a Vip mouse imaged on Multiscope


```python
# get all Vip sessions in the Multiscope project code
vip_sessions = ophys_sessions[(ophys_sessions.cre_line=='Vip-IRES-Cre')&
                             (ophys_sessions.project_code=='VisualBehaviorMultiscope')&
                             (ophys_sessions.prior_exposures_to_image_set==0)]

# ophys_session_id is the index of the ophys_session_table
ophys_session_id = vip_sessions.index.values[0]
```


```python
# look at info for this ophys session
vip_sessions.loc[ophys_session_id]
```

### Plot the average dF/F trace for each of the experiments in this session for a 5 minute time period 


```python
# get all the ophys_experiment_ids (corresponding to imaging planes) for this session
ophys_experiment_ids = vip_sessions.loc[ophys_session_id].ophys_experiment_id
print(ophys_experiment_ids)
```

```python
# create figure axis
fig, ax = plt.subplots(1,1, figsize=(15,4))
# enumerate over experiments in this session
for i, ophys_experiment_id in enumerate(ophys_experiment_ids): 
    # get the dataset object
    dataset = cache.get_behavior_ophys_experiment(ophys_experiment_id=ophys_experiment_id)
    # get ophys timestamps
    ophys_timestamps = dataset.ophys_timestamps
    # get the population average dF/F trace
    dff_traces = dataset.dff_traces
    # dff_traces is a dataframe with a column 'dff'
    # get the values of this column and turn into a matrix of n_cells x timepoints
    dff_traces = np.vstack(dff_traces.dff.values)
    # take the mean over the cell axis
    average_dFF = np.mean(dff_traces, axis=0)
    # get the imaging_depth and targeted_structure for this experiment
    imaging_depth = dataset.metadata['imaging_depth']
    targeted_structure = dataset.metadata['targeted_structure']
    # plot it, including the imaging_depth and targeted_structure in the legend label
    ax.plot(ophys_timestamps, average_dFF, label=targeted_structure+'_'+str(imaging_depth))
    ax.set_title(dataset.metadata['cre_line']+', ophys_session_id: '+str(ophys_session_id))
ax.set_ylabel('dF/F')
ax.set_xlabel('time (seconds)')
ax.set_xlim(5*60, 10*60)
ax.legend()
```

## Ophys Cells Table

```python
cells_table = cache.get_ophys_cells_table()

cells_table.head()
```

### How many cells per experiment?


```python
cell_per_exp = cells_table.groupby('ophys_experiment_id').size()
fig = plt.hist(cell_per_exp, bins=50)
plt.xlabel('Cell count')
plt.ylabel('Number of experiments')
plt.show()
cell_per_exp.describe()
```


Merge the cell counts into the ophys experiments table.


```python
ophys_experiments['n_cells'] = ophys_experiments.index.map(cell_per_exp)
```

Now we can look at the cell count by depth, for example


```python
fig, ax = plt.subplots(figsize=(30, 10))
ax.scatter(ophys_experiments['imaging_depth'], ophys_experiments['n_cells'], alpha=.3)
ax.set_xlabel('Imaging depth (microns)')
ax.set_ylabel('Cell count')
plt.show()
```


Or by cre-line


```python
import seaborn as sns
sns.boxplot(data=ophys_experiments, x='n_cells', y='cre_line')
```

