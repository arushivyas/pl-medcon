pl-medcon
================================

.. image:: https://badge.fury.io/py/medcon.svg
    :target: https://badge.fury.io/py/medcon

.. image:: https://travis-ci.org/FNNDSC/medcon.svg?branch=master
    :target: https://travis-ci.org/FNNDSC/medcon

.. image:: https://img.shields.io/badge/python-3.5%2B-blue.svg
    :target: https://badge.fury.io/py/pl-medcon

.. contents:: Table of Contents


.. code:: 

             _                           _                  
            | |                         | |                 
       _ __ | |______ _ __ ___   ___  __| | ___  ___  _ __  
      | '_ \| |______| '_ ` _ \ / _ \/ _` |/ __|/ _ \| '_ \ 
      | |_) | |      | | | | | |  __/ (_| | (__| (_) | | | |
      | .__/|_|      |_| |_| |_|\___|\__,_|\___|\___/|_| |_|
      | |                                                   
      |_|                                                   


Abstract
--------

An app to covert NIfTI volumes to DICOM files. This is a ChRIS conformant "DS" (Data Synthesis) plugin that wraps around the `medcon` package and provides a thin shim about that executable.


Synopsis
--------

.. code::
 
        python medcon.py                                                \
             -i|--inputFile <inputFile>                                 \
            [-a|--args 'ARGS: <argsToPassTo_medcon>']                   \
            [--do <macro>]                                              \
            [-h] [--help]                                               \
            [--json]                                                    \
            [--man]                                                     \
            [--meta]                                                    \
            [--savejson <DIR>]                                          \
            [-v <level>] [--verbosity <level>]                          \
            [--version]                                                 \
            <inputDir>                                                  \
            <outputDir>

Description
-----------

``medcon.py`` coverts NIfTI volumes to DICOM files. This is a ChRIS
conformant "DS" (Data Synthesis) plugin that wraps around the
medcon package and provides a thin shim about that executable. Using
the ``[--args 'ARGS: <args>']`` CLI, a user can pass any additional 
arbitrary arguments to the underlying `medcon`.

If running this application directly, i.e. outside of its 
docker container, please make sure that the `medcon` application
is installed in the host system. On Ubuntu, this is typically:


.. code::
                    
    sudo apt install medcon

and also make sure that you are in an appropriate python virtual
environment with necessary requirements already installed 
(see the ``requirements.txt`` file).

Please note, however, that running this application from its
docker container is the preferred method and the one documented
here.


Arguments
---------

.. code::

         -i|--inputFile <inputFile>
        Input file to process. This file exists within the explictly provided 
        CLI positional <inputDir>.

        [-a|--args 'ARGS: <argsToPassTo_medcon>']
        Optional string of additional arguments to "pass through" to medcon.

        All the args for medcon are themselves specified at the plugin level
        with this flag. These args MUST be contained within single quotes
        (to protect them from the shell) and the quoted string MUST start with
         the required keyword 'ARGS: '.

        [--do <macro>]
        Optional argument to provide a "macro" type functionality. Using this 
        argument will add the correct underlying arguments to the internal 
        `medcon` binary.

        Currently available:

	        - 'nifti2dicom' : this will silently add the args 
                              '-c dicom -split3d'

        [-h] [--help]
        If specified, show help message and exit.

        [--json]
        If specified, show json representation of app and exit.

        [--man]
        If specified, print (this) man page and exit.

        [--meta]
        If specified, print plugin meta data and exit.

        [--savejson <DIR>]
        If specified, save json representation file to DIR and exit.

        [-v <level>] [--verbosity <level>]
        Verbosity level for app. Not used currently.

        [--version]
        If specified, print version number and exit.

Run
----

While ``pl-medcon`` is meant to be run as a containerized docker image, typcially within ChRIS, it is quite possible to run the dockerized plugin directly from the command line as well. The following instructions are meant to be a psuedo- ``jupyter-notebook`` inspired style where if you follow along and copy/paste into a terminal you should be able to run all the examples.

First, let's create a directory, say ``devel`` wherever you feel like it. We will place some test data in this directory to process with this plugin.

.. code:: bash

    cd ~/
    mkdir devel
    cd devel
    export DEVEL=$(pwd)

Now, we need to fetch sample NIfTI data. 

Pull NIfTI
~~~~~~~~~~

- We provide a sample directory of a .nii volume here. (https://github.com/FNNDSC/SAG-anon-nii.git)

- Clone this repository (``SAG-anon-nii``) to your local computer.

.. code:: bash

    git clone https://github.com/FNNDSC/SAG-anon-nii.git

Make sure the ``SAG-anon-nii`` directory is placed in the devel directory.


Using ``docker run``
~~~~~~~~~~~~~~~~~~~~

To run using ``docker``, be sure to assign an "input" directory to ``/incoming`` and an output directory to ``/outgoing``. *Make sure that the* ``$(pwd)/out`` *directory is world writable!*

- Make sure your current working directory is ``devel``. At this juncture it should contain ``SAG-anon-nii``.

- Create an output directory named ``results`` in ``devel``.

.. code:: bash

    mkdir results && chmod 777 results

- Pull the ``fnndsc/pl-medcon`` image using the following command.

.. code:: bash

    docker pull fnndsc/pl-medcon


Examples
--------

Copy and modify the different commands below as needed

..  code:: bash

    docker run --rm                                                         \
        -v ${DEVEL}/SAG-anon-nii/:/incoming -v ${DEVEL}/results/:/outgoing  \
        fnndsc/pl-medcon medcon.py                                          \
        -i SAG-anon.nii                                                     \
        --do nifti2dicom                                                    \
        /incoming /outgoing

Debug
------

Finally, let's conclude with some quick notes on debugging this plugin. The debugging process is predicated on the idea of mapping a source code directory into an already existing container, thus "shadowing" or "masking" the existing code and overlaying current work directly within the container.

In this manner, one can debug the plugin without needing to continually rebuild the docker image.

So, assuming the same env variables as above, and assuming that you are in the source repo base directory of the plugin code:

.. code:: bash

    git clone https://github.com/FNNDSC/pl-medcon.git
    cd pl-medcon
    docker run --rm -ti                                                     \
           -v $(pwd)/medcon:/usr/src/medcon                                 \
           -v ${DEVEL}/SAG-anon-nii/:/incoming                              \
           -v ${DEVEL}/results/:/outgoing                                   \
           fnndsc/pl-medcon medcon.py                                       \
           -i SAG-anon.nii                                                  \
           --do nifti2dicom                                                 \
           /incoming /outgoing

Of course, adapt the above as needed.
