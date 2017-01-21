# Data LCLS 2017#
Repository containing the data used for the manuscript:

**Accurate prediction of x-ray pulse properties from a free-electron laser using machine learning**

A. Sanchez-Gonzalez, P. Micaelli, C. Olivier, T. R. Barillot, M. Ilchen, A. A. Lutman, A. Marinelli, T. Maxwell, A. Achner, M. Agåker, N. Berrah, C. Bostedt, J. D. Bozek, J. Buck, P. H. Bucksbaum, S. Carron Montero, B. Cooper, J. P. Cryan, M. Dong, R. Feifel, L. J. Frasinski, H. Fukuzawa, A. Galler, G. Hartmann, N. Hartmann, W. Helml, A. S. Johnson, A. Knie, A. O. Lindahl, J. Liu, K. Motomura, M. Mucke, C. O'Grady, J-E. Rubensson, E. R. Simpson, R. J. Squibb, C. Såthe, K. Ueda, M. Vacher, D. J. Walke, V. Zhaunerchyk, R. N. Coffee and J. P. Marangos

Currently under resubmission for Nature Communications.

## Overview of the experiments ##

The results presented on the paper were obtained using experimental data from two different experiments:

1. **amof6215 - Run 236**:
  * Single pulse photon energy prediction.
  * Single pulse spectral shape prediction.
2. **amo86815 - Run 70**:
  * Time-delay prediction.
  * Double pulse photon energy prediction.

## Description of the data files ##

Inside the folders for each experiment there are a series of data files:

XX\_runYY\_ZZ.npz

where XX indicates the type of data contained and YY the run number. For runs containing more than 30000 events, the data is divided in chunks of 30000 events, with ZZ being the chunk number.

The data is stored in the numpy [.npz format](https://docs.scipy.org/doc/numpy/neps/npy-format.html) which can be easily loaded into a python dictionary using:

    import numpy as np
	datadict = np.load(filename)

Common keys to the dictionaries across all the files are:

* `'XXXXXList'`: Data associated to each of the individual events. These are one-dimensional or bi-dimensional arrays where the first index selects different events.

* `'FiducialList'` and `'TimeList'`: This pair of variables together represent a unique identifier of the events. In principle the same order of events is followed across different files belonging to the same run and the same chunk, but these should be checked to ultimately verify that event information from different files truly correspond to the same event.

* `'XXXXXListMask'`: Since sometimes some devices cannot be recorded for some of the events, there must be a way to indicate when the information about an event coming form a particular data source is not valid. For each variable `'XXXXXList' ` containing information about all of the events, there is an associated one dimensional boolean array named `'XXXXXListMask'` which is set to `True` for the events for which the data in `'XXXXXList'` is valid.

* `'XXXXXNames'`: In some cases there are names associated with the multiple variables stored for each event in a multidimensional array. This variable consists of a list of strings with the name of the variables.

The rest of the keys are particular to each of the files:

### EBeam ###

These files contain part of the fast diagnostics used as input for the model coming from electron beam diagnostics for every single shot at 120 Hz:

* `'EBeamParameterNames'`: Contains a list with the names of the different variables that are stored for each event. For more information on the meaning of these variables please visit [the documentation from LCLS](https://pswww.slac.stanford.edu/swdoc/releases/ana-current/psana-doxy/html/classPsana_1_1Bld_1_1BldDataEBeamV7.html).
* `'EBeamValuesList'`: Bi-dimensional array containing the data where the second index corresponds to each of the variables in `'EBeamParameterNames'`.

### GMD ###

These files contain the rest of the fast diagnostics used as input for the model coming from gas monitor detectors for every single shot at 120 Hz:

* `'GMDParameterNames'`: Contains a list with the names of the different detectors that are stored for each event. For more information on the meaning of these variables please visit [the documentation from LCLS](https://pswww.slac.stanford.edu/swdoc/releases/ana-current/psana-doxy/html/classPsana_1_1Bld_1_1BldDataFEEGasDetEnergyV1.html).
* `'GMDValuesList'`: Bi-dimensional array containing the data where the second index corresponds to each of the variables in `'GMDParameterNames'`.

### EPICS ###

These files contain the environmental diagnostics used as input for the model. The variables are updated approximately at 2 Hz, but still stored at 120 Hz.

* `'EPICSParameterNames'`: Contains a list with the names of the different detectors that are stored for each event.
* `'EPICSValuesList'`: Bi-dimensional array containing the data where the second index corresponds to each of the variables in `'EPICSParameterNames'`.

### Delays ###

These files contain single-shot time-delay information extracted from the processing of XTCAV images for every other shot (60 Hz), used as targets for the prediction.

* `'DelayValuesList'`: One-dimensional array containing the time-delay in fs for every event. A positive delay indicates the low energy pulse arriving first. Because XTCAV images are only recorded at 60 Hz, `'DelayValuesListMask'` is `False` for half of the events.

### Optical ###

These files contain single-shot information obtained from an optical spectrometer running at 120 Hz.

* `'UXSProfileList'`: Two-dimensional array containing the spectral profiles across the second dimension for each of the events. The units are arbitrary, but consistent across events. These profiles are the ones used as targets for the single pulse spectral shape prediction.

* `'xUXS'`: One-dimensional array containing the camera pixel values corresponding to each of the points of the spectral profiles in `'UXSProfileList'`.

* `'UXSSingleFitList'`: Gaussian fit performed for each of the events. There are three values per event: the amplitude in arbitrary units, the center in pixels, and the full width half maximum in pixels. `'UXSSingleFitListMask'` is set to `False` in the cases where the fit does not succeed. The center of the fits are the ones used as targets for the single pulse photon energy prediction.

In order to convert from `pixel` into `energy` in eV, the following calibration formula is used: 

    P1 = 466.
    E1 = 531.5
    a = 9.8
    energy = (pixel-P1)/a+E1


### TOF ###

These files contain single-shot information obtained from a time-of-flight spectrometer running at 120 Hz.

* `'TOFProfileList'`: Two-dimensional array containing the spectral profile across the second dimension for each of the events. The units are arbitrary, but consistent across events. 

* `'xTOF'`: One-dimensional array containing the time-of-flight times in quarters of ns corresponding to each of the points of the spectral profiles in `'TOFProfileList'`.

* `'TOFDoubleFitList'`: Double Gaussian fit performed for each of the events. There are six values per event: the amplitude in arbitrary units, the center in eV, and the full width half maximum in eV for the low energy pulse, followed by the same three variables for the high energy pulse. `'TOFDoubleFitListMask'` is set to `False` in the cases where the fit does not succeed. The centers of the fits are the ones used as targets for the double pulse photon energy prediction.

In order to convert from time-of-flight in quarters of ns (`TOF_qns`) into `energy` in eV, the following calibration formula is used: 

	c=0.3
	me=0.511e6/c**2	
	t0=209.25
	L=0.0868

	TOF_ns=TOF_qns/4
	energy=V+(me/2)*(L**2)/(TOF_ns-t0)**2

## Further information ##

For more information, please do not hesitate to contact the authors of the manuscript.