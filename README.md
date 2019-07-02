## Overview

When we got a dog I thought it might be a good idea to see what she was doing while we were at work or at 
night when she was wandering around downstairs.  I probably didn't come up with this great idea soon 
enough otherwise she wouldn't have destroyed two expensive CDs that came through the post or chewed some
coats that were hanging up!

Anyway, I did finally get a basic setup using [zoneminder](https://zoneminder.com) working but found this to 
be a bit flaky and unreliable as well as seeming like an awful lot of services/facilities when all I 
needed was a really simple UI and video/recording.

## Setup

### Topology


```
                                            Home Network

                                                 |
                                                 | 
 	                                 	 +-------+--------+
    	                       	         |      CCTV      |
	                       	        	 |     Server     |
	                       	        	 |     (APU2)     |
	                                 	 +--------+-------+
                        						  |
						                          |
						                          |
                                     +------------+-------------+
           +-------------------------+   8-Port Switch with     |
      	   | 	            +--------+       4-port PoE         +----------+
           |                |        +--------------+-----------+	       |
    	   |		        |			            |	                   |
    	---+--- 	     ---+---		            |	                   |
      -/       \-      -/       \-		            |      	               |
     /   Dome    \    /	 Bullet   \	          +-----+-----+	         +-----+-----+
     |  Camera   |    |	 Camera   |		      | Raspberry |	         | Raspberry |
     \           /    \	          /		      |   Pi 2    |	         |   Pi 2    |
      -\       /-      -\       /-		      +-----------+	         +-----------+
    	-------          -------
```


### Hardware

#### CCTV Server

This is a PC Engines [APU2](https://www.pcengines.ch/apu2.htm) with 4Gb memory 240Gb SSD and an attached
1Tb HDD for storing images/video from the CCTV.

The APU2 has three NICs which allows connection to the home network and cameras to be physically seperate
and for traffic between these to be blocked.  The reason for this paranoia is that I bought cheap IP cameras
and I simply didn't trust that they would be secure or stable enough.

A further reason for this setup is to minimise network traffic on my home network - with 4 cameras there is 
quite alot of data flowing through the Switch and I really don't want that soaking up bandwidth from the 
rest of the devices I have in the house.

The CCTV server is running Ubuntu 18.04 LTS Server.

#### Network switch 

This is a [Netgear GS108PEv3](https://www.amazon.co.uk/NETGEAR-GS108PEv3-Power-Over-Ethernet-Lifetime-Protection/dp/B00LMXBOG8/ref=sr_1_3?keywords=nether+8+port+PoE&qid=1562046271&s=gateway&sr=8-3)

The four PoE ports are used to power the cameras and Raspberry Pi's

#### Cameras

* [SV3C Dome Camera](https://www.amazon.co.uk/gp/product/B06VWLCZTV/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
* [SV3C Camera](https://www.amazon.co.uk/gp/product/B01G1U4MVA/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

Both of these are 1080p and include automatic night vision - essentially they switch on infrared LEDs when
the light level drops, you can here the small click of a relay when this happens.

These are simply setup to stream data, I'm not making use of any other features on these cameras.


#### Raspberry Pi's

These are [Raspberry Pi 2 Model B](https://www.amazon.co.uk/Raspberry-Pi-Model-Desktop-Linux/dp/B00T2U7R7I/ref=sr_1_4?keywords=Raspberry+Pi+2&qid=1562046592&s=electronics&sr=1-4) devices.

They are connected using [PoE Splitter](https://www.amazon.co.uk/gp/product/B01H37XQP8/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

It would be much better now to use [Raspberry Pi 3 Model B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) devices and the [PoE HAT](https://www.raspberrypi.org/products/poe-hat/)



## Configuration

### Raspberry Pi's



### CCTV Software






## References

* [Raspberry Pi Camera Streaming](https://tutorials-raspberrypi.com/raspberry-pi-security-camera-livestream-setup/)
