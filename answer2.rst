Dear Brian,

Thanks you for reviewing this manuscript.
As you stated, the code of pyFAI has been developed for speed, but we hoped not to compromised too much on the clarity.
This focus on speed should also simplify the life of our users as pyFAI is designed to run on a single computer without needing any computer cluster/grid/cloud to get good preformances.

About errors
------------

PyFAI tries to handle errors to the best extends possible and tries to be consistent with other software of reference like Fit2D and SPD.
One should separate the "initialization" of the variance from the propagation of variance.

Initialization of the variance
..............................

PyFAI allows any variance array to be passed as input and have it propagated.
If the experimental setup allows the acquisition of many frames of the same (stable) sample, one can extract the per_pixel variance and propagate it via pyFAI.
This is for example done on the SAXS beamline at the Diamond light source (personal communication from Nick Terrill).
Most of the time one wants to extract the variance from a single frame, for this two options are possible:
* Use the Poisson Law which states that the variance is the raw signal (corrected from any artificial offset and gain effect from the detector).
  This directly applicable for pixel detectors, explaining the success of Pilatus detectors on SAXS beamlines.
  This option is available via the keyword *error_model="poisson"* as option to the azimuthal integrator call.
* One may also measure the variance of intensity within all pixel falling into a single bin during integration (being part of the same ring).
  This method is accessible via the keyword *error_model="azimuthal"* as option to the azimuthal integrator call and
  looks similar to what you describe in your review article "Everything SAXS".
  This option is not widely used as it tend to over-estimate the error (personal communication from Olivier Tache).

Propagation of the error
........................

All *regridding engines* in pyFAI provide the weighted and unweighted histograms in addition to integrated curve and the bin boundaries.
Those values are used especially when propagating errors: sigma = sqrt( variance_weighted_histogram ) / variance_unweighted_histogram

But this is only correct with *regridding engines* without pixel splitting as described in
doi:10.1107/S1600576714010516
As pixel splitting induces bin-correlation, the reported errors are overestimated (~20% if the bin size is a pixel).
One solution would be to take the trace of the covariance matrix as defined in Eq9 of this publication.
While this is probably the way to go, there are currently no resources available to implement it.
Pixel splitting is an issue only if the detector is perfect and has no point spread function which is true only for pixel detectors with large pixels (i.e. Pilatus).

About corrections
-----------------

Yes Brian, many preprocessing corrections are available by default as
dark/flat/polarization/solid angle correction as you described in your review article.

I have the feeling you are looking for something like this:
doi:10.1016/j.procs.2011.04.061
but I have the feeling the iPython notebook gets more momentum recently.

To partially answer your question: when pyFAI saves the (processed) data, most parameters used for the processing are saved as metadata which is already a good start.

Pixel thickness effect p4 l18.
------------------------------
Anything is possible within a HDF5 file. At least little things are impossible with HDF5 and the sensitive layer thickness is clearly possible to add.
While this correction has been requested by high-energy diffractionists I have no idea how to deconvolute
the point-spread function without introducing tons of noise.


Constrained refinement: p6 l55
------------------------------
It is possible to provide a default parameter on the command line (parameter in [dist, poni1, poni2, rot1, rot2, rot3, wavelength]) and also fix or not its value.
By default everything is free except the wavelength.
This can be done, either on the command line (see pyFAI-calib --help) or during the refinement.
For example, in the "new" figure2, the refinement has been perform this way:

? set dist 0.615
? fix dist
? set rot1 0
? fix rot1
? refine

Strains p7 l60
--------------
Calibrant are usually "strain-less". But to answer your question, pyFAI-calib is really used a lot on beamline doing strain measurement.
They use CeO2 which gives very thin lines and provides calibration with a precision of 0.2 pixels for the PONI localization (in the best case).
Mis-calibration leads to an error in sin(chi) whereas strains gives errors in sin(2*chi). So it is possible to distinguish them.

p8 l33
Peak position at the sub-pixel level is especially important in material science where they are looking for sharp features,
so the smallest neighborhood possible has been chosen.

p12 l68
The way FIT2D makes the pixel splitting has been reverse-engineered: we got no support from its author.
For sure,  FIT2D considers the pixel with border parallel to the 2theta/chi axes
The pixel is smaller, maybe it considered as a square in output space with the same area ?
PyFAI considers the bounding rectangle which causes an "over-smearing" of a fraction of a pixel.
The ragged top of the FIT2D bin is also a surprise to me: I have no explanation.

p15 About polarization:
For now the polarization correction is the same as the one from FIT2D
Issue #52 has not (yet) been implemented.

P24 l39:
The precision is "measured", as explained, by the offset measured between the raw image and and the image reconstructed from the 1D curve and the geometry.
One could also take 2 section at 180° appart and measure the offset (planed for implementation).