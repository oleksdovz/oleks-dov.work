# How to Set Up NAT in Proxmox: A Step-by-Step Guide


## **Introduction:**

Network Address Translation (NAT) is a crucial component in modern networking setups, allowing multiple devices within a private network to share a single public IP address for internet access. Proxmox, a powerful open-source virtualization platform, offers robust networking capabilities, including NAT configuration. In this guide, we'll walk you through the process of setting up NAT in Proxmox, enabling efficient communication between virtual machines and the outside world.

## **Prerequisites:**

- Proxmox Virtual Environment installed and running.
- Basic understanding of networking concepts.
- Access to Proxmox web interface or command-line interface.

### **Step 1: Accessing Proxmox Web Interface:**

1. Open your web browser and navigate to the IP address of your Proxmox server, typically https://proxmox-host:8006
2. Log in with your username and password to access the Proxmox web interface.

### **Step 2: Creating a Bridge Interface:**

1. In the Proxmox web interface, navigate to the "**Datacenter**" view.
2. Click on "**Network**" in the left sidebar.
3. Click the "Create" button and select "Linux Bridge."
4. Enter a name for the bridge, such as "**vmbr100**," and click "**Create**."
5. Select the newly created bridge from the list and click "**Edit**."
6. Configure the bridge settings according to your network requirements, including IP address, subnet mask, gateway, and DNS servers.
7. Click "OK" to save the changes.

### **Step 3: Creating Virtual Machines:**

1. Navigate to the "Create VM" wizard in the Proxmox interface.
2. Follow the prompts to create virtual machines as needed, **assigning** them to the newly created bridge interface (**vmbr100**).
3. Ensure that each virtual machine is configured with its own unique internal IP address within the same subnet as the bridge interface.

### **Step 4: Configuring NAT:**

1. SSH into your Proxmox server or access the console directly.
2. Edit the `/etc/network/interfaces` file using your preferred text editor.
3. Add the following lines to the file to configure NAT:

```
auto lo
iface lo inet loopback

iface eno1 inet manual

auto vmbr0
iface vmbr0 inet static
    address 192.168.0.5/24
    gateway 192.168.0.1
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0
############

auto vmbr100
iface vmbr100 inet static
    address 192.168.100.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

    post-up echo 1 > /proc/sys/net/ipv4/ip_forward 
    post-up   iptables -t nat -A POSTROUTING -s '192.168.100.0/24' -o vmbr0 -j MASQUERADE 
    post-up   iptables -A POSTROUTING -t nat -s '192.168.100.0/24' -j MASQUERADE 
    post-down iptables -t nat -D POSTROUTING -s '192.168.100.0/24' -o vmbr0 -j MASQUERADE 

############
```

Replace `192.168.100.1/24` with your server's public IP address. Adjust the subnet (`'192.168.100.0/24'`) as per your internal network configuration.

1. Save the file and exit the text editor.
2. Restart the networking service to apply the changes:

```
sudo systemctl restart networking
```
or 
```
sudo ifdown vmbr100
sudo ifup vmbr100
```

### **Step 5: Testing NAT Configuration:**

1. Start the virtual machines created earlier.
2. Verify that the virtual machines can access the internet by pinging external IP addresses or domain names.
3. Ensure that the virtual machines can communicate with each other within the internal network.


Congratulations! You have successfully set up NAT in Proxmox, enabling internet access for virtual machines while maintaining a secure and isolated internal network environment.
