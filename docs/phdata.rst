2. Photon-HDF5 format definition
================================

An overview of the data format is show in the following figure

.. todo::

    Show a TOC view of a typical file, identifying root data,
    photon group data and subgroups. Mention Metadata.

2.1 Root-level parameters
-------------------------

There are mandatory and optional parameters (scalars or arrays of
scalars) in the root group, as discussed in the next two sections. The
root group also contains one or more ``photon_data`` groups which
contain their own data, as discussed in sections 2.2 & 2.3. Finally,
there are optional groups providing information on the sample and setup,
as well as user-specific information, discussed in the last sections.

2.1.1 Mandatory parameters:
^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  **timestamps_unit**: (float) time in seconds of 1-unit increment
   in timestamps. Normally, timestamps are integers and the unit
   increment is determined by the acquisition electronics. However,
   timestamps can also be floats and express the time in seconds. In
   this case ``timestamps_unit`` will set to 1.

-  **num_spots**: (integer) Normally it is 1 for single-spot
   measurements. In multi-spot measurements contains the number of
   excitation or detection spots.

-  **alex**: (boolean) if True (i.e. = 1), the measurements uses
   alternated excitation.

-  **lifetime**: (boolean) if True (i.e. = 1), the data contains
   nanotime information for each photon (provided by some kind of
   TAC/TDC i.e. TCSPC hardware) in addition to the standard macrotime
   information (typically provided by some kind of digital clock with a
   10-100 ns period).

-  **num_spectral_ch**: (interger) number of different spectral
   bands in the detection channels (i.e. 2 for 2-colors smFRET).

-  **num_polariz_ch**: (interger) number of different polarization
   in the detection channels. The value is 1 if no polarization
   selection is performed and 2 if two orthogonal polarizations are
   recorded.

.. note::

    **OPEN QUESTION**: How to handle the case of 2 laser excitation and
    only 1 laser alternation?

    **ANSWER:** By defining a range for the acceptor excitation period
    (``alex_period_acceptor``) that selects all the photons (i.e.
    ``(0, alex_period)``).

2.1.2 Optional parameters
^^^^^^^^^^^^^^^^^^^^^^^^^

Currently, optional parameters include information necessary to
interpret data acquired with alternating laser excitation (ALEX),
whether μs-ALEX or ns-ALEX (aka PIE, or pulsed-interleaved excitation).
Some of those parameters are mandatory, some other are optional.

**Mandatory** for ALEX data (``ALEX == True``):

-  **alex_period** (integer or float): the duration of one
   excitation alternation period. For μs-ALEX data, it is expressed in
   timestamp units, such that ``alex_period * timestamps_unit`` is the
   alternation period in seconds. For ns-ALEX data
   (``lifetime == True``), ``alex_period`` is expressed in TCSPC bin
   units, such that the alternation period in seconds is
   ``alex_period * tcspc_bin``.

    **OPEN QUESTION**: Why integer OR float and not just float?

**Optional** for ALEX data:

-  **alex_period_donor**: (array with an even-number of elements,
   integers): The start and stop values identifying the donor emission
   period.

-  **alex_period_acceptor**: (array with an even-number of elements,
   integers): The start and stop values identifying the acceptor
   emission period.

.. note::

    For μs-ALEX, ``alex_period_donor`` and
    ``alex_period_acceptor`` are both 2-element arrays. For ns-ALEX,
    they are arrays array with an even-number of elements, comprising as
    many start-stop pairs as the number of excitation periods in the
    TAC/TDC range. In both cases, they have the same unit as
    ``alex_period``.

Note for μs-ALEX
""""""""""""""""

The fields ``alex_period_donor`` and ``alex_period_acceptor`` allow
defining photons detected during donor or acceptor excitation. As an
example, let's define the array

``A`` = ``timestamps`` MODULO ``alex_period``

as the array of timestamps modulo the ALEX alternation period.
Photons emitted during the donor period (respectively acceptor
period) are obtained by applying one of these two conditions:

-  ``(A > start) and (A < stop)`` when ``start < stop`` (*internal
   range*)

-  ``(A > start) or  (A < stop)`` when ``start > stop`` (*external
   range*).

.. todo::

    *this requires a schematic to explain what is meant by
    internal and external range.*

2.2 Photon Data Group(s)
------------------------

A file can contain one or more photon data groups. For instance, a
typical single-spot experiment will contain a single photon-data group,
while a multispot experiment can either have a single photon-data group
(*basic layout*) or as many photon-data groups as there are spots
(*multi-spot layout*). The multispot case is detailed in section
:ref:`sec_multispot`.

The typical content of a photon data group is briefly described in the
next section. The following sections then describe each component in
more details.

2.2.1 Basic layout of photon data groups
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An overview of the photon data group format is show in the following
figure

.. todo::

    *Show a TOC view of a photon data group, identifying arrays
    and specs subgroups*

The photon-data group is named ``/photon_data`` in single-spot data
files.

In the multi-spot case there can be a single photon-data group
(*basic layout*) or *n* groups named ``/photon_data_n``, where *n*
is an integer designing spot number (*multi-spot layout*). The
multispot case is detailed in :ref:`sec_multispot`.

To each photon is associated a fixed number of pieces of information,
this number depending on the experiment. The supported types of
information are described below. For example, timestamp ("timestamps")
and detector ID number ("detectors") would be the minimum number of
pieces of information for each photon. Each type of information is
stored in an array with size equal to the number of photons in the
group.

In addition, parameters (specifications) common to all photons in the
group (scalar or arrays of scalars) are stored within separate
subgroups. Each subgroup's name end with the suffix "\_specs" (for
instance ``detector_specs``).

Finally, flexibility for customization is provided by custom "user"
subgroups, which can reside at all levels of the hierarchy (for instance
``/photon_data/user/``). Those can be a location to save additional
photon or specification information not anticipated by the format.

2.2.2 Mandatory photon data arrays:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  **timestamps**: (array of integers) contains all timestamps.

-  **detectors**: (array of integers) contains the detector ID
   number corresponding to each photon. This array is optional if there
   is a single detector. Each physical detector (for example donor and
   acceptor channels) needs to have a unique label (a positive integer
   including zero). For example, measurements of smFRET with
   polarization anisotropy using a single donor-acceptor pair require 4
   detectors, and therefore need 4 different labels (e.g. 0 - 3). The
   interpretation of what label corresponds to what detector is done
   using information provided in the detectors\_specs subgroup (see
   below).

2.2.3 Optional photon data arrays
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  **nanotimes** (array of integers) contains the TCSPC nanotimes.
   This array is only required if **``lifetime``** is True.

-  **particles**: particle label (or ID number) for each timestamp.
   This optional array is used when the data comes from a simulation
   providing particle ID information.

2.2.4 Photon data specifications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Arrays in the ``photon_data`` group can have additional associated
information that **is not** "photon specific" and therefore does not
justify the use of an array with one value per photon. This data is
instead stored in a subgroup with a ``_specs`` suffix.

2.2.4.1 Detector specifications subgroup
""""""""""""""""""""""""""""""""""""""""

To provide information about whether a photon has been detected in the
donor or acceptor channel, and/or in the parallel or perpendicular
polarization channel, the following arrays are defined inside the
``detectors_specs`` group:

-  **donor**: (array of integers) list of detectors for the donor
   channel. A standard smFRET measurement will have only one value. A
   smFRET with polarization (4 detectors) will have 2 values. For a
   multispot measurement, it will contain the list of donor channel
   detectors (see section 2.3).

-  **acceptor**: (array of integers) list of detectors for the
   acceptor channel. A standard smFRET measurement will have only one
   value. A smFRET with polarization (4 detectors) will have 2 values.
   For a multi-spot measurement it will contain the list of
   acceptor-channel detectors (see section 2.3).

-  **polarization1** (array of integers) list of detectors for the
   "first" polarization. If not specified in the experimental setup
   section, this polarization is assumed parallel to the excitation
   polarization.

-  **polarization2** (array of integers) list of detectors for the
   "second" polarization. If not specified in the experimental setup
   section, this polarization is assumed perpendicular to the excitation
   polarization.

.. note::

    If a single spectral channel is acquired
    (``num_spectral_ch == 1``), the ``donor`` and ``acceptor`` arrays
    can be omitted. If not omitted, the detector(s) ID should go either
    in ``donor`` or ``acceptor``, but not in both.

.. note::

    If a single polarization is acquired
    (``num_polariz_ch == 1``) the polarization fields can be omitted. If
    not omitted, the detector(s) ID number should go either in
    ``polarization1`` or ``polarization2``, but not in both.

2.2.4.2 User defined detector specifications subgroup (optional)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Additional detector specifications can be saved in a dedicated subgroup:
``detectors_specs/user/``.

2.2.4.3 Nanotime specifications subgroup
""""""""""""""""""""""""""""""""""""""""

If a ``nanotimes`` array is present, the following specifications need
to be provided:

-  **tcspc_bin**: (float) TAC/TDC bin size (in seconds).
-  **tcspc_nbins**: (integer) TAC/TDC number of bins.
-  **tcspc_range**: (float) Full-scale range of the TAC/TDC hardware
   in seconds.

.. note::

    The field ``tcspc_range`` is equal to ``tcspc_bin * tcspc_nbins``.

Optionally the following specifications can be provided:

-  **irf_hist_donor**: (array of integers) Instrument Response
   Function (IRF) histogram for the donor detection channel.

-  **irf_hist_acceptor**: (array of integers) Instrument Response
   Function (IRF) histogram for the acceptor detection channel.

-  **calibration_hist**: (array of integers) Histograms of
   uncorrelated counts used to correct the TCSPC non-linearities.

If data comes from simulations, the nanotime specification subgroup can
optionally contain these additional specifications:

-  **tau_accept_only**: (float) Intrinsic Acceptor lifetime
   (seconds).

-  **tau_donor_only**: (float) Intrinsic Donor lifetime (seconds).

-  **tau_fret_donor**: (float) Donor lifetime in presence of
   Acceptor (seconds).

-  **inverse_fret_rate**: (float) FRET energy transfer lifetime
   (seconds). Inverse of the rate of ``D*A`` -> ``DA*``.

Additional specs can be saved in ``nanotimes_specs/user/``.

.. _sec_multispot:

2.3 Multispot layout for photon data
------------------------------------

Multi-spot measurements can be saved using the basic layout described in
previous sections. In this case, the ``timestamps`` array contains all
timestamps from all channels and the ``detectors`` array allows
identifying detectors. In the case of smFRET measurements the
``detectors_specs`` ``donor`` and ``acceptor`` contains an ordered list
of detector numbers, whose length is the number of spots.

This structure is convenient to use when **creating** a data file, as it
uses only two arrays (one for timestamps, one for detectors) and does
not necessitate dispatching each photon in a specific spot photon\_data
subgroup. However, it is not a very efficient data structure for
repeatedly reading multispot data, because, in order to extract
photon-data for a single channel, all ``timestamps`` and ``detectors``
must be first be read and then sorted out. A more efficient way of
storing multispot data, once it has been sorted out, is provided by a
layout variant called "multispot layout".

The "multispot layout" is identical to the basic layout for single-spot
data. The only difference is that, instead of having a single group
``/photon_data``, there are now *N+1* photon data groups
``/photon_data_0`` .. ``/photon_data_N``, one for each spot. Each group
has a suffix indicating the spot number (starting from 0).

2.4 Optional Sample Group
-------------------------

The Photon-HDF5 defines an optional "sample" group where information
about the measured sample can be stored. This data is stored in the
group ``/sample_specs``.

Within ``/sample_specs`` the following fields are defined:

-  **num_dyes**: (integer) number of different dyes present in the
   samples. For a standard single-pair FRET measurement the value is 2.
   For donor-only or acceptor-only measurements the value is 1.

-  **dye_names** (array of string) list of dye names (for example:
   ``['ATTO550', 'ATTO647N']``).

-  **buffer_name** (string) free-form description of the sample
   buffer. For example ``'TE50 + 1mM TROLOX'``.

-  **sample_name** (string) free-form description of the sample. For
   example ``'40-bp dsDNA, D-A distance: 7-bp'``.

2.5 Optional Measurement Setup Group
------------------------------------

The optional group **``/setup_specs``** contains fields describing the
measurement setup:

-  **excitation_wavelengths:** (array of floats): array of
   excitation wavelengths in S.I. units (meters).

-  **excitation_powers** (array of float): array of excitation
   powers (in the same order as ``excitation_wavelengths``). The powers
   are expressed in S.I. units (Watts).

-  **excitation_polarizations** (array of float): polarization angle
   (in degrees), one for each laser.

-  **detection_polarization1** (float): polarization angle (in
   degrees) for what is called ``polarization1``. If this field is not
   specified it is assumed that ``polarization1`` is parallel to the
   excitation polarization of the first laser.
-  **detection_polarization2** (float): polarization angle (in
   degrees) for what is called ``polarization2``. If this field is not
   specified it is assumed that ``polarization2`` is perpendicular to
   the excitation polarization of the first laser.

.. note::

   At the moment, there is no standard way to distinguish
   between linear and elliptically/circularly polarized excitation.

2.6 Optional User Data Group
----------------------------

An unlimited number of user-defined fields are allowed. To make sure
that future versions of this format will not collide with any
user-defined field names, custom data should be contained in a group
named ``user``. A ``user`` group can be placed anywhere in the HDF5
hierachy and should be place wherever it is most logical for the kind of
data stored. As an example, user-data can be stored in ``'/user'``,
``'/photon_data/user'``, ``'/photon_data/nanotimes_specs/user'``,
``'/setup_specs/user'``, etc...

2.7 Metadata
------------

The root node needs to include the following attributes:

-  ``format_name = 'Photon-HDF5'``
-  ``format_title = 'HDF5-based format for time-series of photon data.'``
-  ``format_version = '0.2'``
-  ``format_url = 'http://photon-hdf5.readthedocs.org/'``

Each group or array needs to have a description attribute named
``TITLE`` (following
`the same convention as pytables <http://pytables.github.io/usersguide/file_format.html>`__).

The description attributes for each field are listed in the following table:

=========================   ==================================================================
Field names                 Descriptions used in the TITLE attribute
=========================   ==================================================================
num_spots                   | Number of excitation or detection spots.
num_spectral_ch             | Number of different spectral bands in the detection
                            | channels (i.e. 2 for 2-colors smFRET).
num_polariz_ch              | Number of different polarization in the detection
                            | channels. The value is 1 if no polarization selection is
                            | performed and 2 if two orthogonal polarizations are
                            | recorded.
lifetime                    | If True (or 1) the data contains nanotimes from TCSPC
                            | hardware
alex                        | If True (or 1) the file contains ALternated EXcitation
                            | data.
alex_period                 | The duration of the excitation alternation using the same
                            | units as the timestamps.
alex_period_donor           | Start and stop values identifying the donor emission
                            | period.
alex_period_acceptor        | Start and stop values identifying the acceptor emission
                            | period.
timestamps_unit             | Time in seconds of 1-unit increment in timestamps.
photon_data                 | Group containing arrays of photon-data (one element per
                            | photon)
timestamps                  | Array of photon timestamps
detectors                   | Array of detector numbers for each timestamp
detectors_specs             | Group for detector-specific data.
donor                       | Detectors for the donor spectral range
acceptor                    | Detectors for the acceptor spectral range
polarization1               | Detectors ID for the "polarization1". By default is the
                            | polarization parallel to the excitation, unless specified
                            | differently in the "/setup_specs".
polarization2               | Detectors ID for the "polarization2". By default is the
                            | polarization perpendicular to the excitation, unless
                            | specified differently in the "/setup_specs".
nanotimes                   | TCSPC photon arrival time (nanotimes)
nanotimes_specs             | Group for nanotime-specific data.
tcspc_bin                   | TCSPC time bin duration in seconds (nanotimes unit).
tcspc_nbins                 | Number of TCSPC bins.
tcspc_range                 | TCSPC full-scale range in seconds.
tau_accept_only             | Intrinsic Acceptor lifetime (seconds).
tau_donor_only              | Intrinsic Donor lifetime (seconds).
tau_fret_donor              | Donor lifetime in presence of Acceptor (seconds).
inverse_fret_rate           | FRET energy transfer lifetime (seconds). Inverse of the
                            | rate of D*A -> DA*.
particles                   | Particle label (integer) for each timestamp.
excitation_wavelengths      | Array of excitation wavelengths (meters).
excitation_powers           | Array of excitation powers (in the same order as
                            | excitation_wavelengths). Units: Watts.
excitation_polarizations    | Polarization angle (in degrees), one for each laser.
detection_polarization1     | Polarization angle (in degrees) for "polarization1".
detection_polarization2     | Polarization angle (in degrees) for "polarization2".
=========================   ==================================================================

Additional attributes are allowed in any node but they should not
overlap with standard `pytables
attributes <http://pytables.github.io/usersguide/file_format.html>`__.

