NIC Software
===============
This repository contains drivers, utilities and development libraries for Silicom ultra-low-latency network cards (fb2CG@KU15P and others). 

What's an FPGA-based NIC?
-----------------
The NIC range from Silicom feature world-leading latency performance, precision timing, a simple and flexible programming interface, and true hardware extensibility through FPGA based reconfiguration.

Once the drivers are installed, the NICs present as normal network cards under Linux and many of the features are available through standard Linux APIs, however there are also additional tools and libraries that unlock the full performance and feature set.

An **nic-config** utility provides an overview of device configuration and status at a glance.

An **nic-capture** utility is provided for packet capture.  With appropriate configuration, NICs can provide lossless line rate capture at 10G.  Accurate hardware timestamps are provided for each packet, to 3.2ns resolution for most Silicom NICs.

For low latency applications, Linux sockets applications can be accelerated with the **sock** wrapper that hooks sockets calls and sends data directly to the card, bypassing the kernel.  No recompilation is necessary.  Alternatively, developers can access the card directly, including sending and receiving packets, through the **libnic** API.  Sock also provides an extensions API that allows a hybrid model, where sockets are used for the majority of TCP functions but bypassed on the critical path.

Hardware traffic filtering and steering features are available to reduce host application load.

Advanced users with specific network processing needs can also program the onboard FPGA to develop custom network functions in hardware.

Repository Contents
-------------------
- **modules** - NIC and sock drivers for Linux
- **libs** - libnic and sock libraries
- **util** - NIC utilities including nic-config, nic-capture, nic-clock-sync and nic-fwupdate
- **scripts** - sock wrapper script
- **perf_test** - tools for performing performance tests with NICs as well as with devices from other vendors
- **examples** 
	- **sock** - sock related examples (multicast receive, timestamping, extensions API)
	- **filters** - filtering and traffic steering examples
- **debian** - Debian/Ubuntu packaging
- **nic.spec** - Redhat/CentOS packaging

Installation
------------
To install from source please run ``make`` and ``sudo make install`` from the top level.

Support
-------
For other questions and comments, you can contact us directly.

