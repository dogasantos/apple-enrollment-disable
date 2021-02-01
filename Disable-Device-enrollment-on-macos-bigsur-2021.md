# Disable Device Enrollment Notification on MacOS BIGSUR 2021

### Background

After reading, testing and retesting, modifying steps and retesting, I finally managed to get it done.
Here are some links with a few sources that can help:


https://gist.github.com/henrik242/65d26a7deca30bdb9828e183809690bd#gistcomment-3565896
https://github.com/dogasantos/apple-enrollment-disable/blob/main/disable-notifications.md
https://github.com/dogasantos/apple-enrollment-disable/blob/main/Disable%20DEP%20mac.md
https://www.tonymacx86.com/threads/solved-disable-system-file-protection-in-big-sur.302406/
https://duo.com/labs/research/mdm-me-maybe

### Scenario

I've worked on a comapny with no location/office near me (or anywhere in this country), so they just wiped the notebook and I've be instructed to keep or recycle the notebook.
They did not remove the serial from Apple's managed program (Can't find any reason for this), so I had to find a way to keep the laptop for my personal use.

I've tested 3 to 5 different guides without a *full* working solution. Then I've tested different combinations util I found one that got me covered 100%.

My laptop config:
```
macOS Big Sur 11.1
MacBook Pro (13-inch, 2017, Four thinderbolt 3 ports)
```

### WARNING 

You'll need to full erase/format the laptop, install MacOS BigSur (11.1 tested) 
After the first reboot after macos install, you *must* get into recovery mode and *don't* connect the laptop on network (internet)
Again, you must *NOT CONNECT THE INTERNET* until the entire procedure is done. 


### First step

Full erase/format the laptop plus install MacOS BigSur 11.1 via usb stick

  ```
   a. Boot into recovery using command-R during reboot, wipe the harddrive using Disk Utility, and select reinstall macOS

   b. Initial installation will run for approximately 1 hour, and reboot once

   c. It will then show a remaining time of about 10-15 minutes

   d. When it reboots again, be sure to press command-R to boot into recovery and proceed with the next step
```

### 2nd step

After the macos installation (before the machine setup), you *MUST* get into the recovery mode.

Open Utilities → Terminal and type:
```
# csrutil disable
# csrutil authenticated-root disable
# reboot
```
When it reboots again, be sure to press command-R to boot into recovery (again) 

### 3rd step
Goal here is to remount the main partition as read and write so we can modify our root files. 

Before we do this, check if the last procedure has been done, otherwise, repeat 2nd step:
```
# csrutil status
# csrutil authenticated-root status
```

If everything is ok (disabled), you must take note on wich disk your "Machintosh HD" partition is. You can do this by typing (example): 

```
# mount
/dev/disk3s1 on / (apfs, local, read-only, journaled)
....
/dev/disk2s5 on /Volumes/Machintosh HD (apfs, sealed, local, read-only, journaled)
...
```

Here in this example, "Machintosh HD" is on /dev/disk2s5.

Take note, then run:

```
# umount /Volumes/Machintosh\ HD 
# cd /Volumes
# mkdir disco
# mount -t apfs -rw /dev/disk2s5 disco
```

Note: the only reason to create "disco" is to have a short name without spaces for ease of use.

### 4th step: THE MAIN

Now, we have our root partition on read and write mode, with both csrutil/authenticated-root disabled. 
We'll be able to modify Core System files in the following order:

1) Patch hosts file - your macos won't be able to call home to fetch a profile
2) Patch mdmclient binary file - In case your hosts file revert for some reason, we'll have this fallback measure
3) Move mdm program related system files in order to disable the daemon entirely.


NOTE: In case you apply a deep system update, you might need to redo some (or all) steps. Small updates/modifications won't affect the whole system as you have 3 layers here

#### 1) Patch hosts file


Just type each command in terminal:

```
# cd /Volumes/disco/etc/"
# echo "127.0.0.1 iprofiles.apple.com" >> hosts
# echo "127.0.0.1 mdmenrollment.apple.com" >> hosts
# echo "127.0.0.1 deviceenrollment.apple.com" >> hosts
# echo "127.0.0.1 gdmf.apple.com" >> hosts
```

#### 2) Patch mdmclient binary file

mdmclient will trigger cloudconfigurationd and fetch the profile directly from Apple.

Just type each command in terminal:

```
# mv /Volumes/disco/usr/libexec/mdmclient /Volumes/disco/usr/libexec/mdmclient.disabled
# echo '#!/bin/bash' > /Volumes/disco/usr/libexec/mdmclient 
# echo 'Activation record: { }' >> /Volumes/disco/usr/libexec/mdmclient 
# chmod 755 /Volumes/disco/usr/libexec/mdmclient 
```

#### 3) Move mdm files

```
# cd /Volumes/disco/System/Library"
# mkdir LaunchDaemons.disabled LaunchAgents.disabled
# mv LaunchDaemons/com.apple.ManagedClient* LaunchDaemons.disabled/
# mv LaunchAgents/com.apple.ManagedClient* LaunchAgents.disabled/
```

### 5th step: make it worth

If you reboot your machine at this point, you'll notice that all the modification you just did won't be there anymore. 
That happens because APFS filesystem uses a snapshot to record a "valid" system state and boot using it. So you must create a new snapshot and make it the primary one.

To list available snapshots, you'll need the partition path you took note in the 3rd step:

```
# diskutil apfs listSnapshots /dev/disk2s5
```

Let's create a new snapshot and give it a name (no mdm snap will be the name):
```
# /Volume/disco/System/Library/Filesystems/apfs.fs/Contents/Resources/apfs_systemsnapshot -v /Volumes/disco -s nomdmsnap


You can find different commands to do this step, like:

```
sudo bless --folder /Volume/disco/System/Library/CoreServices --bootefi --create-snapshot
```

List snapshots to see your new nomdmsnap in place:

```
# diskutil apfs listSnapshots /dev/disk2s5
```

Ok, let's make it the snapshot MacOS should use in the future:
```
# /Volume/disco/System/Library/Filesystems/apfs.fs/Contents/Resources/apfs_systemsnapshot -v /Volumes/disco -r nomdmsnap
``` 

Done!

### 6th step: Finish

In terminal:

```
# csrutil enable
# reboot
```

You'll boot in normal mode (reboot once again in case your mac boot in recovery mode by default)

If you come to the “Choose your country/location” dialogue, *make sure* to not select a wireless network, but “continue without an internet connection” (or equivalent option)

finish the setup

Use your new mac normally.


# FINAL NOTE

As I've wrote this guide 3 minutes after a sucessful procedure, some details might be not exact, but are enough to put you in the right path.
