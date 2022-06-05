
.. _driver-interfaces:

Driver Interfaces
=================

This is an incomplete unofficial document describing driver interfaces present in OpenVR.

.. _vrproperties-interface:

IVRProperties Interface
^^^^^^^^^^^^^^^^^^^^^^^

WIP


IVRSettings Interface
^^^^^^^^^^^^^^^^^^^^^

Current version: ``IVRSettings_003``

Allows the user to fetch, write (and also remove keys or entire sections) the last overridden value of a specific key from a specific section.
The values can be overridden by any ``*.vrsettings`` file (which is picked up by either SteamVR, like ``<steam install path>/configs/steamvr.vrsettings`` or your driver using the ``Prop_AdditionalDeviceSettingsPath_String`` prop) with a matching section & key pair.

Example usage:

You're writing a driver and you need to have some sort of configuration/persistence.
Your needs can be easily satisfied with ``IVRSettings``. First create a file called ``default.vrsettings`` and place it in ``<your driver dir>/resources/settings/``, then put your default configuration inside this file, using json syntax and the following scheme:

.. code-block:: json

	{
		"my_very_own_driver_section" : {
			"my_very_usefull_key_float" : 0.5,
			"my_very_usefull_key_int" : 15,
			"my_very_usefull_key_bool" : false,
			"my_very_usefull_key_str" : "blah blah blah text here blah blah blah"
		}
	}

Then in your driver you can access the values for your section & key pairs (and other section & key pairs) by using the ``IVRSettings`` interface:

.. code-block:: c++

	float fVal = vr::VRSettings()->GetFloat("my_very_own_driver_section", "my_very_usefull_key_float");
	
	int32_t iVal = vr::VRSettings()->GetInt32("my_very_own_driver_section", "my_very_usefull_key_int");
	
	bool bVal = vr::VRSettings()->GetBool("my_very_own_driver_section", "my_very_usefull_key_bool");
	
	char buff[255];
	memset(buff, 0, sizeof(buff));
	vr::VRSettings()->GetString(
		"my_very_own_driver_section",
		"my_very_usefull_key_str",
		buff,
		sizeof(buff)
	);
	std::string sVal(buff, sizeof(buff));

You can also write to section & key pairs, but be aware that if you write to a section & key pair that never existed before, it'll get create, except in ``<steam install path>/config/steamvr.vrsettings`` instead:

.. code-block:: c++

	vr::VRSettings()->SetFloat("my_very_own_driver_section", "my_very_usefull_key_float", 1.5);
	vr::VRSettings()->SetInt32("my_very_own_driver_section", "my_very_usefull_key_int", 420);
	vr::VRSettings()->SetBool("my_very_own_driver_section", "my_very_usefull_key_bool", true);
	vr::VRSettings()->GetString(
		"my_very_own_driver_section",
		"my_very_usefull_key_str",
		"new very kewl text mmmmmmmmmmm yes"
	);
	vr::VRSettings()->SetInt32("my_very_own_driver_section", "my_key_that_didnt_exist_int", 420);

You can also remove sections or keys in a sections, but be extra careful not to remove a section or key that is used by a different driver.

.. code-block:: c++

	// this will only remove this specific key from this section
	vr::VRSettings()->RemoveKeyInSection("my_very_own_driver_section", "my_key_that_didnt_exist_int");
	// this will remove all keys in the section and delete the section itself
	vr::VRSettings()->RemoveSection("my_very_own_driver_section");

Optionally you can also pass a pointer to ``vr::EVRSettingsError`` as the last argument to capture the error, for how to process those refer to the ``vr::EVRSettingsError`` enum.


VRInput Interface
^^^^^^^^^^^^^^^^^

WIP


IVRDriverLog Interface
^^^^^^^^^^^^^^^^^^^^^^

Current version: ``IVRDriverLog_001``

A pretty straightforward interface allowing you to log something in SteamVR's log.
The logs will appear in the developer web console as well as in ``<steam install path>/logs/vrserver.txt``.

Example usage:

.. code-block:: c++

	VRDriverLog()->Log("my very cool log");

This is the only method it has and it accepts a null terminated string, we recommend you write your own wrapped around it to allow formatting.

ITrackedDeviceServerDriver Device Interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

WIP


Components
^^^^^^^^^^

WIP


IServerTrackedDeviceProvider Driver Interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

WIP


IVRWatchdogProvider Driver Interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

WIP


IVRCompositorPluginProvider Driver Interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

WIP


