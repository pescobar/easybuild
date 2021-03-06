.. _using_external_modules:

Using external modules
======================

Since EasyBuild v2.1, support for using modules that were not installed via EasyBuild is available.
We refer to such modules as *external modules*.

This can be useful to reuse vendor-supplied modules, for example on Cray systems.


Using external modules as dependencies
---------------------------------------

External modules can be used as dependencies, by including the module name in the ``dependencies`` list (see
:ref:`dependency_specs`), along with the ``EXTERNAL_MODULE`` constant marker.

For example, to specify the readily available module ``fftw/3.3.4.2`` as a dependency::

  dependencies = [('fftw/3.3.4.2', EXTERNAL_MODULE)]

For such dependencies, EasyBuild will:

* load the module before initiating the software build and install procedure
* include a '``module load``' statement in the generated module file (for non-build dependencies)

If the specified module is not available, EasyBuild will exit with an error message stating that the dependency can
not be resolved because the module could not be found. It will *not* search for a matching easyconfig file in order to
try and install the module to resolve the dependency.


Metadata for external modules
-----------------------------

Since very little information is available for external modules based on the dependency specification alone (i.e., only
the module name), metadata can be supplied to EasyBuild for external modules.

Using the ``--external-modules-metadata`` configuration option, the location of one or more files can be specified that
provide such metadata. The files are expected to be in INI format, with a section per module name and key-value
assignments denoting the metadata specific to that module.

Format::

  [modulename]
  key1 = value1
  key2 = value2, value3

For example, the following file provides metadata for a handful of modules that may be provided on Cray systems::
 
  [cray-hdf5/1.8.13]
  name = HDF5
  version = 1.8.13
  prefix = HDF5_DIR

  [cray-hdf5-parallel/1.8.13]
  name = HDF5
  version = 1.8.13
  prefix = /opt/cray/hdf5-parallel/1.8.13/GNU/49

  [cray-netcdf/4.3.2]
  name = netCDF, netCDF-Fortran
  version = 4.3.2, 4.3.2
  prefix = NETCDF_DIR


The following keys are supported:

* ``name``: specifies the software name(s) that is (are) provided by the module
* ``version``: specifies the software version(s) that is (are) provided by the module
* ``prefix``: specifies the installation prefix of the software that is provided by the module, or the name of an
  environment variable defined by the module that contains the installation prefix
 
Any other keys are simply ignored.

.. note::
  When both ``name`` and ``version`` are specified, they must provide an *equal number of values*
  (see for example the ``cray-netcdf`` example above).

Using the provided metadata, EasyBuild will define environment variables that are also defined by modules that are
generated by EasyBuild itself. In particular, for each software name that is specified:

* the corresponding environment variable ``$EBROOT<NAME>`` is defined to the specified ``prefix`` value (if any)
* the corresponding environment variable ``$EBVERSION<NAME>`` is defined to the corresponding ``version`` value (if any)

For example, for the external modules for which metadata is provided in the example above, the following
environment variables are set in the build environment when the module is used as a dependency:

* for ``cray-hdf5/1.8.1.13``:

  * ``$EBROOTHDF5`` = ``$HDF5_DIR``
  * ``$EBVERSIONHDF5`` = ``1.8.13``

* for ``cray-hdf5-parallel/1.8.13``:

  * ``$EBROOTHDF5`` = ``/opt/cray/hdf5-parallel/1.8.13/GNU/49``
  * ``$EBVERSIONHDF5`` = ``1.8.13``

* for ``cray-netcdf/4.3.2``:

  * ``$EBROOTNETCDF`` = ``$NETCDF_DIR``
  * ``$EBROOTNETCDFMINFORTRAN`` = ``$NETCDF_DIR``
  * ``$EBVERSIONNETCDF`` = ``4.3.2``
  * ``$EBVERSIONNETCDFMINFORTRAN`` = ``4.3.2``

The ``get_software_root`` and ``get_software_version`` functions that are commonly used occasionally in easyblocks
pick up the ``$EBROOT*`` and ``$EBVERSION*`` environment variables, respectively.
