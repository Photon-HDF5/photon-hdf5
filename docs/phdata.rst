Photon-HDF5 format definition
=============================

Overview
--------

An overview of the data format is shown in the following figure
(**figure outdated**):

.. figure:: /images/alex-photon-hdf5.png
    :align: center

A Photon-HDF5 is a standard HDF5 file with a predefined structure.

Every Photon-HDF5 file has a **/photon_data** group that
contains the photon timestamps and other per-photon data.
In the most basic form **/photon_data** contains only the per-photon data
(timestamps, detectors, nanotimes, etc...). However, in order to correctly
interpret the data, additional information is needed (for example
which detector is donor/acceptor in a 2-colors smFRET experiment, or the
alternation period in a us-ALEX experiment). If available, these additional
specifications are contained inside **/photon_data** in the
**measurement_specs** sub-group.

Other optional groups are:

- **/indentity**: information about the data file
- **/provenance**: information about the original data file
- **/setup**: fundamental parameters of the experimental setup
- **/sample**: sample information.

The following sections describe in details all the Photon-HDF5
groups and fields.

Root-level parameters
---------------------

- **/acquisition_time**: (float) the measurement duration in seconds.
- **/comment**: (string) a user defined comment.


Photon-data group
-----------------

This section describes the layout and fields in the **/photon_data** group.

Mandatory fields:

- **timestamps**: (array) the photon timestamps.
- **timestamps_specs/**
    - **timestamps_unit**: (float) timestamps units in *seconds*.

Optional if there is only 1 detector, otherwise mandatory:

- **detectors**: (array of integers) the detector ID for each timestamp.

When the dataset contains nanotime information, the following
fields must be present:

- **nanotimes**:(array of integers) the TCSPC nanotimes.
 - **nanotimes_specs/**
    - **tcspc_unit**: (float) TAC/TDC bin size (in seconds).
    - **tcspc_range**:(float) Full-scale range of the TAC/TDC (in seconds).
    - **tcspc_num_bins**: (integer) TAC/TDC number of bins.

Optionally ``nanotimes_specs`` can also contain:

-  **irf_donor_hist**: (array of integers) Instrument Response
   Function (IRF) histogram for the donor detection channel.
-  **irf_acceptor_hist**: (array of integers) Instrument Response
   Function (IRF) histogram for the acceptor detection channel.
-  **calibration_hist**: (array of integers) Histograms of
   uncorrelated counts used to correct the TCSPC non-linearities.

Finally, if the data come from a simulation, ``/photon_data`` may contain:

-  **particles**: (array of integers) a particle ID (integer) for each
   timestamp.

Measurement specs
^^^^^^^^^^^^^^^^^

The optional **/photon_data/measurement_specs** group contains additional
information that allows to unambiguously interpret the data for each specific
type of measurement. This group is optional, but if present it must be complete.

- **measurement_type**: (string) the type of the measurements. Valid names
  are:

  - "smFRET" (1 excitation color, 2 detection colors)
  - "smFRET-usALEX" (2 excitation colors, 2 detection colors)
  - "smFRET-usALEX-3c" (3 excitation colors, 3 detection colors)
  - "smFRET-nsALEX" (2 excitation colors, 2 detection colors)

  New names can be created for different kind of measurements and we
  encourage users to submit new names requests.

The field *measurement_type* represents the name of the specific measurement
that is saved into the file. It is an important field that allows the sofware
library that reads and saves Photon-HDF5 files to perform strict consistency
checks.
In fact, each *measurement_type*, has an associated set of mandatory fields
that must be present to assure the all the information needed to
unambiguously interpret the data is present.
For example, for a 2-color smFRET measurement the software can check if
the specification of which detector represents the donor or acceptor channel
is present. If not present, it can throw and error or warn the user so that
this important information can be added before saving the file.
This prevents the accidental creation of incomplete or inconsistent files.

For us-ALEX, 2, 3 or N colors:

- **alex_period**: (integer or float) the duration of one complete excitation
  alternation period expressed in timestamp units, such that
  ``alex_period * timestamps_unit`` is the alternation period in seconds.

For ns-ALEX (or lifetime with no alternation):

- **laser_pulse_rate**: (float) excitation laser pulse repetition rate in
  *Hertz*.

For 2-color (or more) us-ALEX and ns-ALEX (optional):

- **alex_period_spectral_ch1**: (array with an even-number of interger
  elements) the start and stop values identifying the *spectral_ch1*
  (i.e. *donor* for smFRET measurements) emission period.

- **alex_period_spectral_ch2**: (array with an even-number of interger
  elements) the start and stop values identifying the *spectral_ch2*
  (i.e. *acceptor* for smFRET measurements) emission period.

- etc...

.. note::

    For μs-ALEX, *alex_period_donor* and *alex_period_acceptor*
    are both 2-element arrays. In this case these values are expressed in
    *timestamps_units*.
    For ns-ALEX (or PIE), they are arrays with an even-number of elements,
    comprising as many start-stop pairs as the number of excitation periods
    in the TAC/TDC range. In this case these values are expressed in
    *nanotimes_units*.

Note for μs-ALEX
""""""""""""""""

The fields *alex_period_donor* and *alex_period_acceptor* allow
defining photons detected during donor or acceptor excitation. As an
example, let's define the array

``A`` = ``timestamps`` *MODULO* ``alex_period``

as the array of timestamps modulo the μs-ALEX alternation period.
Photons emitted during the donor period (respectively acceptor
period) are obtained by applying one of these two conditions:

-  ``(A > start) and (A < stop)`` when ``start < stop`` (*internal
   range*)

-  ``(A > start) or  (A < stop)`` when ``start > stop`` (*external
   range*).

.. figure:: /images/alternation_range.png
    :alt: Illustration of the internal and external ranges
    :align: center

    Alternation histogram showing selection for the donor and acceptor periods.
    In this case the donor period is defined as an "external range" (2850, 580)
    while the acceptor period as an "internal range" (900, 2580).


Detectors specs
"""""""""""""""

Inside **measurement_specs**, the sub-group **detectors_specs/**
contains the mapping between the each pixel ID and the detection channels
(i.e. spectral bands, polarizations or beam-split channels).

Note that a detector ID can be a single integer of a n-tuple of integers,
to support the case of 2-D detector arrays. Therefore an array of detector
IDs can be either a 1-D or a 2-D array, in the latter case it is one row
per detector.

When a measurement records more than 1 spectral band, the fields:

- **spectral_ch1**
- **spectral_ch2**
- etc...

specify which detector is employed in each spectral band. When the measurement
records only 1 spectral band these fields may be omitted. The spectral bands
are strictly ordered for increasing wavelenghts. For example, for 2-color
smFRET measurements ``spectral_ch1`` and ``spectral_ch2`` represent the donor
and acceptor channel respectively.

When a measurement records more than 1 polarization state, the fields:

- **polarization_ch1**
- **polarization_ch2**

specify which detector is employed for each polarization. When the measurement
records only one polarization these fields may be omitted.

When the detection path is split in 2 channels through a non-polarizing
beam splitter the fields:


- **split_ch1**
- **split_ch2**

specify which detector is employed in each of the "beam-split" channels.

All the previous fields are arrays containing one or more detector IDs.
For example, a 2-color smFRET measurement will have only one value in
``spectral_ch1`` (i.e. donor) and one value in ``spectral_ch2``
(i.e. acceptor). A 2-color smFRET measurement with polarization
(4 detectors) will have 2 values in each of the ``spectral_chX`` and
``polarization_chX`` fields.
For a multispot smFRET measurement, ``spectral_chX`` will contain the list
of donor/acceptor detectors (see section 2.3).

Finally, a label (i.e. a string) can be associated to each detector through
the optional field *labels*:

- **labels**: (optional) a table with 2 columns: detector ID and detector
  label (a string).

For 2-color smFRET measurements it is recommended to use the labels "donor"
and "acceptor" for the respective detectors. Note, however, that these
labels only represent an additional user-defined metadata and are not
necessary for the interpretation of the measurement.
When detector ID is a *n*-tuple, ``labels`` has *n+1* columns
(*n* for the ID and 1 for the labels).


setup group
-----------

The **/setup** group contains information about the measurement setup:

- **num_pixels**: (integer) total number of detector's pixels.

- **num_spots**: (integer or "none") the number of excitation (or detection)
  "spots" in the sample. This field is 1 for all the measurements using a
  single confocal excitation volume. When not applicable, for example under
  widefield illumination with 2-D imaging detectors, this value must be
  the string "none".

- **num_spectral_ch**: (integer) number of distinct detection spectral
  channels. For example, in a 2-color smFRET experiment there are 2
  detection spectral channels (donor and acceptor) so this value is 2.
  When there is only a single detection channel or all the channels receive
  the same spectral band this value is 1.

- **num_polarization_ch**: (integer) number of distinct detection polarization
  channels. For example, in polarization anysotropy measurements this value
  is 2.
  When there is only a single detection channel or all the channels receive
  the same polarization (even when no polarization selection is performed)
  this value is 1.

- **num_split_ch**: (integer) number of distinct detection channels that
  receive the same spectral and polarization band. For example, when a
  non-polarizing beam-splitter is employed in the detection path, this value
  is 2. When no polarization- and spectral-insensitive splitting is performed
  this value is 1.

- **modulated_excitation**: (boolean) *True* (i.e. 1) if there is any form of
  excitation modulation either in wavelength (like in us-ALEX or PAX) or in
  polarization. This field is also *True* for pulse-interleaved excitation
  (PIE) or ns-ALEX measurements.

- **lifetime**: (boolean) *True* (i.e. 1) if the measurements includes a
  *nanotimes* array of (usually sub-ns resolution) photon arrival times
  respect to a laser pulse (like in TCSPC measurements).

- **excitation_cw**: (array of booleans) for each excitation source,
  this field indicates whether it is continuous wave (CW), *True*, or pulsed,
  *False*.
  The order of excitation sources is the same as in
  ``excitation_wavelengths`` and it is in increasing order of wavelengths.

- **excitation_wavelengths**: (array of floats) list of excitation wavelengths
  (center wavelength if braod-band) in increasing order. Units are in *meters*.

The following fields are optional and not all all them are relevant for a
particular experimental configuration. If not-relevant these field should be
omitted.

- **excitation_polarizations**: (arrays of floats) list of polarization
  angles (in degrees) for each excitation source.
  The order of excitation sources is the same as in
  ``excitation_wavelengths`` and it is in increasing order of wavelengths.

- **detection_wavelengths**: (arrays of floats) reference wavelengths (in
  *meters*) for each detection spectral band.
  This array is ordered in increasing order of wavelengths.
  The first element refers to ``detectors_specs/spectral_ch1``, the second to
  ``detectors_specs/spectral_ch2`` and so on.

- **detection_polarizations**: (arrays of floats) polarization angles
  for each detection polarization band.
  The first element refers to ``detectors_specs/polarization_ch1``, the second
  to ``detectors_specs/polarization_ch2`` and so on.
  This field is not-relevant if no polarization selection is performed.

- **excitation_powers**: (array of floats) excitation power in *Watts*
  for each excitation source.

- **detection_split_ch_ratios**: (array of floats) power fractions detected
  by each "beam-split" channel (i.e. independent detection channels
  obtained through a non-polarizing beam splitter). For 2 beam-split
  channels that receive the same power this array should be *[0.5, 0.5]*.
  The first element refers to ``detectors_specs/split_ch1``, the second to
  ``detectors_specs/split_ch2`` and so on.
  This field is not-relevant when no polarization- and spectral-insensitive
  splitting is performed.


identity group
--------------

The **identity/** group contains info about the specific Photon-HDF5 file:

- **filename**: (string)
- **full_filename**: (string)
- **creation_time**: (string) Creation time as "YYYY-MM-DD HH:MM:SS".
- **software**: (string)
- **software_version**: (string)
- **format_name**: (string) This must always be "Photon-HDF5"
- **format_version**: (string) "0.3"
- **format_url**: (string) A URL pointing to the Photon-HDF5 documentation.

provenance group
----------------

The **provenance/** group contains info about the original file that has
been converted to Photon-HDF5 file. This group is optionla but reccomended.

- **author**: (string)
- **affiliation**: (string)
- **filename**: (string)
- **full_filename**: (string)
- **creation_time**: (string)
- **modification_time**: (string)
- **software**: (string)
- **software_version**: (string)

sample group
------------

The **/sample** group contains information related to the measured sample.
This group is optional.

- **num_dyes**: (integer) number of different dyes present in the samples.
- **dye_names**: (array of string) list of dye names (for example: ['ATTO550', 'ATTO647N'])
- **buffer_name**: (string) a user defined description for the buffer.
- **sample_name**: (string) a user defined description for the sample.
