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
    |       Host         |
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

This lab demonstrates how to configure a Linux machine as a router, providing NAT and DHCP services to an internal network.

**Topology**
   - External interface: wlo1 (Internet)
   - Internal interface: enp44s0 (LAN 192.168.137.0/24)

**Steps:**

1. Open Linux Network Settings.

2. Assign a static IPv4 address to the internal network interface.
```
192.168.137.1/24
```

3. To enable IPv4 forwarding, edit "/etc/sysctl.conf" and add the following line at the end of the file:
```
net.ipv4.ip_forward=1
```

4. Apply the changes:
```
sudo sysctl -p
```

5. Configure NAT (Masquerading) to allow internal clients to access the internet.

   - _Note: To make these rules persistent, configure them to load at boot._

```
sudo iptables -t nat -A POSTROUTING -o wlo1 -j MASQUERADE
sudo iptables -A FORWARD -i wlo1 -o enp44s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i enp44s0 -o wlo1 -j ACCEPT
```

6. If the client devices are not configured with manual IP addresses, install the "isc-dhcp-server" package to provide DHCP services.

7. Check and add configurations in "/etc/dhcp/dhcpd.conf".
```
(comment both lines)
#option domain-name "example.org";
#option domain-name-servers ns1.example.org, ns2.example.org;

(uncomment the line)
authoritative;

(add to the end file)
subnet 192.168.137.0 netmask 255.255.255.0 {
  range 192.168.137.100 192.168.137.200;
  option routers 192.168.137.1;
  option domain-name-servers 8.8.8.8;
}
```

8. Bind the DHCP service to the correct interface by editing "/etc/default/isc-dhcp-server".
```
INTERFACESv4="enp44s0"
```

9. Enable and start the DHCP service.
```
sudo systemctl enable --now isc-dhcp-server
```

10. Done!


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
