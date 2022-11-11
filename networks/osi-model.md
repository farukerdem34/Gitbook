# OSI Model

The Open Systems Interconnection (OSI) Model is a logical model that is used to represent the data that flows from our computer to another computer. This is commonly used to troubleshoot network issues and is a very useful tool in understanding how networks pack and unpack data.

The OSI Model is structured in 7 layers as shown below:

<figure><img src="../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

When traffic is routed from our computer to another device, it is first 'packed' from Layer 7 down to Layer 1 from the sender, and then flows from Layer 1 to Layer 7 on the recipient's machine. Each layer of the model would sometimes add headers to the packet about the sender and recipient.

There are 3 sections of the OSI Model:

* Hardware Layers
  * Physical
  * Data Link
  * Network
* Transport - Heart of OSI
* Software Layers
  * Session
  * Presentation
  * Application

Each layer has their own data format of which they transfer information, and it varies from layer to layer.

## Physical

The Physical layer represents the actual physical connection between machines that transmits data. This can refer to the copper or fiber optic wires that used to transmit information in the form of **bits**. This layer would transfer stuff in 1s and 0s for Layer 2 to put back together.

Examples of Physical Layer devices are:

* Copper wiring
* Shielded Twisted Pair cables
* RJ-45 Connectors&#x20;

## Data Link

The Data Link layer is responsible for the node-to-node delivery of the message. The main function of this layer is to ensure that the data transfer occurs without error when going from one node to another through the Physical Layer. When a packet arrives in a network, the Data Link Layer would transmit it to the Host using its MAC address.

This layer can thus be sub-divided into 2 sub-layers:

* Logical Link Control&#x20;
* Media Access Control

The packet received is **divided into frames,** which is the main medium of transfer in this layer. This layer would encapsulate the Sender and Receiver's MAC address into the packet header.&#x20;

Layer 2 technologies include:

* Switches
* MAC Addresses
* Address Resolution Protocol (ARP)

ARP is a protocol that is used to link IP addresses with MAC addresses (see Packets). As such, the main purpose of this layer is **framing and adding physical addresses for the frames.**

## Network

The Network layer is in charge fo transmission of data from one host to another host that exists in a different network. This takes cares of the routing needed to move the packets to their destination.&#x20;

The sender's IP address is also placed on the header by this layer.

Layer 3 Technologies include:

* Routers
* IP addresses
* Routing protocols, like OSPF and EIGRP
* Ping Packets / ICMP
* VPN and IPSec

The main data being transmitted&#x20;

## Transport

## Session

## Presentation

## Application

