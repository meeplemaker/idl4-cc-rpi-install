# idl4-cc-rpi-install

These instructions will guide you through the installation of your Raspberry Pi as how we will use it during the IDL4-CC-labs.

## Prerequisites

1. Raspberry Pi 4B
2. A MicroSD with a freshly installed Raspberry Pi OS using [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
3. Acces to an RJ-45 connector on your Mac or PC
4. Ethernet cable
5. Screen, keyboard & mouse (basic installation only)
6. [VNC® Viewer](https://www.realvnc.com/en/connect/download/viewer/) installed on your Mac or PC (optional)

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
    
### Step 2 (optional)

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

    interface wlan0
    metric 200
    
    interface eth0
    metric 300
    static ip_address=192.168.100.1/24
    static routers=192/168.100.1
    static domain_name_servers=8.8.8.8

So what does it do? First, we give the **wlan0** interface a metric of 200 so outgoing network traffic (internet) is routed to this interface because the Raspberry Pi is connected to the internet with WIFI. Then, we also specify the metric of the **eth0** interface and give it a fixed IP-address of **192.168.100.1** with a subnet of **255.255.255.0**. From our Mac or PC, we will be able to reach the Raspberry Pi using this IP-address.

Close the editor by pressing **ctrl-x**

![dchpcd.conf step 1](/img/dhcpcd-conf-1.png)

Confirm by pressing **y**

![dchpcd.conf step 2](/img/dhcpcd-conf-2.png)

Finally press **return**

![dchpcd.conf step 3](/img/dhcpcd-conf-3.png)

Time to reboot once more!

    sudo reboot
    
### Step 4

Now it's time to connect your Mac or PC with the ethernet-cable to the Raspberry Pi.

Give your Mac or PC a fixed IP-address in the same network range as the Raspberry PI. I use **192.168.100.2** with a subnet of **255.255.255.0** but any other valid IP-address in the same network will work. Use the IP-address from the Raspberry PI - **192.168.100.1** as the router-address.

![network settings step 1](/img/network-settings-1.png)

Set the DNS-server to **8.8.8.8**

![network settings step 2](/img/network-settings-2.png)

Time to test the network connection. Open a terminal window on the Raspberry Pi. Type

    ping 192.168.100.2
    
You should get the following result.

![ping](/img/ping.png)

From now on it's possible to use [SSH](https://en.wikipedia.org/wiki/Secure_Shell) to access our Raspberry Pi. On our Mac or PC open a terminal window and type

    ssh pi@192.168.100.1
    
Type in the password of the Raspberry Pi (the default password: raspberry)
   
![ssh step 1](/img/ssh-1.png)

Et voilà, from now we have remote access to the Raspberry Pi.

![ssh step 2](/img/ssh-2.png)

If you have enabled VNC and installed VNC Viewer on your Mac or PC it's also possible to work on the remote desktop of the Raspberry Pi if you prefer to work with a GUI.

### Step 5

Almost there! Our Raspberry Pi has internet connection but our Mac or PC has only access to the local network. We need to setup the Raspberry Pi so it shares it's internet connection. This is done by enable ip-forwarding.

Open the file /etc/sysctl.conf in nano

    sudo nano /etc/sysctl.conf
    
Look for the line **#net.ipv4.ip_forward=1** and uncomment it by removing the #-symbol

Save the file by pressing **ctrl-x**

![sysctl](/img/sysctl.png)

Our Raspberry Pi has a public IP-address on the **wlan0** interface. We have to set up [**Network Address Translation**](https://en.wikipedia.org/wiki/Network_address_translation) or **Masquerading** so the internet traffic is directed to our private network. This is done by adding a line to **iptables**, the Linux firewall.

Type in the following command

    sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
    
Next, type    
   
    sudo iptables -t nat -L

You should get a similar result

![iptables](/img/iptables.png)

Now it's time to test if we have an internet connection. Fire up a browser on your Mac or PC and see if you can browse the internet. Alternatively you can open up a terminal window and type

    ping 8.8.8.8
    
You should get a window like this

![ping 8.8.8.8](/img/ping-internet.png)


We will save these rule to a ruleset so we can use it whenever we want.

First we make a directory where we will store the ruleset

    sudo mkdir /etc/iptables

Then, we make a file to copy the ruleset into

    sudo touch /etc/iptables/rules.v4
    
Finally, save the current rules into the ruleset

    sudo iptables-save | sudo tee /etc/iptables/rules.v4
  
Every time we boot up our Raspberry Pi we will copy these rules and add them to the firewall. Open the file /etc/rc.local

    sudo nano /etc/rc.local

And add the following line just before **exit0**

    iptables-restore < /etc/iptables/rules.v4

It should look similar like this

![rc.local](/img/rc-local.png)

Reboot your Raspberry Pi a final time

    sudo reboot
    
Check a final time if you still have internet access on your Mac or PC.    

## Node-JS

...

## Node-RED

...
