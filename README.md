# sharing-wifi-to-cable

## Overview

Let’s assume you have two devices in your lab. One of them needs to be connected using an Ethernet cable, while the other device has two network options available: Wi-Fi and Ethernet.

At first, creating a network bridge between Wi-Fi and Ethernet might seem like the correct solution. However, in practice, this usually does not work as expected. Wi-Fi operates on the IEEE 802.11 standard, which handles frames differently from traditional Ethernet (IEEE 802.3). Because of this difference, most Wi-Fi adapters running in client mode cannot forward Layer-2 broadcast traffic — such as DHCP requests — through a software bridge.

As a result, the bridged device often fails to receive an IP address from the router, making the bridge approach unreliable in typical lab environments.

To solve this limitation, we will use Network Address Translation (NAT) instead of a Layer-2 bridge. With NAT enabled, the device connected via Ethernet does not need to receive an IP address directly from the main router. Instead, it obtains a private IP address from the host device, which then forwards the traffic to the Wi-Fi network and the internet.

In this setup, the host machine effectively acts as a small gateway, providing DHCP, routing, and outbound connectivity for the Ethernet-connected device. Although this approach does not place both devices in the same broadcast domain, it is reliable, simple to configure, and perfectly suitable for lab environments or temporary installations.

This makes NAT the most practical solution when only one physical wired connection is available and the primary network access is through Wi-Fi.

## Diagram
```
        +-----------+
        | Internet  |
        +-----------+
               |
            Ethernet
               |
        +-----------+
        |  Router   |
        +-----------+
               |
 ((( ((( Wi-Fi signal ))) )))
               |
    +--------------------+
    |      Host          |
    |  WI-FI + Ethernet  |
    |    (NAT/DHCP)      |
    +--------------------+
               |
            Ethernet
               |
        +-----------+
        |  Client   |
        +-----------+
```

## Linux

_Creating a Lab Network_

## Windows

Let’s follow the steps:

1. Press WIN + R, or open the Start Menu → Run.

2. Type ncpa.cpl and press Enter.

3. Locate both network interfaces: Wi-Fi and Ethernet, and make sure they are active.

4. Right-click the Wi-Fi interface and choose Properties.

5. Open the Sharing tab and enable "Allow other network users to connect through this computer’s Internet connection.", and click OK.

6. Read the alert message informing that 192.168.137.1 will be assigned as the IP address.
   - This address will act as the gateway.
   - The internal network will be 192.168.137.0/24.
   - Windows will automatically create a DHCP service and may require removing any manual IPv4 configuration on the second device.

7. On the second device connected via Ethernet, ensure the network interface is set to automatic (DHCP) and verify that a new IP address in the range 192.168.137.x is assigned.

8. After receiving the IP address, configure the DNS server (for example, 8.8.8.8) if it is not set automatically.

9. Well done! The device should now have Internet connectivity through the Windows host.
