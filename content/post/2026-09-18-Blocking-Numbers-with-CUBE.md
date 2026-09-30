+++
title ='Blocking Numbers with CUBE'
date = 2026-09-18
draft = true
tags= ["CUBE", "UC"]
+++


Long ago, one may have received a spam call maybe once a month. Advancing technology introduced robo calling and number spoofing. The ability to automate unsolicited calls for marketing or spam has increased the need for better posturing and filtering at the edge. This post will cover an easy way to block those calls from your network.   
<!--more-->  


## Contents
- [Contents](#contents)
- [Overview](#overview)
- [Components Used](#components-used)
- [Configuration](#configuration)
- [Verification](#verification)
- [References](#references)

## Overview
This post is to demonstrate a way of blocking unsolicited calls at your gateway before they enter your network. While these calls can be blocked on CUCM with some strategic partition, calling search space, and route pattern configurations, the traffic is already in your system and inside network. The overhead of dropping that traffic falls on your CUCM with a high probability of causing a DoS event to your users. A better solution is to block this traffic at the gateway before the calls enter your network.

## Components Used
This configuration can be implemented on Cisco routers running CUBE.

## Configuration
First we need to define the number(s) we intend to block. Think of the voice translation rules as an ACL (access control list) in which the list is assessed from the top-down format. The following configuration blocks *8675309*. Unlike ACLs, there is no implicit deny at the end. If a number does not match, it is simply not translated or, in our case, it is not blocked. Additionally, the `/.*/` can be used to block all numbers/matches.
```
voice translation-rule 3001
 rule 1 reject /8675309/
```
Alternatively, for a longer list of numbers, an e164-pattern-map is more efficient than individual reject rules. The main configuration below uses the pattern map.
```
voice class e164-pattern-map 3001
 url http://<server>/pattern-map.cfg
```



**Basic Dial Peer Configuration Prior to Blocking**
```
dial-peer voice 1 voip
 description SIP from ISP
 session protocol sipv2
 voice-class sip bind control source-interface GigabitEthernet0/0/1
 voice-class sip bind media source-interface GigabitEthernet0/0/1
 dtmf-relay rtp-nte
 no vad
```
In the configuration that follows, we are matching inbound calling numbers against our e164 pattern map file. After they match the dial-peer and before they are forwarded, the translation profile "Inbound-CallBlock" is engaged. This, in turn, references translation-rule 3002, which rejects all numbers with the wildcard match of ".*" This can be read as match "any digit repeating any number of times".

```
voice translation-rule 3002
 rule 1 reject /.*/

voice translation-profile Inbound-CallBlock
 translate calling 3002
```

**Working Dial Peer With Blocking Added**
```
dial-peer voice 1 voip
 description Block from ISP with Blocking
 call-block translation-profile incoming Inbound-CallBlock
 call-block disconnect-cause incoming call-reject
 session protocol sipv2
 incoming calling e164-pattern-map 3001
 voice-class sip bind control source-interface GigabitEthernet0/0/1
 voice-class sip bind media source-interface GigabitEthernet0/0/1
 dtmf-relay rtp-nte
 no vad
```

We still need a dial-peer to match legitimate traffic destined for our network. The following dial-peer satisfies this requirement. The `.T` matches any calling number, but the pattern map on dial-peer 1 is a more specific match, so only numbers not in the pattern map land on dial-peer 2.

**Working Dial Peer to Match Legitimate Traffic**
```
dial-peer voice 2 voip
 description Legitimate Traffic from ISP
 session protocol sipv2
 answer-address .T
 voice-class sip bind control source-interface GigabitEthernet0/0/1
 voice-class sip bind media source-interface GigabitEthernet0/0/1
 dtmf-relay rtp-nte
 no vad
```

**Alternative: Blocking a Small List of Numbers on a URI-Matched Dial Peer**

If your dial-peers already use SIP URI matching instead of ANI-based matching (for example, to separate SIP trunks or tenants), you can add call blocking to that existing dial-peer directly, without needing an e164-pattern-map or a second dial-peer. This fits a small, static list of numbers to block, since the reject patterns live inline in the translation-rule.

```
voice class uri 201 sip
 host ipv4:10.10.10.1
```

```
voice translation-rule 3003
 rule 1 reject /8675309/

voice translation-profile URI-CallBlock
 translate calling 3003
```

```
dial-peer voice 3 voip
 description SIP from ISP (URI Matched)
 session protocol sipv2
 incoming uri via 201
 call-block translation-profile incoming URI-CallBlock
 call-block disconnect-cause incoming call-reject
 voice-class sip bind control source-interface GigabitEthernet0/0/1
 voice-class sip bind media source-interface GigabitEthernet0/0/1
 dtmf-relay rtp-nte
 no vad
```

Note: `call-block` only applies to the dial-peer it's configured on. If multiple URI-matched dial-peers exist (for example, one per tenant or trunk), the `call-block` commands must be added to each dial-peer where blocking should apply, adding it to just one does not protect the others.

## Verification
You don't need a live call to confirm a number will be blocked. The `test voice translation-rule` command runs a number against a translation-rule and reports the result.

A number that matches a reject rule:
```
Router# test voice translation-rule 3003 8675309
 blocked on rule 1
```

A number that does not match any rule:
```
Router# test voice translation-rule 3003 5551234
5551234 Didn't match with any of rules
```

To confirm the rules and profiles are configured as expected:
```
show voice translation-rule 3003
show voice translation-profile URI-CallBlock
```

If you are using an e164-pattern-map, confirm the file loaded from the URL and check the number of entries:
```
show voice class e164-pattern-map 3001
```
Note: testing translation-rule 3002 isn't useful here, since `/.*/` blocks every number. In the e164-pattern-map design, the pattern map decides which numbers are blocked, so check the pattern map instead.

## References
- [Understand IOS and IOS XE Call Routing](https://www.cisco.com/c/en/us/support/docs/voice/ip-telephony-voice-over-ip-voip/211306-In-Depth-Explanation-of-Cisco-IOS-and-IO.html)
- [Determine Voice Translation Rules](https://www.cisco.com/c/en/us/support/docs/voice/call-routing-dial-plans/61083-voice-transla-rules.html)
- [Configure Number Translation with Voice Translation Profiles (call-block feature)](https://www.cisco.com/c/en/us/support/docs/voice/call-routing-dial-plans/64020-number-voice-translation-profiles.html)
- [Configuring Multiple Pattern Support on a Voice Dial Peer](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/voice/cube_fund/configuration/xe-3s/cube-fund-xe-3s-book/cube-fund-xe-3s-book_chapter_01000.pdf)
- [Cisco IOS Voice Command Reference – show voice class e164-pattern-map](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/voice/vcr4/vcr4-cr-book/vcr-s9.html)
- [Configuring an Inbound Dial Peer to Match on a URI](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/voice/cube_fund/configuration/xe-3s/cube-fund-xe-3s-book/voi-inbnd-dp-match-uri.pdf)


