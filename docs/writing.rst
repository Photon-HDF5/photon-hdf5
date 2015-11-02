.. _writing:

Writing Photon-HDF5 files
=========================

To create Photon-HDF5 files, users can convert existing files or 
save directly from suitably modified acquisition software. The 
conversion option is generally the simplest approach and, when 
using closed-source acquisition software, also the only one available
(until vendors start supporting Photon-HDF5). 

To simplify saving (and converting) Photon-HDF5 files we developed and maintain, 
`phconvert <http://photon-hdf5.github.io/phconvert/>`_, an open-source 
python library serving as reference implementation for the 
Photon-HDF5 format. While Photon-HDF5 
:ref:`can be created without phconvert <writing_from_scratch>`, 
using only a HDF5 library, we recommend taking advantage of phconvert 
to simplify the writing step and to make sure that the saved file
conforms to the specifications. Phconvert, in fact, checks that all mandatory 
fields are present and have correct names and types, and adds a description 
to each field. Phconvert can be directly used in programs written in Python
or other languages that allow calling Python code (see next sections).
phconvert permissive license (MIT) allows integration with both open and 
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
See also the notebook 
`Writing Photon-HDF5 files <https://github.com/Photon-HDF5/phconvert/blob/master/notebooks/Writing%20Photon-HDF5%20files.ipynb>`_
(`view online <http://nbviewer.ipython.org/github/Photon-HDF5/phconvert/blob/master/notebooks/Writing%20Photon-HDF5%20files.ipynb>`_).

We encourage interested users to contribute to phconvert so that 
out-of-the-box support for conversion of the largest number of formats can 
be provided. If you have an input file format not supported by phconvert
please open a `new issue <https://github.com/Photon-HDF5/phconvert/issues>`__ 
on GitHub.

.. _save_photon_hdf5_script:

Save Photon-HDF5 from a third party-software
--------------------------------------------

To directly save Photon-HDF5 files from within an acquisition software, 
there are several options. For programs written in Python, the obvious option
is using phconvert which makes simple creating Photon-HDF5 files while
assuring the validity of the output file. See for example the notebook 
`Writing Photon-HDF5 files <https://github.com/Photon-HDF5/phconvert/blob/master/notebooks/Writing%20Photon-HDF5%20files.ipynb>`_
(`view online <http://nbviewer.ipython.org/github/Photon-HDF5/phconvert/blob/master/notebooks/Writing%20Photon-HDF5%20files.ipynb>`_).

For acquisition software written in other languages(e.g. C, MATLAB or LabVIEW), 
it is in principle possible to call python using the [Python C API](https://docs.python.org/3.4/c-api/index.html#c-api-index) 
(see `Embedding Python in Another Application <https://docs.python.org/3.4/extending/embedding.html>`__).
However understanding the Python C API requires a fairly good proficiency in C 
(and probably python).

In order to make it easy to create valid Photon-HDF5 in any language 
(without duplicating the effort of creating a library like phconvert 
in every language) we devised an alternative approach. The user can 
save the :ref:`photon-data arrays <photon_data_group>`(timestamps, detectors, nanotimes, etc…) 
in a plain HDF5 file. The remaining metadata is written in a simple 
text file (`YAML <https://en.wikipedia.org/wiki/YAML>`__). Next, a script called 
`phforge <http://photon-hdf5.github.io/phforge/>`__ reads the metadata and 
the photon-data arrays and creates a valid Photon-HDF5 file using phconvert. 
In this way, at the cost of a small inefficiency (writing some 
temporary files), a user can easily and reliably generate Photon-HDF5 
files from any language.
The metadata file is a text-based representation of the full Photon-HDF5
structure, excluding the photon-data arrays and some other field 
automatically filled by phconvert. To store this metadata we use YAML markup 
(a superset of JSON) for its simplicity and ability to describe hierarchical 
structures. For example, a minimal metadata file describing only mandatory
fields is the following::

    description: This is a dummy dataset which mimics smFRET data.
 
    setup:
        num_pixels: 2                # using 2 detectors
        num_spots: 1                 # a single confocal excitation
        num_spectral_ch: 2           # donor and acceptor detection
        num_polarization_ch: 1       # no polarization selection
        num_split_ch: 1              # no beam splitter
        modulated_excitation: False  # CW excitation, no modulation
        lifetime: False              # no TCSPC in detection
    
    photon_data:
        timestamps_specs:
            timestamps_unit: !!float 10e-9  # 10 ns

To save the photon-data arrays the user needs to call the HDF5 library 
for the language of choice. For example, in MATLAB timestamps and detectors 
arrays can be saved with the following commands:

::
    h5create()
    h5write()
    h5create()
    h5write()

Finally, once metadata and photon-data files have been saved, a Photon-HDF5 
file can be created calling the phforge script as follows::

    phforge metadata.yaml photon-data-arrays.h5 photon-hdf5-output.hdf5

Note that the file generate with this minimal metadata, does not contain 
the :ref:`measurement_specs group <measurement_specs_group>` which is 
in general necessary for a user to analyze the data.  

The phforge script is available at http://photon-hdf5.github.io/phforge/.
More examples of metadata files, including non mandatory fields 
and measurement_specs group, are available at
https://github.com/Photon-HDF5/phforge/tree/master/example_data.

Please `use the mailing list <https://groups.google.com/forum/#!forum/photon-hdf5>`__
if you have any questions.

Saving Photon-HDF5 from MATLAB
------------------------------

Creating Photon-HDF5 in MATLAB is easy using the approach described in the
previous section, i.e. calling the script `phforge <http://photon-hdf5.github.io/phforge/>`__.

Complete MATLAB examples can be found at http://photon-hdf5.github.io/photon-hdf5-matlab-write/.

In principle, it should be possible using a recent release of MATLAB (R2014b or later) to 
`directly call python functions <http://www.mathworks.com/help/matlab/call-python-libraries.html>`__. 
Therefore it should be possible to directly call phconvert.
However, in our recent attempt, we weren't able to configure MATLAB in order 
to load the correct dynamic libraries (i.e. the HDF5 C library) required by phconvert.

.. _writing_from_scratch:

Saving Photon-HDF5 from scratch using only an HDF5 library
-----------------------------------------------------------

To create Photon-HDF5 files from languages different than python
the easiest option, by far, is calling the phforge script
as described in previous section :ref:`save_photon_hdf5_script`.

If for some reason you cannot use phforge or phconvert, you have to implement
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

Finally, you can verify that generate files are compliant with the
Photon-HDF5 specifications by using the *phconvert* function
`phconvert.hdf5.assert_valid_photon_hdf5_tables() <http://phconvert.readthedocs.org/en/latest/hdf5.html#phconvert.hdf5.assert_valid_photon_hdf5>`_. 
This function will raise errors or warnings if the input file does not follows the specs.


