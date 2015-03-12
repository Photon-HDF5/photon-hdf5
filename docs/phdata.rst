Photon-HDF5 format definition
=============================

Overview
--------

An overview of the data format is shown in the following figure
(**figure outdated**):

.. figure:: /images/alex-photon-hdf5.png
    :align: center

A Photon-HDF5 is a standard HDF5 file with a predefined structure.

Every Photon-HDF5 file has a :ref:`/photon_data <photon_data_group>`
group that contains the photon timestamps and other single photon data.
In the most basic form */photon_data* contains only the per-photon data
(timestamps, detectors, nanotimes, etc...). However, in order to correctly
interpret the data, additional information is needed (for example
which detectors correspond to the donor and acceptor channels in a 2-colors
smFRET experiment, or the
alternation period in a μs-ALEX experiment). If available, these additional
specifications are contained within the */photon_data* field in the
:ref:`measurement_specs sub-group <measurement_specs_group>`.

Other optional groups are:

- :ref:`/identity <identity_group>`:
  Information about the data file

- :ref:`/provenance <provenance_group>`:
  Information about the original data file (when the Photon-HDF5 file
  has been converted from another format)

- :ref:`/setup <setup_group>`:
  Description of the experimental setup

- :ref:`/sample <sample_group>`:
  Description of the measured sample

The following sections describe the Photon-HDF5
groups and fields.

Root-level parameters
---------------------

- **/acquisition_time**: (float) the measurement duration in seconds.
- **/comment**: (string) a user-defined comment.


.. _photon_data_group:

Photon-data group
-----------------

This section describes the layout and fields in the **/photon_data** group.

Mandatory fields:

- **timestamps**: (array) photon timestamps.
- **timestamps_specs/**
    - **timestamps_unit**: (float) timestamp units in *seconds*.

Optional if there is only 1 detector, otherwise mandatory:

- **detectors**: (array of integers) detector ID for each timestamp.

When the dataset contains nanotime information (i.e. arrival time of each
photon with respect to a laser pulse), the following
fields must be present:

- **nanotimes**:(array of integers) TCSPC nanotimes.
 - **nanotimes_specs/**
    - **tcspc_unit**: (float) TAC/TDC bin size (in seconds).
    - **tcspc_range**:(float) full-scale range of the TAC/TDC (in seconds).
    - **tcspc_num_bins**: (integer) number of TAC/TDC bins.
    - **time_reversed**: (boolean) *True* if nanotimes contains the
      time elapsed between a photon and the next laser pulse. *False*
      if it contains the time elapsed between a laser pulse and a photon.

Finally, if the data come from a simulation, ``/photon_data`` may contain:

-  **particles**: (array of integers) a particle ID (integer) for each
   timestamp.


.. _measurement_specs_group:

Measurement specs
^^^^^^^^^^^^^^^^^

The optional **/photon_data/measurement_specs** group contains additional
information allowing unambiguous interpretration of the data for each specific
type of measurement.

- **measurement_type**: (string) the type of the measurements. Valid names
  are:

  - "smFRET" (1 excitation color, 2 detection colors)
  - "smFRET-usALEX" (2 excitation colors, 2 detection colors)
  - "smFRET-usALEX-3c" (3 excitation colors, 3 detection colors)
  - "smFRET-nsALEX" (2 excitation colors, 2 detection colors)

  New names can be created for different kind of measurements and we
  encourage users to submit new name requests.

The *measurement_type* field describes the type of measurement
saved within the file. It is an important field allowing sofware
packages reading and saveing Photon-HDF5 files to perform consistency
checks (see also :ref:`measurement_type`).

For μs-ALEX, 2, 3 or N colors:

- **alex_period**: (integer or float) duration of one complete excitation
  alternation period expressed in timestamp units. The alternation period
  is equal to ``alex_period * timestamps_unit``.

For ns-ALEX (or lifetime with no alternation):

- **laser_pulse_rate**: (float) excitation laser pulse repetition rate in
  *Hertz*.

For 2-color (or more) μs-ALEX and ns-ALEX (optional):

- **alex_period_spectral_ch1**: (array with an even-number of integer
  elements) start and stop values identifying the *spectral_ch1*
  (i.e. *donor* for smFRET measurements) emission period (see note below for
  more details).

- **alex_period_spectral_ch2**: (array with an even-number of interger
  elements) start and stop values identifying the *spectral_ch2*
  (i.e. *acceptor* for smFRET measurements) emission period (see note below for
  more details).

- etc...

.. note::

    For μs-ALEX, both *alex_period_donor* and *alex_period_acceptor*
    are 2-element arrays. In this case, these values are expressed in
    *timestamps_units*.
    For ns-ALEX (also known as PIE), they are arrays with an even-number
    of elements, comprising as many start-stop nanotime pairs as
    there are excitation periods within the TAC/TDC range.
    In this case these values are expressed in *nanotimes_units*.

    For more details see :ref:`alex_period_def`.


.. _detectors_specs_group:

Detectors specs
"""""""""""""""

Within **measurement_specs**, the **detectors_specs/** sub-group
contains all the :ref:`detector ID <detector_ids>`--detection channel
associations, i.e. spectral bands, polarizations or
:ref:`beam-split channels <beam_split_ch>`.

When a measurement records more than 1 spectral band, the fields:

- **spectral_ch1**
- **spectral_ch2**
- etc...

specify which detector is employed in each spectral band. When the measurement
records only 1 spectral band these fields may be omitted. The spectral bands
are strictly ordered for increasing wavelenghts. For example, for 2-color
smFRET measurements ``spectral_ch1`` and ``spectral_ch2`` represent the
*donor* and *acceptor* channel respectively.

If a measurement records more than 1 polarization states, the fields:

- **polarization_ch1**
- **polarization_ch2**

specify which detector is used for each polarization. When the measurement
records only one polarization, these fields may be omitted.

When the detection light is split into 2 channels using a non-polarizing
beam-splitter the fields:


- **split_ch1**
- **split_ch2**

specify which detector is used in each of the "beam-split" channels.

All previous fields are arrays containing one or more detector IDs.
For example, a 2-color smFRET measurement will have only one value in
``spectral_ch1`` (donor) and one value in ``spectral_ch2``
(acceptor). A 2-color smFRET measurement with polarization
(4 detectors) will have 2 values in each of the ``spectral_chX`` and
``polarization_chX`` fields (where X=1 or 2).
For a multispot smFRET measurement, ``spectral_chX`` will contain the list
of donor/acceptor detectors (see :ref:`multi_spot`).

Finally, a label (string) can be associated to each detector using
the optional *labels* field:

- **labels**: (optional) table with 2 columns: detector ID (integer)
  and detector label (string).

For 2-color smFRET measurements, it is recommended to use the "donor"
and "acceptor" labels for the respective detectors. Note, however, that these
labels only represent an additional user-defined metadata and are not
necessary for the interpretation of the measurement.
When detector ID is a *n*-tuple, ``labels`` has *n+1* columns
(*n* for the ID and 1 for the labels).


.. _setup_group:

setup group
-----------

The **/setup** group contains information about the measurement setup:

- **num_pixels**: (integer) total number of detector pixels. For example,
  for a single-spot 2-color smFRET measurement using 2 single-pixel SPADs as
  detectors this field is 2.

- **num_spots**: (integer) the number of excitation (or detection)
  "spots" in the sample. This field is 1 for all the measurements using a
  single confocal excitation volume. When not applicable, for example under
  widefield illumination with 2-D imaging detectors, this field is omitted.

- **num_spectral_ch**: (integer) number of distinct detection spectral
  channels. For example, in a 2-color smFRET experiment there are 2
  detection spectral channels (donor and acceptor), therefore its value is 2.
  When there is a single detection channel or all channels detect
  the same spectral band, its value is 1.

- **num_polarization_ch**: (integer) number of distinct detection polarization
  channels. For example, in polarization anysotropy measurements, its value
  is 2.
  When there is a single detection channel or all channels detect
  the same polarization (including when no polarization selection is performed)
  its value is 1.

- **num_split_ch**: (integer) number of distinct detection channels
  detecting the same spectral band **and** polarization state. For example,
  when a non-polarizing beam-splitter is employed in the detection path,
  its value is 2. When no splitting
  is performed, its value is 1.

- **modulated_excitation**: (boolean) *True* (or 1) if there is any form of
  excitation modulation either in the wavelength space (as in μs-ALEX or PAX)
  or in the polarization space. This field is also *True* for
  pulse-interleaved excitation (PIE) or ns-ALEX measurements.

- **lifetime**: (boolean) *True* (or 1) if the measurements includes a
  *nanotimes* array of (usually sub-ns resolution) photon arrival times with
  respect to a laser pulse (as in TCSPC measurements).

- **excitation_wavelengths**: (array of floats) list of excitation wavelengths
  (center wavelength if broad-band) in increasing order (unit: *meter*).

- **excitation_cw**: (array of booleans) for each excitation source,
  this field indicates whether excitation is continuous wave (CW), *True*,
  or pulsed, *False*.
  The order of excitation sources is the same as that in
  ``excitation_wavelengths`` and is in increasing order of wavelengths.

The following fields are optional and not necessarly relevant for
all experiments. If the associated information is irrelevant or not available,
these fields are omitted.

- **excitation_polarizations**: (arrays of floats) list of polarization
  angles (in degrees) for each excitation source.
  The order of excitation sources is the same as in
  ``excitation_wavelengths`` and is in increasing order of wavelengths.

- **excitation_input_powers**: (array of floats) excitation power in *Watts*
  for each excitation source. This is the excitation power entering
  the optical system.

- **excitation_intensity**: (array of floats) excitation intensity in the
  sample for each excitation source (units: *Watts/meters²*).
  In the case of confocal excitation this is the peak PSF intensity.

- **detection_wavelengths**: (arrays of floats) reference wavelengths (in
  *meters*) for each detection spectral band.
  This array is ordered in increasing order of wavelengths.
  The first element refers to ``detectors_specs/spectral_ch1``, the second to
  ``detectors_specs/spectral_ch2`` and so on.

- **detection_polarizations**: (arrays of floats) polarization angles
  for each detection polarization band.
  The first element refers to ``detectors_specs/polarization_ch1``, the second
  to ``detectors_specs/polarization_ch2`` and so on.
  This field is not relevant if no polarization selection is performed.

- **detection_split_ch_ratios**: (array of floats) power fraction detected
  by each "beam-split" channel (i.e. independent detection channels
  obtained through a non-polarizing beam splitter). For 2 beam-split
  channels that receive the same power this array should be ``[0.5, 0.5]``.
  The first element refers to ``detectors_specs/split_ch1``, the second to
  ``detectors_specs/split_ch2`` and so on.
  This field is not relevant when no polarization- and spectral-insensitive
  splitting is performed.


.. _identity_group:

identity group
--------------

The **identity/** group contains information about the specific Photon-HDF5
file. If some information is not availble the relative field may be omitted.

- **author**: (string) the author of the measurement (or simulation).
- **author_affiliation**: (string) the company or institution the *author*
  is affiliated with.

- **creator**: (string) the Photon-HDF5 file creator. Used when the data
  was previously stored in another format and the conversion is perfomed by
  a different person than the author.
- **creator_affiliation**: (string) the company or institution the *creator*
  is affiliated with.

- **url**: (string) URL that allow to download the Photon-HDF5 data file.
- **doi**: (string) Digital Object Identifier (DOI) for the Photon-HDF5
  data file.

- **filename**: (string) Photon-HDF5 file name at creation time. This field
  saves the original file name even is the file is later on renamed on disk.
- **full_filename**: (string) Photon-HDF5 file name (including the full path)
  at creation time.
- **creation_time**: (string) the Photon-HDF5 file creation time with
  the following format: "YYYY-MM-DD HH:MM:SS".
- **software**: (string) name of the software used to create
  the Photon-HDF5 file.
- **software_version**: (string) version of the software used to create
  the Photon-HDF5 file.

- **format_name**: (string) this must always be "Photon-HDF5"
- **format_version**: (string) for the current version it  must be "0.3"
- **format_url**: (string) A URL pointing to the Photon-HDF5 specification
  document.


.. _provenance_group:

provenance group
----------------

The **provenance/** group contains info about the original file that has
been converted into a Photon-HDF5 file.
If some information is not availble the relative field may be omitted.

- **filename**: (string)
- **full_filename**: (string)
- **creation_time**: (string)
- **modification_time**: (string)
- **software**: (string)
- **software_version**: (string)


.. _sample_group:

sample group
------------

The **/sample** group contains information related to the measured sample.
This group is optional.

- **num_dyes**: (integer) number of different dyes present in the samples.
- **dye_names**: (array of string) list of dye names (for example:
  ``['ATTO550', 'ATTO647N']``)
- **buffer_name**: (string) a user defined description for the buffer.
- **sample_name**: (string) a user defined description for the sample.


.. _glossary:

Additional notes and definitions
--------------------------------

.. _detector_ids:

Detector IDs
^^^^^^^^^^^^

A detector ID is the "name" of each pixels and is typically a single
integer (when all the pixels are numbered with a progressive index).
In some case (when using detector arrays) the detector ID
can be a *n*-tuple of integers. This allow to specify, for each pixel,
the module number and the X, Y location, for example. Therefore, an
array of detector IDs can be either a 1-D column array or a 2-D array.
In either cases, each row identifies a detector.


.. _beam_split_ch:

Beam-split channels
^^^^^^^^^^^^^^^^^^^

When the emitted light path is split in 2 or more detection paths by using
a non-polarizing beam splitter the measurement has so called
beam-split channels. The fields `split_ch1` and `split_ch2` contains
the list of detector IDs for each beam-split channel
(see :ref:`detectors_specs_group`).


.. _alex_period_def:

Definition of alternation periods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Note for μs-ALEX
""""""""""""""""

The *alex_period_donor* and *alex_period_acceptor* fields allow
defining photons detected during donor or acceptor excitation. As an
example, let's define the array

``A`` = ``timestamps`` *MODULO* ``alex_period``

as the array of timestamps modulo the μs-ALEX alternation period.
Photons emitted during the donor period (respectively, acceptor
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
    while the acceptor period is defined as an "internal range" (900, 2580).
    This situation is due to the ALEX period being out of phase with respect
    to the time stamping clock.


.. _measurement_type:

Measurement type
^^^^^^^^^^^^^^^^

Each *measurement_type* has an associated set of mandatory fields
which must be present to ensure that all information needed to
unambiguously interpret the data is present.
For example, for a 2-color smFRET measurement, a software package creating
a file should check that
the association between detector and donor or acceptor channel
is present. If some necessary field is absent, the software package
should warn the user in order that this information is added before
saving the file.


.. _multi_spot:

Multi-spot measurements
^^^^^^^^^^^^^^^^^^^^^^^

TODO
