---
---

## Functional (fMRI) Pipeline

1. Prepare fieldmaps for correction of susceptibility distortions

    1. Estimate field map from the two “best” spin-echo volumes (1 per
    phase-encode direction) using FSL TOPUP<sup>1</sup>.  “Best” is
    defined by smoothness in the z-dimension (stdev of the slice-to-slice
    difference in the z-dimension)

    2. If visual inspection indicates that the spin-echo has significant 
    motion contamination then use the dual-echo-time-derived fieldmap instead
    of the spin-echo-derived fieldmap

2. Registration

    1. Boundary-based registration (BBR, FSL FLIRT<sup>3</sup>) of the fieldmap magnitude to
    the T2w structural

    2. Boundary-based registration (BBR, FSL FLIRT<sup>3</sup>) of the sbref to the
    T2 structural incorporating field map-based distortion correction of
    the sbref

    3. Linear registration (6-dof, corratio, FSL FLIRT<sup>3</sup>) of the first volume
    of the functional multiband EPI to the sbref

    4. After susceptibility and motion correction (see below), linear registration (6-dof,
    corratio, FSL FLIRT<sup>3</sup>) of the temporal mean of the motion and distortion
    corrected functional multiband EPI to the distortion corrected sbref

    5. Nonlinear diffeomorphic multimodal registration of the age-matched
    T2w and GM/WM tissue probability from the dHCP volumetric atlas<sup>7</sup> to
    the subjects T2w and GM/WM tissue probability using ANTs SyN<sup>6</sup>. The GM/WM probability for each subject was calculated as the partial volume estimate from a tool called [Toblerone](https://toblerone.readthedocs.io/en/latest/). We have [augmented](https://git.fmrib.ox.ac.uk/seanf/dhcp-resources/-/blob/master/docs/dhcp-augmented-volumetric-atlas-extended.md) 
    the dHCP volumetric atlas with week-to-week
    nonlinear transforms estimated using a diffeomorphic T2w
    registration (ANTs SyN).  The appropriate transforms are then combined
    to yield a (40 week) template-to-structural transform

    6. From these primary registrations the following composite transforms
    are calculated:

        1. Fieldmap to native functional

        2. Motion and distortion corrected functional to 40-week template
        from the dHCP volumetric atlas

3. Susceptibility and motion correction

    1. Slice-to-volume motion correction and motion-by-susceptibility
    correction is performed using FSL EDDY<sup>2</sup>

4. ICA Denoising

    1. Temporal high-pass filter (150s high-pass cutoff) and ICA denoising
    using FSL FIX<sup>4</sup>, pre-trained with manually-labelled data from 50 dHCP
    neonatal subjects, to identify artefactual ICs (accuracy: median TPR=98%,
    median TNR=96%). The ICA dimensionality was capped at 600 ICs.

    2. Noise ICs and motion parameters regressed from motion and distortion
    corrected functional multiband EPI.

## fMRI QC

1. Numerous quality assurance metrics are calculated during the
pre-processing. Six of these are specifically compared against the population
distribution to flag outliers for manual inspection and potential exclusion:

    1. Mean DVARS<sup>5</sup> of the ICA denoised functional EPI; provides a snapshot of output signal quality
    
    2. Mean tSNR of the ICA denoised functional EPI; provides a snapshot of output signal quality
    
    3. Normalised mutual information of the source (moving) image, re-sampled
    to reference space, and the reference (fixed) image, for each of the
    primary registrations:

        1. Fieldmap to structural T2w

        2. Native functional to sbref

        3. Motion and distortion corrected functional to sbref

        4. Sbref to structural T2w

        5. Age-matched atlas template T2w to native structural T2w

2. All QA measures were converted to Z-scores and flipped as necessary so
that positive z-scores are good and negative bad.  Subject/sessions with
a z-score < -2.5 on any QC metric were flagged for further inspection.

> 28 subject/sessions were flagged for futher inspection (see figure).  All flagged subject/sessions are included in the release.

![](fmri_qc_z_distns.png)

## References

1. Andersson, J. L., Skare, S. and Ashburner, J. **How to correct susceptibility
distortions in spin-echo echo-planar images: application to diffusion tensor
imaging** *Neuroimage (2003), 20: 870-888.* DOI: 10.1016/S1053-8119(03)00336-7

2. Andersson, J. L. and Sotiropoulos, S. N. **An integrated approach
to correction for off-resonance effects and subject movement in
diffusion MR imaging** *Neuroimage (2016), 125: 1063-1078.* [DOI:
10.1016/j.neuroimage.2015.10.019](https://doi.org/10.1016/S1053-8119(03)00336-7)

3. Jenkinson, M., and Smith, S. **A global optimisation
method for robust affine registration of brain images**
*Medical image analysis (2001), 5(2): 143–156.* [DOI:
10.1016/S1361-8415(01)00036-6](https://doi.org/10.1016/j.neuroimage.2015.10.019)

4. Salimi-Khorshidi, G., Douad, G, Beckman, C. F., Glasser, M. F.,
Griffanti, L., and Smith, S. M. **Automatic denoising of functional
MRI data: combining independent component analysis and hierarchical
fusion of classifiers** *NeuroImage (2014), 90: 449–468.* [DOI:
10.1016/j.neuroimage.2013.11.046](https://doi.org/10.1016/S1361-8415(01)00036-6)

5. Power, J. D., Barnes, K. A., Snyder, A. Z., Schlaggar,
B. L.,  and Petersen, S. E. **Spurious but Systematic
Correlations in Functional Connectivity MRI Networks Arise from
Subject Motion** *NeuroImage (2012), 59(3): 2142–54.* [DOI:
10.1016/j.neuroimage.2011.10.018](https://doi.org/10.1016/j.neuroimage.2011.10.018)

6. Avants, B.B., Epstein, C.L., Grossman, M., Gee, J.C. **Symmetric diffeomorphic image registration with cross-correlation: Evaluating automated labeling of elderly and neurodegenerative brain.** Med. Image Anal. (2008), 12, 26–41. https://doi.org/10.1016/j.media.2007.06.004

7. Andreas Schuh, Antonios Makropoulos, Emma C. Robinson, Lucilio Cordero-Grande, Emer Hughes, Jana Hutter, Anthony N. Price, Maria Murgasova, Rui Pedro A. G. Teixeira, Nora Tusor, Johannes Steinweg, Suresh Victor, Mary A. Rutherford, Joseph V. Hajnal, A. David Edwards, and Daniel Rueckert. **Unbiased construction of a temporally consistent morphological atlas of neonatal brain development**, bioRxiv, 2018. doi: https://doi.org/10.1002/mrm.26796

