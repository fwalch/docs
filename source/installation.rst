.. highlight:: shell

Installation instructions
=========================

This chapter describes how to set up a real-time kernel and install ``libfranka`` and
``franka_ros``, either as binary packages or by building from source. ``franka_ros`` is only
required if you want to control your robot using `ROS <http://www.ros.org/>`_.

.. note::

   While ``libfranka`` and the ``franka_ros`` packages should work on different Linux distributions,
   official support is currently only provided for Ubuntu 16.04 LTS `Xenial Xerus` and ROS
   `Kinetic Kame`. The following instructions might therefore only work in this environment.


Setting up a real-time kernel
-----------------------------

In order to control your robot using ``libfranka``, the controller program on the workstation
PC must run with `real-time priority` under a ``PREEMPT_RT`` kernel. The procedure of patching a
kernel to support ``PREEMPT_RT`` and creating an installation package is described by the
following online resources:

 * `Installing a Kernel with the RT Patch
   <http://home.gwu.edu/~jcmarsh/wiki/pmwiki.php%3Fn=Notes.RTPatch.html>`_
 * `Howto setup Linux with PREEMPT_RT properly
   <https://wiki.linuxfoundation.org/realtime/documentation/howto/applications/preemptrt_setup>`_

.. _installation-real-time:

Allow user to set real-time permissions for its processes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After the ``PREEMPT_RT`` kernel is installed and running, add a group named `realtime` and
add the user controlling your robot to this group::

    sudo addgroup realtime
    sudo usermod -a -G realtime $(whoami)

Afterwards, add the following limits to the `realtime` group in ``/etc/security/limits.conf``::

    @realtime soft rtprio 99
    @realtime soft priority 99
    @realtime soft memlock 102400
    @realtime hard rtprio 99
    @realtime hard priority 99
    @realtime hard memlock 102400


Installing from the ROS repositories
------------------------------------

Binary packages for ``libfranka`` and ``franka_ros`` are available from the ROS repositories.
After `setting up ROS Kinetic <http://wiki.ros.org/kinetic/Installation/Ubuntu>`__, execute::

    sudo apt install ros-kinetic-libfranka ros-kinetic-franka-ros


Building from source
--------------------

This section describes how to build ``libfranka`` and ``franka_ros``.

Building libfranka
^^^^^^^^^^^^^^^^^^

To build ``libfranka``, install the following dependencies from Ubuntu's package manager::

    sudo apt install build-essential cmake git libpoco-dev

Then, download the source code by cloning ``libfranka`` from
`GitHub <https://github.com/frankaemika/libfranka>`__::

    git clone --recursive https://github.com/frankaemika/libfranka
    cd libfranka

In the source directory, create a build directory and run CMake::

    mkdir build
    cd build
    cmake -DCMAKE_BUILD_TYPE=Release ..
    cmake --build .

.. _libfranka_systemwide:

If a systemwide installation into ``/usr`` is desired, execute the following instructions::

    cd libfranka/build
    cpack -G DEB
    sudo dpkg -i libfranka-*.deb

Building the ROS packages
^^^^^^^^^^^^^^^^^^^^^^^^^

After `setting up ROS Kinetic <https://wiki.ros.org/kinetic/Installation/Ubuntu>`__, create a Catkin
workspace in a directory of your choice:

.. code-block:: shell

    cd /path/to/desired/folder
    mkdir -p catkin_ws/src
    cd catkin_ws
    source /opt/ros/kinetic/setup.sh
    catkin_init_workspace src

Then clone the ``franka_ros`` repository from `GitHub <https://github.com/frankaemika/franka_ros>`__
, install any missing dependencies and build the packages:

.. code-block:: shell

    git clone --recursive https://github.com/frankaemika/franka_ros src/franka_ros
    # Install all missing dependencies
    rosdep install --from-paths src --ignore-src --rosdistro kinetic -y --skip-keys libfranka
    catkin_make -DCMAKE_BUILD_TYPE=Release -DFranka_DIR:PATH=/path/to/libfranka/build
    source devel/setup.sh

.. hint::
    If you compiled and installed ``libfranka`` systemwide as
    :ref:`described above <libfranka_systemwide>`, specifying ``Franka_DIR`` is not necessary.
    However, in this case, if you also installed ``ros-kinetic-libfranka``, ``libfranka`` might be
    picked up from ``/opt/ros/kinetic`` instead of from your custom ``libfranka`` installation in
    ``/usr``!
