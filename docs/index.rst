.. Photon-HDF5 documentation master file, created by
   sphinx-quickstart on Tue Dec 09 10:48:26 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Photon-HDF5 file format documentation
======================================

:Authors: Antonino Ingargiola, Xavier Michalet, Ted Laurence
:Contact: tritemio@gmail.com
:Version: 0.2

.. note ::
   This document describes the version "0.2-Draft" of the
   **Photon-HDF5 file format**. The latest version of this document
   can be found at 
   `http://photon-hdf5.readthedocs.org <http://photon-hdf5.readthedocs.org>`__.

Photon-HDF5 is a file format for a class of single-molecule spectroscopy 
experiments that requires storing "photon streams" (a list of per-photon data). 
A noteworthy example is represented by confocal single-molecule FRET (smFRET) 
experiments.

For a brief introduction what the HDF5 format is and why it is
important for single-molecule data see
`Why an HDF5-based smFRET file format <http://fretbursts.readthedocs.org/en/latest/HDF5_format.html>`__.

.. warning::

    This specification is still a draft. Comments and suggestions
    (including typos fixes) are encouraged.


Table of contents
-----------------

.. toctree::
    :maxdepth: 3

    phdata
