# How to Install and run Pi-Hole on a physical server as a recursive ad filter

How to set up Pi-Hole with Unbound to make a recursive DNS sinkhole. This project was inspired by Craft Computing on YouTube: https://www.youtube.com/watch?v=FnFtWsZ8IP0
He uses VMs for his server, I happened to have an extra Opitplex MF lying around so I repurposed it.

## Objective
Set up pi-hole via Ubuntu Server to block ads network-wide and increase privacy. Pi-hole will become your DNS server or a DNS sinkhole that will intercept and redirect DNS requests to a non-routable address for certain set of domain names, aka your filter list.
Our pi-hole will also be using unbound as a validating, recursive, caching DNS resolver.

### How it works
A client asks the pi-hole server for a domain.
Pi-hole will check its cache first for any blocked ad and/or tracker domains.
If it isn't cached, pi-hole will check its blocking list.
If the domain isn't blocked, pi-hole forwards the request to unbound.
Unbound resolves the query by consulting authoritative DNS servers across the internet. The authoritative DNS server provides true DNS records for specific domain names and IP addresses.
The answer recursively gets answers back 
Client > pi-hole > unbound > DNS server > unbound > pi-hole > client

### Skills Learned


### Tools Used
Hardware Requirements
- Dell OptiPlex 3060 MFF (alternatively, raspberry-pi, or any device with at least 4GB RAM)
    - This device should be able to run 24/7
- 4 GB RAM minimum
- USB thumb-drive
- Ethernet Cable, optional

Software Requirements 
- Ubuntu Server iso
- Rufus
- PuTTY, or any other method for SSH

### Alternative to using a physical server
Alternative to running a physical server, or installing Ubuntu Server on a device, you can use a hypervisor to great a virtual machine of Ubuntu Server that will run pi-hole.
Choose a Hypervisor and install it
   - I am using VWWare Workstation Pro, but you can use Virtual Box, Proxmox, etc.

Load the Ubuntu Server into VMWare
   - set VM to 4GB RAM
   - Set 2 Cores
   - everything else, settings set to default
   - Note IP

Install PuTTY for SSH

SSH into server, install pi-hole starting with step 6 below.

## Steps
1. Download Ubuntu Server 22.04 iso

2. Download Rufus 4.5 portable

3. Run Rufus portable executable to load the Ubuntu Server iso onto the thumb drive.

4. USB Boot the OptiPlex to install Ubuntu Server 22.04
    - Write down the IP address of the server during install
    - Keep most settings default, just set your admin account to whatever name and password you so choose
    - When prompted, install OpenSSH
    - Select reboot at the very end to reboot and update server

5. Log into the server
    - You can also SSH in from another device; I use PuTTY

6. Install Pi-Hole with the following command:
    sudo curl -sSL https://install.pi-hole.net | bash
        - Use all the default settings
        - Install the web interface

7. Create custom web interface password using the following command replacing the "enter password" with your chosen password without the <> brackets
    pihole -a -p <enter password>

8 Log into pi-hole web interface to check that it is up and running
    - Using a web browser navigate to your noted IP address: X.X.X.X/admin

9. Install Unbound with the following commands:
    - sudo apt update
    - sudo apt install unbound


10. Unbound needs a config file. Create the config file using the following command:
    sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf

11. Use pi-hole's example config file for unbound from pi-hole's website. Copy and paste the config from pi-hole into the server
    - To exit, press CTRL+X
    - To save, press Y
    - Press Enter to finally exit

12. Finalize pi-hole's setting. Go back into pi-hole web interface > settings > DNS Tab > Unchecked Google DNS > Enter Upstream DNS Server Custom 1:
    127.0.0.1#5335
    Click Save

13. Reboot the server

14. Change the DNS on your router to the IP of your Ubuntu Server.

15. Time to test pi-hole
    - Go to any website with a lot of ads, in this case, we'll be using cnn.com, buzzfeed.com, and we see that there are ads everywhere. Specifically, CNN has an advertisement banner at the very top. If pi-hole is running, you will see a grey box that just says "advertisement."
    - Go back to pi-hole and you should see queries and blocked domains in your pi-hole's dashboard

~fin
