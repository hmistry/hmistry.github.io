---
layout: post
title:  "Cleaning Xcode files"
date:   2017-10-09
categories: software
tags: xcode
excerpt_separator: <!--more-->
---

This is a summary from the following articles on recovering disk space from Xcode:
1. [Xcode users can free up space on your Mac by Ajith Nayak](http://ajithrnayak.com/post/95441624221/xcode-users-can-free-up-space-on-your-mac)
2. [How To Recover Disk Space From Xcode by Sascha Holesch](http://blog.neverthesamecolor.net/how-to-recover-disk-space-from-xcode/)

<br>
#### 1. Derived Data
**Location:** `~/Library/Developer/Xcode/DerivedData/`
<br>
**Purpose:** Contains data about the projects which includes index, build output, debug/release built targets, and logs. Data can be recreated.
<br>
<!--more-->
**Delete:** Safe to delete. Can be done through Xcode Projects or in Finder by deleting the folders of projects no longer needed.

<br>
#### 2. Archives
**Location:** `~/Library/Developer/Xcode/Archives/`
<br>
**Purpose:** Project targets created for beta testing or App Store submissions are archived. The dSYM data is crucial for debugging crash reports.
<br>
**Delete:** Not safe to delete if you need the dSYM data. Can be done through Xcode Organizer or in Finder by deleting the folders of projects no longer needed - be sure to only delete if you don't need the dSYM data.

<br>
#### 3. iOS Device Support
**Location:** `~/Library/Developer/Xcode/iOS DeviceSupport/`
<br>
**Purpose:** Contains data stored for every device with every iOS version combination used in development. Initial device connection takes longer to copy all this info. Data can be recreated by connecting device to Mac.
<br>
**Delete:** Safe to delete. Use Finder to delete the folders of device with iOS version no longer needed.

<br>
#### 4. Core Simulator
**Location:** `~/Library/Developer/Xcode/CoreSimulator/Devices/`
<br>
**Purpose:** Contains simulators for each version of iOS + device and represented by UUID's which can be deciphered using the file `.default_created.plist`.
<br>
**Delete:** Safe to delete. Can be done through Xcode Projects or in Finder by deleting the folders of projects no longer needed.

<br>
#### 5. Documentation
**Location:** `~/Library/Developer/Shared/Documentation/DocSets/`
<br>
**Purpose:** Contains documentation sets stored for Xcode.
<br>
**Delete:** Safe to delete. Delete file in Finder.

<br>
#### 6. Cache
**Location:** `~/Library/Caches/com.apple.dt.Xcode`
<br>
**Purpose:** Some kind of Xcode cache in form of a packaged file. File size is relatively small.
<br>
**Delete:** Safe to delete. Delete file in Finder.

<br>
#### 7. Device Backup
**Location:** `~/Library/Application Support/MobileSync/Backup/`
<br>
**Purpose:** Contains a backup sync data of some files and settings of an iOS device that was connected to Mac. This maybe a legacy thing because it's related to iTunes mobile device sync. Needs verification.
<br>
**Delete:** Safe to delete through iTunes device section in preference if you don't need it.
