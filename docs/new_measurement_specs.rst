.. _new_measurement_specs:

Defining new measurement types
==============================

If you need to save a type of measurement not included in the current list 
supported measurement types, you may want to use a 
:doc:`"generic" measurement type <generic>` instead.

If the "generic" type is for some reason unsuitable you may propose a new
"specific" measurement type following the instruction in this page.

A new "measurement_type" will define what kind of fields will be present
in the :ref:`measurement_specs group <measurement_specs_group>`
in order to make the dataset self-contained.


What to put in measurement_specs
--------------------------------

First, a new measurement_specs needs to have an official name.
The name of the measurement-type is a string stored
in `measurement_specs/measurement_type`. This string is used to differentiate
between different measurement_specs. For example, names of already defined
measurment types are: `smFRET`, `smFRET-usALEX`, `smFRET-usALEX-3c` and
`smFRET-nsALEX` (see :ref:`here <measurement_specs_group>`).

Beyond the name, measurement_specs group should contain all the metadata
needed to for unambiguous analysis of the dataset. For example, in smFRET experiments
the measurement_specs contains the IDs of donor and acceptor detection
channels. In Alternated Excitation (ALEX) measurements it will also contains
definition of laser alternation (period, range for donor and acceptor, etc...).

In most cases, measurement_specs will contain the association between
detector IDs physical detection channels (:ref:`detectors_specs_group`).
Detection channels may differ in
the detected spectral band and in detected polarization, called
`spectral_chX` and `polarization_chX` (where *X* is 1, 2, ...). When using a
non-polarizing beam splitter and both spectral band and polarization are
equal, the detection channels are names `split_chX`
(see :ref:`beam_split_ch`).

If possible, new measurement_specs should follow naming conventions of
other official measurement_specs.

For any question please use the
`Photon-HDF5 Google Group <https://groups.google.com/forum/#!forum/photon-hdf5>`_.


How to propose a new measurement_specs
--------------------------------------

Logistically, the process works as follows.

1. Users propose new `measurement_specs` using the
   `Photon-HDF5 Google Group <https://groups.google.com/forum/#!forum/photon-hdf5>`_.
2. After discussion, the measurement_specs will be advertised on the
   Photon-HDF5 website in order to gather opinions from different parties.
3. Discussion continues and modifications are proposed until consensus
   is reached.
4. Finally, the new measurement_specs becomes official part of Photon-HDF5:
   it is added to the Photon-HDF5 documentation and software support is
   added to phconvert.

For any question please use the `Photon-HDF5 Google Group <https://groups.google.com/forum/#!forum/photon-hdf5>`_.
