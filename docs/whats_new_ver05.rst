.. _version0.5:

What changes in version 0.5
===========================

- New field ``/setup/excitation_alternated`` (array of booleans) that is True
  for all the excitation sources that are intensity-modulated.
  This field allows to distinguish between ALEX and PAX measurements (i.e.
  when only one laser is alternated).

- New group ``/setup/detectors`` containing arrays of per-pixel information.
  See :ref:`setup_detectors_group`.

- Support for cases where the TCSPC specs change for each pixel with
  dedicated fields in ``/setup/detectors``.
  See :ref:`Group /setup/detectors <setup_detectors_group>`.

- A new measurement_type "generic" is added. In a "generic" measurement, the
  specifications are determined by the fields in ``/setup``.



The "generic" measurement type
------------------------------

For measurement types that cannot be described by any other official
``measurement_type`` in Photon-HDF5, it is possible "generic" and
relying on the ``/setup`` fields to describe the exact configuration.

Here we list a few examples of measurements described with the "generic"
measurement_type.

2-Polarizations
^^^^^^^^^^^^^^^

A setup detecting two polarizations in the detection path will have::

    /setup/num_spectral_ch = 1
    /setup/num_split_ch = 1
    /setup/num_polarization_ch = 2

and will also specify the detector used for each polarizations::

    /photon_data/measurement_specs/detectors_specs/
        polarization_ch1 = 0
        polarization_ch2 = 1

(where the values 0 and 1 are only examples). In addition ``/setup/lifetime``
will tell if the measurement has TCSPC data or not. Finally, the optional field
``/setup/excitation_polarizations`` can be used to save the angle of each
polarization.

2-Split channels
^^^^^^^^^^^^^^^^

This example is a setup in which the detection (emission) path is split
in two using a non-polarizing beam splitter. In this case we have::

    /setup/num_spectral_ch = 1
    /setup/num_split_ch = 2
    /setup/num_polarization_ch = 1

and the measurement_specs will contain::

    /photon_data/measurement_specs/detectors_specs/
        split_ch1 = 0
        split_ch2 = 1

(where the values 0 and 1 are only examples). In addition ``/setup/lifetime``
will tell if the measurement has TCSPC data or not. Finally, the optional
field ``/setup/detection_split_ch_ratios`` can be used to store the
"splitting-ratio" of each channel. For example, with a 70-30% beam splitter
this field will be ``[0.7, 0.3]``.

2-Spectral channels
^^^^^^^^^^^^^^^^^^^

This case is already covered by the "smFRET" ``measurement_type`` both for CW and
TCSPC measurements (the two are distinguished with the value in
``/setup/lifetime``).

If a measurement has two spectral detection bands but it is fundamentally
different from a smFRET measurement, then the "generic" ``measurement_type``
can be used. In this case the ``measurement_specs`` fields will be the same
as in the "smFRET" ``measurement_type``. In particular::

    /setup/num_spectral_ch = 2
    /setup/num_split_ch = 1
    /setup/num_polarization_ch = 1

and::

    /photon_data/measurement_specs/detectors_specs/
        spectral_ch1 = 0
        spectral_ch2 = 1

TCSPC measurements
^^^^^^^^^^^^^^^^^^

In all the previous examples, a variant with TCSPC data will have
``/setup/lifetime = True`` and the field
``/photon_data/measurement_specs/laser_repetition_rate`` specifying the
laser pulse rate. When the laser is pulsed but the acquisition hardware is
not TCSPC the ``/setup/excitation_cw`` contains False and
``/setup/lifetime = False``.

2-Spectral channels + polarization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a smFRET measurement where the donor and acceptor emission paths
are further split in parallel and perpendicular polarization. In this case
we have::

    /setup/num_spectral_ch = 2
    /setup/num_split_ch = 1
    /setup/num_polarization_ch = 2

Assuming that detectors 0, 1 are used for the two polarizations in
the donor channel and detectors 2, 3 for the acceptor channel, we have::

    /photon_data/measurement_specs/detectors_specs/
        spectral_ch1 = [0, 1]
        spectral_ch2 = [2, 3]
        polarization_ch1 = [0, 2]
        polarization_ch2 = [1, 3]


ALEX + polarization
^^^^^^^^^^^^^^^^^^^

In this measurement we have μs-ALEX excitation and four detectors for the
two polarizations for donor and acceptor paths. This is the same as the previous
example but with two CW alternating lasers.

In this case we have::

    /setup/num_spectral_ch = 2
    /setup/num_split_ch = 1
    /setup/num_polarization_ch = 2
    /setup/excitation_alternated = [True, True]

and, using the detector number of the previous example::

    /photon_data/measurement_specs/
        alex_period = 4000
        detectors_specs/
            spectral_ch1 = [0, 1]
            spectral_ch2 = [2, 3]
            polarization_ch1 = [0, 2]
            polarization_ch2 = [1, 3]

Note that when there is a True in ``/setup/excitation_alternated`` then
``/photon_data/measurement_specs/alex_period`` need to be present.
