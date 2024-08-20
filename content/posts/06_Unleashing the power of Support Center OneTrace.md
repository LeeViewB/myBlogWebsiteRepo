+++
title = "Unleashing the power of Support Center OneTrace"
description = ""
type = ["posts","post"]
tags = [
    "ConfigMgr",
    "Windows",
    "Log reader",
    "Support Center"
]
date = "2023-05-19"
categories = [
    "ConfigMgr",
    "Windows"
]
[ author ]
  name = "Liviu"
+++

{{< image src="/img/blogpost6/01.jpg" position="center" style="border-radius: 8px;" >}}

## Introduction

Support Center OneTrace is an invaluable diagnostic and troubleshooting tool offered by Microsoft specifically for Microsoft Endpoint Configuration Manager (formerly known as SCCM or System Center Configuration Manager). It serves as a powerful resource, empowering IT professionals to delve deep into the intricacies of Configuration Manager clients, facilitating seamless issue analysis and resolution.

[Support Center OneTrace](https://learn.microsoft.com/en-us/mem/configmgr/core/support/support-center-onetrace) has been around for quite a few years now. Microsoft created it to replace CMTrace. Initially, it faced the challenge of dethroning its predecessor, and maybe it still does, but I do think it has risen above expectations and established itself as an admin must-have when it comes to troubleshooting tools. Although I was reluctant at first to give it a chance, as I clung to the all-familiar CMTrace, my perception changed when I decided to revisit Ontrace just last year.

After I familiarized myself with its interface and its functions, it left me no room to ever consider going back to CMTrace. I now consider it a remarkable tool that boasts an impressive range of features that ensure a seamless troubleshooting experience. Let's review some of the fantastic features it has to offer.

## OneTrace Installation

Before we get into its features, I'd like to mention that this tool is not publicly available for download.
If you want to get your hands on it, you'll need a Configuration Manager environment. Should you already have one, on your site server you will find the SupportCenterInstaller.msi at the following path:

```
%configMgr_Installation_Directory%\cd.latest\SMSSETUP\Tools\SupportCenter\SupportCenterInstaller.msi
```

If you don't have a ConfigMgr setup and you don't want to go through the complicated process of creating one just to get your hands on the installer, no worries. You can always register for a complete, preconfigured, and free [Windows and Office 365 lab environment](https://learn.microsoft.com/en-us/microsoft-365/enterprise/modern-desktop-deployment-and-management-lab?view=o365-worldwide).

## OneTrace Setup and Configuration
Every time I install Support Center OneTrace on any of my new test devices, I like to enable some settings. This is entirely subjective, but I do feel I get a better overview of the logs this way.

The first thing I like to enable is **Dark Mode**, which is in Preview at the moment. It can be enabled from View -> Dark Mode.

Having **Log Entry** enabled will allow me to see the entire text included in a row, even if it doesn't fit in the **Log Viewer** window. In the top right corner of **Log Entry**, you will find a :pushpin: Pin button, which will allow you to pin the windows so it won't minimize.

I like to use the **Advanced Filters** as they allow me to filter or highlight specific words, filter the log to include a timeframe, exclude rows, and a lot more.

{{< image src="/img/blogpost6/02.png" position="center" style="border-radius: 8px;" >}}

## View Status Messages

As this is a ConfigMgr tool, we can use it to view the Status Messages of all the ConfigMgr components.
You can use this option either locally on your site server, or you can connect remotely to verify the status of the components.

{{< image src="/img/blogpost6/03.png" position="center" style="border-radius: 8px;" >}}
{{< image src="/img/blogpost6/04.png" position="center" style="border-radius: 8px;" >}}

The **View Status Messages** option is a ConfigMgr-specific feature, but other than this, OneTrace can be used with any log files.

## Create groups of log files

While you can still open multiple log files in the same tab, just as in CMTrace, now you have the ability to create a group of files, to make it easier to open them.
This can be handy when you open the same log files over and over again.

{{< image src="/img/blogpost6/05.png" position="center" style="border-radius: 8px;" >}}
{{< image src="/img/blogpost6/06.png" position="center" style="border-radius: 8px;" >}}

OneTrace will then allow you to open all the logs from your group at once. The group can be always modified or deleted, if required.

{{< image src="/img/blogpost6/07.png" position="center" style="border-radius: 8px;" >}}

## Highlight log entries

This is my favorite OneTrace option 🥰. What better way to explain it than using a real-life troubleshooting example? During a support case, I was investigating the failure of an app, to see when it tried to install on a specific device managed by Intune.\
I collected all the IME logs from the device and opened all of them in the same tab, and then took advantage of the **Advanced filters** to make it easier to go through the IME logs.

{{< image src="/img/blogpost6/08.png" position="center" style="border-radius: 8px;" >}}

With this feature, you will be able to **Include**, **Exclude** or **Highlight** specific rows based on keywords. These 3 options can be used not just with **Log Text**, they can be used with other columns such as **File Name**, **Severity**, **Component**, **Thread**, **Time** and **Source File**.
Regular Expressions are supported as well.

{{< image src="/img/blogpost6/09.png" position="center" style="border-radius: 8px;" >}}

The best part is that you can export your configuration to your own filter set, and you will then be able to easily import the same configuration every time it's needed.

{{< image src="/img/blogpost6/10.png" position="center" style="border-radius: 8px;" >}}

The Ribbon - other tips and tricks
With such great features already listed, I'd like just to mention some last useful options OneTrace has:

1. Jump between the **next / previous error**:

{{< image src="/img/blogpost6/11.png" position="center" style="border-radius: 8px;" >}}

2. **Enable / Disable Hints** - I would recommend keeping this enabled. When using Filters to highlight specific words with different colours, next to the scroll bar you will find another bar with the matches OneTrace found based on your Filter configuration.

{{< image src="/img/blogpost6/12.png" position="center" style="border-radius: 8px;" >}}

3. **Enable / Disable** live updating of the log file at will.

{{< image src="/img/blogpost6/13.png" position="center" style="border-radius: 8px;" >}}

4. **Decode certificate** - you will be able to paste the serialized certificate value for any certificate on the client. Once you Process the information, OneTrace will give you general information and details about the certificate. You will also be able to export the certificate to a .cer file.

{{< image src="/img/blogpost6/14.png" position="center" style="border-radius: 8px;" >}}

5. **Error lookup** - This is available in CMTrace too. It will allow you to enter an error code and search for what it means. Here is an example of a lookup of a 32-bit hexadecimal error code. Signed and unsigned 32-bit integer codes are also supported.

{{< image src="/img/blogpost6/15.png" position="center" style="border-radius: 8px;" >}}

## Final thoughts

OneTrace encompasses a large variety of functions that can give you a tremendous advantage when it comes to troubleshooting issues and going through log files.
However, I think there are also a couple of downsides to this tool:

- If you still image devices with ConfigMgr, you won't be able to use this tool during WinPE. It uses Windows Presentation Foundation (WPF) - a component that isn't available in WinPE. Microsoft recommends keeping using CMTrace in such cases.
- The installer only comes with a ConfigMgr setup. It is not available to those who don't have one.

This being said, I do think a key strength of Support Center OneTrace lies in its centralized interface, which conveniently collects and analyzes a myriad of log files. This centralized approach enables swift identification and troubleshooting of any issue, offering an efficient pathway to resolution.

By incorporating Support Center OneTrace into your toolbox, you can optimize your troubleshooting efforts, and solve any issues you might encounter faster than ever before.