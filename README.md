# Contents

# Overview
This repository contains code used for utilizing machine-learning models to design enzyme sequences with improved activity. The models rely on upper confidence bound (UCB) optimization, a strategy that balances exploiting regions of the sequence-function landscape that are highly active, and exploring new regions of the landscape.
# Requirements
* python 3.7.4, including the following packages
  * numpy 1.17.2
  * scikit-learn 0.21.3
  * scipy 1.2.1
  * matplotlib 3.1.3
  * pandas 1.0.3
# Installation Guide
* The contents are available on GitHub at https://github.com/jgreenhalgh2/UCB_optimization_ACRs
* The scripts used in this repository take very little time to download. The run time for the scripts in UCB_tools.py depends on the operation being performed and the number of datapoints. Cross validation can take a few minutes with a lot of data, but other components are fairly quick. 

