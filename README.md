## Overview


In this hands-on lab, you will explore, configure, and analyze a simple Local Area Network (LAN) using the GNS3 network emulator. The lab will walk you through launching GNS3, examining network settings, troubleshooting connectivity, and configuring firewall rules. You will also establish a new subnet within the network.

#### 1. Launch GNS3:

  * Open a terminal and enter gns3 & to launch GNS3 in the background.
  * Ignore any update check prompts.

#### 2. Network Topology:

  * Three hosts: Linux-1, Linux-2, and Firefox
  * Two switches, one router, and a firewall
  * An ISP (Internet Service Provider) node
  * An Internet-facing web server (Web)

![Image](https://github.com/user-attachments/assets/b38b37cb-3252-45e5-af63-cbf90f52bdb1)

## **Part 1: Examine the Firewall Device**

In this part of the lab, you will examine the pfSense firewall device. In this topology, the pfSense firewall device is a multi-purpose network appliance that provides firewall, router, and DHCP server functionality. In this topology, the pfSense firewall device functions as the LAN's primary router and default gateway.

First, you will need to boot up the firewall and power on the simulated network.

#### 1. Power on the firewall:

  * Right-click the pfSense device in GNS3 and select Start.
  * Wait until network interfaces turn green.
  * Open a console connection by right-clicking the firewall and selecting Console.

![Image](https://github.com/user-attachments/assets/53a8d848-f0f6-417a-885b-1ff0542612f8)

Note: The firewall can take up to 3 minutes to fully boot up.

On the GNS3 control bar, click the Start button (the large green arrow) to power on all devices.

In the GNS3 topology window, you will see all connections turn green, indicating the hosts are powered on and connected to the network.

![Image](https://github.com/user-attachments/assets/a9528900-3218-48db-b0bd-44a0cd4cb64b)

#### 2. Access the Web GUI:

  * Open a terminal on the firefox host and launch its VNC console.
  * Use the Firefox browser and navigate to pfSense.cybrary.lab.
  * Sign in using saved credentials.

**OBS:** As the host's name indicates, the TigerVNC console will launch with a Firefox window open. You will use the Firefox browser to connect to the pfSense appliance's web GUI. Although you could interact with the pfSense appliance directly through the command line interface that you saw when you started the device, the web GUI is easier to use for instructional purposes.



#### 3. Examine Network Interfaces:

  * From the menu bar at the top, Navigate to **Interfaces > LAN** and view the firewall's internal IP.

![Image](https://github.com/user-attachments/assets/1f63e1da-f8ed-47b4-871c-94007ff532de)

  * Navigate to **Interfaces > WAN**, scroll down see the firewall's internal IPv4 address.

![Image](https://github.com/user-attachments/assets/a31b8242-38b5-499a-9e87-d746b0d331c1)


Take note of the IPv4 Upstream gateway. The firewall will pass all non-local traffic to this gateway, which is the device labeled ISP. This simulated LAN connects to the "Internet" using an Internet Service Provider (ISP). The ISP's job is to route traffic to and from the Internet on behalf of the firewall. The firewall's job is to manage Internet traffic on behalf of the LAN.

#### 4. Testing Connectivity

  * From the menu bar at the top, click Diagnostics and select Ping to open the Ping diagnostic tool.
  * On the **Diagnostics > Ping page**, type 66.67.68.1 in the Hostname field, then click Ping to ping the ISP device.

A successful ping indicates proper WAN connectivity.

![Image](https://github.com/user-attachments/assets/ff717b06-e046-4c9d-a4b5-0b41aa8e9c53)

![Image](https://github.com/user-attachments/assets/16be216d-6f43-4cc7-8a49-fff0acdb230d)

## **Part 2: Examine the DHCP Server and Host Network Configurations**

In this part of the lab, you will examine the DHCP Server running on the pfSense firewall appliance. DHCP stands for Dynamic Host Configuration Protocol. DHCP grants clients an IP Address (and other networking options) from a predefined pool of addresses. Hosts on the LAN configured to use DHCP will automatically be assigned an IP address when they boot up.

#### 1. Check DHCP Settings:

  * From the menu bar at the top, click Services and select DHCP Server.
  * On the **Services > DHCP Server > LAN page**, notice that DHCP is enabled.

![Image](https://github.com/user-attachments/assets/feb549fc-2387-421e-aa14-0c4ab326dfea)

  * On the Linux-1 Console, type **cat /etc/network/interfaces** and press Enter to display the contents of the network configuration file for Linux-1.

<img width="723" alt="Image" src="https://github.com/user-attachments/assets/81558380-d129-44ae-9e74-482c482e2e9b" />

Notice that the Static configuration is commented out, but the DHCP configuration section is not. Linux-1 should get an IP Address and a gateway from the firewall's DHCP server. Let's confirm.

  * On the Linux-1 tab, type **ifconfig** and press Enter to display the network configuration. Feel free to use the more up to date ip a command as well.

![Image](https://github.com/user-attachments/assets/9471c67b-316b-4b03-bc6b-0aa50e2386f2)

Notice that Linux-1 indeed has an IP Address from the address pool on the firewall.

  * On the Linux-1 tab, type **netstat -rn** and press Enter to confirm the default gateway.

<img width="718" alt="Image" src="https://github.com/user-attachments/assets/ccb7b1d9-d106-4570-9b0a-5d14fcc8f6b3" />

**OBS**: The 0.0.0.0 means **"everything"**. Thus we are sending "everything" not local to the default gateway.

In the TigerVNC console, right-click the M icon in the lower left-hand corner, then select terminal to open a new LXTerminal inside the TigerVNC console. Repeat **cat /etc/network/interfaces** and **netstat -rn** to confirm the network configuration for the firefox host.

![Image](https://github.com/user-attachments/assets/26363bb6-eab5-41d9-9d31-ee5fccd215a6)

You will see that DHCP has also provided the firefox host an IP address and default gateway.

## **Part 3: Firewall Rules & Traffic Flow**

In this part of the lab, you will examine the pfSense appliance's firewall feature. You will begin by reviewing the rules table for the WAN interface.

#### 1. Examine Firewall Rules:
  * In the TigerVNC console, scroll up to the menu bar at the top in the open web browser, then click **Firewall > Rules**.

![Image](https://github.com/user-attachments/assets/327acc54-730c-492e-a855-3750f135180b)

Notice there are no rules configured for the WAN side of the firewall. Although you don't see any rules, there is always one rule present but hidden on every firewall interface: "deny everything". Thus, ALL inbound traffic on the WAN side is blocked.

**OBS:** PfSense is a stateful firewall; as such, it knows when a valid request is made from an internal client and will allow the reply traffic into the WAN interface without needing a written rule. Thus, traffic that is in reply to a valid request (made from an internal client) is allowed.

Click the LAN tab.

![Image](https://github.com/user-attachments/assets/328f1a84-35fe-4419-b66a-1b7f70a7a3dc)

You should see three rules on the **Firewall > Rules > LAN page**.

  * The first rule ensures we can connect to the firewall's web configuration page on port 80. 
  * The second rule allows all IPv4 traffic from the LAN network to any destination. 
  * The third rule allows all IPv6 traffic from the LAN Network to any destination.

**OBS:** While this default firewall configuration is fine for a lab, a production firewall should limit outbound traffic to only what is needed. For example, many organizations only allow secure web traffic to exit the network. Imagine the havoc malware could wreak if given free outbound Internet access from a contaminated host!

  * In the GNS3 topology window,  **Web > Console** 

  * In the Web tab, type **ifconfig** and press Enter to see the Web device’s IP Address.

![Image](https://github.com/user-attachments/assets/284a0edb-003e-4087-b4ff-b7d90860c6b0)

#### 2. Test Traffic Flow:

  * In the Linux-1 tab, ping the Web device’s IP address.
  * Given the firewall rules table, you should be able to ping the external Web device from the internal Linux-1 device.
  * In the Web tab, ping the ISP device (66.67.68.1), then ping the firewall's WAN interface.

![Image](https://github.com/user-attachments/assets/126ae03b-960a-4c91-a3be-6b76ab212fb2)


## **Part 4: Configure a New Subnet**
Next, let's focus on the Router and Linux-2 devices in our LAN. Although the Router appears to be connected to the rest of the network via Switch 1, its network configuration file is actually empty. In the next steps, we will configure the Router to accommodate a new network subnet that serves Switch2 and Linux-2: 10.10.10.0/24. The Router will therefore need two NICs: one in the 192.168.1.0/24 network and one in the new 10.10.10.0/24 network.

#### 1. Configure Router Interfaces:

  * In the GNS3 topology window, right-click the **Router > Auxiliary console** (not Console).

  * type cat **/etc/network/interfaces** and press Enter to display the router’s configuration file. 

<img width="723" alt="Image" src="https://github.com/user-attachments/assets/8497f35c-2e1f-47a9-9374-41ea33b7a6ee" />

While we could edit /etc/network/interfaces using the vi text editor, let's take advantage of this virtual environment and use a GNS3 shortcut instead.

**OBS:** You can use the vi editor if you are comfortable with it.

  * In the GNS3 topology, right-click the **Router > Configure** from the context menu.
  * At the bottom of the Node properties window, next to Network configuration, click **Edit**.

The Router interfaces editor will open.

In the Router interfaces editor, configure the router as follows, then click Save.

| Interface | IP Address      | Netmask         | Gateway       |
|-----------|---------------|----------------|--------------|
| eth0      | 192.168.1.254 | 255.255.255.0  | 192.168.1.1  |
| eth1      | 10.10.10.1    | 255.255.255.0  | N/A          |

<img width="771" alt="Image" src="https://github.com/user-attachments/assets/2db1a0ed-8568-4e07-abc8-338b375e1c23" />


#### 2. Restart Router & Verify Configuration:

  * In the GNS3 topology, right-click the Router device and select **Stop** from the context menu, then right-click the Router device again and select **Start** to restart the router.
  * In the GNS3 topology, right-click the **Router > Auxiliary console** from the context menu to re-open the Router tab in the terminal.
  
On the Router tab, type **ifconfig** and Press Enter to confirm the new IPs have been configured for eth0 and eth1 (ignore eth2).

![Image](https://github.com/user-attachments/assets/f82dd051-0b9f-405e-8c80-65b59f9b9f7b)

On the Router tab, use the **netstat -rn** command to verify that the firewall LAN address (192.168.1.1) is the default gateway. 

![Image](https://github.com/user-attachments/assets/93df63d7-94c1-4d60-9382-32fbdbdebf75)

#### 3. Configure Linux-2:

As you did with the Router, use GNS3 to configure Linux-2 as follows:

Static config for eth0
eth0 address: 10.10.10.100
eth0 netmask: 255.255.255.0
eth0 gateway: 10.10.10.1

| Interface | IP Address     | Netmask        | Gateway      |
|-----------|--------------|---------------|-------------|
| eth0      | 10.10.10.100 | 255.255.255.0 | 10.10.10.1  |

Open a Console for Linux-2. ping the Router's internal IP address (10.10.10.1), then ping the firewall's LAN IP address (192.168.1.1).

![Image](https://github.com/user-attachments/assets/3d196502-a31c-457a-a886-7794645868f5)

You should find that the Router replies, but the Firewall does not. This does not mean that your ping requests did not reach the firewall - after all, Linux-2 is aware of the Router device as its gateway, and the Router device is aware of the Firewall device as its gateway. Rather, it means that the Firewall does not know where to send its response.

The Firewall device knows about the two networks it’s connected to (192.168.1.0/24 and 66.67.68.0/24) and where to send all other traffic (66.67.68.1), but it does not know about 10.10.10.0/24, nor will 66.67.68.1 know anything about an internal network. To resolve this, we will need to tell the Firewall where to route traffic for 10.10.10.0/24. We will accomplish this by adding a routing statement that instructs the firewall to send all traffic destined for 10.10.10.0/24 to the Router.

#### 4. Add Static Route to Firewall:

* On the **System > Routing > Gateways**, click Add to add a new gateway.

18. On the **System > Routing > Gateways > Edit** page, configure the new gateway as shown below, then scroll down to the bottom and click Save.

| Disabled | Interface | Address Family | Name   | IP Address    | Gateway       |
|----------|----------|---------------|--------|---------------|---------------|--------------|
| Off      | LAN     | IPv4          | Router    | 192.168.1.254 | 

<img width="734" alt="Image" src="https://github.com/user-attachments/assets/639a6f25-8b98-486f-984c-ddc1482d256d" />

The new gateway should appear on the **System > Routing > Gateways** page.

  * On the **System > Routing > Gateways** page, select the **Static Routes** tab.
  * On the **System > Routing > Static Routes** page, click Add to add a new static route.

Configure the new static route as follows, then click Save.

| Destination network | Gateway | 
|----------|----------|
| 10.10.10.0 / 24      | Router - 192.168.1.254     | 


<img width="761" alt="Image" src="https://github.com/user-attachments/assets/8669a974-9b83-42e6-9d67-360434795e4f" />

On the **System > Routing > Static Routes** page, click Apply Changes and wait for the changes to be applied.

  * On the **Firewall > Rules > LAN** page, click the Edit button (the Pencil) for the second rule.
  * On the **Firewall > Rules > Edit** page, scroll down and change the Source from LAN net to any, then scroll down and click Save.

<img width="738" alt="Image" src="https://github.com/user-attachments/assets/555489ca-2a1a-4379-8b44-fa315c7c7ba3" />

  * On the **Firewall > Rules > LAN page**, click Apply Changes.

We should now be able to reach the firewall and beyond from the 10.10.10.0/24 LAN.

**OBS**: The first few pings may fail if you try to test too quickly after adding the new network to the firewall.
