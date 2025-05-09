+++
title = 'Dial Peers Unencrypted Post Upgrade'
date = 2025-01-17
draft = false
tags= ["CUBE", "Crypto", "UC", "SRTP"]
+++

I ran in to an issue in a customer's production environment a few months back with a voice stream not negotiating Secure Real Time Transport Protocol (SRTP) even though it was configured correctly on the customer's edge router, or so we thought.

<!--more-->

## Contents
- [Contents](#contents)
- [Overview](#overview)
- [Components Used](#components-used)
- [Issue](#issue)
- [Feature Change](#feature-change)
- [References](#references)

## Overview
As much as we would love to have the time to check every little feature, syntax change, and widget that may or not be changed between versions of code, things slip through the cracks at times. This goes for all code version changes to include server applications, routers, and phones. This is especially true when some configurations have been around for years and have never had previous issues.
After a recent upgrade of IOS on the CE router, the router was no longer sending the encrypted attributes, although it was present in the running configuration.

## Components Used
The information in this document is based on these software and hardware versions:
- Cisco 44XX series routers
- IOS versions
  - 17.5 and above
  - 17.6 and below

## Issue
This is the existing cofiguration on the CE router for the dial-peer with address information removed, of course.
```
dial-peer voice 100 voip
  description Interface to/from Service Provider
  session protocol sipv2
  session target ipv4:x.x.x.x
  session transport tcp tls
  destination e164-pattern-map 1
  incoming uri via PROVIDER
  voice-class codec 1
    voice-class sip srtp-auth sha1-32 sha1-80
  voice-class sip early-offer forced
  voice-class sip profiles 1
  voice class sip bind control source-interface GigabitEthernet0/0/1
  voice class sip bind media source-interface GigabitEthernet0/0/1
  dtmf-relay rtp-nte
  srtp
  no vad
```

The traffic was as follows for the SIP traffic between the CE and PE routers.
First, is the received SIP INVITE on the CE from the PE router:
```
RCECEIVED INVITE: (all crpto avail)
Received: 
INVITE sip:3157146976@domain n:5061;maddr=x.x.x.x
Via: SIP/2.0/TLS y.y.y.y:5061
o=CiscoSystemsSIP-GW-UserAgent 4516 4203 IN IP4 x.x.x.x
a=crypto:1 AEAD_AES_256_GCM inline:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
a=crypto:2 AEAD_AES_128_GCM inline:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
a=crypto:3 AES_CM_128_HMAC_SHA1_80 inline:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
a=crypto:4 AES_CM_128_HMAC_SHA1_32 inline:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Next, we see that the ACK is sent back to the PE router:

```
SENT OK Back: (ack that Pac wants to use AES 256)
Sent: 
SIP/2.0 200 OK
Via: SIP/2.0/TLS y.y.y.y:5061;
v=0
o=CiscoSystemsSIP-GW-UserAgent 8722 23 IN IP4 x.x.x.x
a=crypto:1 AEAD_AES_256_GCM inline:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```  
  
The provider then told us that they did not currently support AES_256, even though they were sending it out in there intiial INVITE. 
What we couldn't figure out, is that once that AES_256 was removed from ```srtp-ath``` comamnd, why was the router not ACK'ing the SHA-80 even though it was in the running configuration under the dial-peer and simply ACK'ing the AESA-256.


```
  voice-class codec 1
    voice-class sip srtp-auth sha1-32 sha1-80
```
 The documentation states the the router will accept the following crypto suits by default:
```
	
From Cisco IOS XE Everest Release 16.5.1b onwards, the following crypto suites are enabled by default on the SRTP leg:

AEAD_AES_256_GCM
AEAD_AES_128_GCM
AES_CM_128_HMAC_SHA1_80
AES_CM_128_HMAC_SHA1_32
``` 
  
So the router is working as designed by default, but not how it was being configured?
(Insert lots of head scratching here)

 ## Feature Change
After checking, double checking, and triple checking, we finally found a nice little footnote in one of the configuration guides. 

```
Effective Cisco IOS XE Everest Releases 16.5.1b, srtp-auh command is deprecated. 
Although this command is still available in Cisco IOS XE Everest software, executing 
this command does not cause any configuration changes. Use voice class srtp-crypto command 
to configure the preferred cipher-suites for the SRTP call leg (connection). For more 
information, see SRTP-SRTP Interworking.
```

Since this was a recently upgraded router, the ```srtp-auth``` command was left over from previous versions of code. Even though it was still present in the running configuration, it is essentially doing nothing.

The newer verbiage of syntax would need the following for the dial-peer to configure a prefferred order of crypto suites for SRTP:  

  
Newer Syntax Version
```
In Gloal Configuration Mode

voice class srtp-crypto tag
crypto preference cipher-suite

In dial-peer configuration Mode
dial-peer voice tag voip

voice-class sip srtp-crypto crypto-tag
```


Older Syntax Version
```
In dial-peer configuration mode:

dial-peer voice tag voip
voice-class sip srtp-auth {sha1-32 | sha1-80 | system}
```



## References
[Cisco Unified Border Element Configuration Guide Through Cisco IOS XE 17.5](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/voice/cube/configuration/cube-book/srtp-rtp-interworking.html)
[Cisco Unified Border Element Configuration Guide - Cisco IOS XE 17.6 Onwards](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/voice/cube/ios-xe/config/ios-xe-book/m_voi-srtp-rtp-int.html)
