# OCI Windows In-Place Migration Guide

This README provides a step-by-step guide on performing an in-place migration of a Windows instance on Oracle Cloud Infrastructure using an OCI-Included license.

# Overview

This migration guide demonstrates the process of upgrading an existing Windows instance in OCI through in-place migration techniques. 
The focus is to ensure a smooth transition with minimum downtime and maximal compatibility with the new environment.

Watch the full demonstration in [this video](https://frmlfp7pqh4h.objectstorage.eu-frankfurt-1.oci.customer-oci.com/p/rqda58064fqp1Cbscaux1albjmxyI-UBqGkrHW5VZ2y8MdVTxpG-1h5biKkTgVWE/n/frmlfp7pqh4h/b/WinMig/o/OCI_WIndows_In-Place_Migration.mp4).

# Migration Steps:


### Verify Current Windows Edition

Execute the following command in PowerShell prompt:

```ruby
Get-ComputerInfo -Property WindowsProductName
```

### Update Windows Instance

Apply the latest updates to your Windows instance.


### Backup

Create a [full backup of the boot and attached block volumes.](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumebackups.htm)


### Prepare Installation Media

Ensure the proper Windows Server ISO is available on a block volume.


### Mount the ISO

Mount the ISO locally on the instance.


### Initiate Setup

Run setup.exe from the mounted ISO.
If prompted for a product key, enter it. 

***Note:*** The product key will be removed during KMS activation.


### Select Windows Edition

Choose the correct image, 
for example, ***"Windows Server 2022/2025 Standard with Desktop Experience"***


### Configuration Retention

Select what to keep. 
If ***"Keep files, settings and apps"*** option is greyed out, ***immediately abort the upgrade to prevent data loss.***


### Monitor Upgrade

While setup is updating its binaries:

Go to your OCI Console:
	
- Instance Details 
	
- [Create a Console Connection](https://docs.oracle.com/en-us/iaas/Content/Compute/References/serialconsole.htm)

This will allow continuous access and monitoring without RDP.

Connect via VNC to the Console connection.

### Start Installation process

Click ***Install*** to start the upgrade.

### Post-Installation Steps:

When installation has completed, open a PowerShell Prompt with Admin Rights and verify the upgraded Windows version using:

```ruby
Get-ComputerInfo -Property WindowsProductName
```

***Remove the existing product key***

Open a Command prompt with Admin Rights

```ruby
slmgr /upk
```

***Clear the license key from the registry***

```ruby
slmgr /cpky 
```

***Restart instance***

```ruby
shutdown /r /t 0
```

***Register the OCI KMS Server***

```ruby
slmgr /skms 169.254.169.253:1688
```

***Reset Activation State***

```ruby
slmgr /rearm
```

***Restart instance***

```ruby
shutdown /r /t 0
```

***Retrieve the proper Generic Volume License Key***

[Key Management Services (KMS) client activation and product keys](https://learn.microsoft.com/en-us/windows-server/get-started/kms-client-activation-keys?tabs=server2022%2Cwindows1110ltsc%2Cversion1803%2Cwindows81)


***Install the KMS Client Key***

Example using GVL key for Windows 2022 datacenter:

```ruby
slmgr /ipk WX4NM-KYWYW-QJJR4-XV3QB-6VM33
```

***Force activation***

```ruby
slmgr /ato
```

***Check the activation status***

```ruby
slmgr /xpr
```

***Get detailed information***

```ruby
slmgr /dlv
```

### Upgrade completed:

The compute instance is now upgraded and uses ***OCI-KMS*** with an ***OCI product key.***

## Why OCI-Provided licenses use KMS ?

Using a ***Key Management Service (KMS)*** allows assigning a temporary key that is periodically renewed, rather than a permanent key.

This activation model is dynamic and designed to support enterprises managing a large number of licenses across multiple servers and client computers in an internal network environment.

***How KMS Works:***
	
	- Temporary Key:

		- The key used for activation via KMS is not permanent and is regularly reactivated by an internal KMS server.
		
		- This key must be renewed at regular intervals to ensure that the operating system remains activated without interruption.

	- Renewal Period:
	
		- By default, a machine activated via KMS must "renew" or "revalidate" its activation on a regular basis. (Default: 6 months)
		
		- However, the attempts to reactivate are much more frequent.

	- Renewal Tolerance:
	
		- While the activation interval is set at 180 days, systems attempt to reactivate every 7 days.
		
		- This helps prevent service interruptions in case of KMS server issues or other network errors that could impede activation during this time.

	- Why a Renewal Model?
	
		- Using KMS as an activation method is particularly beneficial for large organizations for several reasons:
		
		- CENTRALIZED MANAGEMENT: KMS allows administrators to manage activations through an internal network, centralizing license administration without requiring an external connection for each activation.

		- LICENSE COMPLIANCE: This model helps businesses stay compliant with Microsoft licensing agreements, as it ensures only verified and approved machines are activated.

		- FLEXIBILITY: It provides flexibility in license management, particularly useful in cases of frequent hardware or software configuration changes, common in large IT environments.

This repeated temporary approach ensures that any machine no longer part of the network or organization loses its activation, thereby helping to secure and regulate the use of software licenses.

