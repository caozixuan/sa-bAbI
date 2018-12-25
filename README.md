This repository contains code for sa-bAbI code generator and for open source code tranformation.

Our team project is based on cmu-sei/sa-bAbi. We add 5 kinds of new vulnerabilities and try applying the model to some transformed open source code.

Contributors:
* [caozixuan](https://github.com/caozixuan)
* [Endless-Fighting](https://github.com/Endless-Fighting)
* [rsy56640](https://github.com/rsy56640)
* [Evan-Choo](https://github.com/Evan-Choo)
* [RickyZhang1998](https://github.com/RickyZhang1998)
* [YINNNER](https://github.com/YINNNER)

# Quickstart

## Generate sa data

A minimal example to get you an end to end analysis of generated code.

Git clone or unzip the archive and descend into the working directory.
```
$ unzip SA-bAbI-<version>.zip
$ cd sa-babi-<version>
```

Build the docker images and setup the workspace
```
$ docker-compose build
$ mkdir working
```

Generate two datasets, a training set and test set. The `SA_SEED` variable
is optional, but used here for reproducibility.
```
$ SA_SEED=0 ./sa_e2e.sh working/sa-train-1000 1000
$ SA_SEED=1 ./sa_e2e.sh working/sa-test-100 100
```

or use `./sa_e2e_no_tools.sh` with no tools analyzing, and this will be much faster.

```
$ SA_SEED=0 ./sa_e2e_no_tools.sh working/sa-train-1000 1000
$ SA_SEED=1 ./sa_e2e_no_tools.sh working/sa-test-100 100
```

Within the output directory you will find:

* `alerts/` containing CSV files of the alerts each static analyzer found per file
* `clang_sa/` containing XML output from clang
* `cppcheck/`containing XML output from cppcheck
* `frama-c/` containing text output from frama-c
* `src/` containing the generated sa-bAbI .c files
* `tokens/` containing the tokenized .c files
* `tool_confusion_matrix.csv` reporting results on the whole dataset
* `tool_confusion_matrix_sound.csv` reporting results on the sound subsample of the dataset

## Deep learning quickstart

Build and activate the conda environment for the deep learning component
```
$ cd pipeline
$ conda env create -f environment.yml
$ source activate sa_babi
```

Train the deep learning model on the training data.
Note that the `pipeline/constants.py` file is already setup for
`working/sa-train-1000`. If you wish to
use different data, modify `pipeline/constants.py` appropriately before
running the following command.
```
$ python train.py
```

The current validation script only serves the needs of the arXiv.org paper.
TODO: Develop a more general validation script.

## Predict true c files

1. predict directly: we have put `models`, `vocal.pkl`, `instances.npy` in `working/sa-train-10000`, these files will be used to predict. You can use them to predict directly.

```
$ ./sa_gen_tokens.sh working/sa-test-trueC
$ cd pipeline
$ source activate sa_babi
$ python test_trueC.py
```

2. retrain model and then predict: generate new sa train data and train the model, and then predict.

```
$ SA_SEED=0 ./sa_e2e_no_tools.sh working/sa-train-10000 10000
$ cd pipeline
$ source activate sa_babi
$ python train.py

$ cd ..
$ ./sa_gen_tokens.sh working/sa-test-trueC
$ cd pipeline
$ source activate sa_babi
$ python test_trueC.py
```



# Docker Infrastructure

This repository, in part, contains an automated system for:

1) Generating source code
2) Generating tokens from the source
3) Running open-source static analysis tools on the source
4) Scoring the tool outputs against ground truth

This system is built with docker and docker compose.

## Prerequisites
This system has been primarily run on Linux (Ubuntu 16.04), but should work on
any Linux-like system (including Mac), assuming the following are present:
1) docker
2) docker-compose
3) bash
4) realpath (obtainable through the coreutils homebrew package on Mac)

## Building

The necessary docker images can be built with
```
docker-compose build
```

### Running the SA Pipeline

The `sa_e2e.sh` script runs the tool pipeline end-to-end. The usage
for this script is:
```
bash sa_e2e.sh <working_dir> <num_instances>
```
Where `<working_dir>` is the path to a local directory where outputs will be
stored (will be created if it doesn't exist), and `<num_instances>` is the
number of testcases that will be generated.

After this script is run, `<working_dir>` will contain:
1) Generated source files in the `src` directory
2) Tokenized source files in the `tokens` directory.
3) Raw tool outputs in directories named with tool names (e.g. cppcheck)
4) Aggregated tool alerts in the `alerts` directory. The alerts are in a common csv format.
5) Confusion matrices for tools in `tool_confusion_matrix.csv` and `tool_confusion_matrix_sound.csv`

#### Setting the RNG seed
The testcases are randomly generated based on a seed. By default, this
seed is set to a random value, but you can set it to a specific value
by setting the `SA_SEED` environment variable, e.g.

```
SA_SEED=10 bash sa_e2e.sh <working_dir> <num_instances>
```

## Version 0.1

Initial release

