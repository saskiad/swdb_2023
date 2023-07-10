# Visual Behavior 2-photon


The Visual Behavior 2P project used in vivo 2-photon calcium imaging (also called optical physiology, or “ophys”) to measure the activity of genetically identified neurons in the visual cortex of mice performing a go/no-go visual change detection task. This dataset can be used to evaluate the influence of experience, expectation, and task engagement on neural coding and dynamics in excitatory and inhibitory cell populations.

We used single- and multi-plane two- imaging approaches to record the activity of populations of neurons across multiple cortical depths and visual areas during change detection behavior. Each population of neurons was imaged repeatedly over multiple days under different sensory and behavioral contexts, including familiar and novel stimuli, as well as active behavior and passive viewing conditions.

The full dataset includes neural and behavioral measurements from 107 mice during 704 in vivo 2-photon imaging sessions from 326 unique fields of view, resulting in longitudinal recordings from 50,482 cortical neurons. Neural activity was measured as calcium fluorescence from GCaMP6 expressing cells in populations of excitatory (Slc17a7-IRES2-Cre;Camk2a-tTA;Ai93(TITL-GCaMP6f) or Ai94(TITL-GCaMP6s)), Vip inhibitory (Vip-IRES-Cre;Ai148(TIT2L-GCaMP6f-ICL-tTA2)), and Sst inhibitory (Sst-IRES-Cre;Ai148(TIT2L-GCaMP6f-ICL-tTA2)) neurons. Imaging took place between 75-400um below the cortical surface.


Different imaging configurations and stimulus sets were used in different groups of mice, resulting in four unique datasets (indicated by their `project_code` in SDK metadata tables). Two single-plane 2-photon datasets were acquired in the primary visual cortex (VISp). In the VisualBehavior dataset, mice were trained with image set A and tested with image set B which was novel to the mice. In the VisualBehaviorTask1B dataset, mice were trained with image set B and tested with image set A as the novel image set. One multi-plane dataset (VisualBehahviorMultiscope) was acquired at 4 cortical depths in 2 visual areas (VISp & VISl) using image set A for training and image set B for novelty. Another multi-plane dataset (VisualBehaviorMultiscope4areasx2d) was acquired at 2 cortical depths in 4 visual areas (VISp, VISl, VISal, VISam). In this dataset, two of the images that became highly familiar during training with image set G were interleaved among novel images in image set H.


 

## Background

## Technique

