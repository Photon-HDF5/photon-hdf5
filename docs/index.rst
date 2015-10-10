.. Photon-HDF5 documentation master file, created by
   sphinx-quickstart on Tue Dec 09 10:48:26 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Photon-HDF5 file format documentation
======================================

:Authors: Antonino Ingargiola, Xavier Michalet, Ted Laurence
:Contact: tritemio@gmail.com
:Version: 0.4


Photon-HDF5 is a file format for freely diffusing single-molecule spectroscopy
experiments such as single-molecule FRET (smFRET) (with or without lifetime),
Fluorescence Correlation Spectroscopy (FCS) and related techniques.

Any dataset containing photon timestamps and other per-photon data
can be stored in Photon-HDF5 files.
Photon-HDF5 is designed for long-term preservation and data sharing and
can therefore store experimental details and metadata such as
setup configurations, sample information, authorship and provenance.

The present document contains the reference documentation for the Photon-HDF5 format.

Other related resources:

- |example_files|, view them with |hdfview|.
- |read_examples|
- |phconvert|: reference library to create and convert Photon-HDF5 files
- |photon-hdf5_paper| (*coming soon*).


.. |example_files| raw:: html

   <a href="http://figshare.com/articles/Example_smFRET_data_files_in_Photon_HDF5_format/1456362" target="_blank">Example Photon-HDF5 data files</a>

.. |hdfview| raw:: html

   <a href="https://www.hdfgroup.org/products/java/hdfview/" target="_blank">HDFView</a>

.. |read_examples| raw:: html

   <a href="https://github.com/Photon-Data/photon_hdf5_reading_examples" target="_blank">Code examples of reading Photon-HDF5 files in multiple languages</a>

.. |phconvert| raw:: html

   <a href="http://photon-hdf5.github.io/phconvert/" target="_blank">phconvert</a>

.. |photon-hdf5_paper| raw:: html

   <a href="" target="_blank">Photon-HDF5: an open format for single-molecule spectroscopy using photon-counting devices </a>


.. note ::
   This document describes **Photon-HDF5 version 0.4**.
   The latest version of this document can be found at
   `http://photon-hdf5.org <http://photon-hdf5.org/>`__.
   To send correction or getting involved in the file format
   development please see the :ref:`contributing <contributing>` section.


Table of contents
-----------------

.. toctree::
    :maxdepth: 3
    :numbered:

    intro
    phdata
    reading
    writing
    limitations
    contributing
