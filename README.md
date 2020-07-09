# NAT - Network Address Translation


## Caveats I

There are exceptions to all of this, e.g. see Caveats II below, but the basic scheme described here is how the 'net works.

## General background

### All hosts in an IP network addressing scheme are peers

That is, each host communicates, at the IP level, with all other hosts in the same way, which is via TCP/IP connection.

### Each host, specifically each network device (ethernet card) on a host, is a member of exactly one LAN (Local Area Network).
An alternate name for a LAN is a broadcast domain.  A LAN is a group of network devices that can communicate via broadcast; that is, one host on a LAN can commumicate with one target host on the same LAN by broadcasting data to that LAN with enough address information that the target host will be able to determine that the data are directed to the target host, while all other hosts on the LAN will know to ignore those data.

### One of the ways that each host is a peer is that it only needs to know a few things to communicate with any other host:
+ It knows its own IP address
+ It knows the netmask of the LAN it is connected to
+ Using the netmask, can determine if an IP address is local i.e. is on the same LAN
+ For all other, non-local, IP addresses, it knows the IP address of a router on the same LAN

### A router is a special-purpose host, with multiple network devices on multiple LANs
+ Routers pass data between LANs in a way that the peer-to-peer nature of networking is maintained
+ Broadcast data does not generally cross LAN boundaries

This peer-to-peer model, with minimal configuration that can be automated, is perhaps the main reason for the Internet's success and ubiquity:  there is only one fundamental way to connect any host to any other host on the planer.

## NAT-specific background

### A TCP/IP connection is used for dialog between two hosts, and is identified by four parameters:
+ The IP address and the port number of one host e.g. 192.168.1.123:12345, or A:B
+ The IP address and the port number of the other host e.g. 123.1.168.192:54321, or C:D.

Typically, one of the hosts, the client (e.g. A above) initiates a dialog by connecting from a random port (B) on itself to a Known Port (D) on the other host, called the server (C).

If the server is "listening" on that Known Port D, it accepts the connection and dia

## Why NAT exists

Not every host (or LAN) needs to, nor should, be directly accessible from every other host on the planet.
+ For one thing, there are not enough IP addresses (~4 billion) in the 32-bit IPV4 address space.
+ Directly accessible means to be able to initiate a connection to a host on a non-local LAN through a router.
+ E.g. the outside world, or even the people in accounting, do not need to initiate a connection to a PLC the production LAN.
+ However, the people in accounting will need to initiate a connection out to the internet e.g. to a remote headquarters.
+ Or an HMI will need to send timestamped event data, to a host outside the production LAN.
+ Or a SCADA system on the production LAN communicating with PLCs will need to send data outside the production LAN.

What this boils down to is the need for a special-purpose, __one-way__ router controlling traffic to and from production LANs
+ Hosts on the production LAN need to be able to directly access the business LAN and beyond,
+ But the production LAN hosts need to be invisible to direct access from the business LAN

## How NAT works

"There is no problem than cannot be solved in software with another layer of indirection."






## Caveats II

This discussion is very much a simplified view e.g. it
+ Provides a definition of LAN that is functional, fuzzy, and not detailed
+ Blurs the lines between the Data Link, Network, and Transport layers
+ Assumes TCP/IP, but other protocols (UDP, etc.) use the same principles.
+ Mostly excludes other protocols and e.g. ethernet, DHCP, DNS
+ For the most part either treats hosts as if they have only one network device, ...
+ Or blurs the line between a host and its network device(s)
+ Although all routers will have multiple network devices, not all hosts with multiple devices are routers

