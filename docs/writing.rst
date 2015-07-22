.. _writing:

Writing Photon-HDF5 files
=========================

The recommended way to write (or convert) Photon-HDF5 files is by using the
`phconvert <https://github.com/Photon-HDF5/phconvert>`_ python library.

However, some user may want to save Photon-HDF5 files directly from an
acquisition software not written in python (as opposed to converting the files
later on). Both use cases are illustrated below.

Using phconvert
---------------

Users wanting to convert existing files to Photon-HDF5 can use phconvert.
`phconvert homepage <http://photon-hdf5.github.io/phconvert/>`_ has
instructions on how to convert files.

If your input file format is not supported by *phconvert* please open a
`new issue <https://github.com/Photon-HDF5/phconvert/issues>`__ on GitHub for phconvert.
Adding a new file format is quite straightforward
and we want to support as many format as possible out of the box.

When saving a new file, *phconvert* performs a validation the user input
therefore assuring that the generated file is a valid Photon-HDF5 file.
Furthermore a description attribute (*TITLE*) is automatically added to every
field making easier for a user to interactively inspect the file
(for example with `HDFView <https://www.hdfgroup.org/products/java/hdfview/>`__).

Please refer to the *phconvert notebook* for examples on how to use phconvert to
create Photon-HDF5 files. If something is not clear please ask questions
by opening an `issue on GitHub <https://github.com/Photon-HDF5/phconvert/issues>`_.


Using third party software
--------------------------

Even if using *phconvert* is the recommended way to save or convert Photon-HDF5
files, some users may prefer saving file directly from their non-python
applications. This approach is needed for non-python acquisition
applications that want to natively save to Photon-HDF5.
Note however that, from some languages like MATLAB (since R2014b),
you can directly call python functions.
In this case it is advisable to call the phconvert functions
instead of coding new routines to save Photon-HDF5 files. This will both
save development time and assure compatibility of the generated files.

Note that the user wanting to code writing routines from scratch needs to use
the HDF5 library for the language of choice and needs to be able to create HDF5 files.

To facilitate this task, we provide
`a JSON file <https://github.com/Photon-HDF5/phconvert/blob/master/phconvert/specs/photon-hdf5_specs.json>`_
containing all the official field names, a short description and a generic
type definition (array, scalar, string or group).
This JSON file can be used both to
validate names and types of the data fields and to retrieve the standard short description.
The description string should be saved for all the official fields in
an attribute named "TITLE". For compatibility with h5labview, we recommend to 
use a single-space string (" ") for all the user fields that lack a description
(phconvert uses this workaround too).

Furthermore, the :ref:`/identity group <identity_group>` should include
the fields ``software_name`` and ``software_version`` to specify the name
and the version of the software that created the file.

Finally, you can verify that the generate files are compliant with the
Photon-HDF5 specifications by using the *phconvert* function 
``phconvert.hdf5.assert_valid_photon_hdf5_tables()``. This function will 
raise errors or warnings if the input file does not follows the specs.
