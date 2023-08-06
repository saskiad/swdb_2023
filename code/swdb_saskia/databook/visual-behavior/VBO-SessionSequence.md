





## Ophys session sequence 

The `ophys_session_table` only includes sessions with 2-photon imaging data that passed our QC criteria. Behavior sessions that took place during 2-photon imaging but did not pass QC can still be found in the `behavior_session_table`, as it includes the full training history for every mouse. In the `behavior_session_table`, only sessions with passing ophys data will have an `ophys_session_id` (otherwise it will be `NaN`). We can use this information to identify ophys sessions that did not pass QC, but still have behavior data. 

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
# how many valid ophys sessions are there?
print('there are', len(ophys_sessions[ophys_sessions.mouse_id==mouse_id]), 'sessions in the ophys_session_table for this mouse')
print('these are the sessions with valid ophys data')
```

```python
# what session types have ophys data?
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
