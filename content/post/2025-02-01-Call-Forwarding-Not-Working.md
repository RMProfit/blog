+++
title = 'Unable to Call Forward All (CFA) from Phones'
date = 2025-02-01
draft = false
tags= ["CUCM", "UC", "Phones"]
+++

Issues in production are the gifts that keep on giving. Another strange one to look at as phones were unable to perform Call Fortwarding All (CFA) from phones, depending on which server they were registered to.

<!--more-->

## Contents
- [Contents](#contents)
- [Overview](#overview)
- [Components Used](#components-used)
- [Troubleshooting](#troubleshooting)
- [Issue](#issue)
- [References](#references)

## Overview
In a Call Manager cluster running 14SU3 that is split across the WAN, an interesting issue happened wherein phones registered to servers at Data Center 1 (DC1) were unable to set their physical phones to Call Forward All (CFA) when utilizing the soft key. Of course you can press the Call Forward key, and type in the forwarding number, but there were no audible tones or confirmation like there normally would be.  

 However, when phones were reigstered to DC2, everything worked as expected.  
 Also to note is that sometimes the CFA on the phone would be successful but would take up to five minutes to  display success on the phone's screen.

## Components Used
- CUCM Version 14SU3
- 88xx Phones 

## Troubleshooting
Here are the following items that were looked at in a normal round of check on a UC infrastructure.
* ```utils dbreplication runtimestate``` and ```utils dbreplication status```
  * The database output was good and there were no mismatched tables. The Round Trip Times (RTT) times were all 40ms or less between servers.
* ```utils ntp status```
  * Timing good and nothing to see here.
* ```utils diagnose test```
  * No process hogging all the cpu or memory.
* Computer Telephony Interface (CTI) logs yielded no results.

## Issue

There was one thing that was very odd. Two servers in DC1 had their Call Manager service core. This means that something blocked the CUCM service from completing a task, consumned all the memory available for that process, and then forced a restart. This can happen with any number of processes in CUCM.

Upon analyzing the backtrace it seemed that the Signal Distriution Layer (SDL) process, the process that interacts with other process and is part of call processing, had incurred some issues.

```
#3  0x586d00fd n SdlMsgQueueCirc::enqueueSignal ... at SdlMsgQueueCirc.cpp:155
#4  0x586b3045 in enqueueSignal ... ccm/Common/Include/Sdl/SdlMsgQueue.hpp:66
#5  SdlThreadedProcess::inputSignal ... at SdlThreadedProcess.cpp:164
#6  0x58690e3b in SdlRouter::callProcess ... _sdlSignal=_sdlSignal@entry=0xbe368410, _deleteSignal 
```
After some ninja-like searching on the Internet, I came acorss the following Cisco Bug:
[Normal Priority Queue Exahustion Due to Bulk Signals to Update ITL hash in DB](https://bst.cloudapps.cisco.com/bugsearch/bug/CSCwf68099)

The core / backtrace matched what I had seen, and I was also able to confirm another telltale hitting this defect of numerous messages in the CUCM/SDL logs:

```"Device type mismatch between the information contained in the device's TFTP configuration file and what is configured in Unified CM Administration for that device"```

One of the workarounds is to remove devices from the network that are no longer in use. This had also happened recently to this customer environment. A lot of deprectaed phones were recently removed from the CUCM's database, but there is no gurantee they were also physically removed from the network in the various locations.  

In this case, the large quantity of phones are continuously sending registration messages and eventually starving out the SDL queue. This causes User Facing Features (UFF) like call forwarding to not be processed by the server and 'mot work'.

The path of least resistance was to patch/upgrade the servers to 14SU4 instead attempting to verify devices had been removed from the network. The 14SU4 patch modified the error handling of the SDL queue.


## References
[Normal Priority Queue Exahustion Due to Bulk Signals to Update ITL hash in DB](https://bst.cloudapps.cisco.com/bugsearch/bug/CSCwf68099)

