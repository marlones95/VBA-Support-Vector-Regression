# VBA-Support-Vector-Regression
This repository contains MATLAB code for analyzing fMRI data using support vector regression (SVR) in a delayed-match-to-comparison (DMTC) task, as part of the study: "Effector-Dependent Neural Representations of Perceptual Decisions Independent of Motor Actions and Sensory Modalities" (Esmeyer et al.).
The primary goal of the analysis is to investigate effector-dependent neural representations of perceptual decisions, in a task where participants had to compare the frequency of two visual flicker stimuli (f1 and f2). 
Thereby, perceptual decisions (f1 > f2 or f1 < f2) are modelled by the integration of two subjective frequency distributions, which reflect noisy realizations of f1 and f2. To account for the Fechner’s law (Fechner, 1860) frequencies were log-transformed. In addition, the model accounts for the TOE (i.e. contraction bias), which is a commonly found bias in DMTC tasks (Ashourian & Loewenstein, 2011; Pennock et al., 2021; Saarela et al., 2023). The TOE refers to the phenomenon that the mnemonic representation of stimuli tends to regress towards the average of the stimulus set. The Bayesian inference model accounts for the TOE by combining the noisy distribution of the log-transformed f1 (likelihood distribution) with a distribution centred around the mean of a-priori observed log stimuli frequencies (prior distribution) to obtain a posterior distribution (f1’) which is shifted towards the mean of the overall stimulus set (Ashourian & Loewenstein, 2011; Boboeva et al., 2024). Thus, during the decision the subjective, mean-shifted posterior f1’ was compared to the noisy distribution of the log-transformed f2, leading to distinct performance patterns depending on stimulus order and frequency of f1 (f2 is not retained and thus not combined with the prior distribution).
We included 3 free parameters in our model. (1) The variance of the likelihood distribution σ²stim to model the individual’s perceptual precision (1/σ²stim) across all stimulus frequencies. A smaller σ²stim indicated a less noisy perception of the stimulus frequencies. (2) The variance of the prior distribution σ²prior, which models the precision of the prior (1/σ²prior). A higher precision of the prior amplifies the influence of the stimulus mean and thus increases the TOE. (3) The response bias b indicates the participants tendency to prefer f1 > f2 over f1 < f2 or vice versa. These three model parameters were estimated for each participant individually by minimizing the variational free energy in a variational Bayes scheme, as implemented in the VBA toolbox (Daunizeau et al., 2014). 

To quantify if the model with all three parameters (“full model”) provided the most accurate explanation for the behavioural data, we employed Bayesian model selection (BMS; Stephan et al., 2009) to compare it to a set of plausible alternative models. Our three comparison models included the “simple model” incorporating only σ²stim as a free parameter, the “biased simple model” incorporating σ²stim and b as free parameter, and the “unbiased TOE model” incorporating σ²prior and σ²stim as free parameters. In the applied BMS, models were regarded as random effects that could differ between subjects as implemented in the VBA toolbox (Daunizeau et al., 2014) and the variational free energy was used to select the best model (i.e. the model which occurs most frequently in the data set). By punishing for the inclusion of additional parameters, the variational free energy inherently accounts for model complexity and thereby avoids overfitting.

We used the individually estimated measures for the perceptual precision and the TOE to calculate SFDs of all 8 stimulus pairs by subtracting the posterior means of f1’ and f2. Since the means had to be subtracted in different orders, depending on the two varying comparison directions, a total of 16 SFD values (8 stimulus pairs * 2 comparison directions) were extracted for each participant. The SFDs were used to model the strength of the decision (i.e. the subjective decision values) in the subsequent fMRI decoding analysis.

To identify brain regions with activation patterns being predictive of subjective decision values (i.e. SFDs), we applied an MVPA whole-brain searchlight approach for each participant. Voxel-wise activation patterns that exhibited a linear relationship with SFDs were obtained with a linear SVR using version 3.999F of The Decoding Toolbox (TDT; Hebart et al., 2015). A searchlight radius of 5 voxel was chosen as to implement a suitable payoff between sensitivity and specificity, where we confirmed the consistency of the main findings by additional control analyses with searchlight sizes of 4- and 6-voxel radius. Run-wise beta estimates of the GLM analysis were retrieved for all incorporated voxels of each searchlight location. The 16 SFD values from the Bayesian model were used as labels for an SVR. Then, a six-fold leave-one-run-out cross-validation procedure was applied, as implemented in TDT (Hebart et al., 2015). The resulting prediction accuracy maps comprised Fisher-z-transformed correlation coefficients at each searchlight location, depicting the correlation between true and predicted SFD labels. For the subsequent group-level analyses, the single-subject correlation maps were normalized to MNI space, resampled to a voxel size of 2 × 2 × 2 mm³ and spatially smoothed using a 3 mm full width at half maximum Gaussian filter. Correlation maps were then subjected to a one-sample t-test, to test for local brain activation patterns that showed significantly positive prediction accuracies of SFDs on the group level. 

## Scripts
Matlab codes for the analyses.

### VBA
- VBA_MAIN: Batch script for the variational Bayes routine
- f_vDMTC_simple: Evolution function
- g_vDMTC_simple_bias: Observation function
- Plot_TOE: Illustration of the time-order effect
- Figure_2b_c: Script for Figures 2b and 2c
- Figure 2D: Script for Figure 2D
- Supplementary_Figure_1: Script for Supplementary Figure 1
- compute_TOE_performance: Function computing d' and empirical time-order effect
- Model_comparisons: Script performing single subject model comparison and group level Bayesian model selection

### fMRI Analyses
- Decoding_Batch: Batch script running all pre-processing and analysis functions (first level)
- D1_glm_1stLevel: Function running the GLM
- D2_Decoding_SVM: Function running the support vector machine decoding
- Second_Level_Decoding: Script running the 2nd level one-sample t-test

### Required software packages and toolboxes: 
- SPM12: https://www.fil.ion.ucl.ac.uk/spm/software/spm12/
- The Decoding Toolbox (TDT): https://sites.google.com/site/tdtdecodingtoolbox/
- VBA toolbox: https://mbb-team.github.io/VBA-toolbox/