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

Users that want to convert existing files to Photon-HDF5 are strongly
encouraged to use phconvert. Even if phconvert is a python library, no python
knowledge is strictly required as users can simply follows the steps
illustrated in the *phconvert notebooks* (`repository <https://github.com/Photon-HDF5/phconvert/tree/master/notebooks>`__,
`see on-line <http://nbviewer.ipython.org/github/Photon-HDF5/phconvert/tree/master/notebooks/>`__).
For the absolute python beginner,
we recommend `this page <http://fretbursts.readthedocs.org/en/latest/absolute_beginner.html>`__
with installation and first-run instructions
(these instructions are for the FRETBursts software but they are also valid
for phconvert).

If your input file format is not supported by *phconvert* please open a
`new issue <https://github.com/Photon-HDF5/phconvert/issues>`__ on GitHub for phconvert.
Adding a new file format is quite straightforward
and we want to support as many format as possible out of the box.

When saving, *phconvert* performs a validation the user input
therefore assuring that the generated file is a valid Photon-HDF5 file.
Furthermore a description attribute (*TITLE*) is automatically added to every
field making easier for a user to interactively inspect the file
(for example with `HDFView <https://www.hdfgroup.org/products/java/hdfview/>`__).

Please refer to the *phconvert notebook* for examples on how to use phconvert to
create Photon-HDF5 files. If something is not clear you can ask questions
by opening an issue on GitHub.


Using third party software
--------------------------

Even if using *phconvert* is the recommended way to save or convert Photon-HDF5
files, some users may prefer saving file directly from their non-python
applications. This is the only approach for non-python acquisition
applications that want to natively save to Photon-HDF5.

Note that in this case, the user needs to use a HDF5 library for the language
of choice and needs to be able to create HDF5 files.

To facilitate this task, we provide a JSON file containing all the official
fields names and their description. This JSON file can be used both to
validate the names of the fields and to retrieve a standard short description.
The description string should be saved for all the official fields in
an attribute named "TITLE".

Furthermore, the :ref:`/identity group <identity_group>` should include
the fields ``software_name`` and ``software_version`` to specify the name
and the version of the software that created the file.

Finally, please make sure that the generate files are compliant with the
Photon-HDF5 specifications by loading the file with *phconvert*
(using ``phconvert.hdf5.load_photon_hdf5()``). This function will raise errors
or warnings if the files does not follows the specs.
