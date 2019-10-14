
# Dependencies

1. AFNI
2. Python 2.7
3. numpy
4. scipy


# Installation

Install Python and other dependencies. If you have AFNI installed and on your path, you should already have an up-to-date version of ME-ICA on your path. Running `meica.py` without any options will check for dependencies and let you know if they are met. If you don't have numpy/scipy (or appropriate versions) installed, I would strongly recommend using the [Enthought Canopy Python Distribution](https://www.enthought.com/downloads/). Click [here](http://wiki.org/installing.html) for more installation help.

# Important Files and Directories

- `meica.py` : a master script that performs preprocessing and calls the ICA/TE-dependence analysis script `tedana.py`
- `meica.libs` : a folder that includes utility functions for TE-dependence analysis for denoising and anatomical-functional co-registration
- `meica.libs/tedana.py` : performs ICA and TE-dependence calculations

# Usage

fMRI data is called: 		rest_e1.nii.gz, rest_e2.nii.gz, rest_e3.nii.gz, etc. 
Anatomical is:		mprage.nii.gz

meica.py and tedana.py have a number of options which you can view using the -h flag. 

Here's an example use (healthy subject) :

    meica.py -d rest1_e1.nii.gz,rest1_e2.nii.gz,rest1_e3.nii.gz -e 15,30,45 -b 15s -a mprage.nii --MNI --prefix sub1_rest

This means:

    -e 15,30,45   are the echo times in milliseconds
    -d rest_e1.nii.gz,rest_e2...   are the 4-D time series datasets (comma separated list of dataset of each TE) from a multi-echo fMRI acqusition
    -a ...   is a "raw" mprage with a skull
    -b   15 means drop first 15 seconds of data for equilibration
    --MNI   warp anatomical to MNI space using a built-in high-resolution MNI template. 
	--prefix sub1_rest   prefix for final functional output datasets, i.e. sub1_rest_....nii.gz

Here's an example use in patient with brain lesion (i.e. Stroke):

    python2 /where_you_place_me-ica_folder/me-ica-master/meica.py --OVERWRITE -d srestingMecho1.nii,srestingMecho2.nii,srestingMecho3.nii -e 10,20,30 -b 2v -a mprage_brain.nii.gz --space=/usr/share/fsl/data/standard/MNI152_T1_2mm_brain.nii.gz --qwarp --fres 2.5 --prefix=sub1_rest --cpus=16  --coreg_mode=lp-t2s --no_skullstrip --keep_int --tpattern=realignMB2.txt

This means:

    -e 10,20,30 are the echo times in milliseconds    
    -d srestingMecho1.nii,srestingMecho2.nii,srestingMecho3.nii are the 4-D time series datasets (comma separated list of dataset of each TE) from a multi-echo fMRI acqusition
    -a mprage_brain.nii.gz is a previously skullstripped mprage 
    -b 2V   means drop first 2 volumes for equilibration
    --space=/usr/share/fsl/data/standard/MNI152_T1_2mm_brain.nii.gz warp anatomical to MNI space using a built-in high-resolution FSL MNI template. 
    --qwarp type of nonlinear normalization in AFNI
    --fres 2.5 isotropic resample of fucntional data to 2.5mm
    --prefix sub1_rest   prefix for final functional output datasets, i.e. sub1_rest_....nii.gz
    --cpus=16 number of core avaliable  for processing data
    --coreg_mode=lp-t2s  functional to anat pearson correlation
    --no_skullstrip : this avoid skullstripping of anatomical T1w3D as provided in -a ...
    --keep_int : only necessary for debug purpose, mantaining temporary results
    --tpattern=realignMB2.txt : only necessary for specifying slice timing or in case of Multiband acquisition
    
  Example of realignMB2.txt for a MB2, ascending, with 36 slices,TR 1sec:
    
    .055555555555556	0.111111111111111	0.166666666666667	0.222222222222222	0.277777777777778	0.333333333333333	0.388888888888889	0.444444444444444	0.5	0.555555555555556	0.611111111111111	0.666666666666667	0.722222222222222	0.777777777777778	0.833333333333333	0.888888888888889	0.944444444444444	1	0.055555555555556	0.111111111111111	0.166666666666667	0.222222222222222	0.277777777777778	0.333333333333333	0.388888888888889	0.444444444444444	0.5	0.555555555555556	0.611111111111111	0.666666666666667	0.722222222222222	0.777777777777778	0.833333333333333	0.888888888888889	0.944444444444444	1	
	
Again, see `meica.py -h` for handling other situations such as: anatomical with no skull, no anatomical at all, applying FWHM smoothing, non-linear warp to standard space, etc.

Click [here](http://wiki.org/group_analysis.html) more info on group analysis.

## Output

- `./meica.rest1_e1/` : contains preprocessing intermediate files. Click [here](http://wiki.org/meica_preprocessing.html) for detailed listing.
- `sub1_rest_medn.nii.gz` : 'Denoised' BOLD time series after: basic preprocessing, T2* weighted averaging of echoes (i.e. 'optimal combination'), ICA denoising. Use this dataset for task analysis and resting state time series correlation analysis. See [here](http://wiki.org/viewing_results.html#dof) for information on degrees of freedom in denoised data.
- `sub1_rest_tsoc.nii.gz` : 'Raw' BOLD time series dataset after: basic preprocessing and T2* weighted averaging of echoes (i.e. 'optimal combination'). 'Standard' denoising or task analyses can be assessed on this dataset (e.g. motion regression, physio correction, scrubbing, blah...) for comparison to ME-ICA denoising.
- `sub1_rest_mefc.nii.gz` : Component maps (in units of \delta S) of accepted BOLD ICA components. Use this dataset for ME-ICR seed-based connectivity analysis.
- `sub1_rest_mefl.nii.gz` : Component maps (in units of \delta S) of ALL ICA components.
- `sub1_rest_ctab.nii.gz` : Table of component Kappa, Rho, and variance explained values, plus listing of component classifications. See [here](http://wiki.org/viewing_results.html#kappa_spectra) for more info.

For a step-by-step guide on how to assess ME-ICA results in more detail, click [here](http://wiki.org/viewing_results.html)

#Some Notes

- Make sure your datasets have slice timing information in the header. If not sure, specify a `--tpattern` option to `meica.py`. Check AFNI documentation of [3dTshift](http://afni.nimh.nih.gov/pub/dist/doc/program_help/3dTshift.html) to see slice timing codes.
- For more info on T2* weighted anatomical-functional coregistration click [here](http://wiki.org/meica_alignp_mepi_anat.html)
- FWHM smoothing is not recommended. tSNR boost is provided by optimal combination of echoes. For better overlap of 'blobs' across subjects, use non-linear standard space normalization instead with `meica.py ... --qwarp`
