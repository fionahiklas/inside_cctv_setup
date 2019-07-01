## Overview

When we got a dog I thought it might be a good idea to see what she was doing while we were at work or at 
night when she was wandering around downstairs.  I probably didn't come up with this great idea soon 
enough otherwise she wouldn't have destroyed two expensive CDs that came through the post or chewed some
coats that were hanging up!

Anyway, I did finally get a basic setup using [zoneminder](https://zoneminder.com) working but found this to 
be a bit flaky and unreliable as well as seeming like an awful lot of services/facilities when all I 
needed was a really simple UI and video/recording.


## Topology


```
 	                                 	 +----------------+
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



## References

* [Raspberry Pi Camera Streaming](https://tutorials-raspberrypi.com/raspberry-pi-security-camera-livestream-setup/)
