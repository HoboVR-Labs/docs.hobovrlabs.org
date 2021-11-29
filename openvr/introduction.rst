An Introduction to Drivers
==========================

SteamVR uses dynamic libraries to load drivers, and as it turns out, all that a dynamic library needs to do to be recognized as a driver, is export *a single function* ``void* HmdDriverFactory(const char *pInterfaceName, int *pReturnCode)``... *And* have a nice driver directory and be built for both 32 and 64 bit instructions *AND* use very specific interfaces in the driver ABI... In short, its a mess. But it doesn't have to be, hopefully this page helps with that.


.. _opevr-driver-directories:

Driver Directory
^^^^^^^^^^^^^^^^

Driver directories have only 1 mandatory component: a ``driver.vrdrivermanifest`` file, which describes how the contents of the driver directory will be used. See `Driver Manifest File`_.

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

However if your driver directory is indented for an actual driver, you'll need to add sub-directory ``bin``. ``bin`` stores the compiled driver, however SteamVR expects you to put architecture specific versions of your compiled driver into platform specific sub-directories. Resources are still allowed, and function exactly as they do in the "resource only" case. Here are a few examples of resource only driver directories(pulled from SteamVR):

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

This file determines how the driver will be recognized by SteamVR. It needs to be placed in the root of your driver directory and have this name: ``driver.vrdrivermanifest``.

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

Status Icons
------------

First lets talk about something easy and nice, device status icons. If you ever bothered to read the code comments in the official `driver sample <https://github.com/ValveSoftware/openvr/blob/master/samples/driver_sample/driver_sample.cpp#L208>`_, you'd find a rather interesting section that mentions how to configure device status icons outside of driver code using a ``driver.vrresources`` file.

.. code-block:: c

    // Icons can be configured in code or automatically configured by an external file "drivername\resources\driver.vrresources".
    // Icon properties NOT configured in code (post Activate) are then auto-configured by the optional presence of a driver's "drivername\resources\driver.vrresources".
    // In this manner a driver can configure their icons in a flexible data driven fashion by using an external file.
    //
    // The structure of the driver.vrresources file allows a driver to specialize their icons based on their HW.
    // Keys matching the value in "Prop_ModelNumber_String" are considered first, since the driver may have model specific icons.
    // An absence of a matching "Prop_ModelNumber_String" then considers the ETrackedDeviceClass ("HMD", "Controller", "GenericTracker", "TrackingReference")
    // since the driver may have specialized icons based on those device class names.
    //
    // An absence of either then falls back to the "system.vrresources" where generic device class icons are then supplied.
    //
    // Please refer to "bin\drivers\sample\resources\driver.vrresources" which contains this sample configuration.
    //
    // "Alias" is a reserved key and specifies chaining to another json block.
    //
    // In this sample configuration file (overly complex FOR EXAMPLE PURPOSES ONLY)....
    //
    // "Model-v2.0" chains through the alias to "Model-v1.0" which chains through the alias to "Model-v Defaults".
    //
    // Keys NOT found in "Model-v2.0" would then chase through the "Alias" to be resolved in "Model-v1.0" and either resolve their or continue through the alias.
    // Thus "Prop_NamedIconPathDeviceAlertLow_String" in each model's block represent a specialization specific for that "model".
    // Keys in "Model-v Defaults" are an example of mapping to the same states, and here all map to "Prop_NamedIconPathDeviceOff_String".


An all around good explanation, until you look in ``driver.vrresources`` that is provided with the sample driver...

.. code-block:: json

    {
        "jsonid" : "vrresources",
        "statusicons" : {
            "HMD" : {
                "Prop_NamedIconPathDeviceOff_String" : "{sample}/icons/headset_sample_status_off.png",
                "Prop_NamedIconPathDeviceSearching_String" : "{sample}/icons/headset_sample_status_searching.gif",
                "Prop_NamedIconPathDeviceSearchingAlert_String" : "{sample}/icons/headset_sample_status_searching_alert.gif",
                "Prop_NamedIconPathDeviceReady_String" : "{sample}/icons/headset_sample_status_ready.png",
                "Prop_NamedIconPathDeviceReadyAlert_String" : "{sample}/icons/headset_sample_status_ready_alert.png",
                "Prop_NamedIconPathDeviceNotReady_String" : "{sample}/icons/headset_sample_status_error.png",
                "Prop_NamedIconPathDeviceStandby_String" : "{sample}/icons/headset_sample_status_standby.png",
                "Prop_NamedIconPathDeviceAlertLow_String" : "{sample}/icons/headset_sample_status_ready_low.png"
            },

            "Model-v Defaults" : {
                "Prop_NamedIconPathDeviceOff_String" : "{sample}/icons/headset_sample_status_off.png",
                "Prop_NamedIconPathDeviceSearching_String" : "Prop_NamedIconPathDeviceOff_String",
                "Prop_NamedIconPathDeviceSearchingAlert_String" : "Prop_NamedIconPathDeviceOff_String",
                "Prop_NamedIconPathDeviceReady_String" : "Prop_NamedIconPathDeviceOff_String",
                "Prop_NamedIconPathDeviceReadyAlert_String" : "Prop_NamedIconPathDeviceOff_String",
                "Prop_NamedIconPathDeviceNotReady_String" : "Prop_NamedIconPathDeviceOff_String",
                "Prop_NamedIconPathDeviceStandby_String" : "Prop_NamedIconPathDeviceOff_String",
                "Prop_NamedIconPathDeviceAlertLow_String" : "Prop_NamedIconPathDeviceOff_String"
            },

            "Model-v1.0" : {
                "Alias" : "Model-v Defaults",
                "Prop_NamedIconPathDeviceAlertLow_String" : "{sample}/icons/headset_model1_alertlow.png"
            },

            "Model-v2.0" : {
                "Alias" : "Model-v1.0",
                "Prop_NamedIconPathDeviceAlertLow_String" : "{sample}/icons/headset_model2_alertlow.png"
            },

            "Controller" : {
                "Prop_NamedIconPathDeviceOff_String" : "{sample}/icons/controller_status_off.png",
                "Prop_NamedIconPathDeviceSearching_String" : "{sample}/icons/controller_status_searching.gif",
                "Prop_NamedIconPathDeviceSearchingAlert_String" : "{sample}/icons/controller_status_searching_alert.gif",
                "Prop_NamedIconPathDeviceReady_String" : "{sample}/icons/controller_status_ready.png",
                "Prop_NamedIconPathDeviceReadyAlert_String" : "{sample}/icons/controller_status_ready_alert.png",
                "Prop_NamedIconPathDeviceNotReady_String" : "{sample}/icons/controller_status_error.png",
                "Prop_NamedIconPathDeviceStandby_String" : "{sample}/icons/controller_status_standby.png",
                "Prop_NamedIconPathDeviceAlertLow_String" : "{sample}/icons/controller_status_ready_low.png"
            }
        }
    }

What are these ``Prop`` things? What does ``{sample}`` mean? Why are some icons gifs and others pngs?

Lets start with the ``Prop`` things. The full list of acceptable keys can be found in `openvr_driver.h <https://github.com/ValveSoftware/openvr/blob/master/headers/openvr_driver.h#L524>`_ in the ``ETrackedDeviceProperty`` enum:

.. code-block:: c

    // *other enum members*

    // Properties that are used for user interface like icons names
    Prop_IconPathName_String                        = 5000, // DEPRECATED. Value not referenced. Now expected to be part of icon path properties.
    Prop_NamedIconPathDeviceOff_String              = 5001, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others
    Prop_NamedIconPathDeviceSearching_String        = 5002, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others
    Prop_NamedIconPathDeviceSearchingAlert_String   = 5003, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others
    Prop_NamedIconPathDeviceReady_String            = 5004, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others
    Prop_NamedIconPathDeviceReadyAlert_String       = 5005, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others
    Prop_NamedIconPathDeviceNotReady_String         = 5006, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others
    Prop_NamedIconPathDeviceStandby_String          = 5007, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others
    Prop_NamedIconPathDeviceAlertLow_String         = 5008, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others
    Prop_NamedIconPathDeviceStandbyAlert_String     = 5009, // {driver}/icons/icon_filename - PNG for static icon, or GIF for animation, 50x32px for headsets and 32x32px for others

    // *other enum members*

The convenient code comments, which also explain the gif and png situation.

Now onto the ``{sample}`` and ``{driver}`` from prop comments. Which has to do with path resolution. You see SteamVR allows you register your driver from anywhere, you can use ``{name of a driver}`` in the beginning of paths, it will then be replaced with the path to that driver + ``/resources``. See `Driver Relative Paths`_.

Last thing about icons that you need to know is that you can have variants of icons without changing any of the resource configs. You can do that by a ``@2x`` suffix before the extension to the variant icon names to add a 2 times higher res icon, adding a ``.b4bfb144`` suffix to the variant icon will make it the default variant on Windows, and you can also combine them, but ``@2x`` needs to be the first suffix. For good measure here is an example(pulled from SteamVR):

.. code-block:: bash

    tree -L 2 indexhmd/resources/icons/
    indexhmd/resources/icons/
    ├── headset_status_error@2x.b4bfb144.png
    ├── headset_status_error@2x.png
    ├── headset_status_error.b4bfb144.png
    ├── headset_status_error.png
    ├── headset_status_off@2x.6e6c89c9.png
    ├── headset_status_off@2x.png
    ├── headset_status_off.6e6c89c9.png
    ├── headset_status_off.png
    ├── headset_status_ready@2x.b4bfb144.png
    ├── headset_status_ready@2x.png
    ├── headset_status_ready_alert@2x.b4bfb144.png
    ├── headset_status_ready_alert@2x.png
    ├── headset_status_ready_alert.b4bfb144.png
    ├── headset_status_ready_alert.png
    ├── headset_status_ready.b4bfb144.png
    ├── headset_status_ready.png
    ├── headset_status_searching@2x.b4bfb144.gif
    ├── headset_status_searching@2x.gif
    ├── headset_status_searching_alert@2x.b4bfb144.gif
    ├── headset_status_searching_alert@2x.gif
    ├── headset_status_searching_alert.b4bfb144.gif
    ├── headset_status_searching_alert.gif
    ├── headset_status_searching.b4bfb144.gif
    ├── headset_status_searching.gif
    ├── headset_status_standby@2x.b4bfb144.png
    ├── headset_status_standby@2x.png
    ├── headset_status_standby_alert@2x.b4bfb144.png
    ├── headset_status_standby_alert@2x.png
    ├── headset_status_standby_alert.b4bfb144.png
    ├── headset_status_standby_alert.png
    ├── headset_status_standby.b4bfb144.png
    ├── headset_status_standby.png
    └── indexhmd.svg

    0 directories, 33 files


Input Profiles
--------------

Json with a lot of hate (WIP)

Localization
------------

WIP

.. note::

    Writer's note: Ok, i'm gonna be straight with you, this is not mentioned anywhere, we have nothing! Take everything here with a massive grain of salt, figuring out how it works is like piecing together a log out of saw dust.

    You are more than welcome to update our documentation, if you see something inaccurate. See :ref:`contrib-section`.


Render Models
-------------

Obj models with a hint of json hate (WIP)

Firmware
--------

Its not in any examples, it is mentioned a lot in the ``openvr_driver.h`` header though. You might be able to use the "automatic™ SteamVR™ firmware™ updater™ tool™ thingymajig™®©℠℗" using the combination of ``VREvent_FirmwareUpdateStarted`` and ``VREvent_FirmwareUpdateFinished`` events together with ``EVRFirmwareError`` enum, firmware related props and files in ``{driver name}/firmware/``. We don't really now the specific on how to use the "automatic™ SteamVR™ firmware™ updater™ tool™ thingymajig™®©℠℗". What we do know is how to trigger the manual firmware update notification for the user which will redirect the user to a url from ``Prop_Firmware_ManualUpdateURL_String``. Well how do you do that then? Simple just set ``Prop_Firmware_UpdateAvailable_Bool`` and ``Prop_Firmware_ManualUpdate_Bool`` to true using the ``VRProperties`` interface. See :ref:`vrproperties-interface`.

Animations
----------

I got nothing for you yet again x)

The only thing I know about it, is that Index and Oculus controller drivers use them. Animations can be found in ``{driver name}/anims/``. Why are they there? What are they used for? How do *you* use it in *your* driver? And the answers to those are: No idea. No clue. And I wish I knew.

This is not mentioned in any examples, hell there is no mention of the word ``anim`` in ``openvr_driver.h``. As far as i can tell it's not mentioned anywhere, if you have *any* info about it, it would help a lot if you could update this section. See :ref:`contrib-section`.

Update: Actually turns out there is an issue on the topic. See `OpenVR Issue 1552 <https://github.com/ValveSoftware/openvr/issues/1552>`_.


Driver Relative Paths
^^^^^^^^^^^^^^^^^^^^^

After you register a driver SteamVR will interpret ``{name of your driver}`` as ``/path/to/your/driver/resources``, only works when paths are processed by OpenVR/SteamVR tools. Example: When setting icons for devices(either for status icons, bindings, models or other resources), to use icons shipped with your driver, you can use ``{name of your driver}/icons/icon_name.png``.

Surface Level Driver ABI
^^^^^^^^^^^^^^^^^^^^^^^^

To be a functional driver you need to compile your source into a DLL or a Shared Object (depending on which platform you use).

HMD Driver Factory
------------------

Your driver needs to export only a single symbol which is a function that with the following signature: ``void* HmdDriverFactory(const char *pInterfaceName, int *outReturnCode)``

How does it work? Dead simple actually, ``pInterfaceName`` is a null terminated string with the name of the interface SteamVR runtime requests from your driver, if your driver implements the requested interface, return a raw pointer to it, if it does not match any of the interfaces implemented by your driver set ``*outReturnCode`` to ``VRInitError_Init_InterfaceNotFound`` and return ``NULL``.

Why? Interfaces implemented in your driver get to SteamVR runtime through this function.

Driver Interfaces
-----------------

Honestly this topic is too big to be in the general surface level introduction to driver ABI, so it's been moved to it's own page :ref:`driver-interfaces`.