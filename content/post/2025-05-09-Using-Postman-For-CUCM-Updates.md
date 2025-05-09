+++
title = 'Directory Number Update with Postman'
date = 2025-05-09
draft = false
tags= ["CUCM", "UC", "Postman","API","DevNet","Automation"]
+++

You're halfway through the workweek and there's an emergency, a customer needs to migrate thousands numbers from one partition to another or migrate to a new number altogether within 48 hours. What do you do? *Hint: Not Panic*

<!--more-->

## Contents
- [Contents](#contents)
- [Overview](#overview)
- [Components Used](#components-used)
- [Update Line Partition](#update-line-partition)
- [Update Lines with Variables](#update-lines-with-variables)
- [Migrate Lines with Variables](#migrate-lines-with-variables)
- [Video Walkthrough](#video-walkthrough)
- [References](#references)


## Overview
The built-in Bulk Administration Tool (BAT) for Cisco Call Manager (CUCM) is a powerful tool, but there is always a corner case scenario in which you cannot utilize it. BAT can update phones via a search or utilizing a CSV file for the device name.  

Unfortunately, the only way to migrate a list of directory number's (DN) partition is to remove the DN from the system and then re-add it. This can get messy and be prone to issues with CSV formats and nobody wants to be responsible for breaking thousands of phones, especially on a Friday.

With Postman, a few variables in our API call, and a CSV seed file, we can automate this process without fighting CUCM's formatting requirements with BAT.

## Components Used
The information in this document is based on these software versions:  
 CUCM 15  
 Postman 11.34.4

## Update Line Partition
While I will not get in to all the specifics of API calls in this post, we can cover the high level information that is required of us.
Verify you have proper API user credentials defined on the target CUCM system and that your Headers are defined like the following:  

![postman_headers](https://github.com/RMProfit/blog/blob/main/content/post/images/postman_headers.png?raw=true) 

*Note that you may need to disable SSL certificate checking for a successful authentication.*

The Cisco CUCM Administrative XML (AXL) utilizes Simple Object Access Protocol (SOAP) headers to transmit the requested actions to and from the server. The following is the first API call we will look at. 

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns="http://www.cisco.com/AXL/API/15.0">
   <soapenv:Header/>
   <soapenv:Body>
      <ns:updateLine sequence="?">
         <pattern>1001</pattern>
         <routePartitionName uuid="?">new_pt</routePartitionName>
         <newRoutePartitionName uuid="?">old_pt</newRoutePartitionName>
      </ns:updateLine>
   </soapenv:Body>
</soapenv:Envelope>
```

Ensure that the version you are calling is denoted in the first line of the of the SOAP Envelope. You can see in the url that we are using version 15:  
```
/AXL/API/15.0
```   


The body of the SOAP message is what we will focus on. We are going to initiate an updateLine request that matches DN **1001** with Partition **old_pt**. The **new_partition** line is what the DN will be modified to.
```
    <ns:updateLine sequence="?">
         <pattern>1001</pattern>
         <routePartitionName uuid="?">old_pt</routePartitionName>
         <newRoutePartitionName uuid="?">new_pt</newRoutePartitionName>
```   

  If we wanted to match a DN with no route partition assigned, we can leave the space empty between the _greater than_ and _less than_ sign.  


<routePartitionName uuid="?"```><```/routePartitionName>

The previous example is only good to update a single line or DN from one partition to another. Next, we will looking at using Postman variables to update a list of DNs.  

## Update Lines with Variables

We will now modfiy the previous API call with variables instead of a hard-coded DN. This is done simply with utilizing double braces *{{}}* to define a variable.

Here is our updated syntax:
```
      <ns:updateLine sequence="?">
         <pattern>{{dn}}</pattern>
         <routePartitionName uuid="?">old_pt</routePartitionName>
         <newRoutePartitionName uuid="?">new_pt</newRoutePartitionName>
```


Next, we need to prepare our Postman environment. Configure your specific environment variables or global variables.

![postman_variables](https://github.com/RMProfit/blog/blob/main/content/post/images/postman_variables.png?raw=true)
Create your CSV file with the same variable used in the API call and the Postman environment. The first cell in a column will be the variable and the rest of the cells in that column will be the different possbilities of the variable. Or, in other words, what DNs we want to modify.

![single_var_excel](https://github.com/RMProfit/blog/blob/main/content/post/images/single_var_excel.png?raw=true)

Next we will open the "Runner Tab" in Postman and select our CSV seed file. We can also configure a delay, in the event you are modifying thousands of DNs or objects against a server, as to not overrun it.  

![runner_tab](https://github.com/RMProfit/blog/blob/main/content/post/images/runner_tab.png?raw=true)

You can also preview the uploaded CSV and review it was read correctly by Postman.  
![runner_preview](https://github.com/RMProfit/blog/blob/main/content/post/images/runner_preview.png?raw=true)

Now, click **Run New Collection** and if all goes well, you should be greeted with *200 OK*s meaning it was successful.

## Migrate Lines with Variables
Multiple variables can be used within Postman and the CSV file making this tool even more powerful for mass updating of CUCM objects. With the following API call, we can match on an "Old DN" variable and hard coded Partition and migrate to a "New DN" variable while keeping the same Partition.
```
   <soapenv:Body>
      <ns:updateLine sequence="?">
         <pattern>{{oldDN}}</pattern>
         <routePartitionName uuid="?">Partition</routePartitionName>
         <newPattern uuid="?">{{newDN}}</newPattern>
      </ns:updateLine>
```
The CSV is updated accordingly.

![multi_var_excel](https://github.com/RMProfit/blog/blob/main/content/post/images/multi_var_excel.png?raw=true)  

 A video walkthrough of this example can be found at the link below.  


## Video Walkthrough
[Update CUCM Directory Numbers with Postman](https://youtu.be/TDrmnm_ZtiY?si=UU7f2-y4DA-lGETd)

## References
[Version 15 Cisco Unified CM AXL Schema Reference](https://developer.cisco.com/docs/axl-schema-reference/)  
[Cisco Devnet](https://developer.cisco.com/)