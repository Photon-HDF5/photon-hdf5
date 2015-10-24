.. _writing:

Writing Photon-HDF5 files
=========================

To create Photon-HDF5 files, users can convert existing files or 
save directly from suitably modified acquisition software. The 
conversion option is generally the simplest approach and, when 
using closed-source acquisition software, also the only one available
(until vendors start supporting saving Photon-HDF5 files from their software). 

To simplify saving (and converting) Photon-HDF5 files we maintain, 
`phconvert <http://photon-hdf5.github.io/phconvert/>`_, an open-source 
python library serving as reference implementation for the 
Photon-HDF5 format. While Photon-HDF5 can be created without phconvert 
using only a HDF5 library, we recommend taking advantage of phconvert 
to simplify the writing step and to make sure that the saved file
conforms to the specifications. Phconvert, in fact, checks that all mandatory 
fields are present and have correct names and types, and adds a description 
to each field. Phconvert can be directly used in programs written in Python, 
MATLAB or other languages that allow calling Python code (see next sections).
phconvert permissive license (MIT) allows intergration with both open and 
closed source software.

Converting files to Photon-HDF5
-------------------------------

phconvert includes a browser-based interface using 
`Jupyter Notebooks <http://jupyter.org/>`_ to convert vendor-specific file 
formats into Photon-HDF5 without requiring any python knowledge. 
The formats currently supported are HT3 (from PicoQuant 
TCSPC hardware), SPC/SET (from Becker & Hickl TCSPC hardware) as well as SM 
(a legacy file format developed by the WeissLab, UCLA). 

We provide a `demo service <http://photon-hdf5.github.io/Photon-HDF5-Converter/>`_ 
to run these notebooks online and convert one of these formats to Photon-HDF5 
without software installation on the user’s computer. 

Beyond the currently supported ones, other formats can be converted by 
writing a Python function to load the data and by using phconvert to save 
the data to Photon-HDF5. Taking the 
existing `phconvert loader functions <https://github.com/Photon-HDF5/phconvert/blob/master/phconvert/loader.py>`_ 
as examples, this task is relatively easy even for inexperienced Python programmers. 
We encourage interested users to contribute to phconvert so that 
out-of-the-box support for conversion of the largest number of formats can 
be provided. If you have an input file format not supported by phconvert
please open a `new issue <https://github.com/Photon-HDF5/phconvert/issues>`__ 
on GitHub.


Save Photon-HDF5 from a third party-software
--------------------------------------------

To directly save Photon-HDF5 files from within an acquisition software, 
the amount of work will vary. For acquisition software written in Python or MATLAB, 
phconvert functions can be reused and therefore the amount of effort to adapt 
the acquisition software is minimized. For acquisition software written in LabVIEW, 
there is currently no easy way to call phconvert functions therefore all fields 
and metadata will have to be populated and written from scratch (using h5labview), 
making sure that all mandatory fields are present and in the correct position. 
We are also exploring the possibility to wrap phconvert in a pre-compiled dynamic 
library (e.g. DLL on Microsoft Windows) to facilitate writing valid Photon-HDF5 
from LabVIEW and other programming languages. 

Saving Photon-HDF5 from MATLAB
------------------------------

You can used MATLAB for to convert existing files to Photon-HDF5
or to save data directly from an acquisition software.

In both cases the easiest option is calling phconvert’s save function 
from MATLAB itself. This approach of calling a Python function from MATLAB 
requires a recent MATLAB version (R2014b or later, see (see
`<http://www.mathworks.com/help/matlab/call-python-libraries.html>`__) 
but provides all the benefits of using phconvert (simplify Photon-HDF5 
writing, automatic file validation). 

We are exploring the possibility to create a MEX file to support older
MATLAB versions. If you would like this functionality please contact us.

Saving Photon-HDF5 from scratch using only an HDF5 library
-----------------------------------------------------------

To create Photon-HDF5 files the best option is calling the phconvert
from your language of choice. In principle,
you should be able to call python function at least from from C, Java, R
and MATLAB.
Please `use the mailing list <https://groups.google.com/forum/#!forum/photon-hdf5>`__
if you have any questions about using phconvert from those languages.

If calling python is not possible from your language, you have to implement
routines to write Photon-HDF5 files using the HDF5 library for your platform,
taking care of following the Photon-HDF5 specification.
In the following paragraph we provide a few suggestions on how to proceed
in this case.

To facilitate writing valid Photon-HDF5, we provide
`a JSON file <https://github.com/Photon-HDF5/phconvert/blob/master/phconvert/specs/photon-hdf5_specs.json>`_
containing all the official field names, a short description and a generic
type definition (array, scalar, string or group).
This JSON file can be used both to validate names and types of the data fields
and to retrieve the standard short description (this is, in fact, what
`phconvert` does). The developer needs to verify that all the mandatory fields
are present.
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
