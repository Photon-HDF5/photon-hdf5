.. _reading:

Reading Photon-HDF5 files
=========================

You can easily read a Photon-HDF5 file in Python, MATLAB, LabVIEW or any
in any other language that supports HDF5 (C, C++, Java, R, etc...).

We provide
`examples  on reading Photon-HDF5 files <http://photon-hdf5.github.io/photon_hdf5_reading_examples/>`_
in different programming languages.

The next section discuss how to write a generic Photon-HDF5 reader
for a single-molecule FRET analysis program.


Reading Photon-HDF5 in a smFRET analysis program
-------------------------------------------------

In this section describes a scheme can be used to read
Photon-HDF5 files in smFRET analysis programs. This scheme is implement
in the burst analysis program
`FRETBursts <http://tritemio.github.io/FRETBursts/>`_
(`see code <https://github.com/tritemio/FRETBursts/blob/master/fretbursts/loader.py#L226>`_).

The scheme is reported here as an example on how to add support for reading
Photon-HDF5 files to an analysis program.

 Read Photon-HDF5 files
 ~~~~~~~~~~~~~~~~~~~~~~~

1. Get the Photon-HDF5 version from ``/identity/format_version``.
   The version must be ``'0.4'`` or greater.

2. Optionally load metadata from `setup`, `sample`, `provenance` and `identity`
   groups. This is not needed for the analysis. The fields may not be present.

3. Same as in point 2 for root fields `description` and `acquisition_duration`.

4. If there is a ``/photon_data`` group the file is single-spot. Call the
   function to load single-spot data (see next section). If there is
   no ``/photon_data`` the file is a multi-spot one, call the multi-spot read
   function (next section).

Read single-spot function
~~~~~~~~~~~~~~~~~~~~~~~~~

Make a function that takes a channel parameter (`N`) and reads the data of the
corresponding channel (``/photon_dataN``). If `N` is not passed, the function
reads data from ``/photon_data``.

Read the photon-data following these steps:

1. Check if a photon-data group with given `N` is present, if not skip the
   channel. This is needed in cases of missing channels (e.g. dead pixel
   in SPAD array).

2. Load photon-data arrays. Load the array ``timestamps`` and its unit
   (``timestamps+specs/timestamps_unit``). If present, load also
   ``detectors``, ``nanotimes`` and ``particles``. If ``nanotimes`` is present
   load ``nanotime_specs`` (``tcspc_unit`` and ``tcspc_num_bins``).

3. Load measurements-specs. Refer to :ref:`measurement_specs_group`
   documentation for the details. Note that ``measurement_specs`` may be not
   present.

3. If ``measurement_specs`` is present and the measurement-type starts with
   ``smFRET`` load detectors definitions
   (donor: ``measurement_specs/detectors_specs/spectral_ch1``,
   acceptor: ``measurement_specs/detectors_specs/spectral_ch2``).

4. For us-ALEX load ``alex_period`` and for ns-ALEX load ``
   load the period definitions.

5. For both us-ALEX and ns-ALEX load the donor and acceptor
   period definitions (``alex_excitation_period1`` and
   ``alex_excitation_period1``). For us-ALEX also load
   ``alex_offset``. All these field may not be present.

Read multi-spot function
~~~~~~~~~~~~~~~~~~~~~~~~

Implement the single-channel version then:

- Find all the root groups starting with ``photon_data`` and for each group
  load the data for that channel. There can be missing channels
  (e.g. if there are dead pixels).
