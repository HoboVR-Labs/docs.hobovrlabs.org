Getting Started
===============

To get started with ``hobo_vr`` you need two things: our driver and a poser.

Installing The Driver
---------------------
.. note::

    You need Steam and SteamVR installed to use our driver


You can download our installers from `here <https://github.com/HoboVR-Labs/hobo_vr/releases>`_.


Building Driver From Source
---------------------------

.. note::

    We assume you are familiar with command line/terminal, and able to run basic commands.


We use ``CMake`` to build our driver, so you'll need it installed. (On Windows you'll need the command line version.)

You will also need ``Git``, so make sure it's installed as well. (On Windows you'll need the command line version.)

Now you need to clone our repository::
    
    git clone https://github.com/HoboVR-Labs/hobo_vr
    cd ./hobo_vr
    git submodule init
    git submodule update

And since you're building the driver change to it's source directory::

    cd ./driver

Building on Linux
^^^^^^^^^^^^^^^^^

.. note::

    The ``-DINSTALL_X86_DRIVER=1`` argument is not optional for 32bit builds, if it is not added you risk installing a 32bit driver into a 64bit target!

You'll need to compile our driver twice, once 32bit and once 64bit. With the ``Unix Makefiles`` generator you can use the following commands to generate the build files::

    cmake . -DCMAKE_CXX_FLAGS=-m32 -DCMAKE_C_FLAGS=-m32 -B "build32" -DINSTALL_X86_DRIVER=1
    cmake . -DCMAKE_CXX_FLAGS=-m64 -DCMAKE_C_FLAGS=-m64 -B "build64"

To build the driver you need to run these commands::

    cmake --build build32 --config Release
    cmake --build build64 --config Release

Now you can run these commands to move the built binaries into the driver folder::

    cmake --install build32
    cmake --install build64

See :ref:`opevr-driver-directories`.


Building on Windows
^^^^^^^^^^^^^^^^^^^

.. note::

    The ``-DINSTALL_X86_DRIVER=1`` argument is not optional for 32bit builds, if it is not added you risk installing a 32bit driver into a 64bit target!

To build on windows you'll need Visual Studio with C++ desktop development package installed.

You'll need to compile our driver twice, once 32bit and once 64bit. With the ``Visual Studio 16 2019`` generator you can use to following commands to generate the build files::
    
    cmake -G "Visual Studio 16 2019" -A Win32 . -B "build32" -DINSTALL_X86_DRIVER=1
    cmake -G "Visual Studio 16 2019" -A x64 . -B "build64"

To build the driver you need to run these commands::

    cmake --build build32 --config Release
    cmake --build build64 --config Release

Now you can run these commands to move the built binaries into the driver folder::

    cmake --install build32
    cmake --install build64

See `Openvr Driver Folders <#>`_.

Installing Your Build
^^^^^^^^^^^^^^^^^^^^^

.. note::
    
    You only need to install your driver build once, unless you move your install directory.


To install your build of the ``hobo_vr`` driver, you need to do a few things.


First and foremost locate ``<your SteamVR install directory>/bin/<your platform>/vrpathreg``. It's a binary file tool that will allow you to register your driver build. We'll refer to it as ``vrpathreg`` from now on.

Now you need to make sure a different version of our driver is not installed, you can do that by running this ``vrpathreg`` command::

    vrpathreg show

.. note::
    
    If you are on Linux and get this error: ``error while loading shared libraries: libopenvr_api.so: cannot open shared object file: No such file or directory``, see `SteamVR Linux issue#478 <https://github.com/ValveSoftware/SteamVR-for-Linux/issues/478>`_.

Checking the paths in the ``External Drivers:`` section, if any the paths end with ``hobovr`` (used by our installers) you need to run this ``vrpathreg`` command::

    vrpathreg removedriver <path to the other versions of hobo_vr>

Now you can install your build by running yet another ``vrpathreg`` command::

    vrpathreg adddriver <path to your built driver directory>

Congratulations, you installed your very own build of the ``hobo_vr`` driver!


What Is a Poser
----------------

A poser is what we call a process that controls our driver. On its own our driver will not do anything, hell it won't even start if a poser process is not running.

Examples
^^^^^^^^

You can find poser examples for Python and C++ on `our GitHub repository <https://github.com/HoboVR-Labs/hobo_vr/tree/master/bindings>`_.

But here is a simple example for Python:

.. code-block:: python
    
    import struct
    import socket
    import math as m
    import time

    TERMINATOR = b'\n'
    SEND_TERMINATOR = b'\t\r\n'
    MANAGER_UDU_MSG_t = struct.Struct("130I")
    POSE_t = struct.Struct("13f")
    CONTOLLER_t = struct.Struct("22f")


    # bind and start listening to the poser address
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    serversocket.bind(('', 6969))
    serversocket.listen(2)  # driver connects with 2 sockets

    #######################################################################
    # now lets accept both of them and resolve

    print("waiting for driver to connect...")

    client_a = serversocket.accept()
    client_b = serversocket.accept()


    print("waiting for driver resolution...")

    resp_a = client_a[0].recv(50)
    resp_b = client_b[0].recv(50)

    if TERMINATOR in resp_a:
        id_msg_a, resp_a = resp_a.split(TERMINATOR, 1)

    if TERMINATOR in resp_b:
        id_msg_b, resp_b = resp_b.split(TERMINATOR, 1)


    if id_msg_a == b"hello" and id_msg_b == b"monky":
        tracking_socket = client_a[0]
        manager_socket = client_b[0]

    elif id_msg_b == b"hello" and id_msg_a == b"monky":
        tracking_socket = client_b[0]
        manager_socket = client_a[0]

    else:
        print("bad connection")
        client_a[0].close()
        client_b[0].close()

        serversocket.close()

        exit()

    input("press anything to start...")

    #######################################################################
    # tell the manager about current device setup

    device_list = MANAGER_UDU_MSG_t.pack(
        20,  # HobovrManagerMsgType::Emsg_uduString
        3,   # 3 devices - 1 hmd, 2 controllers
        0, 13,  # device description
        1, 22,  # device description
        1, 22,  # device description
        *np.zeros((128 - 2 * 3), dtype=int)
    )

    manager_socket.sendall(device_list + SEND_TERMINATOR)


    try:
        i = 0
        while 1:
            controller_z = m.sin(i / 180 * m.pi) * 3
            right_pose = pose = CONTOLLER_t.pack(
                0.2, 0, controller_z,
                1, 0, 0, 0,
                int(i < 10), 0, 0,
                0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0, 0
            )

            left_pose = CONTOLLER_t.pack(
                -0.2, 0, controller_z,
                0, 0, 0, -1,
                int(i < 10), 0, 0,
                0, 0, 0,
                0, 0, 0, 0, 0, 0, 0, 0, 0
            )

            hmd_pose = POSE_t.pack(
                int(i < 10), 0, 0,
                int(controller_z <= 0), 0, -int(controller_z > 0), 0,
                int(i < 10), 0, 0,
                0, 0, 0
            )

            tracking_socket.sendall(
                hmd_pose + right_pose + left_pose + SEND_TERMINATOR
            )

            time.sleep(1 / 60)

            i += 1

    except KeyboardInterrupt:
        print("interrupted, exiting...")


    #######################################################################
    # the end, time to die ^-^

    client_a[0].close()
    client_b[0].close()

    serversocket.close()


How It Works
^^^^^^^^^^^^

For a process to be acknowledged as a poser by our driver, it needs to bind and listen  on ``tcp://127.0.0.1:6969``, and then accept 2 sockets. The two sockets will identify themselves with ``"hello\n"`` and ``"monky\n"`` as the first message sent after establishing a connection.

The socket that sent ``"monky\n"`` is used as a general driver control channel, we call it the manager channel though. This socket allows for changing the device live list, changing some device settings, etc. See `Manager Protocol`_.

The socket that sent ``"hello\n"`` is used as a device control channel, we call it tracking channel though. This socket is meant for controlling the live device list, mostly through tracking. See `Tracking Protocol`_.



Poser Protocols
---------------

When sending messages, the poser process has to fill the ``terminator[3]`` field with ``"\t\r\n"``.

Manager Protocol
^^^^^^^^^^^^^^^^
Current manager protocol consists of the following structs:

.. code-block:: C

    // manager command type
    enum HobovrManagerMsgType
    {
      Emsg_invalid = 0,
      Emsg_ipd = 10,
      Emsg_uduString = 20,
      Emsg_poseTimeOffset = 30,
      Emsg_distortion = 40,
      Emsg_eyeGap = 50,
      Emsg_setSelfPose = 60,
    };


    // manager command structs

    #pragma pack(push, 1)

    // changes the ipd for hmd devices
    struct IpdMessage {
        uint32_t type;  // has to be Emsg_ipd
        uint32_t nominator;
        uint32_t denominator;
        uint32_t rest[127];
        char terminator[3];
    };

    // updates device list live
    struct UduStringMessage {
        uint32_t type;  // has to be Emsg_uduString
        uint32_t len; // number of devices
        struct {
            uint32_t device_type; // h - 0, c - 1, t - 2
            uint32_t device_len; // number of floats for this device
        } devices[64];
        char terminator[3];
    };

    // updates pose time offsets
    struct PoseTimeOffsetMessage {
        uint32_t type;  // has to be Emsg_poseTimeOffset
        uint32_t nominator;
        uint32_t denominator;
        uint32_t rest[127];
        char terminator[3];
    };

    // updates distortion parameters, will require a restart to take effect
    struct DistortionMessage {
        uint32_t type;  // has to be Emsg_distortion
        uint32_t k1_nominator;
        uint32_t k1_denominator;
        uint32_t k2_nominator;
        uint32_t k2_denominator;
        uint32_t zoom_width_nominator;
        uint32_t zoom_width_denominator;
        uint32_t zoom_height_nominator;
        uint32_t zoom_height_denominator;
        uint32_t rest[121];
        char terminator[3];
    };

    // updates the hobovr_comp_extendedDisplay eye gap setting,
    // will require a restart to take effect
    struct EyeGapMessage {
        uint32_t type;  // has to be Emsg_eyeGap
        uint32_t width; // in pixels
        uint32_t rest[128];
        char terminator[3];
    };

    // updates the location of the virtual base station device (which manager runs as)
    struct SetSelfPoseMessage {
        uint32_t type;  // has to be Emsg_setSelfPose
        uint32_t x_nominator;
        uint32_t x_denominator;
        uint32_t y_nominator;
        uint32_t y_denominator;
        uint32_t z_nominator;
        uint32_t z_denominator;
        uint32_t rest[123];
        char terminator[3];
    };

    #pragma pack(pop)

Any of them can be sent using the manager socket.

Tracking Protocol
^^^^^^^^^^^^^^^^^
The tracking protocol is, unfortunately, not as simple as the manager protocol. Depending on the current driver device list, the message structure will change. Here are some examples (device structs are explained later):



.. code-block:: C
    
    // Driver device list in this example will be(in that order): hmd, controller_right, controller_left

    #pragma pack(push, 1)

    struct driver_packet {
        pose_t hmd;
        // controller sides are order sensitive
        controller_pose_t controller_right;
        controller_pose_t controller_left;
        char terminator[3];
    };

    #pragma pack(pop)


Now this message struct can be using the tracking socket. However if the device list changes and the message struct is not changed accordingly the driver will ignore messages coming from this socket. *The poser will not be notified about that*.

The rule for constructing tracking message structs is, for each device in the device list(conserving the order) you choose one of the structs:

.. code-block:: C
    
    // tracking only pose, eligible for HMDs and Trackers
    struct pose_t {
        float position[3];  // 3D vector
        float orientation[4];  // quaternion
        float velocity[3];  // 3D vector
        float angular_velocity[3];  // 3D vector
    };

    // tracking + controller inputs, eligible for Controllers only
    struct controller_pose_t {
        pose_t pose;
        float inputs[9];
        // vive wand style inputs

        // inputs[0] - grip button, recast as bool
        // inputs[1] - SteamVR system button, recast as bool
        // inputs[2] - app menu button, recast as bool
        // inputs[3] - trackpad click button, recast as bool
        // inputs[4] - trigger value, one sided normalized  scalar axis
        // inputs[5] - trackpad x axis, normalized two sided scalar axis
        // inputs[6] - trackpad y axis, normalized two sided scalar axis
        // inputs[7] - trackpad touch signal, recast as bool
        // inputs[8] - trigger click button, recast as bool
        
    }

The constructed message struct can be sent then sent out using the tracking socket.

.. note::

    To not be ignored posers should signal their desired device list using the ``UduStringMessage`` manager message.

.. note::

    The tracking protocol is old and terrible, but its stable, so its gonna stay while we're working on a v2.
