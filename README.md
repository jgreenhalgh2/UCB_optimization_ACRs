# Overview
This repository contains code used for utilizing machine-learning models to design enzyme sequences with improved activity. The models rely on upper confidence bound (UCB) optimization, a strategy that balances exploiting regions of the sequence-function landscape that are highly active, and exploring new regions of the landscape.
# Software Requirements
The algorithms in these scripts have been tested on the following systems:
* macOS: High Sierra (10.13)
* macOS: Catalina (10.15.7)
## Python Dependencies
* numpy 1.17.2
* scikit-learn 0.21.3
* scipy 1.2.1
* matplotlib 3.1.3
* pandas 1.0.3
* seaborn 0.10.0
# Installation Guide
* The contents are available on GitHub at https://github.com/jgreenhalgh2/UCB_optimization_ACRs
* The time to download the scripts and any necessary Python dependencies should be fairly short. 
# Demo:
## To analyze GC data:
* Package the raw chromatograph data using either `package_gc_data.py` or `package_gc_data_v2.py`. `package_gc_data.py` must be run from within an environment, such as a `jupyter-notebook` or `spyder`. `package_gc_data_v2.py` can be run directly from the command line. The input for the function is the name of the folder containing the raw text files of the chromatographs. A command such as `python package_gc_data_v2.py gc_07-23-2020` would package all of the chromatographs into a datafile called `gc_07-23-2020.p`.
* The gc_tools package can then be used to process the data. This can be done in a script within spyder (as was done in `gc_07-23-2020.py`), in a jupyter notebook, or by running the script via the terminal (`python gc_07-23-2020.py`, though this command won't produce plots used in the course of the analysis). Briefly:
 * The pickled data file generated from the packaging function is loaded 
 * Parameters relevant to convert peak areas (such as extraction volumes, retention time boundaries, etc.) are specified
 * The peaks are retrieved (using a function like `gc_tools.get_peaks_manual`)
 * The retention time windows are adjusted manually as needed
 * The peaks are integrated 
 * Concentrations are calculated using internal standards 
### Expected Output of GC Script
* This script should produce a csv file containing concentrations of various fatty alcohols, with their corrseponding labels

### Expected Run Time: 
* The script takes 30-60 seconds to run. The analysis may take longer if manual adjustment of retention times is necessary

## To Run UCB Algorithm:

### Preprocessing:
* If multiple batches of data are available (i.e. multiple GC runs), these must be combined.
* Any replicates must be averaged. This can be done either in Python or in Microsoft Excel (or a similar program).
* A block alignment file must be supplied as a pickled file. A csv block alignment is included. 
* If using a structural representation, a list of contacting residues must also be included. This file should be a pickled dictionary with each entry in the format `{(resi_x,resi_y): weight}`, where `resi_1` and `resi_2` are residue numbers, and `weight` is the importance of the contact, or it's abundance in an ensemble of structural models. 

### Initialization:
* A representation should be selected (either structural, sequence, or block), and the appropriate function from `UCB_tools` should be selected to load that representation. 
* For example, in the demo script, the command is `UCB_tools.structure_representation('acr_block_aln.p',contact_dict,csvfile)`, where `csvfile` is the name of the file containing the processed results from GC analysis. This step generates a sequence encoding (X), transforms the titers (act) into the label vector Y (also called activity), and generates a few other intermediary variables. 
* An optional distance matrix can be generated to limit the number of allowed block substitutions for a chimeric sequence (using the `calc_hamming_distance` function provided in the demo script).

### Gaussian Naive Bayes Classification:
* The `function UCB_tools.get_active_sequence` can be used to transform the continuous labels into a binary vector useful for classification
* The Gaussian Naive Bayes Classifier is trained using the binary data and the sequence encoding, and then applied to predict the activity of all untested sequences. 
* `UCB_tools.get_active_predictions` is used to identify the block sequences that are predicted to be active 
* `UCB_tools.gnb_cross_val` utilizes Leave One Out Cross Validation to validate the classifier. This function also produces a ROC curve and confusion matrix.
* Gaussian Process (GP) regression models are then used to map the encodings of active sequences to their activity (Y). Leave one out cross validation is used to tune the models (`UCB_tools.GP_cross_val_scan`).
* The hyperparameter, `lambda`, is selected by either 1) maximizing the correlation coefficient, 2) minimizing the squared error of the model, or 3) arbitrarily selecting a point that comes close to either or both of these objectives but doesn't meet them. 
* New sequences are designed using `UCB_tools.select_new_sequences`. 
* An optional final step, `UCB_tools.check_for_old_assemblies`, can be employed to check the designed sequences against a list of previously attempted sequences (i.e. sequences that were previously designed, but unsuccessfully assembled). 

### Expected Output: 
* This process will produce a ist of sequences that maximize the UCB for the GP models. 

### Expected Run Time: 
* Depending on the number of data points and the encoding method used, the script can take 1-2 minutes to run. 

