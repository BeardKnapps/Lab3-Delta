# Lab Project
 Building small business netowrk.

## Objective:

Starting with a WAN-SWITCH & WAN CLOUD build out the following:

- a secure network
- an internal Windows Domain
- an internal Microsoft IIS webserver
- an internal Windows 10 workstation
- a public webserver
- a public FTP server
- a LAN network on 10.128.0.0/24
- a DMZ network on 10.128.10.0/24
- a GUEST network on 10.128.99.0/24

# Stage1: Network Setup

In Stage1, we will build out the network infrastructure for the small business environment.
The client requested a LAN, Guest, and DMZ network.
We will build these networks on a FortiNet firewall (a FortiGate) & add two ethernet switches and a Win10 workstation.

After adding the above infrastructure, the following changes were made:

Link the WAN port on the firewall to the WAN-SWITCH.
Link the LAN port on the firewall to the LAN-SWITCH.
Link the DMZ port on the firewall to the DMZ-SWITCH.
Link the Win10 workstation to the LAN-SWITCH.   

Taking our GNS3 Topology from this:
Insert picture here

To this:
Insert picture here

## Configure the LAN interface:

On a physical firewall, the LAN interface is usually preconfigured. On a virtualized firewall, you usually have to manually configure it. Once the LAN interface has been configured, you will be able to access the firewall GUI from a device on the LAN.

Per the client's request, we will configure `10.128.0.0/24` as the LAN network.
The LAN gateway will be `10.128.0.1/24`.
The LAN interface is named `port2` on the FortiGate firewall.

!(http://ntt-wiki.dvg.local/lib/exe/fetch.php?w=1000&tok=4f14c4&media=gns3-ntt-lab-stage1-instructions-config-lan-network-fw2.png)

Verify the information is correct:

!(http://ntt-wiki.dvg.local/lib/exe/fetch.php?w=1000&tok=9ff39a&media=gns3-ntt-lab-stage1-instructions-config-lan-network-fw3.png)

## Configure the DHCP server for the LAN interface:

- Enable the DHCP server on the LAN interface.
- The firewall will perform DHCP services for devices on the LAN network.
- Set the DHCP pool scope to `10.128.0.[100-199]`.

!(http://ntt-wiki.dvg.local/lib/exe/fetch.php?w=1000&tok=55ba5b&media=gns3-ntt-lab-stage1-instructions-config-lan-network-fw4.png)

Verify the information is correct:

!(http://ntt-wiki.dvg.local/lib/exe/fetch.php?w=1000&tok=00cca5&media=gns3-ntt-lab-stage1-instructions-config-lan-network-fw5.png)

# Stage1: Network Setup - Add a Win10 Workstation

Check the network settings:

Verify the Win10 workstation has leased a DHCP address from the LAN network.

!(http://ntt-wiki.dvg.local/lib/exe/fetch.php?w=1000&tok=a91c4d&media=gns3-ntt-lab-stage1-instructions-add-win10-1.png)

    - Valid IP range: 10.128.0.[100-199]/24
    - Gateway: 10.128.0.1
    - DHCP server: 10.128.0.1

 Test network connectivity:

!(http://ntt-wiki.dvg.local/lib/exe/fetch.php?w=1000&tok=14b16a&media=gns3-ntt-lab-stage1-instructions-add-win10-2.png)

# Stage1: Network Setup - Connect to the Firewall GUI

## Connect to the GUI

Open the web browser on the Win10 workstation and connect to the firewall GUI.

## Make System Changes

- Set the hostname and timezone.
- Enable the NTP server service. This will allow LAN and DMZ devices to sync time with the firewall.
- Set idle logout to 60 minutes. By default, the GUI will auto logout after 5 minutes of inactivity.
- Enable auto file system check. This tells the firewall to run a system check during boot-up. Enabling this prevents a nagging warning message that pops up during login after an unclean shutdown (power loss/forced reboot/etc). 

Apply the changes.

Backup the Configuration

Reboot the Firewall

# Configure Network Interfaces

Per the client's request, The following needs to be configured:

- `10.128.0.0/24` as the LAN network
- `10.128.99.0/24` as the GUEST network
- `10.128.10.0/24` as the DMZ network

This will provide each network with 254 available addresses: `X.X.X.[1-254]`.
The "dot 1 address" `X.X.X.[1]` will be the firewall's address for each interface.

1. edit port1
alias = WAN
role = WAN
#APPLY-THE-CHANGES

2. edit port2
alias = LAN
role = LAN
create address object matching subnet = enabled
administrative access = https, http, ping, ssh
dns server = same as interface IP
expand advanced
ntp server = local
#APPLY-THE-CHANGES

3. edit port3
alias = GUEST
role = LAN
ip/network mask = 10.128.99.1/24
create address object matching subnet = enabled
administrative access = ping, ssh
dhcp server = enabled
dns server = same as interface IP
expand advanced
ntp server = local
#APPLY-THE-CHANGES

4. edit port4
alias = DMZ
role = DMZ
ip/network mask = 10.128.10.1/24
create address object matching subnet = enabled
administrative access = ping
#APPLY-THE-CHANGES

5. dns database = enabled
#APPLY-THE-CHANGES

6. dns servers = specify
primary dns server = 8.8.8.8
secondary dns server = 1.1.1.1
#APPLY-THE-CHANGES

7. dns server = same as interface IP

8. Create New
interface = LAN(port2)
mode = recursive
#APPLY-THE-CHANGES

9. Create New
interface = GUEST(port3)
mode = recursive
#APPLY-THE-CHANGES

10. Create New
interface = DMZ(port4)
mode = recursive
#APPLY-THE-CHANGES

11. Create New > Service Group
name = LAN-services-group
members = ALL_ICMP, NTP, RDP, SSH, Web Access, Windows AD

12. Create New > Service Group
name = DMZ-services-group
members = ALL_ICMP, FTP, RDP, SSH, Web Access

13. Create New
name = LAN-to-WAN
incoming interface = LAN
outgoing interface = WAN
source = port2 address
destination = all
service = all
NAT = enabled
#APPLY-THE-CHANGES
