# Disable Device Enrollment Program (DEP) notification on macOS Catalina.md

## With full reinstall (recommended)

&nbsp;&nbsp;&nbsp;a. Boot into recovery using `command-R` during reboot, wipe the harddrive using Disk Utility, and select reinstall macOS

&nbsp;&nbsp;&nbsp;b. Initial installation will run for approximately 1 hour, and reboot once

&nbsp;&nbsp;&nbsp;c. It will then show a remaining time of about 10-15 minutes 

&nbsp;&nbsp;&nbsp;d. When it reboots again, be sure to press `command-R` to boot into recovery and continue with **Main procedure**

## Without full reinstall 

Boot to Recovery Mode by holding `command-R` during restart and continue with **Main procedure**

## Main procedure

1. Open Utilities → Terminal and type
```
$ csrutil disable
$ reboot
```

2. Hold `command-R` during the reboot to enter Recovery Mode again

3. Enter Disk Utility, and mount the `Macintosh HD` volume (or whatever your main volume is named).  (It might already be mounted.)

4. Exit Disk Utility, open Utilities → Terminal, and type
```
$ cd "/Volumes/Macintosh HD/System/Library"
$ mkdir LaunchDaemons.disabled LaunchAgents.disabled
$ mv LaunchDaemons/com.apple.ManagedClient* LaunchDaemons.disabled/
$ mv LaunchAgents/com.apple.ManagedClient* LaunchAgents.disabled/
$ cd ../../etc
$ echo "0.0.0.0 iprofiles.apple.com" >> hosts
$ echo "0.0.0.0 mdmenrollment.apple.com" >> hosts
$ echo "0.0.0.0 deviceenrollment.apple.com" >> hosts
$ echo "0.0.0.0 gdmf.apple.com" >> hosts
$ csrutil enable
$ reboot
```

5. If you come to the “Choose your country/location” dialogue, make sure to not select a wireless network, but “continue without an internet connection”

6. After a normal boot, you can verify the DEP status in Terminal:
```
$ profiles status -type enrollment
Enrolled via DEP: No
MDM enrollment: No
```
