
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
Raspberry Pi 4 devices 


## Setup

### Topology


```
                                            Home Network
                                                  |
                                 +----------------+-------------+--------------------------------+----------- .....
                                 |                              |                                |
                                 |                              |                                |
                         +-------+--------+             +-------+--------+               +-------+--------+
                         |   Tweedledum   |             |   Tweedledum   |               |   Tweedledum   |     
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

This started out as a PC Engines [APU2](https://www.pcengines.ch/apu2.htm) with 4Gb memory 240Gb SSD and an attached
1Tb HDD for storing images/video from the CCTV.

The APU2 has three NICs which allows connection to the home network and cameras to be physically seperate
and for traffic between these to be blocked.  The reason for this paranoia is that I bought cheap IP cameras
and I simply didn't trust that they would be secure or stable enough.

A further reason for this setup is to minimise network traffic on my home network - with 4 cameras there is 
quite alot of data flowing through the Switch and I really don't want that soaking up bandwidth from the 
rest of the devices I have in the house.

The CCTV server is running Ubuntu 18.04 LTS Server.

As of writing I'm now upgrading to using a home-built PC, AMD Phenom X6, 12Gb memory, 500Gb for boot drive and two 
further 500Gb HDD for video/images.  The reason for the upgrade is that I'm expanding the CCTV to cover outside as 
well as in and the APU2 is running at 300% CPU and response on the UI is slow.


#### Network switch 

Originally I was using a [Netgear GS108PEv3](https://www.amazon.co.uk/NETGEAR-GS108PEv3-Power-Over-Ethernet-Lifetime-Protection/dp/B00LMXBOG8/ref=sr_1_3?keywords=nether+8+port+PoE&qid=1562046271&s=gateway&sr=8-3)

The four PoE ports are used to power the cameras and Raspberry Pi's

I've now switched (no pun intended) to using two [Netgear GS503P](https://www.amazon.co.uk/NETGEAR-Gigabit-Ethernet-Internet-Splitter/dp/B072BDGQR8/ref=redir_mobile_desktop?ie=UTF8&aaxitk=vHZTFEIydaJReqxvAAobuw&hsa_cr_id=2001183330402&ref_=sb_s_sparkle) switches which means I can run a max of 8 cameras (currently have 5, will be adding more later).


#### Cameras

* [SV3C Dome Camera](https://www.amazon.co.uk/gp/product/B06VWLCZTV/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
* [SV3C Camera](https://www.amazon.co.uk/gp/product/B01G1U4MVA/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

Both of these are 1080p and include automatic night vision - essentially they switch on infrared LEDs when
the light level drops, you can here the small click of a relay when this happens.

These are simply setup to stream data, I'm not making use of any other features on these cameras.

I've also got three Hikvision cameras from my old outside CCTV system that was professional wired in (and then rather 
unprofessionally unwired by me when we moved house)


#### Raspberry Pi's

These are [Raspberry Pi 2 Model B](https://www.amazon.co.uk/Raspberry-Pi-Model-Desktop-Linux/dp/B00T2U7R7I/ref=sr_1_4?keywords=Raspberry+Pi+2&qid=1562046592&s=electronics&sr=1-4) devices.

They are connected using [PoE Splitter](https://www.amazon.co.uk/gp/product/B01H37XQP8/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

It would be much better now to use [Raspberry Pi 3 Model B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) devices and the [PoE HAT](https://www.raspberrypi.org/products/poe-hat/)



## Configuration

### Server Network

Ubuntu 18.04 uses [netplan](https://netplan.io) the configuration files for the main network interface 
(`enp6s0`) and the two further connections (`enp5s0` and `enp4s0`) which form a bridge are under 
`server/etc/netplan`


### Iptables

To prevent the cameras from talking via the server to the internet or internal network I've added firewall 
rules to restrict the traffic.  These can be found under `server/etc/iptables`.


### CCTV Software


#### Running

Motioneye can be Docker

Run the following command

```
docker run --name="motioneye"  \
 -p 8765:8765 --user 3010:3010 \
 --hostname="motioneye" \
 -v /etc/localtime:/etc/localtime:ro \
 -v /home/motioneye/etc:/etc/motioneye \ 
 -v /home/motioneye/var/run:/var/run \
 -v /home/motioneye/var/log:/var/log \
 -v /vol/events/motioneye:/var/lib/motioneye \
 --restart="always" \
 --detach=true \ 
 ccrisan/motioneye:master-amd64
```


### Raspberry Pi's





## Notes

### Recording with VLC

As an interim measure it's possible to [record an RTSP stream with VLC](https://confluence.bethel.edu/display/ITSKB/Recording+a+Network+Stream+with+VLC+Player)



## References

* [Raspberry Pi Camera Streaming](https://tutorials-raspberrypi.com/raspberry-pi-security-camera-livestream-setup/)
* [RTSP URL for HikVision Cameras](https://www.use-ip.co.uk/forum/threads/hikvision-rtsp-stream-urls.890/)
