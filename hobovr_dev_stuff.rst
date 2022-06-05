Miscellaneous Development Docs for HoboVR
=========================================

Here is a collection of some of the things I found myself re-explaining a lot. Hopefully this will clear things up enough going forward.

Installing Dev Versions of Our Driver
-------------------------------------

Download the latest driver artifact for your platform from the CI run you want to use.

Unpack it somewhere in your system and register the unpacked folder as an external driver with SteamVR, either by using the ``register_driver`` script included with the driver or doing it by hand. See :ref:`driver-reg`.

Also I strongly recommend you read the ``README.md`` file included with the driver.

After that's done you need to verify that it's registered name is matching ``hobovr`` and registered path is matching the path of the folder you unpacked, this can be done with ``<vrpathreg> show``.

.. note::

	Replace ``<vrpathreg>`` with the path to the ``vrpathreg`` executable.

For more info see :ref:`vrpathreg`.

.. _driver-reg:

Using Posers and Testing Our Drivers
------------------------------------

Using Posers
^^^^^^^^^^^^

All posers we ship currently need to be started prior to starting SteamVR.

Also most of our posers are simple executables with command line controls, when you get to a point where the poser is waiting for the driver to connect you can start SteamVR.

Testing Our Drivers
^^^^^^^^^^^^^^^^^^^

Test posers can be downloaded from the same CI run you downloaded the driver.

Test posers are normal posers like any other, except they are not tied to any hardware, just start em' like any other poser then start SteamVR.
If everything is working correctly (and if the poser you are testing has an HMD device) you should see a viewport from which you can see the view for both eyes which is a bit distorted and all devices appear green as well as being visible in the viewport.

Registering External Drivers
----------------------------

This applies to every other external driver, not just ours.

After you have your driver folder somewhere on your system (e.g. ``/opt/my_kewl_driver/``) you need to register it, so that SteamVR will try to load it.

To do that use the ``<vrpathreg> adddriver <driver path>`` command, where you replace ``<driver path>`` with the path to the driver folder on your system (e.g. ``/opt/my_kewl_driver/``).

.. note::

	Replace ``<vrpathreg>`` with the path to the ``vrpathreg`` executable.

For more info see :ref:`vrpathreg`.



Troubleshooting External Driver Registry
----------------------------------------

This is mostly aimed at addressing common faults with our dev driver register scripts (which fails half the time on Windows for no god damn reason), but the in general this also applies to other drivers too.

Generally there are 3 steps to this

	1) Check that the path and name match with what you expect using the ``<vrpathreg> show`` command
	2) Make sure there are no null entries, for more info see `OpenVR Issue 1653 <https://github.com/ValveSoftware/openvr/issues/1653>`_
	3) If the previous two steps passed but SteamVR is still not loading your driver it's time to look at the logs, SteamVR will log the load status of all registered driver paths, including load errors. See :ref:`steamvr-logs`.



Uninstalling External Drivers
-----------------------------

Sometimes one needs to uninstall a driver, be it to install a newer one or because it's not needed anymore, this generally applies to all external drivers.

The process itself is split into two parts

	1) Remove it from the external driver registry

	2) Delete all driver files

.. note::

	You don't need to re-register the driver if you are, for example, simply updating the dev version of our driver with a newer build, just replace the files in the driver folder, no further setup required.


Removing Drivers From the External Driver Registry
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To do this you need to use ``vrpathreg``'s ``removedriver`` command or ``removedriverwithname`` if you wish to remove it by name,
if you are not sure you can always double check by looking for the driver you wish to remove in the list of all external drivers acquired with the ``<vrpathreg> show`` command.

For example you have a driver installed with this path in the registry ``/opt/my_kewl_driver`` and named ``my_kewl_driver``.
Now to remove this driver from the registry by name you need to execute this command:

.. code-block:: bash

	<vrpathreg> removedriverwithname my_kewl_driver

And if you want to remove it by using the registered path you need to execute this command:

.. code-block:: bash

	<vrpathreg> removedriver "/opt/my_kewl_driver"

.. note::

	Replace ``<vrpathreg>`` with the path to the ``vrpathreg`` executable.

For more info see :ref:`vrpathreg`.


Deleting Driver Files
^^^^^^^^^^^^^^^^^^^^^

Before you do this you need to make sure the driver folder(s) are no longer registered, so that SteamVR will not try to load them after you deleted them, you can do that by using the ``show`` command from ``vrpathreg``.

Actually deleting them is pretty straight forward, just delete the driver folder, in case of the driver in ``/opt/my_kewl_driver`` from the previous example you just need to delete the ``/opt/my_kewl_driver/`` folder.



.. _steamvr-logs:

Pulling Driver Logs From SteamVR
--------------------------------

SteamVR stores driver logs in ``<steam install path>/logs/vrserver.txt``, but all logs are also accessible from the developer's web console live, while SteamVR is running.



.. _vrpathreg:

What in the Living Hell Is ``vrpathreg``?!
------------------------------------------

It's a registry tool. SteamVR registry tool to be exact, it allows you to change paths to some core components of the SteamVR system as well as register/unregister external drivers.

To use it you'll have to open your terminal of choice in ``<SteamVR install path>/bin/`` and here is where the usage differs from Windows to Linux a bit... You see on windows it's an exe file in ``<SteamVR install path>/bin/win32/`` and on Linux it a bash script in ``<SteamVR install path>/bin/`` wrapper around the binary executable to setup the environment for it to work properly.

So whenever you see ``<vrpathreg>`` replace it with the path to the executable thing for your platform, the arguments stay the same (and if any of the paths have spaces in them put them in double quotes).


.. code-block:: bash

	<vrpathreg> help
	Commands:
		show - Display the current paths
		setruntime <path> - Sets the runtime path
		setthis - Sets the runtime path to the runtime that vrpathreg lives in
		setconfig <path> - Sets the config path
		setlog <path> - Sets the log path
		adddriver <path> - Adds an external driver
		removedriver <path> - Removes an external driver
		removedriverswithname <name> - Removes all external drivers with a given name
		finddriver <name> - Tries to find a driver by name

		Return Code:
		0 : Success
		1 : ( finddriver only ) Driver not present
		2 : ( finddriver only ) Error, driver installed more than once
		-1 : Configuration or permission problem
		-2 : Argument problem

.. note::

	Return codes are fetched differently on Windows and Linux.

	On Linux it's as simple as running ``echo $?`` after the command.

	And on Windows to see the return code of the last command you need to run ``echo %errorlevel%``.

Your go to commands for driver development are ``adddriver``, ``removedriver`` and ``show``, but not so long ago ``removedriverswithname`` and ``finddriver`` have been added.

``show`` takes no arguments, displays the current registry, example (Linux):

.. code-block:: bash

	<vrpathreg> show
	Runtime path = /home/<user>/.local/share/Steam/steamapps/common/SteamVR
	Config path = /home/<user>/.local/share/Steam/config
	Log path = /home/<user>/.local/share/Steam/logs
	External Drivers:
		hobovr : /home/<user>/Documents/hobo_vr/hobovr/


``adddriver`` takes a driver folder path as the first argument, it registers a driver folder for SteamVR, but be careful to not register a null path, see `OpenVR Issue 1653 <https://github.com/ValveSoftware/openvr/issues/1653>`_.

.. code-block:: bash

	<vrpathreg> adddriver <driver folder path>

``removedriver`` takes a driver folder path as the first argument, it unregisteres a driver folder for SteamVR.

.. code-block:: bash

	<vrpathreg> removedriver <driver folder path>

``removedriverswithname`` takes a driver name as the first argument, it unregisteres the driver with a matching name.

.. code-block:: bash

	<vrpathreg> removedriverswithname <driver name>

``finddriver`` takes a driver name as the first argument, outputs the path for the driver matching the input name, outputs an empty line if there is no matching driver.

.. code-block:: bash

	<vrpathreg> finddriver <driver name>

Now, the rest of the commands are useful only for messing with SteamVR itself, i don't recommend you ever try this, but here is a short description of what these commands do:

``setruntime <path>`` sets the SteamVR runtime path, by default it's ``<steam install path>/steamapps/common/SteamVR``


``setconfig <path>`` sets the SteamVR's config path, by default it's ``<steam install path>/config``

``setlog <path>`` sets the path where SteamVR bumps logs, by default it's ``<steam install path>/logs``

``setthis`` no idea what it does, never seen it used
