---
title: 'audiomate: A Python package for working with audio datasets'
tags:
    - Python
    - audio
    - speech
    - music
    - corpus
    - dataset
authors:
    - name: Matthias Büchi
      orcid: 0000-0003-0207-5711
      affiliation: 1
    - name: Andreas Ahlenstorf
      affiliation: 1
affiliations:
    - name: ZHAW, Winterthur, Switzerland
      index: 1
date: 20 December 2019
bibliography: paper.bib
---

# Summary
Machine Learning has a variety of applications in different fields,
including audio-related tasks. But especially with deep learning the need for data has been increased dramatically.
Therefore data has to be prepared carefully before the actual problem can be tackled.
Audiomate aims to reduce the effort for preparation of audio datasets,
by providing a consistent way to work with them.

Audio datasets have various formats, almost every single dataset has it's own format.
So everytime a new dataset is used, a piece of code has to be written to load and parse the data.
Having the data loaded, often some kind of preprocessing is necessary.
Finally, if the data is intended to be used with some kind of machine learning toolkit,
it has to be saved in another specific format.

Audiomate is designed to help with all those problems.
It provides an uniform way to work with the data without knowing specific details about the file-system
representation of the data.
This way datasets of different formats can be loaded and used in the same way.
Furthermore, the data can be stored in different ways.
For example, a machine learning toolkit like Kaldi [@Povey2011TheKS] requires a specific format.
Audiomate automatically exports the data in the correct format,
without the user having to know how that format looks like.
Apart from reading and writing data,
audiomate also provides tools to work with the loaded data including:

* Access all meta-data, like speakers, labels
* Read audio data (single files, batches of files)
* Retrieve information about the data (e.g. amount of speakers, total duration, ...)
* Merge data of multiple datasets (e.g. Combine two speech datasets)
* Split data into smaller subsets (e.g. Create train/dev/test splits with a reasonable distribution of classes)
* Validate data for specific requirements (e.g Check if all samples have assigned a label)

# Use cases
Audiomate has a lot of use cases.
Two of them are described here.

## Training ASR Model with Mozilla's DeepSpeech
For training an automatic speech recognition model, for example Mozilla's implementation of DeepSpeech
(https://github.com/mozilla/DeepSpeech) can be used.
In order to do that, we first need to convert our dataset to the format of the DeepSpeech implementation.
Using audiomate, we can load the given dataset and store it in the specific dataset.
Of course, it requires audiomate to have the specific reader and writer.
But, once having them, they can be used in many ways.
Assume, we decide to change to Kaldi instead of DeepSpeech,
we just change one line to use the audiomate method to store the data in Kaldi's format.

## Training a Neural Network recognizing Music
We want to train a neural network, that detects segments in an audio stream that contains music.
For that we want to use the MUSAN dataset [@musan2015] and the GTZAN dataset [@GTZAN].
To train the DNN we use for example PyTorch (https://pytorch.org/).
With audiomate, we can loaded both datasets with a single line of code.
Furthermore, we can merge them to a single set.
In order to test our model, we also can split the data into two subsets with audiomate.
Without knowing how the audio is stored on the filesystem,
we can load the samples and labels just by iterating over all utterances.

# Implementation
Audiomate is implemented with the goal to make it simple to add new data formats.
A new format can be added by implementing any of three abstract interfaces.

* Reader: A reader defines the procedure to load data from a specific format.
          It stores the data in a universal data structure.

* Writer: A writer defines the procedure to store data in a specific format.
          It gets data from the universal data structure and stores it in the specific format.

* Download: A downloader can be used to download a dataset.
            It downloads all required files automatically.

A specific format, rarely defines interfaces for all of those tasks.
Mostly, readers and downloaders are implemented for certain datasets
and writers are implemented for machine learning toolkits.
Audiomate already provides a bunch of implementation for various datasets and toolkits.

# Acknowledgements
Parts of audiomate were developed during the project Keyword Spider (KWS) funded by INNOSUISSE (26567.1 PFES-ES).

# References
