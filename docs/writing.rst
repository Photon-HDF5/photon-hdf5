.. _writing:

Writing Photon-HDF5 files
=========================

The recommended way to write (or convert) Photon-HDF5 files is by using the
`phconvert <https://github.com/Photon-HDF5/phconvert>`_.
`phconvert` is a python library that can be used as a stand-alone program 
by executing one of the provided 
`Jupyter notebooks <https://github.com/Photon-HDF5/phconvert/tree/master/notebooks>`_.
Phconvert's MIT license is compatible with inclusion in both open and 
closed source software.

Some users may want to save Photon-HDF5 files from an acquisition software 
not written in python (as opposed to converting the files
later on). Also this use case is illustrated below.

From Python, or stand-alone converter
-------------------------------------

Python users, or any user wanting a stand-alone application to convert a file 
to Photon-HDF5, can use ``phconvert``. Instructions on the
`phconvert homepage <http://photon-hdf5.github.io/phconvert/>`_ should be 
detailed enough to guide new users in setting-up the Jupyter Notebook
and running a notebook to perform the conversion. If something is not 
clear, please open an `phconvert Issue <https://github.com/Photon-HDF5/phconvert/issues>`_
or ask on `Photon-HDF5 mailing list <https://groups.google.com/forum/#!forum/photon-hdf5>`_.
The `Jupyter notebooks <https://github.com/Photon-HDF5/phconvert/tree/master/notebooks>`_
included with phconvert also serve as examples on how to use the library, in case
you want to import it in other applications.

If your input file format is not supported by *phconvert* please open a
`new issue <https://github.com/Photon-HDF5/phconvert/issues>`__ on GitHub for phconvert.
Adding a new file format is quite straightforward
and we want to support as many format as possible out of the box.

One advantage of using phconvert is that files are validated and guaranteed 
to conform to the Photon-HDF5 specifications.
Furthermore, standard description attributes (*TITLE*) are automatically added to every
field making easier for a user to interactively inspect the file
(for example with `HDFView <https://www.hdfgroup.org/products/java/hdfview/>`__).

From MATLAB
-----------

If you are a MATLAB user, the easiest way to create Photon-HDF5 files is calling the 
python library ``phconvert`` (see previous section). In fact, starting from  
version R2014b, MATLAB can easily call python code (see 
`<http://www.mathworks.com/help/matlab/call-python-libraries.html>`__).
This approach will save development time and assure compatibility 
of the generated files with the Photon-HDF5 specifications. 
If you encounter any difficulties please contact us 
(`Photon-HDF5 google group <https://groups.google.com/forum/#!forum/photon-hdf5>`__),
we are committed to fully support this approach.

From other languages
---------------------

The best option is calling the ``phconvert`` python library
from your language of choice. Note that phconvert licence (MIT) is compatible
with both opensource and closed source applications. In principle,
you should be able to call python function at least from from C, Java, R
and MATLAB.
Please `use the mailing list <https://groups.google.com/forum/#!forum/photon-hdf5>`__
if you have any questions about using phconvert from those languages.
We are willing to modify phconvert to maximize its inter-language compatibility.

If calling python is not possible from your language, you have to implement 
routines to write Photon-HDF5 files using the HDF5 library for your platform,
taking care of following the Photon-HDF5 specification.

In the next paragraph we provide a few suggestions on how to implement 
Photon-HDF5 writing routines. For more info or questions
please contact us on the public mailing list 
(`Photon-HDF5 google group <https://groups.google.com/forum/#!forum/photon-hdf5>`__).

To facilitate writing Photon-HDF5, we provide
`a JSON file <https://github.com/Photon-HDF5/phconvert/blob/master/phconvert/specs/photon-hdf5_specs.json>`_
containing all the official field names, a short description and a generic
type definition (array, scalar, string or group).
This JSON file can be used both to validate names and types of the data fields 
and to retrieve the standard short description (this is, in fact, what 
`phconvert` does).
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
