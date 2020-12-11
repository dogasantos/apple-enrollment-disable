Original: https://gist.github.com/sghiassy/a3927405cf4ffe81242f4ecb01c382ac

# Disable Device Enrollment Notification on Mac.md

## Restart the Mac in Recovery Mode by holding `Comment-R` during restart

#### Open Terminal in the recovery screen and type

```
csrutil disable
```

Restart computer

## Edit `com.apple.ManagedClient.enroll.plist`

In the terminal, type

```
sudo open /Applications/TextEdit.app /System/Library/LaunchDaemons/com.apple.ManagedClient.enroll.plist
```

change

```
<key>com.apple.ManagedClient.enroll</key>
        <true/>
```

to

```
<key>com.apple.ManagedClient.enroll</key>
        <false/>
````

### Restart Computer again

So that the changes take effect
