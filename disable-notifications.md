# Disable Device Enrollment Notification on MacOS BIGSUR

### Restart the Mac in Recovery Mode by holding `Command-R` during restart

### Open Terminal in the recovery screen and type:
```
csrutil disable
csrutil authenticated-root disable
reboot
```

### Restart Computer and get into normal boot mode

Find your root mount's device - run mount and chop off the last s, e.g. if your root is /dev/disk1s2s3, you'll need the "/dev/disk1s2" part

Using the values from above create a new directory, for example ~/mountRun then:
```
sudo mount -o nobrowse -t apfs DISK_PATH MOUNT_PATH
```
Now you can modify the files under the mounted directory, which are the ROOT FILES. Don't mess with them if you're not sure what you're doing. Otherwise, you will break your macos.

In order to disable the ManagedClient daemon, enter the new MOUNT_PATH directory (~/mountRun in the example):
```
 sudo bash
 cd ~/mountRun
 mv System/Library/LaunchAgents/com.apple.ManagedClientAgent.agent.plist System/Library/LaunchAgents/com.apple.ManagedClientAgent.agent-disabled 
 mv System/Library/LaunchAgents/com.apple.ManagedClientAgent.enrollagent.plist System/Library/LaunchAgents/com.apple.ManagedClientAgent.enrollagent-disabled 
 mv System/Library/LaunchDaemons/com.apple.ManagedClient.cloudconfigurationd.plist System/Library/LaunchDaemons/com.apple.ManagedClient.cloudconfigurationd-disabled 
 mv System/Library/LaunchDaemons/com.apple.ManagedClient.enroll.plist System/Library/LaunchDaemons/com.apple.ManagedClient.enroll-disabled 
 mv System/Library/LaunchDaemons/com.apple.ManagedClient.plist System/Library/LaunchDaemons/com.apple.ManagedClient-disabled 
 mv System/Library/LaunchDaemons/com.apple.ManagedClient.startup.plist System/Library/LaunchDaemons/com.apple.ManagedClient.startup-disabled  
```

Now, run:
```
sudo bless --folder MOUNT_PATH/System/Library/CoreServices --bootefi --create-snapshot
```
### Restart the Mac in Recovery Mode by holding `Command-R` during restart

### Enable SIP again
```
csrutil emable
csrutil authenticated-root emable
reboot
```

the changes will take place

