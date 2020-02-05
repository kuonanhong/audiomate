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
    - name: ZHAW Zurich University of Applied Sciences, Winterthur, Switzerland
      index: 1
date: 20 December 2019
bibliography: paper.bib
---

# Summary

Machine learning tasks in the audio domain frequently require large datasets with training data.
Over the last years, numerous datasets have been made available for various purposes, for example, [@musan2015] and [@ardila2019common].
Unfortunately, most of the datasets are stored in widely differing formats.
As a consequence, machine learning practitioners have to convert datasets into other formats before they can be used or combined.
Furthermore, common tasks like reading, partitioning, or shuffling of datasets have to be developed over and over again for each format and require intimate knowledge of the formats.
We purpose Audiomate, a Python toolkit, to solve this problem.

Audiomate provides a uniform programming interface to work with numerous datasets.
Knowledge about the structure or on-disk format of the datasets is not necessary.
Audiomate facilitates and simplifies a wide range of tasks:

* Reading and writing of numerous dataset formats using a uniform programming interface, for example [@musan2015], [@Panayotov2015LibrispeechAA] and [@ardila2019common]
* Accessing metadata, like speaker information and labels
* Reading audio data (single files, batches of files)
* Retrieval of information about the data (e.g., number of speakers, total duration).
* Merging of multiple datasets (e.g., combine two speech datasets).
* Splitting data into smaller subsets (e.g., create training, validation, and test sets with a reasonable distribution of classes).
* Validation of data for specific requirements (e.g., check whether all samples were assigned a label)

# Use Cases

To illustrate Audiomate's capabilities, we present two typical applications where Audiomate significantly simplifies the task of a developer: Training a speech recognition model with Mozilla's implementation of DeepSpeech and training a deep neural network to recognize music.

## Training an Automatic Speech Recognition Model

In this example, we use Mozilla's implementation (https://github.com/mozilla/DeepSpeech) of DeepSpeech [@deepspeech] to train an automatic speech recognition model.
To do so, we use the LibriSpeech dataset [@Panayotov2015LibrispeechAA].
The first step is to convert the dataset into the csv-format expected by DeepSpeech (@@Link? There is no link afaik, just reverse engineering their code).
Using audiomate, we can load the given dataset and store it in the format of DeepSpeech.
Of course, it requires audiomate to have the specific reader and writer.
But, once having them, they can be used in many ways.
Assume, we decide to change to Kaldi instead of DeepSpeech,
we just change one line to use another audiomate writer that stores the data in Kaldi's format.

```python
import audiomate
from audiomate.corpus import io

downloader = io.LibriSpeechDownloader()
downloader.download('/local/data/librispeech')

# librispeech = audiomate.Corpus.load('/local/data/librispeech', reader='librispeech')
reader = io.LibriSpeechReader()
librispeech = reader.load('/local/data/librispeech')

# Save in format of deepspeech
# librispeech.save_at('/local/data/librispeech_ds', writer='mozilla-deepspeech')
writer = io.MozillaDeepSpeechWriter()
writer.save(librispeech, '/local/data/librispeech_ds')

# Or with kaldi
# librispeech.save_at('/local/data/librispeech_kaldi', writer='kaldi')
writer = io.KaldiWriter()
writer.save(librispeech, '/local/data/librispeech_kaldi')
```

## Training a Neural Network Recognizing Music
We want to train a neural network detecting segments in an audio stream that contains music.
To do so, we intend to use the MUSAN dataset [@musan2015] and the GTZAN dataset [@GTZAN].
For training the DNN, we use for example PyTorch (https://pytorch.org/).
With audiomate, we can load both datasets with a single line of code.
Furthermore, we can merge them to a single set.
In order to test our model, we also can split the data into two subsets with audiomate.
Without knowing how the audio is stored on the filesystem,
we can load the samples and labels just by iterating over all utterances or using a method to load the samples in batches.

```python
import audiomate
from audiomate.corpus import io
from audiomate.corpus import subset

musan_dl = io.MusanDownloader()
musan_dl.download('/local/data/musan')

gtzan_dl = io.GtzanDownloader()
gtzan_dl.download('/local/data/gtzan')

musan = audiomate.Corpus.load('/local/data/musan', reader='musan')
gtzan = audiomate.Corpus.load('/local/data/gtzan', reader='gtzan')

full = audiomate.Corpus.merge_corpora([musan, gtzan])

splitter = subset.Splitter(full, random_seed=222)
subviews = splitter.split(proportions={
    'train': 0.8,
    'test': 0.2,
})

for utterance in subviews['train'].utterances.values():
    samples = utterance.read_samples()
    labels = utterance.label_lists[audiomate.corpus.LL_DOMAIN]
```

# Implementation
Audiomate is implemented with the goal to make it simple to add new data formats.
A new format can be added by implementing one or more of three available abstract interfaces.

* Reader: A reader defines the procedure to load data from a specific format.
          It stores the data in a universal data structure.

* Writer: A writer defines the procedure to store data in a specific format.
          It gets data from the universal data structure and stores it in the specific format.

* Download: A downloader can be used to download a dataset.
            It downloads all required files automatically.

One certain format rarely defines interfaces for all of those tasks.
Mostly, readers and downloaders are implemented for datasets,
while writers are implemented for machine learning toolkits.
Audiomate already provides a bunch of implementations for various datasets and toolkits.

# Acknowledgements
Parts of audiomate were developed during the project Keyword Spider (KWS) funded by INNOSUISSE (26567.1 PFES-ES).

# References
