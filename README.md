# idl4-cc-rpi-install

These instructions will guide you through the installation of your Raspberry Pi as we will use it during the IDL4-CC-labs.

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

To configure your WIFI on your Raspberry Pi there are 2 options. The first one is to configure it using the GUI. The wifi-settings can be reached using the menu.

![Desktop WIFI](/img/desktop-wifi.png)

Another option is by adding it manually to the file **/etc/wpa_supplicant/wpa_supplicant.conf**. To adapt it we open it with the built-in text-editor **nano**.

    sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
    
Add the following lines on the bottom of this file.

    network={
            ssid="THE_NAME_OF_YOUR_WIFI_NETWORK"
            psk="THE_PASSWORD_OF_YOUR_WIFI_NETWORK"
            key_mgmt=WPA-PSK
    }

Of course you have to change the **ssid** and **psk** with your own network settings.

Your file should look similar like this. Close the editor by pressing **ctrl-x**

![wpa_supplicant.conf step 1](/img/wpa-1.png)

Confirm by pressing **y**

![wpa_supplicant.conf step 2](/img/wpa-2.png)

Finally press **return**

![wpa_supplicant.conf step 3](/img/wpa-3.png)

It's possible to add multiple networks. Just add them to the bottom of the file.

### Step 2

On your Raspberry Pi, open a terminal window and type

    sudo apt update
    
Next, type

    sudo apt upgrade

When this is finished, your system is up-to-date. It's wise to reboot your Raspberry Pi using

    sudo reboot
    
### Step 3 (optional)

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

### Step 4

Next we will change some network settings on the Raspberry Pi. We will give the **eth0** interface a fixed IP-address. I suggest to use **192.168.100.1** but any other valid IP-address works as well of course. The settings are stored in /etc/dhcpcd.conf. So type

    sudo nano /etc/dhcpcd.conf
    
Copy-paste the following lines to the bottom of the file.

    interface wlan0
    metric 200
    
    interface eth0
    metric 300
    static ip_address=192.168.100.1/24
    static routers=192.168.100.1
    static domain_name_servers=8.8.8.8

So what does it do? First, we give the **wlan0** interface a metric of 200 so outgoing network traffic (internet) will use this interface instead of the **eth0** interface. We also specify the metric of the **eth0** interface and give it a fixed IP-address of **192.168.100.1** with a subnet of **255.255.255.0**. From our Mac or PC, we will be able to reach the Raspberry Pi on this IP-address.

Close the editor by pressing **ctrl-x**

![dchpcd.conf step 1](/img/dhcpcd-conf-1.png)

Confirm by pressing **y**

![dchpcd.conf step 2](/img/dhcpcd-conf-2.png)

Finally press **return**

![dchpcd.conf step 3](/img/dhcpcd-conf-3.png)

Time to reboot once more!

    sudo reboot
    
### Step 5

Now it's time to connect your Mac or PC with the ethernet-cable to the Raspberry Pi.

Give your Mac or PC a fixed IP-address in the same network range as the Raspberry Pi. I use **192.168.100.2** with a subnet of **255.255.255.0** but any other valid IP-address in the same network will work. Use the IP-address from the Raspberry PI - **192.168.100.1** as the router-address.

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

### Step 6

Almost there! Our Raspberry Pi has internet connection but our Mac or PC has only access to the local network. We need to setup the Raspberry Pi so it shares it's internet connection. This is done by enabling ip-forwarding.

Open the file /etc/sysctl.conf in nano

    sudo nano /etc/sysctl.conf
    
Look for the line **#net.ipv4.ip_forward=1** and uncomment it by removing the #-symbol

Save the file by pressing **ctrl-x**

![sysctl](/img/sysctl.png)

We have to reboot so the Raspberry Pi is using the new settings.

    sudo reboot

Our Raspberry Pi has a public IP-address on the **wlan0** interface. We have to set up [**Network Address Translation**](https://en.wikipedia.org/wiki/Network_address_translation) or **Masquerading** so the internet traffic is directed to our private network. This is done by adding a line to **iptables**, the Linux firewall.

With the Raspberry Pi rebooted, type in the following command

    sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
    
Next, type    
   
    sudo iptables -t nat -L

You should get a similar result

![iptables](/img/iptables.png)

Now it's time to test if we have an internet connection. Fire up a browser on your Mac or PC and see if you can browse the internet. Alternatively you can open up a terminal window and type

    ping 8.8.8.8
    
You should get a window like this

![ping 8.8.8.8](/img/ping-internet.png)

We will save this rule to a ruleset so we can use it whenever we want.

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

## Visual Studio Code

Installing applications is a bit different on the Raspberry Pi. It's done using **apt** - a command-line utility for installing, updating, removing, and otherwise managing deb packages on Ubuntu, Debian, and related Linux distributions.

You have already used it to update your system with the command

    sudo apt update

and 

    sudo apt upgrade

Before we install new package, it's always wise to run these 2 commands.

To install Visual Studio Code you type

    sudo apt install code

Of course it's only available on the Raspberry Pi's GUI but it's a handy application. You can find it under Menu -> Programming -> Visual Studio Code.

![Visual Studioc Code](/img/code.png)

## Node-JS

Next we will install Node-.js.

First we have to add the repository for Node.js to your Raspberry Pi. You have 2 options. For the current release run the command.

    curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -

For the current Long-Time-Release run the command

    curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

Now you can install Node.js by running the command

    sudo apt install nodejs

Afterwards you can check if Node.js is correctly installed by typing

    node -v

## Node-RED

Node-RED is a programming tool for wiring together hardware devices, APIs and online services in new and interesting ways.

It provides a browser-based editor that makes it easy to wire together flows using the wide range of nodes in the palette that can be deployed to its runtime in a single-click.

It's a great tool to run on the Raspberry Pi since it can easyly be used as a back-end for your multimedia installation. There are multiple ways to run Node-RED but we will run it locally on our Raspberry Pi. More information and the complete installation procedure can be found on https://nodered.org/docs/getting-started/raspberrypi

There's a handy script that does most of the work for us. Let's run it by typing

    bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)

The script will ask if we want to run the Pi-specific nodes. This is handy if we want to use the GPIO-pins. So press **Y**.

![Node-RED script 1](/img/node-red-script-1.png)

It will ask if we want to use specific settings - we'll leave this up for now. So press **N**

![Node-RED script 2](/img/node-red-script-2.png)

You can run Node-RED by typing

    node-red

Next, open up a browser-window on your Mac or PC and go to

    192.168.100.1:1880

That's it! As you can see Node-RED is running on your Raspberry Pi.

![Node-RED window 1](/img/node-red-1.png)

You can stop Node-RED by pressing **ctrl-c**, Node-RED will be offline in your browser-window.

It's better to run Node-RED in the background. And this can be done by running it as a service.

You can start it by typing

    node-red-start

If we press **ctrl-c** you can notice Node-RED is still running in the background in your browser window.

We can stop it by typing

    node-red-stop

To have access to the Node-RED's log type

    node-red-log

You can also start the Node-RED service on the Raspberry Pi OS Desktop by selecting the Menu -> Programming -> Node-RED menu option. It will open up Node-RED's log as well.

To autostart Node-RED when we boot the Raspberry Pi we have to enable it using **systemctl**

To enable it type

    sudo systemctl enable nodered.service

Disable the service can be done with

    sudo systemctl disable nodered.service
