+++
title ='Blocking Numbers with CUBE'
date = 2025-06-01
draft = true
tags= ["CUBE", "UC"]
+++


#Insert introduction here.
Long ago, one may have recieved a spam call maybe once a month. Advancing technology introduced robo calling and number spoofing. The abilty to automate unsolicited calls for marekting or spam and the  
ability to hide or change the caller ID of the number you are calling, respectively. This post will cover an easy way at blocking those calls from your network.   
<!--more-->  


## Contents
- [Contents](#contents)
- [Overview](#overview)
- [Components Used](#components-used)
- [Configuration](#configuration)
- [Verification](#verification)
- [References](#references)

## Overview
This post is to demonstrate a way of blocking unsolicited calls at your gateway before they enter your network. While these calls can be blocked on CUCM with some strategic partition, calling search space, and route pattern configurations, the issue is at that point the traffic is already in your system. The overhead of dropping that traffic on falls on your CUCM with a high probability of causing a DoS event to your users. A better solution is to block this traffic at your gateway before the calls enter your network.

## Components Used
This configuration can be implemented on Cisco routers running CUBE.

## Configuration
#Working Dial Peer
```
dial-peer voice 3000 voip
 description SIP from BT
 translation-profile incoming BT-Inbound-Translation
 preference 5
 session protocol sipv2
 answer-address .T
 voice-class codec 1  
 voice-class sip copy-list 3000
 voice-class sip bind control source-interface GigabitEthernet0/0/1
 voice-class sip bind media source-interface GigabitEthernet0/0/1
 voice-class sip audio forced
 dtmf-relay rtp-nte
 no vad
```

```
voice class e164-pattern-map 3001
 url http://<server>/pattern-map.cfg
```
or

```  
voice class e164-pattern-map 3001
 url http://<server>/pattern-map.txt
```

```
voice translation-rule 3001
 rule 1 reject /.*/

voice translation-profile BT-Inbound-CallBlock
 translate calling 3001

dial-peer voice 3002 voip
 description SIP from ISP
 translation-profile incoming BT-Inbound-Translation
 call-block translation-profile incoming BT-Inbound-CallBlock
 call-block disconnect-cause incoming call-reject
 session protocol sipv2
 incoming calling e164-pattern-map 3001
 voice-class codec 1  
 voice-class sip bind control source-interface GigabitEthernet0/0/1
 voice-class sip bind media source-interface GigabitEthernet0/0/1
 voice-class sip audio forced
 dtmf-relay rtp-nte
 no vad
```

## Verification
## References


