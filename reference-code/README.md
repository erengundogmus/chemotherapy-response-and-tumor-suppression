# Reference Implementation

The original implementation is included as a Git submodule:

https://github.com/kisblilla/piecewise-pinn-tumor-growth

## Setup

After cloning or pulling this project, initialize the reference code with:

git submodule update --init --recursive

## Observation Window Changes

The observation-window.patch file contains the changes used for the 105, 100, 50, 30, and 20-day experiments.

Apply the patch with:

git -C reference-code/piecewise-pinn-tumor-growth apply ../observation-window.patch

The observation period can then be selected by changing STUDY_END_DAY in main.py.
