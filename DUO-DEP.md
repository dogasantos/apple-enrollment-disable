full original version:

https://duo.com/labs/research/mdm-me-maybe

# 03. DEP APIs
We begin our journey of understanding DEP by looking at the APIs used to interface with the DEP service. There are three distinct DEP APIs.

The so-called DEP "cloud service" API. This is used by MDM servers to associate DEP profiles with specific devices.
The DEP API used by Apple Authorized Resellers to enroll devices, check enrollment status, and check transaction status.
The undocumented private DEP API. This is used by Apple Devices to request their DEP profile. On macOS, the cloudconfigurationd binary is responsible for 
communicating over this API.
Unless otherwise noted, the API we're referring to throughout this paper is the private DEP API.

# 04. How Does DEP Enrollment Work?
Before a device can automatically enroll into its organization's MDM server through DEP, a bootstrapping process must be completed. 
This bootstrapping process involves several asynchronous steps:

 1. Apple, (or an Apple Authorized Reseller), creates a device record through the DEP API.
 2. The organization using DEP to bootstrap MDM enrollment assigns the device to their MDM server in Apple Business Manager.
 3. The MDM server retrieves the device record through the DEP API, then creates a DEP profile.
 4. The device authenticates to the DEP API, then retrieves its Activation Record.
 5. The device authenticates to the MDM server to retrieve a Configuration Profile containing an MDM Enrollment Payload, Certificate Payload, and SCEP Payload.
 6. The device requests and receives its client certificate through SCEP.
 7. The device authenticates to the MDM server with the client certificate retrieved during step six. MDM commands can then be sent to the device via APNs.
 8. Our research focused mostly on step four, the DEP check-in process. For the curious reader, the device 
 bootstrapping process is described in greater detail in Jesse Endahl and Max Bélanager's paper, A Deep Dive into macOS MDM (and how it can be compromised).
 
# 05. DEP Authentication
Due to the way DEP is implemented, it effectively only uses the system serial number to authenticate devices prior to enrollment. 
This was - as far as we know - originally discovered by Pepijn Bruienne, Jesse Peterson and Victor Vrantchan in 2015. Pepijn, 
Jesse, and Victor needed a way to quickly test DEP, so they configured Virtual Machines to use DEP-registered serial numbers 
to simulate the DEP enrollment flow.

This is problematic, though, because an attacker armed with only a valid, DEP-registered serial number can potentially enroll a 
rogue device into an organization's MDM server, or use the DEP API to glean information from enrolled devices.


# 06. Binaries Involved in DEP and MDM
There are several client binaries involved in DEP and MDM enrollment and management on macOS. Throughout our research, we explored the following:

mdmclient: Used by the OS to communicate with an MDM server. On macOS 10.13.3 and earlier, it can also be used to trigger a DEP check-in.
profiles: A utility that can be used to install, remove and view Configuration Profiles on macOS. It can also be used to trigger a 
DEP check-in on macOS 10.13.4 and newer.


cloudconfigurationd: The Device Enrollment client daemon, which is responsible for communicating with the DEP API and retrieving 
Device Enrollment profiles.

When using either mdmclient or profiles to initiate a DEP check-in, the CPFetchActivationRecord and CPGetActivationRecord functions 
are used to retrieve the Activation Record. CPFetchActivationRecord delegates control to cloudconfigurationd through XPC, which then 
retrieves the Activation Record from the DEP API.

CPGetActivationRecord retrieves the Activation Record from cache, if available. These functions are defined in the private Configuration 
Profiles framework, located at /System/Library/PrivateFrameworks/Configuration Profiles.framework.


# 07. Research Methodology
To test whether a serial number is indeed the only identifier needed to perform a DEP check-in, we manually inserted a known DEP-registered Mac serial number in a VMware VMX file, then booted the virtual machine.

`serialNumber = "<serial_number>"`

With the Virtual Machine using the serial number we wrote to the VMX file, we’re able to confirm that the machine is DEP-registered by issuing the following command on devices running macOS 10.13.3 or earlier:


`sudo /usr/libexec/mdmclient dep nag`
This outputs a pretty-printed Activation Record if the device’s serial number is registered in DEP. Also, if DEP enrollment hasn't already completed, a notification will be displayed stating that "Company X can automatically configure your Mac."

`
Activation record: {
AllowPairing = 1;
AnchorCertificates =     (
);
AwaitDeviceConfigured = 0;
ConfigurationURL = "https://example.com/enroll";
IsMDMUnremovable = 1;
IsMandatory = 1;
IsSupervised = 1;
OrganizationAddress = "123 Main Street, Anywhere, , 12345 (USA)";
OrganizationAddressLine1 = "123 Main Street";
OrganizationAddressLine2 = NULL;
OrganizationCity = Anywhere;
OrganizationCountry = USA;
OrganizationDepartment = "IT";
OrganizationEmail = "dep@example.com";
OrganizationMagic = 105CD5B18CE24784A3A0344D6V63CD91;
OrganizationName = "Example, Inc.";
OrganizationPhone = "+15555555555";
OrganizationSupportPhone = "+15555555555";
OrganizationZipCode = "12345";
SkipSetup =     (
TapToSetup,
Payment,
Zoom,
Biometric,
<snip>
);
SupervisorHostCertificates =     (
);
}
`
Machines that are not DEP-registered will display an empty Activation Record.

`
Activation record: {
}
`
On machines running macOS 10.13.4 and later, running the above command will return the following error:

[ERROR] Use 'sudo profiles renew -type enrollment' instead
However, this doesn’t mean we’re unable to view information on DEP-registered devices on macOS systems above 10.13.4. There are two known workarounds:

Copy the mdmclient binary from another Mac running macOS 10.13.3 or earlier and run it on the macOS 10.13.4 or later machine.
Enable Managed Client (MCX) and MDM debug logging by installing this Configuration Profile. The Activation Record will not be printed to standard output, but will instead be written to /Library/Logs/ManagedClient/ManagedClient.log.
Once we were able to confirm that only the serial number is required to authenticate the device for the initial DEP enrollment in a Virtual Machine, we set out to produce a reliable, automated method of controlling the serial number that's submitted when initiating a new DEP enrollment. Since there are a number of different ways this can be achieved, we explored a handful of them initially. These are:

Man-in-the-middle (MITM) network requests to modify the system serial number before it is submitted to iprofiles.apple.com.
Reverse engineer the communication protocol between cloudconfigurationd and the DEP API, which is internally referred to as "Tesla," as well as the scheme used to uniquely identify devices, known as "Absinthe." Understanding how these protocols work would allow us to create a macProfile payload with an arbitrary serial number and submit the request.
Instrument the binaries that interact with the DEP API to insert arbitrary device serial numbers before the request is made.
It would have also been possible to automate modifying the VMX file to include an arbitrary serial number, then start the VM and run the enrollment command. We didn’t seriously consider this because of performance reasons, but it could have been used as a last resort.





