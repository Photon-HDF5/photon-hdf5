.. _version0.5:

What changed in version 0.5
===========================

- A new field: ``/setup/excitation_alternated`` (array of booleans), was added, which is True
  for all excitation sources which are intensity-modulated.
  This field allows distinguishing between ALEX and PAX measurements (i.e.
  when only one laser is alternated).

- A New group: ``/setup/detectors`` containing arrays of per-pixel information, was added.
  See :ref:`setup_detectors_group`.

- Support was added for cases where TCSPC specifications change for each pixel, by addition of 
  dedicated fields in ``/setup/detectors``.
  See :ref:`Group /setup/detectors <setup_detectors_group>`.

- A new measurement_type "generic" was added. In a "generic" measurement, the
  specifications are determined by the fields in the ``/setup`` group.
  For details see :doc:`generic`.
