# Wakeword-benchmark &nbsp; [![Tweet](https://img.shields.io/twitter/url/http/shields.io.svg?style=social)](https://twitter.com/intent/tweet?text=Performance%20comparison%20of%20different%20wakeword%20engines%20for%20Alexa&url=https://github.com/Picovoice/wakeword-benchmark&hashtags=wake-word,voice,AI,Alexa)

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/Picovoice/wakeword-benchmark/blob/master/LICENSE)

The primary purpose of this benchmark framework is to provide a scientific comparison between different wake-word
detection engines. Currently, the framework is configured for **Alexa** as the test wake-word. But it can be
configured for any other wake-words as described [here](#how-can-i-reproduce-the-results).

# Why did we make this?

The benchmark framework provides a definitive answer to the question
"which engine provides the best performance for a given wake-word?". While working on
[Porcupine](https://github.com/Picovoice/Porcupine) we noted that there is a need for such tool to empower customers to make
data-driven decisions. The framework

- uses hundreds of crowd-sourced utterances of wake-word.
- uses tens of hours of data made publicly available by [Common Voice](https://voice.mozilla.org/en) as background model
(i.e. what is not the wake-work and needs to be ignored).
- allows simulating real-world conditions by adding noise to clean speech.
- runs wake-word engines at different detection thresholds (aka sensitivities) which results in different
miss detection and false alarm rates. Finally, it creates an
[ROC](https://en.wikipedia.org/wiki/Receiver_operating_characteristic) curve for each engine.

# Data

[Common Voice](https://voice.mozilla.org/en) is used as background dataset, i.e., dataset without utterances of the
wake-word. It can be downloaded from [here](https://voice.mozilla.org/en/data). Only recordings with at least two up-votes
and no down-votes are used (this reduces the size of the dataset to ~125 hours).

Furthermore, 369 recording of word Alexa from 89 distinct speakers are used. The recordings are crowd-sourced using an
Android mobile application. The recordings can be downloaded [here](https://www.kaggle.com/aanhari/alexa-dataset).

In order to closely simulate real-world situations, the data is mixed with noise. For this purpose, we use
[DEMAND](https://asa.scitation.org/doi/abs/10.1121/1.4799597) dataset which has noise recording in 18 different
environments (e.g. kitchen, office, traffic, etc.). It can be downloaded from
[here](https://www.kaggle.com/aanhari/demand-dataset).

# Wake-word engines

Three wake-word engines are used in this benchmark. [PocketSphinx](https://github.com/cmusphinx/pocketsphinx) which can
be installed using [PyPI](https://pypi.org/project/pocketsphinx/). [Porcupine](https://github.com/Picovoice/Porcupine)
and [Snowboy](https://github.com/Kitt-AI/snowboy) which are included as submodules in this repository. 

# Metric

We measure the performance of the wake-word engines using false alarm per hour and miss detection rates. The false alarm
per hour is measured as a number of false positives in an hour. Miss detection is measured as the percentage of wake-word
 utterances an engine rejects incorrectly. Using these definitions we compare the engines for a given false alarm, and therefore the
engine with a smaller miss detection rate has a better performance.

# Usage

### Prerequisites

The benchmark has been developed on Ubuntu 16.04 with Python 3.5. It should be possible to run it on a Mac machine
or different distributions of Linux but has not been tested. Clone the repository using

```bash
git clone --recurse-submodules git@github.com:Picovoice/wakeword-benchmark.git
```

Install SoX for Ubuntu using
```
sudo apt-get install sox
```

Make sure the Python packages in the [requirements.txt](/requirements.txt) are properly installed for your Python version.
Then install the mp3 handler for SoX

```bash
sudo apt-get install libsox-fmt-mp3
```

Python bindings are used for running Porcupine and Snowboy. The repositories for these are cloned in [engines](/engines).
Make sure to follow the instructions on their repositories to be able to run their Python demo before proceeding to the next step.

### Running the benchmark

Usage information can be retrieved via

```bash
python benchmark.py -h
```

The benchmark can be run using the following command from the root directory of the repository

```bash
python benchmark.py --common_voice_directory <root directory of Common Voice dataset> --alexa_directory <root directory of Alexa dataset> \
--demand_directory <root directory of Demand dataset>
```

This runs the benchmark for a clean environment and creates the ROC curves for different engines. When
`--output_directory <output directory to save the results>` is passed to command line the framework 
saves the results in CSV format. This is going to take a while (it takes 48 hours on a quad-core Intel machine).

To run the benchmark in the noisy environment pass the `--add_noise` and it randomly picks a noise
from DEMAND dataset and adds it to the audio samples. We are adding noise to the audio samples with the SNR of 10dB which
simulates environments with moderate noise.

# Results

Below is the result (ROC curve) of running the benchmark framework for clean and noisy environments. As expected, for a
given false alarm rate the miss rate increases across different engines when noise is added to data.

![](doc/img/benchmark_roc.png)

A more illustrative way of comparing the results is to compare the miss rates given a fixed false alarm per hour value. The
engine with smallest miss rate is performing the best. This is shown below for clean speech scenario

![](doc/img/benchmark_clean_bar.png)

Also below is the result in presence of noise

![](doc/img/benchmark_noisy_bar.png)

# FAQ

### How can I reproduce the results?

The results presented above are completely reproducible given that exactly same datasets and engines are used. 

Datasets are taken from
* [Common Voice Dataset](https://voice.mozilla.org/en/data) is an active project and evolving over time. But this should
not significantly affect the results. In case you are interested to use the exact version as used to produce above results, 
we keep a copy of dataset used internally. (please send an email to <help@picovoice.ai>).
* [Alexa Dataset](https://www.kaggle.com/aanhari/alexa-dataset).
* [Demand Dataset](https://www.kaggle.com/aanhari/demand-dataset).

The engines used in the benchmark are:
* PocketSphinx 0.1.3 from [PyPI](https://pypi.org/project/pocketsphinx/).
* Snowboy is cloned from its repository on [commit](https://github.com/Kitt-AI/snowboy/commit/52723960c5c98969085dd91de69c48287c1f8c1a).
* Porcupine is cloned from its repository on [commit](https://github.com/Picovoice/Porcupine/commit/e54c73322cb258017b4a44a19eb762b8f81ccf2f).

### How can I use the framework for my wake-word?

The framework is currently configured for Alexa as the wake-word. It can be configured for a different wake-word by
following steps below

1. Collect recordings of your wake-word with the sample rate of 16000 in WAV format.
2. Implement `Dataset` interface for the new collection of wake-word recordings.
3. Instantiate the newly-created dataset instead of `AlexaDataset`.
4. Instantiate `CommonVoiceDataset` with `exclude_words` constructor parameter set to the new test word.
3. Assure all wake-word engines have a model for the newly-selected wake-word. Refer to the [engine.py](/engine.py) and
add your models to the engines.
