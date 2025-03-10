The ExaNIC documentation used to be found online at https://exablaze.com/docs/exanic/

The Nexus SmartNIC User Guide is here: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/internals.html
The V5P card general specs are here: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/v5p.html
The V9P card is here: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/v9p.html
HPT info is here: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/exanic-hpt.html
ExaNIC GM is here: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/exanic-gm.html
JTAG for X10 and X40 is here: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/exanic-x10x40.html


Step 1: Program exasock allows existing socket programs to use kernel bypass without modifying them: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/sockets.html

Step 2: The SmartNIC User Guide for libexanic when modifying existing programs: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/libexanic.html

Step 3: Write your own FPGA code with the FDK: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/fdk/user-guide/cisco-nexus-3550-fdk-user-guide.html


ExaNIC Internals: https://www.cisco.com/c/en/us/td/docs/dcn/nexus3550/smartnic/sw/user-guide/cisco-nexus-smartnic-user-guide/internals.html
One of the key differences between a Cisco Nexus SmartNIC (formerly ExaNIC) and most other network cards is that the SmartNIC is optimised for the lowest possible latency. This focus on latency comes with a few design decisions that differ from conventional NICs.

Receive Architecture
To keep latency low, there is very little buffering done by SmartNIC cards, with data received from the network sent through to host memory as soon as possible. This could be considered an extreme version of "cut-through networking".

In host memory, the receive (RX) buffers are implemented as 2 megabyte ring buffers. There are up to 33 ring buffers per port: the default buffer (0) and up to 32 flow steering buffers.

As a packet comes off the wire, data is sent to the host in 'chunks' of 128 bytes, consisting of 120 bytes of data and 8 bytes of metadata. (This size was chosen as a trade-off between bandwidth and latency, and is assumed throughout the current architecture; it is not user-configurable.) Software normally waits for a whole frame to arrive before starting processing, however for some applications it may be useful to process frames chunk-by-chunk, which is possible with the SmartNIC API.

If the hardware gets more than 2 megabytes ahead of software, packets may be lost (this is described in the API as a "software overflow", or informally as "getting lapped"). The lack of flow control allows multiple applications to receive packets from the same receive buffers with no performance penalty. However, it means that keeping up with the packet rate matters more than for conventional cards.

It is possible to achieve loss-free 10G line rate reception from a single buffer if attention is paid to CPU allocation (e.g. pinning the receive thread to a free CPU) and if the work done by the receive thread is kept to a minimum. Larger receive buffers are a common request and may be implemented in the future for capture applications. Note however that if an application is lagging by 2 MiB of data, then the packets it is receiving are many milliseconds old. Thus larger buffers are not a good solution for low-latency applications. Drops in a low-latency application are an indication of application slowness or hiccups that should be investigated.

Flow steering and flow hashing can be used to distribute load to multiple buffers and increase the effective buffer space while still maintaining low latency for important flows. Note that, for latency and PCI Express bandwidth reasons, the SmartNIC only ever delivers each packet to one buffer. If one application needs to listen to flows A and B and another needs to listen to flows B and C, then either all the flows need to be directed to one buffer, or each application needs to listen on two buffers.

Transmit Architecture
Unlike the receive buffers which are located in host RAM, the transmit (TX) buffers are located on the SmartNIC itself. They can be written directly by software via a memory mapping (but not read). To transmit a packet, software writes the packet data into the transmit buffer (together with a short header), and then writes a register on the SmartNIC to start packet transmission.

The amount of transmit buffer memory varies from card to card, further details can be found here.

Applications can request parts of the transmit buffer, at a minimum granularity of 4KiB. The SmartNIC driver usually also acquires one 4KiB chunk for packets being sent from the kernel, although this can be disabled by setting the interface to bypass-only mode with exanic-config. (Note that exasock will not function correctly in this mode, however, since it relies on the kernel driver for TCP slow path processing; it is only useful when using libexanic exclusively.)

The packet header written to the TX buffer together with each packet contains a feedback slot index and feedback ID. After sending the packet, the card writes the given ID to the given slot (within a feedback structure in host memory). This allows software to determine when packet transmission is complete.
