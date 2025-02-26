Building from source
====================

Have Pip build from source
--------------------------

Useful if the binaries don’t work on your system, or you want to use a newer version of Raylib.

Make sure Raylib is installed and then:

::

   pip3 install --no-binary raylib --upgrade --force-reinstall raylib

Build from source manually
--------------------------

Useful if the Pip build doesn’t work and you want to debug it, or you want to contribute to the
project.

.. attention::
   If the Pip build doesn’t work, please submit a bug. (And if you have
   fixed it, a PR.)

Manual instructions follow, but see also how we actually build the wheels
at https://github.com/electronstudio/raylib-python-cffi/blob/master/.github/workflows/build.yml

Windows manual build
~~~~~~~~~~~~~~~~~~~~

Clone this repo including submodules so you get correct version of
Raylib.

::

   git clone --recurse-submodules https://github.com/electronstudio/raylib-python-cffi

Open Visual C++ command shell.

Fix the symlink that doesnt work on Windows

::

   cd raylib-python-cffi
   copy raylib-c\src\raylib.h raylib\raylib.h

Build and install Raylib from the raylib-c directory.

::

   cd raylib-python-cffi/raylib-c
   mkdir build
   cd build
   cmake -DWITH_PIC=on -DCMAKE_BUILD_TYPE=Release ..
   msbuild raylib.sln /target:raylib /property:Configuration=Release
   copy raylib\Release\raylib.lib ..\..
   cd ..\..



To build a binary wheel distribution:

::

   rmdir /Q /S build
   pip3 install cffi
   pip3 install wheel
   python setup.py bdist_wheel

.. TODO::
   There's a hardcoded path (to the raylib header files) in `raylib/build.py` you will probably need to edit.
   Would be useful if some Windows user could figure out how to auto detect this.


Then install it:

::

   pip3 install dist\raylib-3.7.0-cp37-cp37m-win_amd64.whl

(Note: your wheel’s filename will probably be different than the one
here.)

Linux manual build
~~~~~~~~~~~~~~~~~~~~~~

Clone this repo including submodules so you get correct version of
Raylib.

::

   git clone --recurse-submodules https://github.com/electronstudio/raylib-python-cffi

Build and install Raylib from the raylib-c directory.

.. note::
   You can instead use a different version of Raylib you installed from elsewhere, and it should still
   work!

::

   sudo apt install cmake libasound2-dev mesa-common-dev libx11-dev libxrandr-dev libxi-dev xorg-dev libgl1-mesa-dev libglu1-mesa-dev
   cd raylib-python-cffi/raylib-c
   mkdir build
   cd build
   cmake -DWITH_PIC=on -DCMAKE_BUILD_TYPE=Release ..
   sudo make install

.. note:: Optional: Build the Raylib shared libs, if you plan to use
   ``raylib_dynamic`` binding.

   ::

      rm -rf *
      cmake -DWITH_PIC=on -DBUILD_SHARED_LIBS=on -DCMAKE_BUILD_TYPE=Release ..
      make
      sudo make install

::

   cd ../..


Build

::

   pip3 install cffi
   rm -rf build raylib/_raylib_cffi.*
   python3 raylib/build.py

..  note:: (Optional) To update the Linux dynamic libs (names will be different on other platforms):

    ::

       rm dynamic/raylib/*.so*
       cp -P /usr/local/lib/libraylib.so* dynamic/raylib/

To build a binary wheel distribution:

::

   pip3 install wheel
   python3 setup.py bdist_wheel


Then install it:

::

   pip3 install dist/raylib*.whl

To build a complete set of libs for Python 3.6, 3.7, 3.8 and 3.9:

::

   ./raylib/build_multi.sh

.. warning::
   pypi wont accept Linux packages unless they are built
   ``--plat-name manylinux2014_x86_64`` so on linux please run
   ``./raylib/build_multi_linux.sh`` )

.. TODO::
   Separate the instructions for preparing the dynamic module
   from the instructions for building the static module!



Macos manual build
~~~~~~~~~~~~~~~~~~~~~~

These instructions have been tested on Macos 10.14.

Clone this repo including submodules so you get correct version of
Raylib.

::

   git clone --recurse-submodules https://github.com/electronstudio/raylib-python-cffi

Build and install Raylib from the raylib-c directory.

::

    cd raylib-python-cffi/raylib-c/
    mkdir build
    cd build
    cmake -DWITH_PIC=on -DCMAKE_BUILD_TYPE=Release ..
    make
    sudo make install
    cd ../..


Build and install module.

::

   pip3 install cffi
   rm -rf build raylib/_raylib_cffi.*
   python3 raylib/build.py
   pip3 install wheel
   python3 setup.py install



Raspberry Pi
~~~~~~~~~~~~

The integrated GPU hardware in a Raspberry Pi (“VideoCore”) is rather
idiosyncratic, resulting in a complex set of software options. Probably
the most interesting two options for Raylib applications are:

1. Use the Broadcom proprietary Open GL ES 2.0 drivers, installed by
   Raspbian into ``/opt/vc``. These are 32-bit only, and currently X11
   doesn’t use these for its acceleration, so this is most suitable for
   driving the entire HDMI output from one application with minimal
   overhead (no X11).

2. Use the more recent open-source ``vc4-fkms-v3d`` kernel driver. This
   can run in either 32-bit or 64-bit, and X11 can use these, so using
   X11 is probably the more common choice here.

With option 2, the regular linux install instructions above should
probably work as-is.

For option 1, then also follow the above instructions, but with these
modifications:

-  With ``cmake``, use
   ``cmake -DWITH_PIC=on -DSTATIC=on -DSHARED=on -DPLATFORM='Raspberry Pi' ..``

(See
`here <https://github.com/electronstudio/raylib-python-cffi/issues/31#issuecomment-862078330>`__
for a Raspberry Pi wheel)
