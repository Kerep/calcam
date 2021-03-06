====================
Installation & Setup
====================

Dependencies
-------------
Calcam is known to work under Python versions from 2.7 -  3.6, and probably works with newer versions but has not been explicitly tested. In addition to NumPy and SciPy, it also requires the following less standard Python packages to be installed:

	- OpenCV (cv2) 2.4+ [Some features only available with 3.0+]
	- PyQt4 or PyQt5
	- VTK 6.0+ [Tested with versions up to 8.1. Must be built with Qt support enabled]
	
These cannot be reliably installed automatically by the setup script on all platforms / environments, so the setup script will merely check if these are working and will issue an error or warning if not. It is highly recommended to get these libraries installed and working before installing Calcam. The easiest way to install them is usually through your OS's package manager, if applicable, or to use a Python distibution such as `Enthought Canopy <https://www.enthought.com/product/canopy/>`_ or `Python (x,y) <https://python-xy.github.io/>`_. which can provide these packages. It can be very tricky to build these libraries and their python bindings from source and get everything working properly together.

Note: If you only need to work with already created calibration results, it is possible to use some parts of Calcam, e.g. the :class:`calcam.Calibration` class, without having VTK available.


Download & Installation
-----------------------
If you have Git available, the very latest version of the code can be cloned from the GitHub repository::

	git clone https://github.com/euratom-software/calcam.git

Alternatively, the latest release version of the code can be downloaded from: `<https://github.com/euratom-software/calcam/releases>`_ .

Once you have the calcam repository files on your computer, the package is installed using the included ``setuptools`` setup script::

	python setup.py install

This will copy Calcam to the appropriate Python library path and create a launcher script for the Calcam GUI. If installing on a system where you do not have the relevant admin/root permissions to install python packages globally, adding the ``--user`` switch to the above command will install the package under your user account specifically.

After the setup script is finished you can delete the downloaded calcam files, should you want.

**Note for Windows users:** If installing Calcam on Windows, the setup script will finish by printing the location of the GUI launcher ``calcam.exe`` which is created during the installation. It is recommended to make a shortcut to this in a place of your choosing for easy access to the Calcam GUI.

Installing in Development mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If you want to be able to edit the Calcam source code without having to run the install again to have your changes go live, you can use the below alternative command to install calcam in development mode::

	python setup.py develop

In this case the copy of the code you are installing from remains the "live" version and can be used for development. Again, the ``--user`` switch can be added to install in the current user's library path rather than the system one.


CAD Models and Image Sources
----------------------------

Setting up CAD Model Definitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Camera calibration in Calcam is based on matching features between camera images and a CAD model of the scene viewed by the camera. As such, it is necessary to define one or more CAD models for use in calcam. The current version supports importing ``.stl`` or ``.obj`` format 3D mesh files. It's usually convenient to split the model in to several individual mesh files containing different parts of the scene, and these can then be turned on or off individually when working with the model. Calcam packages these mesh files in to a custom zipped file format (.ccm) along with various metadata to create a Calcam CAD model file. You can have several such files and easily switch between them at any time. It is recommended to read the :ref:`cadmodel_intro` section in concepts and conventions, then consult the user guide for the :doc:`gui_settings` interface for details of how to set up CAD model definitions.

Setting up custom image sources (optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
As standard, Calcam can load camera images from most common image file formats. If desired, you can set up additional custom "image sources", which are user-defined Python modules for loading camera images in to Calcam. For example you may want to load camera data directly from a central data server, or read images from an unusual file format. This can be done by writing a small python module which plugs in to calcam and handles the image loading. A full guide to writing such modules can be found in the :doc:`dev_imsources` developer documentation page. Once written, they can be added to Calcam with the :doc:`gui_settings` interface.



Upgrading from Calcam 1.x
--------------------------
The update from Calcam 1.x to Calcam 2 includes large overhauls to the file formats, file storage conventions and Python API. This section covers the main things users need to know when upgrading from Calcam 1.x to Calcam 2.

File Storage
~~~~~~~~~~~~
In Calcam 1, CAD model definitions, other user-defined code, calibration input and results files were stored in a pre-prescribed directory structure. In Calcam 2 this is no longer the case; these files can be stored wherever you want and are opened either by graphical file browsing in the Calcam GUI or by file path in the Calcam API. The main change required to code calling Calcam to accommodate this will be that calibration results will now need to be loaded by supplying the relative or full path to the results file, rather than just the identifying name as before.

File Formats
~~~~~~~~~~~~
Whereas in Calcam 1, imported images, point pairs, calibration and fit results were all stored in separate files, in Calcam 2 all of these elements are stored together as a calibration. This is to maintain better traceability of calcam calibrations and make it easier for users to share data. Except for `.csv` point pair files, Calcam 2 is not backwards compatible with Calcam 1 files, therefore to use existing data from Calcam 1 you must convert your Calcam 1 data to the new Calcam 2 formats. This can be done in bulk using the file converter utility provided in the ``calcam1_file_converter`` directory of the calcam 2 repo. Running ``convert_files.py`` from this directory as a script will open the tool, which is shown below:

.. image:: images/screenshots/file_converter.png
   :alt: Calcam 1.x file converter screenshot

At the top of this window, the "Source Directory", where the tool will look for Calcam 1.x files to convert, is displayed. This is typically detected automatically, but you can also manually set the source directory manually using the :guilabel:`Browse...` button (this should be the complete Calcam 1.x data directory, i.e. the location of the `FitResults`, `Images`, `PointPairs` etc directories). 

Below this are 2 main sections: the top section for converting existing calibrations, and the bottom section for converting existing CAD model definitions. When the :guilabel:`Convert!` button is clicked in the relevant section, the large status bar at the bottom of the window will show the current progress during the conversion. The three text boxes containing file paths are used to specify where the output Calcam 2 calibration files should be saved to, since in Calcam 2 this can be wherever you want.

When converting calibrations, if the :guilabel:`Try to match with image files based on name` checkbox is ticked, the tool will try to match up calibration results with images by looking for Calcam image save files whose name also appears in the name of the calibration result being converted. If such an image is found, the image will be added to the resulting Calcam 2 save file. To disable this auto-matching, un-tick this checkbox, and Calcam 2 calibration results converted from Calcam 1 files will simply not contain any images.

**Note:** the conversion process does not alter or remove any of the original Calcam 1 data, so if anything goes wrong and you have to, or want to, go back to using Calcam 1.x, the data will still be intact, and it is left to the user to remove the old Calcam 1 data when you feel sufficiently comfortable to do so.


API Changes
~~~~~~~~~~~
The change from Calcam 1 to Calcam 2 includes several compatibility breaking API changes. The main changes to the API are:

* The old :class:`calcam.CalibResults` class has been superceded by the new :class:`calcam.Calibration` class. This maintains the methods for working with calibration results which existed in :class:`calcam.CalibResults`, with the addition that :class:`calcam.Calibration` now contains data on the entire calibration process: image, point pairs, fit results and metadata. 

* The old :class:`calcam.VirtualCalib` class has been removed: virtual calibration results are now represented by the new :class:`calcam.Calibration` class, meaning all types of calibration use the same class in Calcam 2.

* The :class:`RayCaster` class has been removed. This is because although more functionality was originally envisaged for this class, that additional functionality is no longer planned for Calcam and therefore only a single method of this class was ever useful. In addition, the important element of this class' state was already being held by other objects. The functionality of the :class:`RayCaster` class has been moved to the function :func:`calcam.raycast_sightlines()`

* Naming conventions: throughout the API (except for in the geometry matrix module which will updated in a future point release), argument or function names which previously used capital letters and PascalCase or camelCase have been changed to lowercase with underscores.

For more information, see the full API documentation in :doc:`api_analysis` and the :doc:`api_examples` .