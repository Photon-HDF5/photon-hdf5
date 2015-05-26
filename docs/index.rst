.. Photon-HDF5 documentation master file, created by
   sphinx-quickstart on Tue Dec 09 10:48:26 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Photon-HDF5 file format documentation
======================================

:Authors: Antonino Ingargiola, Xavier Michalet, Ted Laurence
:Contact: tritemio@gmail.com
:Version: 0.3

Photon-HDF5 is a file format single-molecule experiments
that requires storing photon stream data (e.g. photon timestamps).
Typical examples are freely-diffusing single-molecule FRET (smFRET)
(with or without lifetime), Fluorescence Correlation Spectroscopy (FCS)
and related techniques.

Any dataset containing a photon timestamp and other per-photon information
can be stored using the Photon-HDF5 format.
Additionally, for some predefined types of measurements, it is possible to
store all associated metadata needed for a complete interpretation
of the dataset. For example, a dataset created in a single confocal spot,
freely-diffusing smFRET experiment,
will contain a description of the relation between detector IDs and
donor/acceptor channels (called ``spectral_ch1`` and ``spectral_ch2``
respectively) allowing for a correct interpretation of the dataset
by a burst analysis program.

The present document contains the reference documentation for the Photon-HDF5 format.

Related resources:

- `Examples of reading Photon-HDF5 files in multiple laguages <https://github.com/Photon-Data/photon_hdf5_reading_examples>`__
- `phconvert <https://github.com/Photon-Data/phconvert>`__: reference library to write and convert Photon-HDF5 files
- `Why an HDF5-based smFRET file format <http://fretbursts.readthedocs.org/en/latest/HDF5_format.html>`__
- Photon-HDF5: an open format for single-molecule spectroscopy
  using photon-counting devices (*coming soon*).



.. note ::
   This document describes **Photon-HDF5 version 0.3**, the third iteration of the *Photon-HDF5 file format*.
   The latest version of this document can be found at
   `http://photon-hdf5.readthedocs.org <http://photon-hdf5.readthedocs.org/>`__.

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
    contributing
