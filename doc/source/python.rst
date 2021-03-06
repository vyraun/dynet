Installing the Python DyNet module.
===================================

(for instructions on installing on a computer with GPU, see below)

Python bindings to DyNet are currently only supported under python 2.

TL;DR
-----

(see below for the details)

.. code:: bash

    # Installing python DyNet on a machine with python 2.7:

    pip install cython # if you don't have it already.
    mkdir dynet-base
    cd dynet-base
    # getting dynet and eigen
    git clone https://github.com/clab/dynet.git
    hg clone https://bitbucket.org/eigen/eigen
    cd dynet
    mkdir build
    cd build
    # without GPU support:
    cmake .. -DEIGEN3_INCLUDE_DIR=../eigen -DPYTHON=`which python`
    # or with GPU support:
    cmake .. -DEIGEN3_INCLUDE_DIR=../eigen -DPYTHON=`which python` -DBACKEND=cuda

    make -j 2 # replace 2 with the number of available cores
    cd python
    python setup.py install  # or `python setup.py install --user` for a user-local install.

Detailed Instructions
---------------------

First, get DyNet:

.. code:: bash

    cd $HOME
    mkdir dynet-base
    cd dynet-base
    git clone https://github.com/clab/dynet.git
    cd dynet
    git submodule init # To be consistent with DyNet's installation instructions.
    git submodule update # To be consistent with DyNet's installation instructions.

Then get Eigen:

.. code:: bash

    cd $HOME
    cd dynet-base
    hg clone https://bitbucket.org/eigen/eigen/

We also need to make sure the ``cython`` module is installed. (you can
replace ``pip`` with your favorite package manager, such as ``conda``,
or install within a virtual environment)

.. code:: bash

    pip install cython

To simplify the following steps, we can set a bash variable to hold
where we have saved the main directories of DyNet and Eigen. In case you
have gotten DyNet and Eigen differently from the instructions above and
saved them in different location(s), these variables will be helpful:

.. code:: bash

    PATH_TO_DYNET=$HOME/dynet-base/dynet/
    PATH_TO_EIGEN=$HOME/dynet-base/eigen/

Compile DyNet.

This is pretty much the same process as compiling DyNet, with the
addition of the ``-DPYTHON=`` flag, pointing to the location of your
python interpreter.

If boost is installed in a non-standard location, you should add the
corresponding flags to the ``cmake`` commandline, see the `DyNet
installation instructions page <install.md>`__.

.. code:: bash

    cd $PATH_TO_DYNET
    PATH_TO_PYTHON=`which python`
    mkdir build
    cd build
    cmake .. -DEIGEN3_INCLUDE_DIR=$PATH_TO_EIGEN -DPYTHON=$PATH_TO_PYTHON
    make -j 2

Assuming that the ``cmake`` command found all the needed libraries and
didn't fail, the ``make`` command will take a while, and compile dynet
as well as the python bindings. You can change ``make -j 2`` to a higher
number, depending on the available cores you want to use while
compiling.

You now have a working python binding inside of ``build/dynet``. To
verify this is working:

.. code:: bash

    cd $PATH_TO_DYNET/build/python
    python

then, within python:

.. code:: bash

    import dynet as dy
    print dy.__version__
    model = dy.Model()

In order to install the module so that it is accessible from everywhere
in the system, run the following:

.. code:: bash

    cd $PATH_TO_DYNET/build/python
    python setup.py install --user

(the ``--user`` switch will install the module in your local
site-packages, and works without root privilages. To install the module
to the system site-packages (for all users), run
``python setup.py install`` without this switch)

You should now have a working python binding (the dynet module).

Note however that the installation relies on the compiled dynet library
being in ``$PATH_TO_DYNET/build/dynet``, so make sure not to move it
from there.

Now, check that everything works:

.. code:: bash

    # check that it works:
    cd $PATH_TO_DYNET
    cd pyexamples
    python xor.py
    python rnnlm.py rnnlm.py

Alternatively, if the following script works for you, then your
installation is likely to be working:

::

    from dynet import *
    model = Model()

GPU/MKL Support
---------------

Installing/running on GPU
~~~~~~~~~~~~~~~~~~~~~~~~~

For installing on a computer with GPU, first install CUDA. The following
instructions assume CUDA is installed.

The installation process is pretty much the same, while adding the
``-DBACKEND=cuda`` flag to the ``cmake`` stage:

.. code:: bash

    cmake .. -DEIGEN3_INCLUDE_DIR=$PATH_TO_EIGEN -DPYTHON=$PATH_TO_PYTHON -DBACKEND=cuda

(if CUDA is installed in a non-standard location and ``cmake`` cannot
find it, you can specify also
``-DCUDA_TOOLKIT_ROOT_DIR=/path/to/cuda``.)

Now, build the python modules (as above, we assume cython is installed):

After running ``make -j 2``, you should have the files ``_dynet.so`` and
``_gdynet.so`` in the ``build/python`` folder.

As before, ``cd build/python`` followed by
``python setup.py install --user`` will install the module.

In order to use the GPU support, you can either:

-  Use ``import _gdynet as dy`` instead of ``import dynet as dy``
-  Or, (preferred), ``import dynet`` as usual, but use the commandline
   switch ``--dynet-gpu`` or the GPU switches detailed
   `here <commandline.md>`__ when invoking the program. This option lets
   the same code work with either the GPU or the CPU version depending
   on how it is invoked.


Running with MKL
~~~~~~~~~~~~~~~~

If you've built dynet to use MKL (using -DMKL or -DMKL_ROOT), python sometimes has difficulty finding the MKL shared libraries. You can try setting LD_LIBRARY_PATH to point to your MKL library directory. If that doesn't work, try setting the following environment variable (supposing, for example, your MKL libraries are located at /opt/intel/mkl/lib/intel64):

.. code:: bash

    export LD_PRELOAD=/opt/intel/mkl/lib/intel64/libmkl_def.so:/opt/intel/mkl/lib/intel64/libmkl_avx2.so:/opt/intel/mkl/lib/intel64/libmkl_core.so:/opt/intel/mkl/lib/intel64/libmkl_intel_lp64.so:/opt/intel/mkl/lib/intel64/libmkl_intel_thread.so:/opt/intel/lib/intel64_lin/libiomp5.so


