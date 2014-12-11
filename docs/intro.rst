Introduction
============

1.1 Overview
------------

This document contains the specifications for the Photon-HDF5 format.
This format allows saving single-molecule spectroscopy experiments when
there is at least a stream of photon timestamps. It has been envisioned
as a standard container format for a broad range of experiments
involving confocal microcopy. Notable examples are confocal smFRET
experiments (with or without laser alternation) either with a single or
with multiple excitation spots. It can also store ns-ALEX or FCS
measurements.

What problems are we trying to solve?
-------------------------------------

-  Ensuring long term persistence of the data
-  A space- and speed-efficient file format for repeated use (by opposition to archiving)
-  Facilitating data sharing and interoperability between analysis programs

Features of HDF5
----------------

-  Open, standard and wide-spread used format with open source
   implementations
-  Efficient: the HDF5 format is a binary format that allows compression
   and is fast to read and write
-  Flexible: data arrays can be stored in "groups" (hierarchical
   format). Metadata can be attached to each data entry (attributes). No
   limit in data size. Support for a variety of numeric and non-numeric
   data types.

Photon-HDF5: Design principles
------------------------------

The main design principles we follow are

-  Simplicity
-  Flexibility
-  Compatibility

We aim to define a format that has a minimal set of specifications and therefore
is easy to implement. At the same time, it is important that format can be
expanded to accomodate new use cases while maintaining backward compatibility.

To achieve simplicity, the only required file characteristics are a
general file layout and the presence of a few basic attributes and parameters.
The remaining (small set of) fields here defined will be present only when
they will be needed by a particular measurement.

We retain flexibility by allowing the user to save any arbitrary data
outside the specs of this document. To assure that a future version of
this format will not clash with some user-defined fields, we require
that all the user-defined field be contained in groups named ``user``.
