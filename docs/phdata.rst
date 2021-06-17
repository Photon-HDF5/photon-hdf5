.. _specifications:

Photon-HDF5 format definition
=============================

.. contents::

Overview
--------

A Photon-HDF5 is a HDF5 file with a predefined structure for timestamp-based
data.

A screen-shot of a typical Photon-HDF5 file opened in HDFView is shown here:

.. figure:: /images/hdfview.png
    :align: center
    :width: 400px
    :alt: HDView screenshot showing a Photon-HDF5 file

The previous figure shows the 5 main groups contained in a Photon-HDF5 file.
Of these, :ref:`/photon_data <photon_data_group>` and  :ref:`/setup <setup_group>`
contains the raw data and all the information needed for the analysis.
A schematic overview is shown in the next figure:

.. figure:: /images/format-diagram_first2.png
    :align: center
    :alt: Photon-HDF5 structure: photon_data, setup

The remaining 3 groups provide additional metadata not necessary for data
analysis:

.. figure:: /images/format-diagram_last3.png
    :align: center
    :alt: Photon-HDF5 structure: sample, identity, provenance

A brief description of these 3 metadata groups follows:

- :ref:`/identity <identity_group>`:
  Information about the data file.

- :ref:`/provenance <provenance_group>`:
  Information about the original data file (when the Photon-HDF5 file
  has been converted from another format).

- :ref:`/sample <sample_group>`:
  Description of the measured sample.


.. _root_params:

Root-level parameters
---------------------

The root node ("/") in a Photon-HDF5 file contains the following fields:

- **/acquisition_duration**: (float) the measurement duration in seconds.
- **/description**: (string) a user-defined measurement description.

In addition, the root node has the following attributes which
distinguish a Photon-HDF5 file from other HDF5 files:

- **format_name**: must contain the string "Photon-HDF5"
- **format_version**: (string) the Photon-HDF5 format version.


.. _photon_data_group:

Photon-data group
-----------------

This section describes the layout and fields in the **/photon_data** group.
Note that only the kind of data is specified (i.e. scalar,
integer array, float array), but no data type size is mandated.
For arrays, the most commonly used data-type is indicated.

Mandatory fields:

- **timestamps**: (array) photon timestamps. Typical data-type int64.
- **timestamps_specs/**
    - **timestamps_unit**: (float) timestamp units in *seconds*.

Optional if there is only 1 detector, otherwise mandatory:

- **detectors**: (array of integers) :ref:`detector_ids` for each timestamp.
  Typical data-type uint8.

When the dataset contains TCSPC or nanotime information (i.e. arrival time of each
photon with respect to a laser pulse), the following
fields must be present:

- **nanotimes**:(array of integers) TCSPC nanotimes. Conventionally the time
  axis direction is the "natural" direction, i.e. lifetime decays look
  correctly oriented in time. For more details see :ref:`nanotimes_time_axis`.
  Typical data-type uint16.
- **nanotimes_specs/**
    - **tcspc_unit**: (float) TAC/TDC bin size (in seconds).
    - **tcspc_num_bins**: (integer) number of TAC/TDC bins.
    - **tcspc_range**:(float) (optional) full-scale range of the TAC/TDC (in seconds).
      This a derived field equal to ``tcspc_unit * tcspc_num_bins``.

Finally, if the data come from a simulation, ``/photon_data`` may contain:

-  **particles**: (array of integers) a particle ID (integer) for each
   timestamp. Typical data-type uint8.

.. _measurement_specs_group:

Measurement specs
^^^^^^^^^^^^^^^^^

The group **/photon_data/measurement_specs** group contains additional
information allowing unambiguous interpretation of the data for each specific
type of measurement.

- **measurement_type**: (string) the type of the measurements. Valid names
  are:

    - "smFRET" (1 excitation wavelengths, 2 detection spectral bands). If
      ``/setup/lifetime`` is True the measurement also includes TCSPC nanotimes.
    - "smFRET-usALEX" (2 excitation wavelengths, 2 detection colors)
    - "smFRET-usALEX-3c" (3 excitation wavelengths, 3 detection colors)
    - "smFRET-nsALEX" (2 pulsed-laser wavelengths, 2 detection colors)
    - "generic" a generic measurement defined by fields in ``/setup``
      and ``measurement_specs``.

The *measurement_type* field describes the type of measurement
saved within the file. A "specific" measurement type,
can mandate the presence of certain fields in ``measurement_specs``
(see also :ref:`measurement_type`).

In Photon-HDF5 <0.5, each type of measurement required a distinct
``measurement_type``, which was inconvenient for supporting
many variants of common measurements. In Photon-HDF5 0.5+
we added a :doc:`"generic" measurement type <generic>` which supports
a large combination of setup configurations. In this case, values in
``/setup/`` will determine  mandatory ``measurement_specs`` fields.
If you feel that a new "specific" measurement type is needed
for your application we have :doc:`instructions <new_measurement_specs>`
to propose a new one.
An advantage of a "specific" measurement type compared to "generic",
is that additional fields can be made mandatory.

The following ``measurement_specs`` fields are present in specific
measurement types.

For μs-ALEX, 2, 3 or N colors:

- **alex_period**: (integer or float) duration of one complete excitation
  alternation period expressed in timestamp units. The alternation period
  is equal to ``alex_period * timestamps_unit``.

For ns-ALEX (or lifetime with no alternation):

- **laser_repetition_rate**: (float) excitation laser pulse repetition rate
  in *Hertz*.

For 2-color (or more) μs-ALEX and ns-ALEX (optional):

- **alex_offset**: (scalar) [μs-ALEX only] Time offset (in timestamps units)
  to be applied to the timestamps array before computing the μs-ALEX histogram.
  It is assumed that the μs-ALEX alternation histogram is the histogram of
  (``timestamps`` - ``alex_offset``) **MOD** ``alex_period``.

- **alex_excitation_period1**: (array with an even-number of integer
  elements, normally 2) start and stop values identifying the excitation
  periods for the **first** wavelength in ``/setup/excitation_wavelengths``
  (which is the shortest wavelength).
  In smFRET experiments with 2-colors excitation this field defines the
  *donor excitation period*.
  See also :ref:`wavelengths_order` and note below.

- **alex_excitation_period2**: (array with an even-number of integer
  elements, normally 2) start and stop values identifying the excitation
  periods for the **second** wavelength in ``/setup/excitation_wavelengths``.
  In smFRET experiments with 2-colors excitation this field defines the
  *acceptor excitation period*.
  See also :ref:`wavelengths_order` and note below.

For 3 (or more) colors alternated or interleaved excitation:

- **alex_excitation_period3**: (array with an even-number of integer
  elements, normally 2) start and stop values identifying the excitation
  periods for the **third** wavelength in ``/setup/excitation_wavelengths``.
  See also :ref:`wavelengths_order` and note below.

- etc...

.. note::

    For μs-ALEX, both *alex_excitation_period1* and *alex_excitation_period2*
    are 2-element arrays and are expressed in *timestamps_units*.
    For ns-ALEX (also known as PIE), they are arrays with an even-number
    of elements, comprising as many start-stop nanotime pairs as
    there are excitation periods within the TAC/TDC range.
    In this case the values are expressed in *nanotimes_units*.

    For more details see :ref:`alex_period_def`.

.. _detectors_specs_group:

Detectors specs
"""""""""""""""

Within **measurement_specs**, the **detectors_specs/** sub-group
contains all the :ref:`pixel ID <detector_ids>`--detection channel
associations, i.e. spectral bands, polarizations or
:ref:`beam-split channels <beam_split_ch>`.

When a measurement records more than 1 spectral band, the fields:

- **spectral_ch1**
- **spectral_ch2**
- etc...

specify which detector pixel is employed in each spectral band. When the measurement
records only 1 spectral band these fields may be omitted. The spectral bands
are strictly ordered for increasing wavelengths. For example, for 2-color
smFRET measurements ``spectral_ch1`` and ``spectral_ch2`` represent the
*donor* and *acceptor* channel respectively.

If a measurement records more than 1 polarization states, the fields:

- **polarization_ch1**
- **polarization_ch2**

specify which detector pixel is used for each polarization. When the measurement
records only one polarization, these fields may be omitted.

When the detection light is split into 2 channels using a non-polarizing
beam-splitter the fields:

- **split_ch1**
- **split_ch2**

specify which detector pixel is used in each of the "beam-split" channels.

All previous fields are arrays containing one or more :ref:`detector_ids`.
For example, a 2-color smFRET measurement will have only one value in
``spectral_ch1`` (donor) and one value in ``spectral_ch2``
(acceptor). A 2-color smFRET measurement with polarization
(4 detectors) will have 2 values in each of the ``spectral_chX`` and
``polarization_chX`` fields (where X=1 or 2).
For a multispot smFRET measurement, in each ``photon_dataN`` group,
there will be ``spectral_chX`` fields containing the donor/acceptor
pixels used in that spot (see :ref:`multi_spot`).

If the measurement records space-time markers (timestamps of timing signals
rather than photons; used, for example, in FLIM), the fields (*new in version
0.6*):

- **space_time_marker1**
- **space_time_marker2**
- etc...

specify which :ref:`detector id<detector_ids>` is used for each marker channel.
These fields are single integer values, not arrays.

.. _setup_group:

Setup group
-----------

The **/setup** group contains information about the measurement setup.
This group can be **absent** in some files, an example being
a file containing only detector dark counts, for which the following
fields do not necessarily have a meaning.
When setup is present, the following 9 fields are mandatory:

- **num_spectral_ch**: (integer) number of distinct detection spectral
  channels. For example, in a 2-color smFRET experiment there are 2
  detection spectral channels (donor and acceptor), therefore its value is 2.
  When there is a single detection channel or all channels detect
  the same spectral band, its value is 1.

- **num_polarization_ch**: (integer) number of distinct detection polarization
  channels. For example, in polarization anisotropy measurements, its value
  is 2.
  When there is a single detection channel or all channels detect
  the same polarization (including when no polarization selection is performed)
  its value is 1.

- **num_split_ch**: (integer) number of distinct detection channels
  detecting the same spectral band **and** polarization state. For example,
  when a non-polarizing beam-splitter is employed in the detection path,
  its value is 2. When no splitting
  is performed, its value is 1.

- **num_spots**: (integer) the number of excitation (or detection)
  "spots" in the sample. This field is 1 for all the measurements using a
  single confocal excitation volume.

- **num_pixels**: (integer) total number of detector pixels. For example,
  for a single-spot 2-color smFRET measurement using 2 single-pixel SPADs as
  detectors this field is 2. This field is normally equal to
  ``num_pixels = num_spectral_ch * num_split_ch * num_polarization_ch * num_spot``.
  This equation is not valid when the optical setup has non-symmetric branches,
  for example if the emission path is split in two spectral bands and only
  one of the two is further split in two polarizations.

- **excitation_cw**: (array of booleans) for each excitation source,
  this field indicates whether excitation is continuous wave (CW), *True*,
  or pulsed, *False*.
  The order of excitation sources is the same as that in
  ``excitation_wavelengths`` and is in increasing order of wavelengths.

- **lifetime**: (boolean) *True* (or 1) if the measurements includes a
  *nanotimes* array of (usually sub-ns resolution) photon arrival times with
  respect to a laser pulse (as in TCSPC measurements).

- **modulated_excitation**: (boolean) *True* (or 1) if there is any form of
  excitation modulation either in the wavelength space (as in μs-ALEX or PAX)
  or in the polarization space. This field is also *True* for
  pulse-interleaved excitation (PIE) or ns-ALEX measurements.

- **excitation_alternated**: (array of booleans) *New in version 0.5.*
  For each excitation source,
  this field indicates whether the excitation is alternated (True) or
  not alternated (False). In ALEX measurement all sources are alternated.
  In PAX measurements only one of the two sources is alternated.

.. _detectors_group:

Detectors group
^^^^^^^^^^^^^^^^

*New in version 0.5.* The group **/setup/detectors**
contains arrays with one element per detector's pixel.  The allowed fields are:

    - **id** (array of int): *Mandatory.* Number used by in ``/photon_data/detectors`` to
      identify the pixel.
    - **id_hardware** (array of int): *Optional.* Detector numbers as used by the
      acquisition hardware if different from ``id``.
    - **label** (array of string): *Optional.* A human-readable label for each detector.
    - **counts** (array of int): *Optional.* Number of timestamps counted by each detector.
    - **module** (array of string): *Multispot only, optional.* Name of the module each
      pixel belongs to.
    - **position** (2-D array of int): *Multispot only, optional.* Columns are x,y
      positions of each pixel in the array.
    - **dcr** (array of float): *Optional.* Dark counting rate in Hz for each pixel.
    - **afterpulsing** (array of float): *Optional.* Afterpulsing probability
      for each pixel.
    - **spot** (array of int): *Multispot only, mandatory.* The spot number each pixel is
      used in.
    - **tcspc_unit:** (array of float) array of TAC/TDC bin size (in seconds).
      Present only if ``/setup/lifetime`` is True and if TCSPC info is different
      for each pixel.
    - **tcspc_num_bins:** (integer) array of number of TAC/TDC bins. Present
      only if ``/setup/lifetime`` is True and if TCSPC info is different
      for each pixel.

For more info see
:ref:`Group /setup/detectors <setup_detectors_group>`.

.. _optional_setup_fields:

Optional /setup fields
^^^^^^^^^^^^^^^^^^^^^^

The following ``/setup`` fields are optional and not necessarily relevant for
all experiments. These fields may be not present when the associated
information is irrelevant or not available.

The following fields are arrays, one element per excitation source,
in the order of increasing wavelengths:

- **excitation_wavelengths**: (array of floats) list of excitation wavelengths
  (center wavelength if broad-band) in increasing order (unit: *meter*).

- **laser_repetition_rates**: (array of floats)  *New in version 0.5.*
  The laser repetition rate in Hz for each excitation source.
  This field is mandatory only if there is at least one pulsed excitation
  source. When there are both CW and pulsed lasers, the CW lasers will have
  a value of 0 in this field.

- **excitation_polarizations**: (arrays of floats) list of polarization
  angles (in degrees) for each excitation source.

- **excitation_input_powers**: (array of floats) excitation power in *Watts*
  for each excitation source. This is the excitation power entering
  the optical system.

- **excitation_intensity**: (array of floats) excitation intensity in the
  sample for each excitation source (units: *W/m²*).
  In the case of confocal excitation this is the peak PSF intensity.

The following fields are also arrays:

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

The following fields can be used to indicate the meaning of space-time markers.

- **num_space_time_markers**: (integer) *New in version 0.6.* number of
  space-time markers. Required if space-time markers are recorded. This value
  must match the number of space-time markers specified by
  ``detectors_specs/space_time_marker1``,
  ``detectors_specs/space_time_marker2``, and so on.

- **space_time_markers**: (array of strings) *New in version 0.6.* for each
  space-time marker, this field indicates a standard interpretation. The length
  of this array must equal the value of ``num_space_time_markers``. Currently
  defined interpretations are for indicating raster timings in laser-scanning
  imaging (such as FLIM), and valid element values are:

  - ``"pixel"`` (pixel clock; the marker indicates the beginning timestamp of
    each raster pixel),
  - ``"line"`` (line clock; the marker indicates the beginning timestamp of each
    raster scanline),
  - ``"frame"`` (frame clock; the marker indicates the beginning timestamp of
    each raster frame), and
  - ``""`` (empty string; indicates a marker not used for any of the above
    standard meanings).

.. _sample_group:

Sample group
------------

The **/sample** group contains information related to the measured sample.
This group is optional.

- **num_dyes**: (integer) number of different dyes present in the samples.
- **dye_names**: (string) comma-separated list of dye or fluorophore names
  (for example: ``"ATTO550, ATTO647N"``)
- **buffer_name**: (string) a user defined description for the buffer.
- **sample_name**: (string) a user defined description for the sample.


.. _identity_group:

Identity group
--------------

The **identity/** group contains information about the specific Photon-HDF5
file.

The following fields are mandatory (and automatically added by
`phconvert <http://photon-hdf5.github.io/phconvert/>`__):

- **creation_time**: (string) the Photon-HDF5 file creation time with
  the following format: "YYYY-MM-DD HH:MM:SS".
- **software**: (string) name of the software used to create
  the Photon-HDF5 file.
- **software_version**: (string) version of the software used to create
  the Photon-HDF5 file.

- **format_name**: (string) this must always be "Photon-HDF5"
- **format_version**: (string) the Photon-HDF5 version string (e.g. "0.4")
- **format_url**: (string) A URL pointing to the Photon-HDF5 specification
  document.

The following fields are optional:

- **author**: (string) the author of the measurement (or simulation).
- **author_affiliation**: (string) the company or institution the *author*
  is affiliated with.

- **creator**: (string) the Photon-HDF5 file creator. Used when the data
  was previously stored in another format and the conversion is performed by
  a different person than the author.
- **creator_affiliation**: (string) the company or institution the *creator*
  is affiliated with.

- **url**: (string) URL that allow to download the Photon-HDF5 data file.
- **doi**: (string) Digital Object Identifier (DOI) for the Photon-HDF5
  data file.
- **funding**: (string) Description of funding sources and or grants that
  supported the data collection.
- **license**: (string) License under which the data is released. Many journals
  and funding agencies require or suggest "CC0" (or equivalently "Public Domain")
  for the data.

- **filename**: (string) Photon-HDF5 file name at creation time. This field
  saves the original file name even if the file is later on renamed on disk.
- **filename_full**: (string) Photon-HDF5 file name (including the full path)
  at creation time.


.. _provenance_group:

Provenance group
----------------

The **provenance/** group contains info about the original file that has
been converted into a Photon-HDF5 file.
If the file is directly saved to Photon-HDF5 there is no previous "original"
file and in this case the provenance group may be omitted.
Also, if some information is not available the relative field may be omitted.

- **filename**: (string) File name of the original data file before conversion to Photon-HDF5.
- **filename_full**: (string) File name (with full path) of the original data file before conversion to Photon-HDF5.
- **creation_time**: (string) Creation time of the original data file.
- **modification_time**: (string) Time of last modification of the original data file.
- **software**: (string) Software used to save the original data file.
- **software_version**: (string) Version of the software used to save the original data file.


.. _glossary:

Additional notes and definitions
--------------------------------

.. _detector_ids:

Detector pixel IDs
^^^^^^^^^^^^^^^^^^

A *detector pixel ID* (or simply *pixel ID*) is the "name" of each pixels and
is usually a single integer. Pixels are normally numbered incrementally, but not necessarily so.
In other words, a file containing data taken with 2 single-point (pixel) detectors could have
the first detector labeled "4" and the second detector labeled "6".
In some cases (when using detector arrays) the pixel ID
can be a *n*-tuple of integers. This allow to specify, for each pixel,
the module number and the X, Y location, for example. Therefore, an
array of pixel IDs can be either a 1-D column array or a 2-D array.
In either cases, each row identifies a pixel.


.. _beam_split_ch:

Beam-split channels
^^^^^^^^^^^^^^^^^^^

When the emitted light path is split in 2 or more detection paths by using
a non-polarizing beam splitter the measurement has so called
beam-split channels. The fields `split_ch1` and `split_ch2` contains
the list of :ref:`detector_ids` for each beam-split channel
(see :ref:`detectors_specs_group`).

Beam split channels can receive same or different (depending on whether the
beam splitter is 50-50). The fractional power of each beam split channel
can be saved in the field `detection_split_ch_ratios` in the
:ref:`setup_group`.


.. _wavelengths_order:

Wavelengths and spectral-band order
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In Photon-HDF5, by convention, all the excitation wavelengths and detection
spectral bands are ordered in increasing order: from the shortest to the
longest wavelength. This ordering is strictly followed and removes any
ambiguity in defining first, second, etc... wavelength or spectral band.

For examples, for μs-ALEX and ns-ALEX (or PIE) the excitation wavelengths
(in ``/setup/excitation_wavelenths``) are ordered as

1. *donor excitation wavelength*,
2. *acceptor excitation wavelength*

Similarly, the donor (or acceptor) excitation period range is defined by
``/photon_data/measurement_specs/alex_excitation_period1``
(or ``/photon_data/measurement_specs/alex_excitation_period2``).

Finally the donor (or acceptor) :ref:`detector_ids` number is defined in
``/photon_data/measurement_specs/detectors_specs/spectral_ch1``
(or ``/photon_data/measurement_specs/detectors_specs/spectral_ch2``).


.. _alex_period_def:

Definition of alternation periods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Note for μs-ALEX
""""""""""""""""

The fields *alex_offset*, *alex_excitation_period1* and *alex_excitation_period2*
define the excitation period for each excitation source. The alternation
histogram is the histogram of the following quantity:

``A`` = (``timestamps`` - ``alex_offset``) **MODULO** ``alex_period``

Note that ``alex_offset`` must be a value that shifts the timestamps in a way
that the resulting alternation histogram has uninterrupted excitation periods
for each excitation source. It can be thought as the delay between the start
of the timestamping and the start of the alternation modulation.
In most cases this is just an empirical parameter depending on the
specific setup.

Photons emitted during the donor period (or, respectively, acceptor
period) are obtained by applying the condition:

-  ``(A >= start) and (A < stop)``

.. figure:: /images/alternation_range.png
    :alt: μs-ALEX alternation histogram with marked excitation ranges.
    :align: center

    Alternation histogram showing selection for the donor and acceptor periods.


.. _measurement_type:

Measurement type
^^^^^^^^^^^^^^^^

Each *measurement_type* has an associated set of mandatory fields
which must be present to ensure that all information needed to
unambiguously interpret the data is present.
For example, for a 2-color smFRET measurement, a software package creating
a file should check that
the association between detector-pixel and donor or acceptor channel
is present.

The list of supported measurement types is reported in
:ref:`measurement_specs_group`.


.. _nanotimes_time_axis:

Nanotimes time direction
^^^^^^^^^^^^^^^^^^^^^^^^

In typical TCSPC measurement the *start* and *stop* inputs are inverted,
i.e. the *start* is triggered by the photon and the *stop* by the the laser sync.
This allows to start TAC or TDC measurements only when a photon is
detected and not after each laser sync pulse. However, due to this experimental
configuration, the resulting raw TCSPC histogram looks inverted along the time axis,
with the nanotimes of photons emitted shortly after a laser pulse being larger than
the nanotimes of photons emitted much later.

By convention, the Photon-HDF5 format requires nanotimes to be properly oriented. In other words,
when a `nanotimes` time axis inversion is needed, this correction needs to be performed before
the data is saved into a Photon-HDF5 file. As a corollary, TCSPC histograms computed directly from
`nanotimes` from Photon-HDF5 files are always properly oriented, regardless of
the way the nanotimes were acquired.


.. _setup_detectors_group:

Group /setup/detectors
^^^^^^^^^^^^^^^^^^^^^^
This group is new in version 0.5 and contains fields which are arrays, one
element per detector
(see :ref:`definition <detectors_group>`).
The only mandatory field is ``id`` which contains
all detectors IDs as they appear in ``/photon_data/detectors``.
Within each spot, IDs appear in ``/setup/id`` in increasing order.
All values which appears in
``/photon_data/detectors`` need to be listed here. This includes non-standard
detectors (e.g. a monitor channel to monitor the input power) or space-time markers
saved by the acquisition hardware (for example PicoQuant TCSPC
hardware can save makers for synchronization). However, special
detector IDs used for overflow correction must be removed before
saving a Photon-HDF5 file.

In TCSPC measurements where each pixel has different TCSPC bin width,
the ``/setup/detectors`` group allows to save per-pixel TCSPC info.
In this case the the
``nanotimes_specs`` group is not present in ``photon_data`` and the group
``/setup/detectors`` will contain per-pixel TCSPC info::

    /setup/detectors/tcspc_units
    /setup/detectors/tcspc_num_bins


.. _multi_spot:

Multi-spot measurements
^^^^^^^^^^^^^^^^^^^^^^^

Multi-spot measurements are simply handled by having multiple ``photon_data``
groups, one for each excitation spot. The naming convention is the following::

    photon_data0
    photon_data1
    ...
    photon_data10
    ...
    photon_data100

Note that the enumeration starts from zero and there is no zero filling.
Each ``photon_dataN`` group will have a complete ``measurement_specs``
sub-group so that it can effectively treated as a single-spot measurements
when reading the file.
As a result, even if the ``measurement_type`` field is not expected to change
for different spots, it will be replicated inside each ``photon_dataN``
group.

In version 0.5 and above the ``/photon_dataNN/detectors`` arrays need to
contain detectors identifier which are unique across all the spots. In version
0.4 the same identifiers (e.g. 0 and 1) were allowed in different spots.
