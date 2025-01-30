# OCI Windows In-Place Migration Guide

This README provides a step-by-step guide on performing an in-place migration of a Windows instance on Oracle Cloud Infrastructure using an OCI-Included license.

# Overview

This migration guide demonstrates the process of upgrading an existing Windows instance inOCI through in-place migration techniques.The focus is to ensure a smooth transition with minimum downtime and maximal compatibilitywith the new environment.The final step involves reactivating the Windows License Key.**- For BYOL instances:** Simply verify the activation status and if necessary, re-enteryour product key.**- For OCI-Provided license instances:** You must reset the activation and reapply theproduct key using OCI’s KMS. This step is described later in this document.

[Download PDF Guide](./OCI_Windows_In-Place_Migration_Guide.pdf)


# Migration Steps:


### VERIFY CURRENT WINDOWS EDITION

Execute the following command in PowerShell prompt:

```ruby
Get-ComputerInfo -Property WindowsProductName
```

### UPDATE THE OPERATING SYSTEM

Apply the latest updates to your Windows instance.


### BACKUP YOUR DATA

Create a full backup of [the boot volume](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumebackups.htm) and [attached block volumes.](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/blockvolumebackups.htm)


### PREPARE THE INSTALLATION MEDIA

Download the proper Windows Server ISO locally.
If you don’t have an [attached block volume](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/creatingavolume.htm) yet, it's advisable to attach one to store the installation media efficiently.


### MOUNT INSTALLATION ISO

Mount the ISO locally on the instance.


### INITIATE SETUP

Run setup.exe from the mounted ISO.

If prompted for a product key, enter it. 

**Note:** The product key will be removed during KMS activation.

You should be able to use [a Volume Key provided by Microsoft](https://learn.microsoft.com/en-us/windows-server/get-started/kms-client-activationkeys)***A Volume key is a product key that can be used during setup but can't be activated.***


### SELECT WINDOWS EDITION

Choose the correct image, 
for example, ***"Windows Server 2022 Standard with Desktop Experience"***


### CONFIGURATION RETENTION

Select what to keep.
 
If ***"Keep files, settings and apps"*** option is greyed out, **_immediately abort the upgrade to prevent data loss._**


### MONITOR UPGRADE

Before starting the upgrade: 

- Go to your OCI Console, 
- Instance Details 
- [Create a Console Connection](https://docs.oracle.com/en-us/iaas/Content/Compute/References/serialconsole.htm)

This will allow continuous access and monitoring without RDP.

Connect via VNC to the Console connection.

### START INSTALLATION PROCESS

Click ***Install*** to start the upgrade.

### CONNECT TO THE CONSOLE CONNECTION

Use a VNC connection for Linux, MAc or Windows

Connect via VNC to :

```ruby
127.0.0.1:5900
```

## Post-Installation Steps:

When the installation has completed, 

open a PowerShell Prompt **with Admin Rights** and verify the upgraded Windows version using:

```ruby
Get-ComputerInfo -Property WindowsProductName
```

### REMOVE THE EXISTING PRODUCT KEY


```ruby
slmgr /upk
```

### CLEAR THE LICENSE KEY FROM THE REGISTRY

```ruby
slmgr /cpky 
```

### RESTART THE INSTANCE

```ruby
shutdown /r /t 0
```

### REGISTER THE OCI KMS SERVER

```ruby
slmgr /skms 169.254.169.253:1688
```

### RESET ACTIVATION STATE

```ruby
slmgr /rearm
```

### RESTART THE INSTANCE

```ruby
shutdown /r /t 0
```

### RETRIEVE THE PROPER GENERIC VOLUME LICENSE KEY

[Key Management Services (KMS) client activation and product keys](https://learn.microsoft.com/en-us/windows-server/get-started/kms-client-activation-keys?tabs=server2022%2Cwindows1110ltsc%2Cversion1803%2Cwindows81)


### INSTALL THE KMS CLIENT KEY

Example using GVL key for Windows 2022 Standard:

```ruby
slmgr /ipk VDYBN-27WPP-V4HQT-9VMD4-VMK7H
```

### FORCE ACTIVATION

```ruby
slmgr /ato
```

### CHECK THE ACTIVATION STATUS

```ruby
slmgr /dlv
```

## Why OCI-Provided licenses use KMS ?

Using a **Key Management Service (KMS)** allows assigning a temporary key that is periodically renewed, rather than a permanent key.

This activation model is dynamic and designed to support enterprises managing a large number of licenses across multiple servers and client computers in an internal network environment.

**How KMS Works:**
	
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

