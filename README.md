# idl4-cc-rpi-install

These instructions will guide you through the installation of your Raspberry Pi as how we will use it during the IDL4-CC-labs.

## Prerequisites

1. Raspberry Pi 4B
2. A MicroSD with a freshly installed Raspberry Pi OS using [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
3. Acces to an RJ-45 connector on your Mac or PC
4. Ethernet cable
5. Screen, keyboard & mouse (basic installation only)
6. [VNC® Viewer](https://www.realvnc.com/en/connect/download/viewer/) installed on your Mac or PC

## Basic installation

After these steps your Raspberry Pi will act as a wired router. Network traffic is forwarded in the Raspberry Pi so it is possile to access the internet while your Mac or PC is connected using an ethernet cable with the Raspberry PI.

![General network setup](/img/network.png)

### Step 1

On your Raspberry Pi, open a terminal window and type

    sudo apt update
    
Next, type

    sudo apt upgrade

When this is finished, your system is up-to-date. It's wise to reboot your Raspberry Pi using

    sudo reboot
    
### Step 2

Next we will enable VNC so it is possible to access the Raspberry desktop from the Mac or PC. In the terminal window type

    sudo raspi-config
    
Select **Interace Options**

![raspi-config step 1](/img/raspi-config-1.png)

Select **VNC**

![raspi-config step 2](/img/raspi-config-2.png)

Select **Yes** to enable VNC

![raspi-config step 3](/img/raspi-config-3.png)

**OK!**

![raspi-config step 4](/img/raspi-config-4.png)

Finally close raspi-config by selecting **Finish**

![raspi-config step 5](/img/raspi-config-5.png)

### Step 3

Next we will change some network settings on the Raspberry Pi. We will give the **eth0** interface a fixed IP-address. I suggest to use **192.168.100.1** but any other valid IP-address works as well of course. The settings are store in /etc/dhcpcd.conf. Let's open with the built-in text-editor **nano**.

    sudo nano /etc/dhcpcd.conf
    
Copy-paste the following lines to the bottom of the file.

    interface eth0
    static ip_address=192.168.10.1/24
    static routers=192/168.100.
    static domain_name_servers=8.8.8.8

    interface wlan0
    metric 200

    interface eth0
    metric 300

## Node-JS



## Node-RED
