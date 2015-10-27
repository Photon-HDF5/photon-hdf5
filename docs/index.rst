.. Photon-HDF5 documentation master file, created by
   sphinx-quickstart on Tue Dec 09 10:48:26 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Photon-HDF5 file format documentation
======================================

:Authors: Antonino Ingargiola, Xavier Michalet, Ted Laurence
:Contact: `Photon-HDF5 Google Group <https://groups.google.com/forum/#!forum/photon-hdf5>`_
:Format Version: 0.4


This is the reference documentation for `Photon-HDF5 <http://photon-hdf5.org/>`__,
a file format for timestamp-based single-molecule spectroscopy
experiments such as single-molecule FRET (smFRET) (with or without lifetime),
Fluorescence Correlation Spectroscopy (FCS) and other related techniques.

Any dataset containing photon timestamps and other per-photon data
can be stored in Photon-HDF5 files.
Photon-HDF5 is designed for long-term preservation and data sharing and
can therefore store experimental details and metadata such as
setup configurations, sample information, authorship and provenance.

The present document contains the reference documentation for the Photon-HDF5 format.

**Related resources:**

- |example_files|, view them with |hdfview|.
- |read_examples|
- |phconvert|: reference library to create and convert Photon-HDF5 files
- |photon-hdf5_paper| (BioRxiv manuscript, *submitted for peer-review*).


.. |example_files| raw:: html

   <a href="http://figshare.com/articles/Example_smFRET_data_files_in_Photon_HDF5_format/1456362" target="_blank">Example Photon-HDF5 data files</a>

.. |hdfview| raw:: html

   <a href="https://www.hdfgroup.org/products/java/hdfview/" target="_blank">HDFView</a>

.. |read_examples| raw:: html

   <a href="https://github.com/Photon-Data/photon_hdf5_reading_examples" target="_blank">Code examples of reading Photon-HDF5 files in multiple languages</a>

.. |phconvert| raw:: html

   <a href="http://photon-hdf5.github.io/phconvert/" target="_blank">phconvert</a>

.. |photon-hdf5_paper| raw:: html

   <a href="http://dx.doi.org/10.1101/026484" target="_blank">Photon-HDF5: an open file format for single-molecule fluorescence experiments using photon-counting detectors</a>


Table of contents
-----------------

.. toctree::
    :maxdepth: 3
    :numbered:

    intro
    phdata
    reading
    writing
    new_measurement_specs
    limitations
    contributing
