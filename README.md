<a id="top"/> 
# cob4 Installation

### Contents

1. <a href="#Installation">Installation</a>
2. <a href="#Extra-Installation">Extra Installation</a>
     1. <a href="#Hands">Hand configuration</a>
     2. <a href="#Mimic">Mimic</a>
     3. <a href="#Touch">Calibration touchscreen</a>


### 1. Installation <a id="Installation"/> 

The full Care-O-bot installation can be done using a bash script. The script is in the setup repository, get it using the following command:

```
wget https://raw.githubusercontent.com/ipa320/setup_cob4/master/InstallCob4.sh
chmod +x InstallCob4.sh
```

The installation script needs the parameters robot, ip address and installation mode, where:

 * -r robot: is the robot name (cob4-X)
 * -ip :ip address for your actual installation pc, for example \texttt{10.4.IP.11} for base pc.
 * -m installation mode: on Care-O-bot there are two different types of computers, the master pc and the slave. The master PC has a large hard disk, and works as a NFS system server, the other computers will be the clients

The script allow different types of installation:

+ **Basic Installation** It is composed by the following steps:

  * Install basic tools (vim, meld, terminator ...)
  * Install and configure openssh
  * Allow robot user to execute sudo command without password
  * Setup root user (in this step the user will be asked for a password)
  * Install ROS
  * Setup udev rules
  * Setup bash environment 

+ **Setup NTP and NFS** This option will configure the NFS system depending on the installation mode, it is important that the master pc is already installed and per network reachable before install the slave computers, otherwise the installation process will be cancelled. After this installation it is necessary restart the computer.
 
+ **Full Installation** A full installation means the combination of 1 and 2 (Basic Installation + Setup NTP and NFS)
+ **Cob setup** This step holds the recommended configuration of the robot home directory. This step can only be execute after a full installation.

### 2. Extra Installation <a id="Extra-Installation"/> 
#### 2.1. Hands configuration <a id="Hands"/>
The hands use a bluetooth connection to receive the commands and send the link positions to ROS. This requires the configuration of the bluetooth devices on the hands (Raspberry pcs) and on the torso pc, also some upstart jobs are needed to launch the hand driver on boot.

##### Hand Pcs:

Add a rule (*/etc/udev/rules/98-bluetooth*) to identify the bluetooth device as a serial port:

```
KERNEL=="rfcomm*",GROUP="dialout",MODE="0666",SYMLINK+="ttyBridge"
```

Create the *cob_hand_bridge* service (*/etc/systemd/system/cob_hand_bridge.service*):
```
[Unit]
Description=Virtual Distributed Ethernet

[Service]
ExecStart=/usr/bin/rfcomm watch rfcomm0 1 cob_hand_bridge /dev/ttyBridge

[Install]
WantedBy=multi-user.target
```
And enable the service with the command:
```
sudo systemctl enable cob_hand_bridge.service
```
##### Torso Pcs:
Uninstall the modemmanager package to avoid any bluetooth interference:
```
sudo apt-get purge modemmanager
```
Setup the bluetooth configuration (*/etc/bluetooth/rfcomm.conf*):

```
rfcomm0 {
bind yes;
device **B8:27:EB:67:31:B4**; (hand hci device address)
channel	1;
comment "Bluetooth hand right";
}
```
Add the following upstart job (*/etc/init/cob_hand.conf*):
```
# auto connect cob hand

start on started bluetooth
stop on runlevel [!2345]

respawn
respawn limit 0 10

script
  rfcomm connect rfcomm0 
end script
```

#### 2.2. Mimic <a id="Mimic"/>
The mimic should be installed on head pc. A special user "mimic" has to be created to control the display. After create the user add the following lines to */etc/lightdm/lightdm.conf* :

```
[SeatDefaults]
autologin-user=mimic
autologin-user-timeout=60
```

And also the following autostart job to */u/mimic/.config/autostart/xhost.desktop* :
```
[Desktop Entry]
Type=Application
Exec=xhost +
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
Name[en_US]=mimic
Name=mimic
Comment[en_US]=
Comment=
```

#### 2.3. Calibration touchscreen <a id="Touch"/>

The touchscreen driver can be found under http://zytronic.co.uk/support/downloads/# , after install the driver use the following command to invert the axis and calibrate the panel:
```
 sudo ZyConfig
```
<a href="#top">top</a>
