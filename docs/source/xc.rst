==========================================
ccf(t_spec,t_f_names,vel_width=200.0,f_t):
==========================================
CCF Specifics: In this implementation, the cross correlation
at each point is normalized by the number of flux array values
that went into that point, NOT the total number of point in the
array. This makes the most sense to me, but most formulae you
find do not do this.

**********
Parameters
**********
f_s: array-like

Input flux array from the science spectrum. Array assumes
that the corresponding wavelength array is spaced
logarithmicly, i.e. in linear velocity spacing, and are
continuum normalized and inverted.

f_t: array-like

Input flux array from the template spectrum. Array assumes
that the corresponding wavelength array is spaced
logarithmicly, i.e. in linear velocity spacing, and are
continuum normalized and inverted.

m : int

Number of units in velocity space with which to compute the
cross correlation, must be an odd number.

v_spacing : float

The velocity spacing of the input flux arrays -- must be the
same between them.

*******
Returns
*******
ccf : array-like

The computed cross correlation function

ccf_v : array-like

The velcoity that corresponds to each cross correaltion value
above.



=====================================================================================================================
todcor(t_f_names,t_spec1,t_spec2,vel_width=200.0,stamp_size=5,alpha_fit=True,guess=False,results=True,text_out=True):
=====================================================================================================================
Two-Dimentional Cross Correlation
Algorithm framework - Zucker & Mazeh 1994 (Appendix)
http://adsabs.harvard.edu/abs/1994ApJ...420..806Z

This function preforms a two-dimensional cross correlation on
two input SAPHIRES dictionaries that have been "prepared" by the
saphires.utils.prepare function.

The t_spec1 and t_spec2 files should have already been run through
utils.prepare.

The resultant 2 dimensional distribution is plotted as a contour
plot with the primary RV on the y axis (the spectrum that is
prepared with the template that most closely matches the primary),
and the secondary RV on the x axis. Here you can interactively
zoom aroun with the typical matplotlib funcationallity to find
the peak you want to fit. For a double-lined spectroscopic
binary there are going to be 4 peaks. Two will fall along the
unity line (dotted), which are not the ones you'll want to fit.
Two other peaks should appear on either side of the untity line,
either are possibly the one you want to fit.

- If you have the alpha_fit parameter set to True, you will also have a contour plot of the flux ratio, showing values less than 1.0. One of the suitable peaks will have a flux ratio below 1, i.e. it will have a flux_ratio contour at the todcor peak location, and one will not. As long as you haven't messed up the templates really bad, you'll always want to pick the peak that has a flux ratio value < 1

- If you do not fit the flux ratio value, choose the tallest peak. Sometimes this is difficult to tell, so I recommend setting alpha_fit to True. You select the peak you want to fit by pressing 'm' when the cursor is over it. Pressing 'return' in the terminal will let the function continue.

The procedure described above requires interactive capabilities
that may only work in an ipython session.

Alternatively, you can set the guess parameter and skip the interactive
bit. In this case it is fine, and faster to run in a standard python
session.

Once a peak is selected, a stamp from the full 2D distribution,
with a size defined by the stamp_size parameter, is fit with a
2d gaussian where x and y values of the peak are the secondary and
primary RVs, repsectively.
(There is some code below to do a 2D quadratic fit and the interpolated
maximum below, but I'm going with the 2D gaussian in this version.)

The alpha distributions is interpolated over with a cublic spline
to return the flux ratio value at the peak location.

How do I know I have the best template?

- I typically take a high s/n spectrum where the primary and secondary are very well separated in velocity, and run todcor in the non-interactive mode over a grid of templates. The best pair should produce the highest TOODCOR peak (returned in the 'tod_vals' array). Note that the parameter space can be huge: different temperatures, rotational broadening, etc.

**********
Parameters
**********
t_f_names: array-like

Array of keywords for a science spectrum SAPHIRES dictionary. Output of
one of the saphires.io read-in functions.

t_spec1 : python dictionary

SAPHIRES dictionary for the science spectrum that has been prepared with
the utils.prepare function with a template spectrum. If using different
spectral templates (i.e. different temperatures/spectral types) this
should generally be the hotter of the two. The flux ratio (alpha) assumes
this should be the brighter of the two, which in most cases is the hotter
star.

t_spec2 : python dictionary

SAPHIRES dictionary for the science spectrum that has been prepared with
the utils.prepare function with a template spectrum. If using different
spectral templates (i.e. different temperatures/spectral types) this
should generally be the cooler of the two. The flux ratio (alpha) assumes
this should be the dimmer of the two, which in most cases is the cooler
star.
If you using the same template for both stars, you do not need to
prepare the same science spectrum twice, t_spec1 and t_spec2 can be the
same dictionary

vel_width : float

The range over which to compute the two-dimensional cross correlation.
Larger values take longer to run. Small values could exclude a peak
you care about. The default value is 200 km/s.

stamp_size : int

The size around the peak to fit with a gaussian. Units are in grid
point units, i.e. velocity spacing steps. The default value is 5, but
the ideal value depends on the velocity resolution and if you have
done any oversampling in the utils.prepare step, best to try a few
values.

alpha_fit : bool; float

Whether the flux ratio should be a fit parameter. If True, the flux
ratio is fit, and a contour plot is presented in the interactive mode
to aid in selecting the most appropriate peak. If False, the value
is set to 1. If a float is presented, the alpha value is set to that
float. The default value is True.

guess : bool; array-like

Whether to run todcor in the interactive mode. If False, todcor is
run in the interactive mode where you choose the best peak. The
alternative options is a two element array that give a guess for the
primary and secondary velocities. The default value is False.

results : bool

Option to plot a zoom in of the peak stamp with the best fit peak
location. A nice sanity check. The default value is True.

text_out: bool

Option to output a text file with the results of TODCOR. If True,
the file will have the nomenclature:

[FileName]_[TempName1]_[TempName2]_todcor.dat, and will contain the
RV1, RV2, flux ratio, and todcor peak values. The default value is
True.

*******
Returns
*******
spectra : dictionary

A python dictionary with the SAPHIRES architecture. The output dictionary
will have 2 new keywords as a result of this function. And is a copy of
t_spec1.

['tod_vals']  - An array with the results of the todcor fit:
RV1, RV2, flux ratio, TODCOR peak height.
Note that if you have applied a shift to your spectra
with saphires.utils.apply_shift, that shift is not
accounted for here -- these values are unaware of any
shifts.

['tod_temps'] - The names of the templates used when running TODCOR
