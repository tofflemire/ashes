===============
air2vac(w_air):
===============
Air to vacuum conversion formula derived by N. Piskunov
IAU standard:
http://www.astro.uu.se/valdwiki/Air-to-vacuum%20conversion

**********
Parameters
**********
w_air : array-like

Array of air wavelengths assumed to be in Angstroms

*******
Returns
*******
w_vac : array-like

Array of vacuum wavelengths converted from w_air


==============================================================
apply_shift(t_f_names,t_spectra,rv_shift,shift_style='basic'):
==============================================================
A function to apply a velocity shift to an input spectrum.

The shift is made to the 'nflux' and 'nwave' arrays.

The convention may be a bit wierd, think of it like this:
If there is a feature at +40 km/s and you want that feature
to be at zero velocity, put in 40.

Whatever velocity you put in will be put to zero.

The shifted velocity is stored in the 'rv_shift' header
for each dictionary spectrum. Multiple shifts are stored
i.e. shifting by 40, and then 40 again will result in a
'rv_shift' value of 80.

**********
Parameters
**********
t_f_names: array-like

Array of keywords for a science spectrum SAPHIRES dictionary. Output of
one of the saphires.io read-in functions.

t_spectra : python dictionary

SAPHIRES dictionary for the science spectrum.

rv_shift : float

The velocity (in km/s) you want centered at zero.

shift_style : str, optional

Parameter defines how to shift is applied. Options are 'basic' and
'inter'. The defaul is 'basic'.

- The 'basic' option adjusts the wavelength asignments with the standard RV shift. Pros, you don't interpolate the flux values; Cons, you change the wavelength spacing from being strictly linear. The Pros outweight the cons in most scenarios.

- The 'inter' option leaves the wavelength gridpoints the same, but shifts the flux with an interpolation. Pros, you don't change the wavelength spacing; Cons, you interpolate the flux, before you interpolate it again in the prepare step. Interpolating flux is not the best thing to do, so the less times you do it the better. The only case I can think of where this method would be better is if your "order" spanned a huge wavelength range.

*******
Returns
*******
spectra_out : python dictionary
A python dictionary with the SAPHIRES architecture. The output dictionary
will be a copy of t_specrta, but with updates to the following keywords.

['nwave']    - The shifted wavelength array

['nflux']    - The shifted flux array

['rv_shift'] - The value the spectrum was shifted in km/s



===================
bf_map(template,m):
===================
Creates a two dimensional array from a template by shifting one unit
of the wavelength (velocity) array m/2 times to the right and m/2 to
the left. This array, also known as the design matrix (des), is then
input to bf_solve.

Template is the resampled flux array in log(lambda) space.

**********
Parameters
**********
template : array-like

The logarithmic wavelengthed spectral template. Must have an
even numbered length.

m : int

The number of steps to shift the template. Must be odd

*******
Returns
*******
t : array-like

The design matrix


==========================================================
bf_singleplot(t_f_names,t_spectra,for_plotting,f_trim=20):
==========================================================
A function to make a mega plot of all the spectra in a target
dictionary.

**********
Parameters
**********
t_f_names : array-like

Array of keywords for a science spectrum SAPHIRES dictionary. Output of
one of the saphires.io read-in functions.

t_spectra : python dictionary

SAPHIRES dictionary for the science spectrum. Output of one of the
saphires.io read-in functions.

for_plotting : python dictionary

A dictionary with all of the things you need to plot the fit profiles
from saphires.bf.analysis.

f_trim : int

The amount of points to trim from the edges of the BF before the fit is
made. The edges are usually noisey. Units are in wavelength spacings.
The default is 20.

*******
Returns
*******
None



===============================
bf_solve(des,u,ww,vt,target,m):
===============================
Takes in the design matrix, the output of saphires.utils.bf_map,
and creates an array of broadening functions where each row is
for a different order of the solution.

The last index out the output (m-1) is the full solution.

All of them here for completeness, but in practice, the full
solution with gaussian smoothing is used to derive RVs and flux ratios.

**********
Parameters
**********
des : arraly-like

Design Matrix computed by saphires.utils.bf_map

u : array-like

One of the outputs of the design matrix's singular value
decomposition

ww : array-like

One of the outputs of the design matrix's singular value
decomposition

vt : array-like

One of the outputs of the design matrix's singular value
decomposition

target : array-like

The target spectrum the corresponds to the template spectrum
that was used to make the design matrix.

m : int

Number of pixel shifts to compute

*******
Returns
*******
b_sols : array-like

Matrix of BF solutions for different orders. The last order if the
one to use.

sig : array-like

Uncertainty array for each order.


======================================================================
bf_text_output(ofname,target,template,gs_fit,rchis,rv_weight,fit_int):
======================================================================
A function to output the results of saphires.bf.analysis to a text file.

**********
Parameters
**********
ofname : str

The name of the output file.

target : str

The name of the target spectrum.

template : str

The name of the template spectrum.

gs_fit : array-like

An array of the profile fit parameters from saphire.bf.analysis.

rchis : float

The reduced chi square of the saphires.bf.analysis fit with the data.

fit_int : array-like

Array of profile integrals from the saphires.bf.analysis fit.

*******
Returns
*******
None



===================================================================
cont_norm(w,f,w_width=200.0,maxiter=15,lower=0.3,upper=2.0,nord=3):
===================================================================
Continuum normalizes a spectrum

Uses the bspline_acr package which was adapted from IDL by
Aaron Rizzuto.

Note: In the SAPHIRES architecture, this has to be done before
the spectrum is inverted, which happens automatically in the
saphires.io read in functions. That is why the option to
continuum normalize is available in those functions, and
should be done there.

**********
Parameters
**********
w : array-like

Wavelength array of the spectrum to be normalized.
Assumed to be angstroms, but doesn't really matter.

f : array-like

Flux array of the spectrum to be normalized.
Assumes linear flux units.

w_width : number

Width is the spline fitting window. This is useful for
long, stitched spectra where is doesn't makse sense to
try and normalize the entire thing in one shot.
The defaults is 200 A, which seems to work reasonably well.
Assumes angstroms but will naturally be on the same scale
as the wavelength array, w.

maxiter : int

Number of interations. The default is 15.

lower : float

Lower limit in units of sigmas for including data in the
spline interpolation. The default is 0.3.

upper : float

Upper limit in units of sigma for including data in the
spline interpolation. The default is 2.0.

nord : int

Order of the spline. The defatul is 3.

*******
Returns
*******
f_norm : array-like

Continuum normalized flux array



============================================
d_gaussian_off(x,A1,x01,sig1,A2,x02,sig2,o):
============================================
A double gaussian function with a constant vetical offset.

**********
Parameters
**********
x : array-like

Array of x values over which the Gaussian profile will
be computed.

A1 : float

Amplitude of the first Gaussian profile.

x01 : float

Center of the first Gaussian profile.

sig1 : float

Standard deviation (sigma) of the first Gaussian profile.

A2 : float

Amplitude of the second Gaussian profile.

x02 : float

Center of the second Gaussian profile.

sig2 : float

Standard deviation (sigma) of the second Gaussian profile.

o : float

Vertical offset of the Gaussian mixture.

*******
Returns
*******
profile : array-like

The Gaussian mixture specified over the input x array.
Array has the same length as x.


===========================
gaussian_off(x,A,x0,sig,o):
===========================
A simple gaussian function with a constant vetical offset.
This Gaussian is not normalized in the typical sense.

**********
Parameters
**********
x : array-like

Array of x values over which the Gaussian profile will
be computed.

A : float
Amplitude of the Gaussian profile.

x0 : float

Center of the Gaussian profile.

sig : float

Standard deviation (sigma) of the Gaussian profile.

o : float

Vertical offset of the Gaussian profile.

*******
Returns
*******
profile : array-like

The Gaussian profile specified over the input x array.
Array has the same length as x.


==========================
make_rot_pro_ip(R,e=0.75):
==========================
A function to make a specific rotationally broadened fitting
function with a specific limb-darkening parameter that is
convolved with the instrumental profile that corresponds to
a given spectral resolution.

The output profile is uses the linear limb darkening law from
Gray 2005

**********
Parameters
**********
R : float

The resolution that corresponds to the spectrograph's
instrumental profile.

e : float

Linear limb darkening parameter. Default is 0.75,
appropriate for a low-mass star.

*******
Returns
*******
rot_pro_ip : function
A function that returns the line profile for a rotationally
broadened star with the limb darkening parameter given by the make
function and that have been convolved with the instrumental
profile specified by the spectral resolution by the make function
above.

=========================
rot_pro_ip(x,A,rv,rvw,o):
=========================

**********
Parameters
**********
x : array-like

X array values, should be provided in velocity in km/s, over
which the smooth rotationally broadened profile will be
computed.

A : float

Amplitude of the smoothed rotationally broadened profile.
Equal to the profile's integral.

rv : float

RV center of the profile.

rvw : float

The vsini of the profile.

o : float

The vertical offset of the profile.

*******
Returns
*******
prof_conv : array-like

The smoothed rotationally broadened profile specified by the
paramteres above, over the input x array. Array has the same
length as x.



================================
make_rot_pro_qip(R,a=0.3,b=0.4):
================================
A function to make a specific rotationally broadened fitting
function with a specific limb-darkening parameter that is
convolved with the instrumental profile that corresponds to
a given spectral resolution.

The output profile is uses the linear limb darkening law from
Gray 2005

**********
Parameters
**********
R : float

The resolution that corresponds to the spectrograph's
instrumental profile.

e : float

Linear limb darkening parameter. Default is 0.75,
appropriate for a low-mass star.

*******
Returns
*******
rot_pro_ip : function

A function that returns the line profile for a rotationally
broadened star with the limb darkening parameter given by the make
function and that have been convolved with the instrumental
profile specified by the spectral resolution by the make function
above.

=========================
rot_pro_ip(x,A,rv,rvw,o):
=========================
**********
Parameters
**********
x : array-like

X array values, should be provided in velocity in km/s, over
which the smooth rotationally broadened profile will be
computed.

A : float

Amplitude of the smoothed rotationally broadened profile.
Equal to the profile's integral.

rv : float

RV center of the profile.

rvw : float

The vsini of the profile.

o : float

The vertical offset of the profile.

*******
Returns
*******
prof_conv : array-like

The smoothed rotationally broadened profile specified by the
paramteres above, over the input x array. Array has the same
length as x.



=========================================================
order_stitch(t_f_names,spectra,n_comb,print_orders=True):
=========================================================
A function to stitch together certain parts a specified
number of orders throughout a dictionary.
e.g. an 8 order spectrum, with a specified number of orders
to combine set to 2, will stich together 1-2, 3-4, 5-6, and 7-8,
and result in a dictionary with 4 stitched orders.

If the number of combined orders does not divide evenly, the
remaining orders will be appended to the last stitched order.


=================================================================================================================================
prepare(t_f_names,t_spectra,temp_spec,oversample=1,quiet=False,trap_apod=0,cr_trim=-0.1,trim_style='clip',vel_spacing='uniform'):
=================================================================================================================================
A function to prepare a target spectral dictionary with a template
spectral dictionary for use with SAPHIRES analysis tools. The preparation
ammounts to resampling the wavelength array to logarithmic spacing, which
corresponds to linear velocity spacing. Linear velocity spacing is required
for use TODCOR or compute a broadening function

oversample -
A key parameter of this function is "oversample". It sets the logarithmic
spacing of the wavelength resampling (and the corresponding velocity
'resolution' of the broadening function). Generally, a higher oversampling
will produce better results, but it very quickly becomes expensive for two
reasons. One, your arrays become longer, and two, you have to increase the m
value (a proxy for the velocity regime probed) to cover the same velocity range.

A low oversample value can be problematic if the intrinsitic width of your
tempate and target lines are the same. In this case, the broadening function
should be a delta function. The height/width of this function will depened on
the location of velocity grid points in the BF in an undersampled case. This
makes measurements of the RV and especially flux ratios problematic.

If you're unsure what value to use, look at the BF with low smoothing
(i.e. high R). If the curves are jagged and look undersampled, increase the
oversample parameter.

**********
Parameters
**********
t_f_names: array-like

Array of keywords for a science spectrum SAPHIRES dictionary. Output of
one of the saphires.io read-in functions.

t_spectra : python dictionary

SAPHIRES dictionary for the science spectrum. Output of one of the
saphires.io read-in functions.

temp_spec : python dictionary

SAPHIRES dictionary for the template spectrum. Output of one of the
saphires.io read-in functions.

oversample : float

Factor by which the velocity resolution is oversampled. This parameter
has an extended discussion above. The default value is 1.

quiet : bool

Specifies whether messaged are printed to the teminal. Specifically, if
the science and template spectrum do not overlap, this function will
print and error. The default value is False.

trap_apod : float

Option to apodize (i.e. taper) the resampled flux array to zero near
the edges. A value of 0.1 will taper 10% of the array length on each
end of the array. Some previous studies that use broaden fuctions in
the literarure use this, claiming it reduced noise in the sidebands. I
havent found this to be the case, but the functionallity exisits
nonetheless. The detault value is 0, i.e. no apodization.

cr_trim	: float

This parameter sets the value below which emission features are removed.
Emission is this case is negative becuase the spectra are inverted. The
value must be negative. Points below this value are linearly interpolated
over. The defulat value is -0.1. If you don't want to clip anything, set
this paramter to -np.inf.

trim_style : str, options: 'clip', 'lin', 'spl'

If a wavelength region file is input in the 'spectra_list' parameter,
this parameter describes how gaps are dealt with.

- If 'clip', unused regions will be left as gaps.

- If 'lin', unused regions will be linearly interpolated over.

- If 'spl', unused regions will be interpolated over with a cubic spline. You probably don't want to use this one.

vel_spacing : str; 'orders' or 'uniform', or float

Parameter that determines how the velocity width of the resampled array
is set.
If 'orders', the velocity width will be set by the smallest velocity
separation between the native input science and template wavelength arrays on
an order by order basis.
If 'uniform', every order will have the same velocity spacing. This is useful
if you want to combine BFs for instance. The end result will generally be
slightly oversampled, but other than taking a bit longer, to process, it should
not have any adverse effects.
If this parameter is a float, the velocity spacing will be set to that value,
assuming it is in km/s.
You can get wierd results if you put in a value that doesn't make sense, so
I recommend the orders or uniform setting. This option is available for more
advanced use cases that may only relevant if you are using TODCOR. See
documentation there for a relevant example.
The oversample parameter is ignored when this parameter is set to a float.


*******
Returns
*******
spectra : dictionary

A python dictionary with the SAPHIRES architecture. The output dictionary
will have 5 new keywords as a result of this function.

['vflux'] 		- resampled flux array (inverted)

['vwave'] 		- resampled wavelength array

['vflux_temp']	- resampled template flux array (inverted)

['vel'] 		- velocity array to be used with the BF or CCF

['temp_name'] 	- template name

['vel_spacing'] - the velocity spacing that corresponds to the
resampled wavelength array

It also updates the values for the following keyword under the right
conditions:

['order_flag'] 	- order flag will be updated to 0 if the order has no
overlap with the template. This tells other functions
to ignore this order.




============================
RChiS(x,y,yerr,func,params):
============================
A function to compute the Reduced Chi Square between some data and
a model.

**********
Parameters
**********
x : array-like

Array of x values for data.

y : array-like

Array of y values for data.

yerr : array-like

Array of 1-sigma uncertainties for y vaules.

func : function

Function being compared to the data.

params : array-like

List of parameter values for the function above.

*******
Returns
*******
rchis : float

The reduced chi square between the data and model.



============================================================================================================================================================
region_select_pkl(target,template=None,tar_stretch=True,temp_stretch=True,reverse=False,dk_wav='wav',dk_flux='flux',tell_file=None,jump_to=0,reg_file=None):
============================================================================================================================================================
An interactive function to plot target and template spectra
that allowing you to select useful regions with which to
compute the broadening functions, ccfs, ect.

This funciton is meant for specrta in a pickled dictionary.

For this function to work properly the template and target
spectrum have to have the same format, i.e. the same number
of orders and roughly the same wavelength coverage.

If a template spectrum is not specified, it will plot the
target twice, where it can be nice to have one strethed
and on not.

Functionality:
The function brings up an interactive figure with the target
on top and the template on bottom. hitting the 'm' key will
mark wavelengths dotted red lines. The 'b' key will mark the
start of a region with a solid black line and then the end of
the region with a dashed black line. Regions should always go
from small wavelengths to larger wavelengths, and regions
should always close (.i.e., end with a dashed line). Hitting
the return key over the terminal will advance to the next order
and it will print the region(s) you've created to the terminal
screen that are in the format that the saphires.io.read
functions use. The regions from the previous order will show
up as dotted black lines allowing you to create regions that
do not overlap.

**********
Parameters
**********
target : str

File name for a pickled dictionary that has wavelength and
flux arrays for the target spectrum with the header keywords
defined in the dk_wav and dk_flux arguments.

template : str, None

File name for a pickled dictionary that has wavelength and
flux arrays for the target spectrum with the header keywords
defined in the dk_wav and dk_flux arguments. If None, the
target spectrum will be plotted in both panels.

tar_stretch : bool

Option to window y-axis of the target spectrum plot on the
median with 50% above and below. This is useful for echelle
data with noisey edges. The default is True.

temp_stretch : bool

Option to window y-axis of the template spectrum plot on the
median with 50% above and below. This is useful for echelle
data with noisey edges.The default is True.

reverse : bool

This function works best when the orders are ordered with
ascending wavelength coverage. If this is not the case,
this option will flip them. The default is False, i.e., no
flip in the order.

dk_wav : str

Dictionary keyword for the wavelength array. Default is 'wav'

dk_flux : str

Dictionary keyword for the flux array. Default is 'flux'

tell_file : optional keyword, None or str

Name of file containing the location of telluric lines to be
plotted as vertical lines. This is useful when selecting
regions free to telluric contamination.
File must be a tab/space separated ascii text file with the
following format:
w_low w_high depth(compated the conintuum) w_central
This is modeled after the MAKEE telluric template here:
https://www2.keck.hawaii.edu/inst/common/makeewww/Atmosphere/atmabs.txt
but just a heads up, these are in vaccum.
If None, this option is ignored.
The default is None.

jump_to : int

Starting order. Useful when you want to pick up somewhere.
Default is 0.

reg_file : optional keyword, None or str

The name of a region file you want to overplay on the target and
template spectra. The start of a regions will be a solid veritcal
grey line. The end will be a dahsed vertical grey line.
The region file has the same formatting requirements as the io.read
functions. The default is None.

*******
Returns
*******
None



================================================================================
region_select_vars(w,f,tar_stretch=True,reverse=False,tell_file=None,jump_to=0):
================================================================================
An interactive function to plot spectra that allowing you
to select useful regions with which to compute the
broadening functions, ccfs, ect.

Functionality:
The function brings up an interactive figure with spectra.
Hitting the 'm' key will mark wavelengths with dotted red
lines. The 'b' key will mark the start of a region with a
solid black line and then the end of the region with a
dashed black line. Regions should always go from small
wavelengths to larger wavelengths, and regions should always
close (.i.e., end with a dashed line). Hitting the return
key over the terminal will advance to the next order and it
will print the region(s) you've created to the terminal
screen that are in the format that the saphires.io.read_vars
function can use. The regions from the previous order will
show up as dotted black lines allowing you to create regions
that do not overlap.

**********
Parameters
**********
w : array-like

Wavelength array assumed to be in Angstroms.

tar_stretch : bool

Option to window y-axis of the spectrum plot on the
median with 50% above and below. This is useful for echelle
data with noisey edges. The default is True.

reverse : bool

This function works best when the orders are ordered with
ascending wavelength coverage. If this is not the case,
this option will flip them. The default is False, i.e., no
flip in the order.

tell_file : optional keyword, None or str

Name of file containing the location of telluric lines to be
plotted as vertical lines. This is useful when selecting
regions free to telluric contamination.
File must be a tab/space separated ascii text file with the
following format:
w_low w_high depth(compated the conintuum) w_central
This is modeled after the MAKEE telluric template here:
https://www2.keck.hawaii.edu/inst/common/makeewww/Atmosphere/atmabs.txt
but just a heads up, these are in vaccum.
If None, this option is ignored.
The default is None.

jump_to : int

Starting order. Useful when you want to pick up somewhere.
Default is 0.

*******
Returns
*******
None



======================================================================================================================================================================================================
region_select_ms(target,template=None,tar_stretch=True,temp_stretch=True,reverse=False,t_order=0,temp_order=0,header_wave=False,w_mult=1,igrins_default=False,tell_file=None,jump_to=0,reg_file=None):
======================================================================================================================================================================================================
An interactive function to plot target and template spectra
that allowing you to select useful regions with which to
compute the broadening functions, ccfs, ect.

This funciton is meant for multi-order specrta in a fits file.

For this function to work properly the template and target
spectrum have to have the same format, i.e. the same number
of orders and roughly the same wavelength coverage.

If a template spectrum is not specified, it will plot the
target twice, where it can be nice to have one strethed
and on not.

Functionality:
The function brings up an interactive figure with the target
on top and the template on bottom. hitting the 'm' key will
mark wavelengths dotted red lines. The 'b' key will mark the
start of a region with a solid black line and then the end of
the region with a dashed black line. Regions should always go
from small wavelengths to larger wavelengths, and regions
should always close (.i.e., end with a dashed line). Hitting
the return key over the terminal will advance to the next order
and it will print the region(s) you've created to the terminal
screen that are in the format that the saphires.io.read
functions use (you may have to delete commas and parenthesis
depending on whether you are running this command in python 2
or 3). The regions from the previous order will show
up as dotted black lines allowing you to create regions that
do not overlap.

**********
Parameters
**********
target : str

File name for a pickled dictionary that has wavelength and
flux arrays for the target spectrum with the header keywords
defined in the dk_wav and dk_flux arguments.

template : str, None

File name for a pickled dictionary that has wavelength and
flux arrays for the target spectrum with the header keywords
defined in the dk_wav and dk_flux arguments. If None, the
target spectrum will be plotted in both panels.

tar_stretch : bool

Option to window y-axis of the target spectrum plot on the
median with 50% above and below. This is useful for echelle
data with noisey edges. The default is True.

temp_stretch : bool

Option to window y-axis of the template spectrum plot on the
median with 50% above and below. This is useful for echelle
data with noisey edges.The default is True.

reverse : bool

This function works best when the orders are ordered with
ascending wavelength coverage. If this is not the case,
this option will flip them. The default is False, i.e., no
flip in the order.

t_order : int

The order of the target spectrum. Some multi-order spectra
come in multi-extension fits files (e.g. IGRINS). This
parameter defines that extension. The default is 0.

temp_order : int

The order of the template spectrum. Some multi-order spectra
come in multi-extension fits files (e.g. IGRINS). This
parameter defines that extension. The default is 0.

header_wave : bool or 'Single'

Whether to assign the wavelength array from the header keywords or
from a separate fits extension. If True, it uses the header keywords,
assumiing they are linearly spaced. If False, it looks in the second
fits extension, i.e. hdu[1].data
If header_wave is set to 'Single', it treats each fits extension like
single order fits file that could be read in with saph.io.read_fits.
This feature is useful for SALT/HRS specrtra reduced with the MIDAS
pipeline.

w_mult : float

Value to multiply the wavelength array. This is used to convert the
input wavelength array to Angstroms if it is not already. The default
is 1, assuming the wavelength array is alreay in Angstroms.

igrins_default : bool

The option to override all of the input arguments to parameters
that are tailored to IGRINS data. Keyword arguments will be set
to:
t_order = 0
temp_order = 3
temp_stretch = False
header_wave = True
w_mult = 10**4
reverse = True

tell_file : optional keyword, None or str

Name of file containing the location of telluric lines to be
plotted as vertical lines. This is useful when selecting
regions free to telluric contamination.
File must be a tab/space separated ascii text file with the

following format:

w_low w_high depth(compated the conintuum) w_central
This is modeled after the MAKEE telluric template here:
https://www2.keck.hawaii.edu/inst/common/makeewww/Atmosphere/atmabs.txt
but just a heads up, these are in vaccum.
If None, this option is ignored.
The default is None.

jump_to : int

Starting order. Useful when you want to pick up somewhere.
Default is 0.

reg_file : optional keyword, None or str

The name of a region file you want to overplay on the target and
template spectra. The start of a regions will be a solid veritcal
grey line. The end will be a dahsed vertical grey line.
The region file has the same formatting requirements as the io.read
functions. The default is None.

*******
Returns
*******
None



============================================================================
sigma_clip(data,sig=3,iters=1,cenfunc=np.median,varfunc=np.var,maout=False):
============================================================================
Perform sigma-clipping on the provided data.

This performs the sigma clipping algorithm - i.e. the data will be iterated
over, each time rejecting points that are more than a specified number of
standard deviations discrepant.

.. note::
`scipy.stats.sigmaclip` provides a subset of the functionality in this
function.

**********
Parameters
**********
data : array-like

The data to be sigma-clipped (any shape).

sig : float

The number of standard deviations (*not* variances) to use as the
clipping limit.

iters : int or None

The number of iterations to perform clipping for, or None to clip until
convergence is achieved (i.e. continue until the last iteration clips
nothing).

cenfunc : callable

The technique to compute the center for the clipping. Must be a
callable that takes in a 1D data array and outputs the central value.
Defaults to the median.

varfunc : callable

The technique to compute the variance about the center. Must be a
callable that takes in a 1D data array and outputs the width estimator
that will be interpreted as a variance. Defaults to the variance.

maout : bool or 'copy'

If True, a masked array will be returned. If the special string
'inplace', the masked array will contain the same array as `data`,
otherwise the array data will be copied.

*******
Returns
*******
filtereddata : `numpy.ndarray` or `numpy.masked.MaskedArray`

If `maout` is True, this is a masked array with a shape matching the
input that is masked where the algorithm has rejected those values.
Otherwise, a 1D array of values including only those that are not
clipped.

mask : boolean array

Only present if `maout` is False. A boolean array with a shape matching
the input `data` that is False for rejected values and True for all
others.

Examples
--------
This will generate random variates from a Gaussian distribution and return
only the points that are within 2 *sample* standard deviation from the
median::

.. code-block:: python

  >>> from astropy.stats import sigma_clip

  >>> from numpy.random import randn

  >>> randvar = randn(10000)

  >>> data,mask = sigma_clip(randvar, 2, 1)

This will clipping on a similar distribution, but for 3 sigma relative to
the sample *mean*, will clip until converged, and produces a
`numpy.masked.MaskedArray`::

.. code-block:: python

    >>> from astropy.stats import sigma_clip

    >>> from numpy.random import randn

    >>> from numpy import mean

    >>> randvar = randn(10000)

    >>> maskedarr = sigma_clip(randvar, 3, None, mean, maout=True)



=======================================================
spec_trim(w_tar,f,w_range,temp_trim,trim_style='clip'):
=======================================================
A function to select certain regions of a spectrum with which
to compute the broadedning function.

trim_style - refers to how you want to deal with the bad regions

- 'clip' - remove data all together. This creates edges that can cause noise in the BF

- 'lin'  - linearly interpolates over clipped regions

- 'spl'  - interpolated over the clipped regions with a cubic spline - don't use this option.

Paramters
---------
w_tar : array-like

Wavelength array, must be one dimensional.

f : array-like

Flux array, must be one dimensional.

w_range : str

Wavelength trimming string for the target star.
Must have the general form "w1-w2,w3-w4" where '-'
symbols includes the wavelength region, ',' symbols
excludes them. There can be as many regions as you
want, as long as it ends with an inclusive region
(i.e. cannot end with a comma or dash). Wavelength
values must ascend left to right. The '*' symbol
includes everything.

temp_trim : str

Wavelength trimming string for the template star.
Must have the general form "w1-w2,w3-w4" where '-'
symbols includes the wavelength region, ',' symbols
excludes them. There can be as many regions as you
want, as long as it ends with an inclusive region
(i.e. cannot end with a comma or dash). Wavelength
values must ascend left to right. The '*' symbol
includes everything.


trim_style : str, options: 'clip', 'lin', 'spl'
If a wavelength region file is input in the 'spectra_list' parameter,
this parameter describes how gaps are dealt with.

- If 'clip', unused regions will be left as gaps.

- If 'lin', unused regions will be linearly interpolated over.

- If 'spl', unused regions will be interpolated over with a cubic spline. You probably don't want to use this one.

*******
Returns
*******
w_tar : str

The trimmed wavelength array

f : str

The trimmed flux array


==============================
spec_ccf(f_s,f_t,m,v_spacing):
==============================
This is the "under the hood" cross correlation function
called by xc.todcor and xc.ccf.

If you are looking for a ccf that plays nice with the
SAPHIRES dictionaties, use xc.ccf.

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



=================================================================
td_gaussian(xy_ins,amplitude,xo,yo,sigma_x,sigma_y,theta,offset):
=================================================================
A two-dimensional gaussian fuction, that is tailored to fitting data.

**********
Parameters
**********
xy_ins : array-like

A two dimensional array of the input x and y values. I usually input
something that has been through np.meshgrid, e.g.,
x_in,y_in = np.meshgrid(x,y)

amplitude : float

Amplitude of the 2D gaussian.

xo : float

Center of the 2D gaussian along the x axis.

yo : float

Center of the 2D gaussian along the y axis.

sigma_x : float

Width of the 2D gaussian along the x axis.

sigma_y : float

Width of the 2D gaussian along the y axis.

theta : float

Position angle of the 2D gaussian.

offset : float

Veritcal offset (in the z direction) of the 2D gaussian.

*******
Returns
*******
z : array-like

A "raveled" array of the veritcal points.
Here is how you unravel it:
x_in,y_in = np.meshgrid(x,y)
gauss2d = td_gaussian((x_in,y_in),amplitude,xo,yo,sigma_x,sigma_y,
theta,offset)
gauss = fit_gauss2d.reshape(x.size,y.size)
Now you can plot it like:
ax.contour(x,y,gauss)



========================================================
t_gaussian_off(x,A1,x01,sig1,A2,x02,sig2,A3,x03,sig3,o):
========================================================
A double gaussian function with a constant vetical offset.

**********
Parameters
**********
x : array-like

Array of x values over which the Gaussian profile will
be computed.

A1 : float

Amplitude of the first Gaussian profile.

x01 : float

Center of the first Gaussian profile.

sig1 : float

Standard deviation (sigma) of the first Gaussian profile.

A2 : float

Amplitude of the second Gaussian profile.

x02 : float

Center of the second Gaussian profile.

sig2 : float

Standard deviation (sigma) of the second Gaussian profile.

A3 : float

Amplitude of the third Gaussian profile.

x03 : float

Center of the third Gaussian profile.

sig3 : float

Standard deviation (sigma) of the third Gaussian profile.

o : float

Vertical offset of the Gaussian mixture.

*******
Returns
*******
profile : array-like

The Gaussian mixture specified over the input x array.
Array has the same length as x.


===============
vac2air(w_vac):
===============
Vacuum to air conversion formula from Donald Morton
(2000, ApJ. Suppl., 130, 403) is used for the refraction index,
which is also the IAU standard:
http://www.astro.uu.se/valdwiki/Air-to-vacuum%20conversion

**********
Parameters
**********
w_vac : array-like

Array of vacuum wavelengths assumed to be in Angstroms

*******
Returns
*******
w_air : array-like

Array of air wavelengths converted from w_vac
