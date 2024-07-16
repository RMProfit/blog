+++
title = 'Using TCL to Update E164 Pattern Maps'
date = 2024-07-13
draft = false
tags= ["CUBE", "TCL"]
+++

# Using TCL for E164-Pattern-Map Modification

## Contents
- [Using TCL for E164-Pattern-Map Modification](#using-tcl-for-e164-pattern-map-modification)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Components Used](#components-used)
  - [Create Initial Pattern-Map File and Load into Memory](#create-initial-pattern-map-file-and-load-into-memory)
        - [Detailed Explanation](#detailed-explanation)
      - [File Creation Example with Data](#file-creation-example-with-data)
      - [Append Data to an Existing File](#append-data-to-an-existing-file)
      - [Example: Create the File and Remove a Single Line or Pattern](#example-create-the-file-and-remove-a-single-line-or-pattern)
        - [Creating the file](#creating-the-file)
      - [Removing a Single Line from the File](#removing-a-single-line-from-the-file)
      - [Removing Entries](#removing-entries)
        - [Remove a Single Line from an Existing File](#remove-a-single-line-from-an-existing-file)
        - [Detailed Explanation](#detailed-explanation-1)
      - [Remove Multiple Single Entries](#remove-multiple-single-entries)
        - [Detailed Explanation](#detailed-explanation-2)
      - [Use RegEx to Match Multiple Numbers](#use-regex-to-match-multiple-numbers)
      - [Match an Entry with Wild Cards Present](#match-an-entry-with-wild-cards-present)
        - [Example: Removing Multiple Lines from a File](#example-removing-multiple-lines-from-a-file)
  - [Appendix](#appendix)
    - [Result of Not Reloading the Modified File](#result-of-not-reloading-the-modified-file)

## Introduction

This document provides a detailed overview of utilizing Tool Command Language (TCL) for the effective management of dynamic E164-pattern-maps stored locally on routers. It specifically addresses scenarios typical during migration phases, where number blocks—often noncontiguous—are frequently added to and removed from the system. The syntax and methodologies outlined herein are designed to optimize the handling of such changes, ensuring seamless updates and modifications during transitional periods.

## Prerequisites

- Knowledge of the Cisco router command line interface (CLI).

## Components Used

- This information in this document is based on the Cisco Cloud Services Router (CSR) 16.12.04.
- Variables are referenced in the body of this document and TCL syntax by **`$name`**.
- Commands referenced in the body of this document are denoted by **`name`**.
- Please see inline script #comments as well as the explanation of process steps.

> [!Important]
> After modifying a pattern-map file, reload the file using:
>```javascript
>voice class e164-pattern-map load <pattern-map-group-id>
>```
>Please see the appendix for a detailed example of what occurs when this is not performed.

## Create Initial Pattern-Map File and Load into Memory
```javascript
To create a pattern-map file that exists on the flash: of the router without having to re-import via scp or tftp:
```tcl
tclsh
# Open the file in write-append mode and assign it to a variable
set file_handle [open "flash:filename.cfg" w+]

# Write data to the file
puts $file_handle {
<paste in list>
}

# Close the file to ensure data is written and file integrity is maintained
close $file_handle

# Exit TCL interpreter
tclquit

# Reference the configuration file in the running config (Global Configuration Mode).
voice class e164-pattern-map 3
url bootflash:filename.cfg

#Load the configuration file (Privleged Exec Mode)
voice class e164-pattern-map load 3
```
##### Detailed Explanation
File Handling: The file is opened, and the handle is stored in a variable **$file_handle**. This is crucial for managing file operations correctly.
Writing to the File: Uses puts with the file handle to write data into the file. Replace <paste in list> with the actual data you want to write.
Closing the File: It's important to close the file handle with close to flush the buffer and ensure all data is properly written to the flash memory.
Exiting TCL: The command tclquit is used to correctly exit the TCL interpreter.

#### File Creation Example with Data
```javascript
tclsh
# Open the file in write-append mode and assign it to a variable
set file_handle [open "flash:map.cfg" w+]

# Write data to the file
puts $file_handle {
8138261111
8138262222
8138263333
813826333[567]
}

# Close the file to ensure data is written and file integrity is maintained
close $file_handle
```
#### Append Data to an Existing File
```javascript
#Append data to the end of an existing file
set file_handle [open "flash:map.cfg" a]
puts $file_handle 8138267777
puts $file_handle 8138268888
puts $file_handle {813826333[567]} #Note the syntax for wild cards for appending.
close $file_handle

#These steps are used only if checking the contents while in TCL shell
set file_handle [open "flash:map.cfg" r]
set file_data [read $file_handle]
close $file_handle
puts $file_data
```
#### Example: Create the File and Remove a Single Line or Pattern
##### Creating the file
```javascript
vCUBE1#tclsh
vCUBE1(tcl)#set file_handle [open "flash:map.cfg" w+]
file10
vCUBE1(tcl)#
vCUBE1(tcl)#puts $file_handle {8138261111
+>8138262222
+>8138263333}
vCUBE1(tcl)#
vCUBE1(tcl)#close $file_handle
vCUBE1(tcl)#
vCUBE1(tcl)#set file_handle [open "flash:map.cfg" r]
file10
vCUBE1(tcl)#set file_data [read $file_handle]
8138261111
8138262222
8138263333

vCUBE1(tcl)#close $file_handle
vCUBE1(tcl)#puts $file_data
8138261111
8138262222
8138263333
```
#### Removing a Single Line from the File
```javascript
vCUBE1(tcl)#set file_handle [open "flash:map.cfg" r]
file10
vCUBE1(tcl)#set file_contents [read $file_handle]
8138261111
8138262222
8138263333

vCUBE1(tcl)#close $file_handle
vCUBE1(tcl)#set lines [split $file_contents "\n"]
8138261111 8138262222 8138263333 {}
vCUBE1(tcl)#set new_lines []
vCUBE1(tcl)#foreach line $lines {
+>if {![string match "8138263333" $line]} {
+>lappend new_lines $line
+>}
+>}
vCUBE1(tcl)#set new_file_contents [join $new_lines "\n"]
8138261111
8138262222

vCUBE1(tcl)#set file_handle [open "flash:map.cfg" w]
file10
vCUBE1(tcl)#puts $file_handle $new_file_contents
vCUBE1(tcl)#close $file_handle
vCUBE1(tcl)#
vCUBE1(tcl)#set file_handle [open "flash:map.cfg" r]
file10
vCUBE1(tcl)#set file_data [read $file_handle]
8138261111
8138262222

vCUBE1(tcl)#close $file_handle
vCUBE1(tcl)#puts $file_data
8138261111
8138262222

vCUBE1(tcl)#tclquit
vCUBE1#more flash:map.cfg
8138261111
8138262222
```
#### Removing Entries
##### Remove a Single Line from an Existing File
```javascript
# Open the file for reading
set file_handle [open "flash:map.cfg" r]
set file_contents [read $file_handle]
close $file_handle

# Split the content into lines
set lines [split $file_contents "\n"]

# Initialize an empty list to hold the filtered lines
set new_lines []

# Loop through each line and add it to new_lines if it does not match the pattern
foreach line $lines {
if {![string match "8138263333" $line]} {
lappend new_lines $line
}
}

# Join the filtered lines back into a single string with newline characters
set new_file_contents [join $new_lines "\n"]

# Open the file for writing (this will overwrite the existing content)
set file_handle [open "flash:map.cfg" w]
puts $file_handle $new_file_contents
close $file_handle

# Exit the TCL interpreter
tclquit
```
##### Detailed Explanation
Patterns List: You define a list of strings **$patterns**. Each element in the list represents a pattern that you want to filter out from the file.
Reading the File: Open the file, read its contents into **$file_contents**, and close the file.
Filtering Process: Split the file contents into lines. For each line, check against each pattern in the list. If any pattern matches, mark the line to be excluded with **include_line set to 0**.
Reconstruction: After filtering, the remaining lines $new_lines are joined back into a single string and written back to the file, overwriting the previous content.
Performance Consideration: This method is straightforward and effective for small to medium-sized files. For very large files, consider processing in chunks.

#### Remove Multiple Single Entries
Note: Due to the CLI character buffer limitation, longer lists of patterns to be removed must have the set patterns command broken up after approximately 18 ten-digit numbers. You can achieve this by inserting carriage returns to distribute the numbers across multiple lines while setting the $patterns variable.

```javascript
tclsh
# Define a list of patterns to remove
set patterns {"8138265555" "8138266666" "8138267777"}

# Open the file for reading
set file_handle [open "flash:map.cfg" r]
set file_contents [read $file_handle]
close $file_handle

# Split the content into lines
set lines [split $file_contents "\n"]

# Initialize an empty list to hold the filtered lines
set new_lines []

# Loop through each line
foreach line $lines {
set include_line 1 # Assume the line should be included unless a pattern matches

# Check each pattern against the current line
foreach pattern $patterns {
if {[string match "*$pattern*" $line]} {
set include_line 0
break # If any pattern matches, exclude the line and stop checking
}
}

# If include_line is still 1, append the line to new_lines
if {$include_line} {
lappend new_lines $line
}
}

# Join the filtered lines back into a single string with newline characters
set new_file_contents [join $new_lines "\n"]

# Open the file for writing (this will overwrite the existing content)
set file_handle [open "flash:map.cfg" w]
puts $file_handle $new_file_contents
close $file_handle

# Exit the TCL interpreter
tclquit
```

##### Detailed Explanation
Patterns List: The patterns are defined as a list of strings $patterns. This is a straightforward way to store multiple patterns that need to be removed.
Loop Through Each Line: For each line in the file, a nested loop checks each pattern against the current line.
Filtering: The inner loop stops checking patterns for the current line once a match is found, setting **include_line to 0**. Only lines that aren't matched by any pattern are appended to **$new_lines**.
Reconstruction: The remaining lines are joined back into a single string and written back to the file.

#### Use RegEx to Match Multiple Numbers
```javascript
set pattern {813826333[4-6]}
set file_handle [open "flash:map.cfg" r]
set file_contents [read $file_handle]
close $file_handle

set lines [split $file_contents "\n"]

set new_lines []

foreach line $lines {
# Check if the line matches the regular expression
if {![regexp $pattern $line]} {
lappend new_lines $line
}
}

set new_file_contents [join $new_lines "\n"]
set file_handle [open "flash:map.cfg" w]
puts $file_handle $new_file_contents
close $file_handle
tclquit
```
#### Match an Entry with Wild Cards Present
```javascript
set pattern {813826333[567]}
set file_handle [open "flash:map.cfg" r]
set file_contents [read $file_handle]
close $file_handle

set lines [split $file_contents "\n"]

set new_lines []

foreach line $lines {
if {![string equal $pattern $line]} {
lappend new_lines $line
}
}

set new_file_contents [join $new_lines "\n"]
set file_handle [open "flash:map.cfg" w]
puts $file_handle $new_file_contents
close $file_handle
tclquit
```
##### Example: Removing Multiple Lines from a File
```javascript
vCUBE1(tcl)#set patterns {"8138265555" "8138266666" "8138267777"}
vCUBE1(tcl)#set file_handle [open "flash:map.cfg" r]
file10
vCUBE1(tcl)#set file_contents [read $file_handle]
8138261111
8138262222
8138263333
8138264444
8138265555
8138266666
8138267777

vCUBE1(tcl)#close $file_handle
vCUBE1(tcl)#set lines [split $file_contents "\n"]
8138261111 8138262222 8138263333 8138264444 8138265555 8138266666 8138267777 {} {}
vCUBE1(tcl)#set new_lines []
vCUBE1(tcl)#foreach line $lines {
+> set include_line 1
+>foreach pattern $patterns {
+> if {[string match "*$pattern*" $line]} {
+> set include_line 0
+>break
+>}
+>}
+> if {$include_line} {
+> lappend new_lines $line
+>}
+>}
vCUBE1(tcl)#set new_file_contents [join $new_lines "\n"]
8138261111
8138262222
8138263333
8138264444

vCUBE1(tcl)#set file_handle [open "flash:map.cfg" w]
file10
vCUBE1(tcl)#puts $file_handle $new_file_contents
vCUBE1(tcl)#close $file_handle
vCUBE1(tcl)#tclquit
vCUBE1#more flash:map.cfg
8138261111
8138262222
8138263333
8138264444
```
## Appendix
### Result of Not Reloading the Modified File
The following is what occurs when the local e164-pattern-map file is edited but subsequently NOT reloaded into the router's memory.

The initial file has existing entries, and the router is aware of their presence in the raw file via flash: as well as the existence of valid dial plan numbers.

```javascript
vCUBE1#more flash:map.cfg
8138261111
8138262222
8138263333
813826333[567]

vCUBE1#show voice class e164-pattern-map 3
e164-pattern-map 3
-----------------------------------------
It has 4 entries
It is populated from url flash:map.cfg.
Map is valid.

E164 pattern
-------------------
8138261111
8138262222
8138263333
813826333[567]

vCUBE1#show dialplan number 8138261111
Macro Exp.: 8138261111
VoiceOverIpPeer1
peer type = voice, system default peer = FALSE, information type = voice,
# /output omitted for brevity/
```
The e164-pattern-map file is then updated with an additional entry.
```javascript
vCUBE1#tclsh
vCUBE1(tcl)#set file_handle [open "flash:map.cfg" a]
file10
vCUBE1(tcl)#puts $file_handle 8138267777
vCUBE1(tcl)#close $file_handle
vCUBE1(tcl)#tclquit
vCUBE1#
```
The voice process DOES NOT see the newly added number under the dialplan or pattern-map.
```javascript
vCUBE1#show dialplan number 8138267777
Macro Exp.: 8138267777
No match, result=-1(NO_MATCH)

vCUBE1#show voice class e164-pattern-map 3
e164-pattern-map 3
-----------------------------------------
It has 4 entries
It is populated from url flash:map.cfg.
Map is valid.

E164 pattern
-------------------
8138261111
8138262222
8138263333
813826333[567]
```
However, the newly added entry is present in the file.
```javascript
vCUBE1#more flash:map.cfg
8138261111
8138262222
8138263333
813826333[567]
8138267777
```

To correct this, the file is then reloaded into memory.
```javascript
vCUBE1#voice class e164-pattern-map load 3
url flash:map.cfg loaded successfully
All e164 patterns are valid

vCUBE1#show voice class e164-pattern-map 3
e164-pattern-map 3
-----------------------------------------
It has 5 entries
It is populated from url flash:map.cfg.
Map is valid.

E164 pattern
-------------------
8138261111
8138262222
8138263333
8138267777
813826333[567]

vCUBE1#show dialplan number 8138267777
Macro Exp.: 8138267777
VoiceOverIpPeer1
peer type = voice, system default peer = FALSE, information type = voice,
# /output omitted for brevity/
```
Carriage Returns are Acceptable in a Pattern-Map
In the following output, the pattern map is valid even though viewing it from the flash directly displays empty lines.

```javascript
# Pattern Map Validation
vCUBE1#show voice class e164-pattern-map 3
e164-pattern-map 3
-----------------------------------------
It has 7 entries
It is populated from url flash:map.cfg.
Map is valid.

E164 pattern
-------------------
8138261111
8138262222
8138263333
8138263334
8138263335
8138263336
8138264444

#Viewing the file directly from Flash
vCUBE1#more flash:map.cfg
8138261111
8138262222
8138263333
8138264444

8138263334
8138263335
8138263336

vCUBE1#
```





