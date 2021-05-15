## Overview

When we got a dog I thought it might be a good idea to see what she was doing while we were at work or at 
night when she was wandering around downstairs.  I probably didn't come up with this great idea soon 
enough otherwise she wouldn't have destroyed two expensive CDs that came through the post or chewed some
coats that were hanging up!

Anyway, I did finally get a basic setup using [zoneminder](https://zoneminder.com) working but found this to 
be a bit flaky and unreliable (issues with the cheapo cameras I was using and complexity of the setup) but more 
importantly it was too complicated for the simple needs that I have.

Instead I've switched to using [motioneye](https://github.com/ccrisan/motioneye/wiki) as this is a much simpler 
application but provides just enough features.  

Update: I've also migrated away from using the single board computer (SBC) that was previously running things 
(a [PCEngines APU2](https://www.pcengines.ch/apu2e4.htm)) as this can't cope with 4 video cameras at all.  Now using 
Raspberry Pi 4 devices as these seem to handle video better (built in support for H264 encoding/decoding) and work out
cheaper also.


## Architecture

### Topology


```
                                            Home Network
                                                  |
                                 +----------------+-------------+--------------------------------+----------- .....
                                 |                              |                                |
                                 |                              |                                |
                         +-------+--------+             +-------+--------+               +-------+--------+
                         |   Tweedledum   |             |   Tweedledee   |               |   Whiterabbit  |     
                         |  Raspberry Pi  |             |  Raspberry Pi  |               |  Raspberry Pi  |   ...... 
                         +-------+--------+             +-------+--------+               +-------+--------+ 
                                 |                              |                                |
                                 |         +--------------------+                  +-------------+
                                 |         |                                       |         +------------------- ....
                                 |         |                                       |         |
                       +---------+---------+-----+                       +---------+---------+-----+
                       |                         |                       |                         |   
           +-----------+        POE Switch       |                       |        POE Switch       |
           |           |                         |                       |                         |
           |           +----+----+--------+-----+                        +--------+----+-----------+
           |                |    |        |                                       |    |
           |                |    |        +--------------------+                  |    |
           |                |    +------------+                |                  |    +---------------+
           |                |                 |                |                  |                    |
     +-----+-----+    +-----+-----+     +-----+-----+    +-----+-----+     +------+------+     +-------+------+
     |  Internal |    |  Internal |     |  External |    |  External |     |  Internal   |     |  Internal    |
     |  Camera 1 |    |  Camera 2 |     |  Camera 3 |    |  Camera 4 |     | RPi Camera 1|     | RPi Camera 2 |
     +-----------+    +-----------+     +-----------+    +-----------+     +-------------+     +--------------+
```


### Hardware

#### CCTV Server

Using two Raspberry Pi 4 Model B 2Gb devices and one currently one 4Gb.  Intend to add another 2Gb device.


#### Network switch 

Using two [Netgear GS503P](https://www.amazon.co.uk/NETGEAR-Gigabit-Ethernet-Internet-Splitter/dp/B072BDGQR8/ref=redir_mobile_desktop?ie=UTF8&aaxitk=vHZTFEIydaJReqxvAAobuw&hsa_cr_id=2001183330402&ref_=sb_s_sparkle) 
switches which means I can run a max of 8 cameras (currently have 6, will be adding the remaining two later).


#### Cameras

* [SV3C Dome Camera](https://www.amazon.co.uk/gp/product/B06VWLCZTV/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
* [SV3C Camera](https://www.amazon.co.uk/gp/product/B01G1U4MVA/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

Both of these are 1080p and include automatic night vision - essentially they switch on infrared LEDs when
the light level drops, you can here the small click of a relay when this happens.

These are simply setup to stream data, I'm not making use of any other features on these cameras.

I've also got three Hikvision cameras from my old outside CCTV system that was professional wired in (and then rather 
unprofessionally unwired by me when we moved house)


#### Raspberry Pi Cameras

I have these ready to be setup but currently not connected.

These are [Raspberry Pi 2 Model B](https://www.amazon.co.uk/Raspberry-Pi-Model-Desktop-Linux/dp/B00T2U7R7I/ref=sr_1_4?keywords=Raspberry+Pi+2&qid=1562046592&s=electronics&sr=1-4) devices.

They are connected using [PoE Splitter](https://www.amazon.co.uk/gp/product/B01H37XQP8/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

It would be much better now to use [Raspberry Pi 3 Model B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) devices and the [PoE HAT](https://www.raspberrypi.org/products/poe-hat/)



## Setup

### Initial Setup

Each Raspberry Pi server has a standard Raspbian (buster) install with SSH enabled.

Also make sure that WiFi and bluetooth are disabled

```
rfkill block wifi
rfkill block bluetooth
```


### Install Software

#### Packages

Run the following commands to install the required packages

```
apt install autofs iptables iptables-persistent
```


#### Docker

Run the following as root 

```
apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -

echo "deb [arch=armhf] https://download.docker.com/linux/raspbian/ $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
apt update
apt install docker-ce docker-ce-cli containerd.io
```

### Configuration

#### Autofs

Insert a flash, external SSD/HDD, or other storage drive into a USB3 port and then partition with `fdisk`.  There 
should be a single partition taking up the entire disk.  Format using the following command

```
mkfs -t ext4 /dev/sda1
```

This will give output similar to the following

```
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 30043904 4k blocks and 7512064 inodes
Filesystem UUID: b9bc6b4c-dd6a-4a60-aa0d-d41e45ba9963
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done  
```

Make a note of the `Filesystem UUID`, then create the `/etc/auto.vol` file (see the contents under 
`server/etc/auto.vol`) and insert that UUID value into this file.  Relying on `/dev` names for volumes just 
doesn't seem to work very well, hence the use of UUID.

Also create the file `/etc/auto.master.d/vol.autofs` with the contents from the matching file in this 
repository: `server/etc/auto.master.d/vol.autofs`

Create the `/vol` directory then reload autofs, using these commands

```
mkdir /vol
systemctl enable autofs
syatemctl restart autofs
```

You should now be able to check the existence of the mount and filesystem using `ls /vol/events` - it should only 
show the `lost+found` directory at the moment.  The `df -h /vol/events` command should report the size of the 
filesystem.

#### Firewall setup

I don't trust the cameras to not be trying to connect cloud infrastructure or other dubious activites so I'm being
ultra paranoid and locking the networking down (hopefuly) so only the operations that are needed are permitted.

Copy the `server/etc/iptables/rules.v4` file to `/etc/iptables/` directory.  Then run the following command as root

```
iptables-restore /etc/iptables/rules.v4
```

NOTE: It would be a *REALLY* good idea to have a serial console connection to the Raspberry Pi before mucking 
around with firewall rules, just in case something goes wrong.


#### Creating Motioneye User

Run the following commands

```
groupadd -g 3010 motioneye
useradd -u 3010 -g 3010 -m -c "Motion Eye user" -d /home/motioneye motioneye

```


#### Create Motioneye home files

First become the `motioneye` user

```
sudo bash -l
su - motioneye
```

Next create the var filesystem

```
mkdir -p var/log
mkdir -p var/run
```

Copy all the of the files under `server/home/motioneye/etc/<machine>` to `/home/motioneye/etc` on the server



## Running Motioneye

Motioneye can be run from Docker even on Raspberry Pi, run the following command

```
docker run --name="motioneye"  \
 -p 8081:8081 \
 -p 8765:8765 \
 --user 3010:3010 \
 --hostname="motioneye" \
 -v /etc/localtime:/etc/localtime:ro \
 -v /home/motioneye/etc:/etc/motioneye \ 
 -v /home/motioneye/var/run:/var/run \
 -v /home/motioneye/var/log:/var/log \
 -v /vol/events/motioneye:/var/lib/motioneye \
 --restart="always" \
 --detach=true \ 
 ccrisan/motioneye:master-armhf
```






## Notes

### Recording with VLC

As an interim measure it's possible to [record an RTSP stream with VLC](https://confluence.bethel.edu/display/ITSKB/Recording+a+Network+Stream+with+VLC+Player)

For Hikvision Camera URL is `rtsp://<ip>:554/11`

For Newer SV3C Camera URL is `rtsp://<ip>:554/Streaming/Channel/101`



### Core files

Noticed a core file that was dumped by some process under `/home/motioneye/etc` ran the following to see where it came from

```
file core
core: ELF 32-bit LSB core file, ARM, version 1 (SYSV), SVR4-style, from '/usr/bin/motion -n -c /etc/motioneye/motion.conf -d 5', real uid: 3010, effective uid: 3010, real gid: 3010, effective gid: 3010, execfn: '/usr/bin/motion', platform: 'v7l'
```

This has actually come from the container so need to run `gdb /usr/bin/motion /etc/motioneye/core` from there


## References


* [Raspberry Pi Camera Streaming](https://tutorials-raspberrypi.com/raspberry-pi-security-camera-livestream-setup/)
* [RTSP URL for HikVision Cameras](https://www.use-ip.co.uk/forum/threads/hikvision-rtsp-stream-urls.890/)
