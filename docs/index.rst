.. Photon-HDF5 documentation master file, created by
   sphinx-quickstart on Tue Dec 09 10:48:26 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Photon-HDF5 file format documentation
======================================

:Authors: Antonino Ingargiola, Xavier Michalet, Ted Laurence
:Contact: tritemio@gmail.com
:Version: 0.3

.. note ::
   This document describes **Photon-HDF5 version 0.3**, the third iteration of the *Photon-HDF5 file format*.
   The latest version of this document can be found at
   `http://photon-hdf5.readthedocs.org <http://photon-hdf5.readthedocs.org/>`__.

Photon-HDF5 is a file format for a class of single-molecule spectroscopy
experiments that requires storing photon stream data.
Any dataset containing a photon timestamp and other per-photon information
can be stored using the Photon-HDF5 format.
Additionally, for some predefined types of measurements, it is possible to
store all associated metadata needed for a complete interpretation
of the dataset. For example, a dataset created in a single confocal spot,
diffusing single-molecule FRET (smFRET) experiment,
will contain a description of the relation between detector IDs and donor/acceptor channels
(called ``spectral_ch1`` and ``spectral_ch2`` respectively) allowing for
a correct interpretation of the dataset by a burst analysis program.

For a brief introduction on the HDF5 format and its
use for single-molecule spectroscopy, see
`Why an HDF5-based smFRET file format <http://fretbursts.readthedocs.org/en/latest/HDF5_format.html>`__.

.. warning::

    Although this specification is still a draft, it is the third complete
    rewrite of the file format. Therefore we do not foresee major modifications
    anymore.
    Comments and suggestions (including typos fixes) are encouraged.

    Version 0.3 will be the first version to be declared stable and for which
    we commit to strong backward compatibility.


Table of contents
-----------------

.. toctree::
    :maxdepth: 3
    :numbered:

    intro
    phdata
    limitations
