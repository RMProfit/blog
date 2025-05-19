+++
title ='Blocking Numbers with e164 Pattern Map'
date = 2025-06-01
draft = true
tags= ["CUBE", "UC"]
+++


#Inesert introduction here.

<!--more-->

## Contents
- [Contents](#contents)
- [Overview](#overview)
- [Components Used](#components-used)
- [Configuration](#configuration)
- [Verification](#verification)
- [References](#references)



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
