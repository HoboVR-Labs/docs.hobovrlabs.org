An Introduction To Drivers
==========================

SteamVR uses dynamic libraries to load drivers, and as it turns out, all that a dynamic library needs to do to be recognized as a driver, is export *a single function* ``void* HmdDriverFactory(const char *pInterfaceName, int *pReturnCode)``... *And* have a nice driver directory and be built for both 32 and 64 bit instructions.


.. _opevr-driver-directories:

Driver Directory
^^^^^^^^^^^^^^^^

Driver directories have only 1 mandatory component, a ``driver.vrdrivermanifest`` file, which describes how the contents of the driver directory will be used. See `Driver Manifest File`_.

Everything else is technically optional, but you won't have a driver otherwise so yeah.

Driver directories can be either a "resource only" driver, or a fully fledged driver. That is specified in the ``driver.vrdrivermanifest`` file using the ``"resourceOnly"`` bool field. See `Driver Resources`_.

If your driver directory is "resource only", your driver will only have a single sub-directory ``resources``, again see `Driver Resources`_. Here is a few examples of resource only driver directories(pulled from SteamVR):

.. code-block:: bash

    $ tree -L 2 indexhmd/
    indexhmd/
    ├── driver.vrdrivermanifest
    └── resources
        ├── additional.vrsettings
        ├── driver.vrresources
        ├── firmware
        ├── icons
        ├── index_hmd_overrides.bin
        └── input

    4 directories, 4 files

.. code-block:: bash

    $ tree -L 2 indexcontroller/
    indexcontroller/
    ├── driver.vrdrivermanifest
    └── resources
        ├── anims
        ├── driver.vrresources
        ├── firmware
        ├── icons
        ├── input
        ├── localization
        └── rendermodels

    7 directories, 2 files

.. code-block:: bash

    $ tree -L 2 tundra_labs/
    tundra_labs/
    ├── driver.vrdrivermanifest
    └── resources
        ├── driver.vrresources
        ├── firmware
        ├── icons
        ├── input
        ├── localization
        └── rendermodels

    6 directories, 2 files

However if your driver directory is indented for an actual driver, you'll need to add sub-directory ``bin``. ``bin`` stores the compiled driver, however SteamVR expects you to put architecture specific versions of your compiled driver into platform specific sub-directories. Resources are still allowed, and function exactly as they do in the "resource only" case. Here is a few examples of resource only driver directories(pulled from SteamVR):

.. code-block:: bash

    $ tree -L 2 lighthouse/
    lighthouse/
    ├── bin
    │    ├── linux32
    │    ├── linux64
    │    ├── win32
    │    └── win64
    ├── driver.vrdrivermanifest
    └── resources
        ├── anims
        ├── driver.vrresources
        ├── firmware
        ├── icons
        ├── lighthouse_scale.json
        ├── settings
        ├── webhelperoverlays.json
        └── webinterface

    9 directories, 4 files


.. code-block:: bash

    $ tree -L 2 gamepad/
    gamepad/
    ├── bin
    │    ├── linux32
    │    ├── linux64
    │    ├── win32
    │    └── win64
    ├── driver.vrdrivermanifest
    └── resources
        ├── icons
        ├── input
        └── localization

    7 directories, 1 file


.. code-block:: bash

    $ tree -L 2 null/
    null/
    ├── bin
    │    ├── linux32
    │    ├── linux64
    │    ├── win32
    │    └── win64
    ├── driver.vrdrivermanifest
    └── resources
        └── settings

    5 directories, 1 file


.. code-block:: bash

    $ tree -L 2 hobovr/
    hobovr/
    ├── bin
    │    ├── linux32
    │    ├── linux64
    │    ├── win32
    │    └── win64
    ├── driver.vrdrivermanifest
    └── resources
        ├── driver.vrresources
        ├── icons
        ├── input
        ├── rendermodels
        └── settings

    10 directories, 2 files



Driver Manifest File
^^^^^^^^^^^^^^^^^^^^^

This file determines how the driver be recognized by SteamVR, it needs to be placed in the root of your driver directory and have this name: ``driver.vrdrivermanifest``.

The contents of this file need to look like this:

.. code-block:: json

    {
        "alwaysActivate": false,
        "name" : "example",
        "directory" : "",
        "resourceOnly" : false,
        "hmd_presence" :
        [
            "*.*"
        ],

        "other_presence" : []
    }

And here is what it does:

    * ``"name"``: str - Globally unique name of the driver, this also sets the driver binary name ``driver_<name>.dll`` or ``driver_<name>.so``. The driver name is the name of the directory that contains the ``driver.vrdrivermanifest`` file if it is not specified.

    * ``"alwaysActive"``: bool - If this is true this driver will be activated even if the active HMD is from another driver. Defaults to false.

    * ``"directory"``: str - The name of the directory that contains the rest of the driver files. If this is a relative path it is relative to the directory that contains ``driver.vrdrivermanifest``. Defaults to the full path contains ``driver.vrdrivermanifest`` (literally never seen it used).

    * ``"resourceOnly"``: bool - Set this to true if your driver directory only contains resources.

    * ``"hmd_presence"``: list of str - This is an array of strings that identify the USB VID and PID combinations that indicate an HMD from this driver is probably present. Each entry should be hex values, separated by a period (``*`` means any): ``"<vid>.<pid>"``.

    * ``"other_presence"``: list of str - An array of strings in the same format as hmd_presence that indicates that there is a non-HMD device plugged in (optional, never seen it used).

Driver Resources
^^^^^^^^^^^^^^^^