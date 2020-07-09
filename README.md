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
+ For all other, non-local, IP addresses, it knows the IP address of a device called a "gateway."
+ See Appendix I below for a description of netmask.

### A gateway is a special-purpose host, with multiple network devices on multiple LANs
+ Gateways pass data between LANs in a way that the peer-to-peer nature of networking is maintained
+ Broadcast data do not cross LAN boundaries

## General background summary

This peer-to-peer model, with its minimal, one-time configuration, is perhaps the main reason for the Internet's success and ubiquity:  there is only one fundamental way to connect any two hosts on the planet.

## NAT-specific background

### A TCP/IP connection is used for dialog between two hosts, and is identified by four parameters:
+ The IP address and the port number of one host e.g. 192.168.1.123:12345, or A:B
+ The IP address and the port number of the other host e.g. 123.1.168.192:54321, or C:D.

Ports are 16-bit unsigned integers; many port numbers have specific meanings e.g. 443 is secure web service, 44818 is part of Ethernet/IP.

*N.B. for the purposes of this description, do not get lost in the meaning of the ports e.g. a web browser connection to a web server, or an Ethernet/IP connection; that is for the two hosts to decide; for the topic at hand it is enough to know only that some two-way dialog takes places over a "connection," and that connection is uniquely identified by these four parameters*

Those four parameters {A:B,C:D}, define a unique, __re-usable__ connection between two hosts, and are embedded in every TCP/IP transmission (packet).

Typically, one of the hosts, the client (e.g. A above) initiates a dialog by connecting from a port (B, picked at random) on itself to a Known Port (D) on the other host, called the server (C).  Since the destination IP is embedded in every packet, every gateway between A and C (and there could be dozens) will know the destination IP and will therefore know how to route the data.

### This is another key aspect of the Internet, that each gateway needs to know only two things:
+ Either that a packet is destined for one of the LANs to which the gateway is connected;
+ Or the next gateway to which to pass a packet if not destined for one of its LANs.

## Why NAT exists

Not every host (or LAN) needs to, nor should, be directly accessible from every other host on the planet.
+ For one thing, there are not enough IP addresses (~4 billion) in the 32-bit IPV4 address space.
+ Directly accessible means to be able to initiate a connection to a host on a non-local LAN through a gateway.
+ E.g. the outside world, or accountants on a business LAN, do not need to initiate a connection to a PLC on the production LAN.
+ However, an HMI may to send timestamped event data, to a host outside the production LAN.
+ Or a SCADA system on the production LAN communicating with PLCs may need to send data outside the production LAN.

## What NAT does in a gateway

The why of NAT boils down to the need for a __one-way__ gateway controlling traffic to and from private LANs
+ Hosts on the private (production) LAN need to be able to directly access the public (business) LAN and beyond,
+ But the private LAN hosts need to be invisible to direct access from the public LAN

## How NAT works

### The problem

The one-way nature of such a special-purpose gateway breaks the first background rule (above):  "All hosts in an IP network addressing scheme are peers."  Breaking that rule breaks the networking process.  Because if a host on the private LAN initiates a connection to a host on the business LAN, the public LAN host will not be able to respond with data back through the one-way gateway to take part in the dialog necessary to transmit the data.  However, ...

### "There is no problem than cannot be solved in software with another layer of indirection."

Note that the public-LAN network device of the one-way gateway accessible to all hosts on its public-side LAN.  NAT gateways take advantage of this by modifying packets from and to hosts on the private LAN.  Per the example above, say a connection is initiated from [host:port] = [A:B] the private LAN to [host:port] = [C:D] on the public LAN; the unique connection {A:B,C:D} will be embedded in the packet data.  The NAT gateway replaces the [A:B] i.e. (the private LAN host:port) in the packet with [A':B'], where A' it the NAT gateway's IP address on the public LAN, and B' is a random port, and then sends the packet with modified in source address, to its destination [C:D]; at the same time, it caches all three host:port pairs, [A:B,C:D,A':B'], in a table.  __The modification from [A:B] to [A':B'] is called a Network Address Translation (NAT).__  The public destination host
+ receives the packet,
+ sees [A':B'] as the source address;
+ and embeds source=[C:D] and destination=[A':B'] embedded in its response packet

So the public host has no idea the address was changed, and "believes" it is carrying on a dialog with the NAT gateway, while in reality the NAT gateway is acting as a proxy for the original host on the private LAN.

Since destination [A':B'] is the public interface of the NAT gateway, that packet is delivered as far as the NAT gateway, which then
+ checks the source [C:D] and destination [A':B'] against the table,
+ finds the cached entry,
+ replaces [A':B'] with [A:B],
+ and delivers the packet to the original host on the private LAN.

The original host sees the original [A:B,C:D] pair and is none the wiser, treating it as if both hosts were bi-directionally accessible.

## Appendix I:  netmask

As mentioned above, any host can immediately determine if an IP address is on its local LAN or on another, non-local LAN; the __netmask__ is an essential piece of this determination.

IP addresses comprise 32 bits.  For display, simplicity of entry, and readability by humans, they are typically broken up into four 8-bit octets, expressed in decimal notation and separated by periods (full-stops):  e.g. 192.168.1.16.  The range for each octet is 0 to 255 decimal, 00 to ff hexadecimal.  So 192.168.1.16 could be expressed as c0.a8.1.10 (c0 hex = 192 decimal; a8=168; 1=1; 10=16), or even 11000000.10101000.00000001.00010000 in binary (base 2)

An IP netmask also comprises 32 bits, and a netmask and an IP address define the address range for a single LAN.  Specifically, the 1 bits in the netmask indicate which bits in an IP address compose a unique signature for a LAN; the bit-wise AND result of [IP address,netmask] of each hosts on a LAN must be the same.  In the example above, 192.168.1.16, a typical netmask would be 255.255.255.0, or 11111111.11111111.11111111.00000000 in binary.  So the bit-wise AND result is

    11000000.10101000.00000001.00010000   IP address
    11111111.11111111.11111111.00000000   netmask
    ===================================
    11000000.10101000.00000001.00000000   ANDed result, = c0.a8.1.10 (hex) = 192.168.1.0 (decimal)

The ANDed result, called the network, is often combined with the netmask to express a unique identifier for the network, in this cas

    192.168.1.0/255.255.255.0

or sometimes simply

    192.168.1.0/24

since a netmask almost always comprises some quantity of ones on the left followed by zeros until the 32 bits are complete.

So the range of valid IP addresses on this sample network is 192.168.1.0 through 192.268.1.255;

N.B. there is no requirement that the break between ones and zeros occurs at an octet boundary, e.g. 192.168.0.0/255.255.254.0 = 192.168.0.0/23 is a valid network with IP addresses ranging from 192.168.0.0 to 192.168.1.255; such netmasks are common.  Nor is there a logical requirement that all ones be contiguous, but that is probably never done.


## Caveats II

For the sake of conciseness and clarity, this description is very much a simplified view.  E.g. it
+ Provides a definition of LAN that is functional, fuzzy, and lacking in rigorous detail.
+ Ignores any distinction between routers and gateways.
+ Blurs the lines between the Data Link, Network, and Transport layers
+ Assumes TCP/IP, but other protocols (UDP, etc.) use the same principles.
+ Ignores other protocols involved e.g. ethernet, DHCP, DNS
+ For the most part either treats hosts as if they have only one network device, ...
+ Or blurs the line between a host and its network device(s)
+ Although all gateways will have multiple network devices, not all hosts with multiple devices are gateways.
+ Broadcast data can go inter-LAN in special circumstances
+ IPV4, not IP, addresses are 32-bits
+ Did not mention that .255 is the broadcast IP address


BTC/drbitboy 2020-07-09, cf. post #4 of [this thread](https://www.plctalk.net/qanda/showthread.php?t=125511).
