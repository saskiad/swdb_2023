---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Visual Behavior Ophys Dataset

The main entry point to the VBO dataset is the <code>VisualBehaviorOphysProjectCache</code> class.  This class is responsible for downloading any requested data or metadata as needed and storing it in well known locations.  For detailed info about how to access this data, check out [this tutorial](https://allensdk.readthedocs.io/en/latest/_static/examples/nb/visual_behavior_ophys_dataset_manifest.html)

We begin by importing the <code>VisualBehaviorOphysProjectCache</code>  class.

```{code-cell} ipython3
import pandas as pd
import os

from allensdk.brain_observatory.behavior.behavior_project_cache.\
    behavior_ophys_project_cache \
    import VisualBehaviorOphysProjectCache
```

Now we can specify our cache directory and set up the cache.

```{code-cell} ipython3
# this path should point to the location of the dataset on your platform
cache_dir = '/data/'

cache = VisualBehaviorOphysProjectCache.from_local_cache(
            cache_dir=cache_dir, use_static_cache=True)
```

We can use the <code>VisualBehaviorOphysProjectCache</code> to explore the parameters of this dataset. Let's start by examining the cache metadata tables.

## VBO Metadata
The data manifest comprises 5 tables: 

1. `behavior_session_table`
2. `ophys_session_table`
3. `ophys_experiment_table`
4. `ophys_cells_table`


The `behavior_session_table` contains metadata for every <b>behavior session</b> in the dataset. Some behavior sessions have 2-photon data associated with them, while others took place during training in the behavior facility. The different training stages that mice are progressed through are described by the `session_type`.

The `ophys_session_table` contains metadata for every 2-photon imaging (aka optical physiology, or ophys) session in the dataset, associated with a unique `ophys_session_id`. An <b>ophys session</b> is one continuous recording session under the microscope, and can contain different numbers of imaging planes (aka `ophys_experiments`) depending on which microscope was used. For imaging sessions using the Scientifica single-plane 2P microscope, there will only be one experiment (aka imaging plane) per session. For Multiscope sessions using the multi-plane 2-photon microscope, there can be up to eight imaging planes per session. Quality Control (QC) is performed on each individual imaging plane within a session, so each can fail QC independent of the others. This means that a Multiscope session may not have exactly eight experiments (imaging planes).

The `ophys_experiment_table` contains metadata for every <b>ophys experiment</b> in the dataset, which corresponds to a single imaging plane recorded in a single session at a specific `imaging_depth` and `targeted_structure``, and associated with a unique `ophys_experiment_id`. A key part of our experimental design is targeting a given population of neurons, contained in one imaging plane, across multiple days for several different `session_types` (further described below) to examine the impact of varying sensory and behavioral conditions on single cell responses.

The collection of all imaging sessions for a given imaging plane is referred to as an <b>ophys container</b>, associated with a unique `ophys_container_id`. Each ophys container may contain different numbers of sessions, depending on which experiments passed QC, and how many retakes occured (when a given `session_type` fails QC on the first try, an attempt is made to re-acquire the session_type on a different recording day - this is called a retake, also described further below).

<p>The <code>ophys_cells_table</code> contains the unique IDs of all cells recorded across all experiments. You can use this table to determine which ophys experiments a given cell was matched in.

Now let's look at each of these tables in more detail to get a better sense of the dataset.


## Ophys sessions table
First, let's load the ophys_sessions_table and print the columns:

```{code-cell} ipython3
ophys_sessions = cache.get_ophys_session_table()
ophys_sessions.columns
```

This table gives us lots of useful metadata about each recording session, including the genotype, sex and age of the mouse that was run, what brain areas were recorded and some important info about the stimulus. 

To demystify a few of these columns, let's briefly review the experimental design. Each mouse was trained with one of two image sets (`G` or `H`). For the majority of mice, we recorded two sessions: one with the trained 'familiar' image set and one with a 'novel' image set. Note that two of the eight images were shared across these two image sets as diagrammed below for an example mouse. For this mouse, image set `G` (images on blue and purple backgrounds) was used in training and was therefore 'familiar', while image set `H` (the two holdover images on purple background plus six novel images on red background) was 'novel'. 

![doctask](/images/image_sets_and_training_trajectories_diagram_defaultsdk_and_unfiltered.webp)

Each recording session can be defined by a few parameters, including the `image_set` used (G or H), the `experience_level` of the mouse (indicating whether the mouse had seen the image set in previous training sessions) and the `session_number` (indicating whether it was the first or second recording day for the mouse). In bottom bubble of the above diagram, you can see the three different training/recording trajectories mice in this dataset took:

* Train on G; see G on first recording day; see H on second recording day
* Train on G; see H on first recording day; see G on second recording day
* Train on H; see H on first recording day; see G on second recording day

The numbers in the recording session cells indicate how many of each session type exist in this dataset. The first number is what the SDK returns by default. The second number (in parentheses) is what the SDK returns without filtering for abnormalities (see below as well as the [Data Access tutorial](https://allensdk.readthedocs.io/en/latest/_static/examples/nb/visual_behavior_neuropixels_data_access.html)).


Here is a brief description of each column: 

abnormal_activity
: nan or list List of experiment time stamps when possible epileptic activity was noted.

abnormal_histology
: nan or list List of brain areas where possible damage was noted in post-hoc imaging.

age_in_days
: int age of mouse in days

behavior_session_id
: int unique identifier for a behavior session

channel_count
: float total number of channels on all probes used for experiment

date_of_acquisition
: date time object date and time of data acquisition, yyyy-mm-dd hh:mm:ss.

ecephys_session_id
: int unique identifier for an ecephys recording session

equipment_name
: string identifier for equipment data was collected on

experience_level
: string 'Familiar': image set mouse was trained on, 'Novel': not the image set the mouse was trained on.

file_id
: int lookup id to retrieve NWB file from S3 or the local cache.

genotype
: string full genotype of transgenic mouse

image_set
: string image set shown for a particular behavior session or ephys session

mouse_id
: int unique identifier for a mouse

prior_exposures_to_image_set
: float 64 number of prior sessions (during training or ophys) where the mouse was exposed to the image set shown in the current session. Starts at 0 for first exposure

prior_exposures_to_omissions
: int 64 number of sessions where the mouse was exposed to omissions. Starts at 0 for first exposure. Omissions do not occur during training

prior_exposures_to_session_type
: int 64 Number of previous sessions (during training or ophys) where the mouse was exposed to the current session type. Starts at 0 for first exposure

project_code
: string ‘NeuropixelVisualBehavior’--Project this session belongs to

session_number
: float 64 [1, 2] Indicates whether this session was the first or second recording day for the mouse. Takes values '1' or '2'.

session_type
: string Visual stimulus type displayed during behavior session

sex
: string [‘M’, ‘F’] Sex of the mouse

structure_acronyms
: string List of CCF structures recorded during this experiment

unit_count
: float number of units recorded during this

## Behavior Sessions Table

In this dataset, mice are trained on a visual change detection task. This task involves a continuous stream of stimuli, and mice learn to lick in response to a change in the stimulus identity to earn a water reward. There are different stages of training in this task, described below. The metadata for each behavior session in the dataset can be found in the `behavior_sessions_table` and can be used to build a training history for each mouse. Importantly, this table lists all of the behavior sessions for each mouse from the beginning of its training.

Let's load the table and take a look:

```{code-cell} ipython3
behavior_sessions = cache.get_behavior_session_table()

print(f"Total number of behavior sessions: {len(behavior_sessions)}")

behavior_sessions.head()
```

You can see that there are many more sessions here than in the ecephys_sessions_table. But the columns are defined in the same way.

To see how to use this table to look at the training trajectory for one mouse, check out [this tutorial](https://allensdk.readthedocs.io/en/latest/_static/examples/nb/visual_behavior_neuropixels_analyzing_behavior_only_data.html)

## Units Table

The <b>units</b> metadata table contains important quality and waveform metrics for every unit recorded in this dataset. For more info about how to use these quality metrics to filter units for analysis, check out [this tutorial](https://allensdk.readthedocs.io/en/latest/_static/examples/nb/visual_behavior_neuropixels_quality_metrics.html)

Let's grab this table and take a look at the columns:

```{code-cell} ipython3
units = cache.get_unit_table()
print(f'This dataset contains {len(units)} total units')

units.head()
```

For more information about many of the metrics included in this table and how to use them to guide your analysis, see our [quality metrics tutorial](https://allensdk--2471.org.readthedocs.build/en/2471/_static/examples/nb/visual_behavior_neuropixels_quality_metrics.html). For now, here's a brief description of each column:


**General Metadata**  

`ecephys_channel_id`:                   unique ID of channel on which unit's peak waveform occurred  
`ecephys_probe_id`:                     unique ID for probe on which unit was recorded  
`ecephys_session_id`:                   unique ID for session during which unit was recorded  
`anterior_posterior_ccf_coordinate`:    CCF coord in the AP axis  
`dorsal_ventral_ccf_coordinate`:        CCF coord in the DV axis  
`left_right_ccf_coordinate`:            CCF coord in the left/right axis  
`structure_acronym`:                    CCF acronym for area to which unit was assigned  
`structure_id`:                         CCF structure ID for the area to which unit was assigned  
`probe_horizontal_position`:            Horizontal (perpindicular to shank) probe position of each unit's peak channel in microns  
`probe_vertical_position`:              Vertical (along shank) probe position of each unit's peak channel in microns


**Waveform metrics**: Look [here](https://github.com/AllenInstitute/ecephys_spike_sorting/tree/master/ecephys_spike_sorting/modules/mean_waveforms) for more detail on these metrics and the code that computes them. For the below descriptions the '1D waveform' is defined as the waveform on the peak channel. The '2D waveform' is the waveform across channels centered on the peak channel.

`amplitude`:                            Peak to trough amplitude for mean 1D waveform in microvolts   
`waveform_duration`:                    Time from trough to peak for 1D waveform in milliseconds     
`waveform_halfwidth`:                   Width of 1D waveform at half-amplitude in milliseconds  
`PT_ratio`:                             Ratio of the max (peak) to the min (trough) amplitudes for 1D waveform  
`recovery_slope`:                       Slope of recovery of 1D waveform to baseline after repolarization (coming down from peak)  
`repolarization_slope`:                 Slope of repolarization of 1D waveform to baseline after trough  
`spread`:                               Range of channels for which the spike amplitude was above 12% of the peak channel amplitude  
`velocity_above`:                       Slope of spike propagation velocity traveling in dorsal direction from soma (note to avoid infinite values, this is actaully the inverse of velocity: ms/mm)  
`velocity_below`:                       Slope of spike propagation velocity traveling in ventral direction from soma (note to avoid infinite values, this is actually the inverse of velocity: ms/mm)  
`snr`:                                  signal-to-noise ratio for 1D waveform        


**Quality metrics**: Look [here](https://github.com/AllenInstitute/ecephys_spike_sorting/tree/7e567a6fc3fd2fc0eedef750b83b8b8a0d469544/ecephys_spike_sorting/modules/quality_metrics) for more detail on these metrics and the code that computes them.

`amplitude_cutoff`:                     estimate of miss rate based on amplitude histogram (ie fraction of spikes estimated to have been below detection threshold)  
`cumulative_drift`:                     cumulative change in spike depth along probe throughout the recording  
`d_prime`:                              classification accuracy based on LDA  
`firing_rate`:                          Mean firing rate over entire recording  
`isi_violations`:                       Ratio of refractory violation rate to total spike rate  
`isolation_distance`:                   Distance to nearest cluster in Mahalanobis space   
`l_ratio`:                              The Mahalanobis distance and chi-squared inverse cdf are used to find the probability of cluster membership for each spike.  
`max_drift`:                            Maximum change in unit depth across recording  
`nn_hit_rate`:                          Fraction of nearest neighbors in PCA space for spikes in unit cluster that are also in unit cluster  
`nn_miss_rate`:                         Fraction of nearest neighbors for spikes outside unit cluster than are in unit cluster  
`presence_ratio`:                       Fraction of time during session for which a unit was spiking  
`silhouette_score`:                     Standard metric for cluster overlap, computed in PCA space  
`quality`:                              Label assigned based on waveform shape as described [here](https://github.com/AllenInstitute/ecephys_spike_sorting/tree/7e567a6fc3fd2fc0eedef750b83b8b8a0d469544/ecephys_spike_sorting/modules/noise_templates). Either 'good' for physiological waveforms or 'noise' for artifactual waveforms.


## Probes Table

The <b>probes</b> table contains useful info about every probe insertion in the dataset. 

```{code-cell} ipython3
probes = cache.get_probe_table()
probes.head()
```

Here's a brief description of every column:

ecephys_probe_id
: int Unique identifier for probe insertion

ecephys_session_id
: int Unique identifier for ecephys session for this insertion

name
: string Name of probe indicating which of the six probe positions this probe filled (A-F).

sampling_rate
: float Sampling rate for AP band data

has_lfp_data
: bool Flag indicating whether LFP data was collected on this probe

lfp_sampling_rate
: float Sampling rate for LFP data

phase
: float Neuropixels generation for this probe (all probes in this dataset are Neuropixels 1.0) 

unit_count
: int Number of units recorded for this insertion

channel_count
: int Number of channels available for recording on this insertion

structure_acronyms
: string List of areas recorded for this probe insertion


## Channels Table

Metadata for every probe channel recorded in this dataset. This table can be merged with the units table to get CCF area assignments for each unit.

```{code-cell} ipython3
channels = cache.get_channel_table()
channels.head()
```

Here are the columns:

ecephys_channel_id
: Unique identifier for each channel. This is the id used in the units <code>ecephys_channel_id</code> column.

ecephys_probe_id
: int Unique identifier for probe insertion

ecephys_session_id
: int Unique identifier for ecephys session for this insertion

anterior_posterior_ccf_coordinate
: CCF coord in the AP axis  

dorsal_ventral_ccf_coordinate
: CCF coord in the DV axis  

left_right_ccf_coordinate
: CCF coord in the left/right axis  

structure_acronym
: CCF acronym for area to which channel was assigned  

probe_horizontal_position
: Horizontal (perpindicular to shank) probe position of each unit's peak channel in microns  

probe_vertical_position
: Vertical (along shank) probe position of each unit's peak channel in microns  

probe_channel_number
: Index of channel position on probe (0-383 with 0 at tip of probe)    

unit_count
: int Number of units assigned to this channel