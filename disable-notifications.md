# Disable Device Enrollment Notification on MacOS BIGSUR

### Restart the Mac in Recovery Mode by holding `Command-R` during restart

### Open Terminal in the recovery screen and type:
```
csrutil disable
```

### Restart Computer INTO THE RECOVERY MODE AGAIN
In the terminal, and run the following two commands:
```
sudo mount -uw /Volumes/Macintosh\ HD
```
```
sudo bash 
 mv /System/Library/LaunchAgents/com.apple.ManagedClientAgent.agent.plist /System/Library/LaunchAgents/com.apple.ManagedClientAgent.agent-disabled 
 mv /System/Library/LaunchAgents/com.apple.ManagedClientAgent.enrollagent.plist /System/Library/LaunchAgents/com.apple.ManagedClientAgent.enrollagent-disabled 
 mv /System/Library/LaunchDaemons/com.apple.ManagedClient.cloudconfigurationd.plist /System/Library/LaunchDaemons/com.apple.ManagedClient.cloudconfigurationd-disabled 
 mv /System/Library/LaunchDaemons/com.apple.ManagedClient.enroll.plist /System/Library/LaunchDaemons/com.apple.ManagedClient.enroll-disabled 
 mv /System/Library/LaunchDaemons/com.apple.ManagedClient.plist /System/Library/LaunchDaemons/com.apple.ManagedClient-disabled 
 mv /System/Library/LaunchDaemons/com.apple.ManagedClient.startup.plist System/Library/LaunchDaemons/com.apple.ManagedClient.startup-disabled  
```

### Enable SIP again
```
csrutil enable
```

### Restart Normally and Enjoy
