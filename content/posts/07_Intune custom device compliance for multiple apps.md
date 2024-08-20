+++
title = "Intune custom device compliance for multiple apps in a single compliance rule"
description = ""
type = ["posts","post"]
tags = [
    "Intune",
    "Windows",
    "custom device compliance",
    "compliance rule"
]
date = "2023-08-20"
categories = [
    "Intune",
    "Windows"
]
[ author ]
  name = "Liviu"
+++

{{< image src="/img/blogpost7/01.jpg" position="center" style="border-radius: 8px;" >}}

## Introduction
I love working with apps. It's also my bread and butter.
A short time ago, I needed to look into device custom compliance and check if the installed version on their devices is greater than or equal to specific version (in my case, the latest). Additionally, I needed to this for multiple apps.

Full disclosure, I started checking out community scripts to see if anyone else did this before, and if they did, my intention was to re-use their script.

I found some scripts, but they only checked whether an application was installed, not the version. Also, there were none that would check multiple apps either. Since I didn't want to create multiple compliance rules for each app, I started writing my own script.

In case you're a reader that just gets into device custom compliance, some background information first. Otherwise, if you're a seasoned admin, skip the next chapter.

>**Note**: If this article is useful, make sure to read the Bonus section at the end, it will simplify things even more!

## What is custom device compliance?

The answer is simple. Intune provides a range of pre-configured compliance settings you can use to create a policy and check whether or not the device has the settings you want it to have.

For instance, you can ensure that the device is compliant with a specific minimum OS version, or if Bitlocker is enabled, and so on.

However, the predefined settings are limited, and this is where the custom device compliance comes in - it provides the ability to create tailored compliance policies tailored to your organization's specific needs.

To create a custom compliance policy, you will need two things:

- A PowerShell script - this is the first thing you'll have to define and upload. When this script runs as part of the custom compliance policy, it discovers the settings you specified in the JSON file (second thing needed). More details can be found on [Microsoft Learn](https://learn.microsoft.com/en-us/mem/intune/protect/compliance-custom-script).\
One important aspect is to have the line below as the last line in your script to convert your hash table into a compressed JSON string and then return it.
```
return $hash | ConvertTo-Json -Compress
```

- A JSON file - you will create rules in this file that determine if the device is compliant or not based on the data returned by the PowerShell script. This file will have to be in a specific format for it to work. More details can be found on [Microsoft Learn](https://learn.microsoft.com/en-us/mem/intune/protect/compliance-custom-json).

An even simpler explanation of custom device compliance policies: custom rules created in a JSON file, and a script that checks if devices follow these rules.

## The script

Let's talk about the script. It can be found in my [GitHub Repo](https://github.com/LeeViewB/CheckComplianceScripts/tree/main/Check-ComplianceMultipleApps), along with a couple of other examples.

#### The PS1 file
This PowerShell script will allow you to:

- Specify a list of application names in an array. The script can check for wildcards as well, but I recommend entering the exact name shown in appwiz.cpl - as that is what it is the value returned in the hash table.\
The ARP DisplayName found for that software is going to be used as the SettingName in the JSON file as well.
- Choose whether to check for these applications in the current user's profile (HKCU) or in the machine-wide installation profile (HKLM).\
If you decide to go for HKCU, you'll have to ensure the script is configured to run as the logged-on user when uploaded in Intune. More on that later.
- Decide whether to only check if the application is installed or additionally check its version.

In summary, this script checks for the presence or the version of specified applications installed on a device. Then, it compiles the results into a JSON format, and sends this information to a system like Microsoft Intune for compliance checking.

#### Examples
If you are looking for examples, there are some in the [ReadMe](https://github.com/LeeViewB/CheckComplianceScripts/tree/main/Check-ComplianceMultipleApps) file in the repo. I also included some screenshots that show how the output looks like based on which variables you set in the script.

> **NOTE:** If you're going to check the version and the software is not installed, then it's going to return 0.0.0.0 for it.

## Creating the custom compliance policy

To create a new custom compliance policy, you will first need to upload your script.
Go to the **Intune Admin Center** -> **Devices** -> **Compliance Policies** -> **Scripts** -> Add your "**Windows 10 or later**" script.

{{< image src="/img/blogpost7/02.png" position="center" style="border-radius: 8px;" >}}

Give it a name and hit **Next**.

{{< image src="/img/blogpost7/03.png" position="center" style="border-radius: 8px;" >}}

Paste your script at the next screen.

> **IMPORTANT** If you configure the script to check the HKCU for your app(s) by setting the $userProfileApp variable to $true, make sure to also configure Intune to run the script using the logged on user credentials.

{{< image src="/img/blogpost7/04.png" position="center" style="border-radius: 8px;" >}}

Create your script once pasted. After the script is created, you'll need to create the policy. Ensure that you have modified the JSON file accordingly.\
Go to the **Intune Admin Center** -> **Devices** -> **Compliance Policies** -> Create Policy for **Windows 10 and later**.

{{< image src="/img/blogpost7/05.png" position="center" style="border-radius: 8px;" >}}

Next, expand **Custom Compliance** and configure it to require a **Custom Compliance Script**. Select the script you have just uploaded.\
After that, upload the JSON file you have adjusted to reflect the same setting name as the apps you are checking with your script. See the image below for visual aid.

{{< image src="/img/blogpost7/06.png" position="center" style="border-radius: 8px;" >}}

By clicking **Next** you will be able to choose your desired actions for noncompliance, set assignments, and then finally create the compliance policy.\
Wait for the compliance sync to take place. If you want it to happen faster, go to a targeted device and run this PowerShell one-liner:

```
Start-Process -FilePath "C:\Program Files (x86)\Microsoft Intune Management Extension\Microsoft.Management.Services.IntuneWindowsAgent.exe" -ArgumentList "intunemanagementextension://synccompliance"
```

## The results

Once the sync is complete, if the device is not compliant - the software version specified in the JSON file is not installed - it will inform the user about the non-compliance in the Company Portal.

In the image below, you'll see how the JSON rules impact the messages shown to the user in the Company Portal if the device is not compliant. You can zoom in on the page to see the screenshot better.

{{< image src="/img/blogpost7/07.png" position="center" style="border-radius: 8px;" >}}

If you were to check for any apps in **HKCU**, the result would be the same, assuming you configured the script to run as the logged-on user.

{{< image src="/img/blogpost7/08.png" position="center" style="border-radius: 8px;" >}}

If you configured the same script to only **check for app presence in HKCU**, that's what it will do. The result will look like this:

{{< image src="/img/blogpost7/09.png" position="center" style="border-radius: 8px;" >}}

You get my point. One script. Adjust it. Do your compliance check.

## Bonus
But hey, there's a bonus. If you don't want to make these JSON and PS1 adjustments manually, I have also created a GUI, just for fun. You will find it in the same [GitHub Repo](https://github.com/LeeViewB/CheckComplianceScripts/tree/main/Check-ComplianceMultipleApps).

{{< image src="/img/blogpost7/10.png" position="center" style="border-radius: 8px;" >}}

You'll be able to choose which type of check you're going to do:

{{< image src="/img/blogpost7/11.png" position="center" style="border-radius: 8px;" >}}

If you're going to do an App Version check, you'll be prompted to enter the minimum version of the software to be installed for it to be compliant.

{{< image src="/img/blogpost7/12.png" position="center" style="border-radius: 8px;" >}}

Once you do, you'll be prompted to save a zip file. This archive will include both the PS1 and JSON file adjusted to the settings you picked in the GUI.

That's it! :grin:
