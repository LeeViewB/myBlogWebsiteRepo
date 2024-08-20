+++
title = "Automatically deploy newer versions of an app - IntuneMac"
description = ""
type = ["posts","post"]
tags = [
    "Intune",
    "macOS",
    "apps"
]
date = "2022-09-30"
categories = [
    "Intune",
    "macOS"
]
[ author ]
  name = "Liviu"
+++

{{< image src="/img/blogpost2/1.jpg" position="center" style="border-radius: 8px;" >}}

## Introduction

Nowadays, the number of organizations with an Intune license, which manage their MacOS fleet using it, is starting to grow significantly. This can be practical for a variety of reasons, but the most important one would be cost-saving - why pay for another management platform, when you already have Intune, right?
Well, as useful as it may be, once it's configured, Admins run into an issue when it comes to packaging applications.
Now, some more skeptical might say:

> "Why package applications when you have the AppStore, you can purchase the apps from Apple Business Manager (ABM), have them synced to Intune, and just assign them?"

The answer is simple - not all applications are available in the AppStore for MacOS: Google Chrome and Edge are just 2 examples from a long list of apps.
To avoid always having to manually package up newer versions as they are released, you can automate the process by using Intune Shell Scripts.

Credit to [theneiljohnson](https://github.com/theneiljohnson) for creating the scripts, now we can simply adapt them to the application we want.
Scripts for a good selection of apps are already available in Github, such as Company Portal, Adobe Reader, Edge, Teams, Remote Desktop, and many more. The script can also be customized to reference the app you want.

## Pros and Cons of using Intune Shell Scripts
Of course, using Shell Scripts to deploy applications has its Pros and Cons.

### Pros:
This [ReadMe](https://github.com/microsoft/shell-intune-samples/tree/master/macOS/Apps) ~~already comes~~ (the page was updated in the meantime) with some valid reasons as to why you might use scripts for App Deployment, such as:
• App isn't signed and you don't have access to the correct certificate
• App is complex, has embedded apps or multiple payloads (not compatible with macOS MDM stack pkg or dmg delivery)
• App has complex dependencies, and you want to handle those via a script
• You just don't want the hassle of packaging the app regularly

### Cons:
However, when it comes to Cons, there are none listed, therefore I took the liberty to list a couple:
• They can only be assigned as Required. You cannot assign a script as available. Some users might not want that application installed.
Even if they uninstall it, if you re-run the script automatically, it will simply get re-installed.
• There is no detection, so the IntuneShell script will run even if the application is already installed.
The good news is, the script is written so that it will not attempt the installation if the latest version is already installed.

## DEMO: Using IntuneShell Scripts for App Deployment

For starters, go to Shell-Intune-Samples and copy the [nstall.Edge.sh](https://github.com/microsoft/shell-intune-samples/tree/master/macOS/Apps/Edge) script. We will adapt this to install Google Chrome.
At this time, there isn't one provided by Microsoft in their GitHub repo.

{{< image src="/img/blogpost2/02.png" position="center" style="border-radius: 8px;" >}}

You won't require any scripting skills, as all we need to do is update a few variables (see the above image).

For **weburl** it works without an Azure Blob Storage URL too. It just needs to be a static download URL.
Then we can configure if we want to terminate a conflicting process (defined in the processpath variable) or not, or if the application will autoUpdate or not. If autoUpdate will be set to true, the script will not attempt the install anymore, if any Chrome version is already installed.
In my case, Chrome will auto-update itself, so even if my script will re-run on that device, it will not do anything.

If you want to test the script before you upload it to Intune, you can do so on your mac device.
You will need to first make the script executable, so for that, we will use the [chmod](https://www.computerhope.com/unix/uchmod.htm) command:

> ```chmod +x pathToYourScript```

Then we can run it. I ran my test as the root user, as that's how my script will eventually run from Intune.

{{< image src="/img/blogpost2/03.png" position="center" style="border-radius: 8px;" >}}

It ran successfully, so now let's go to Intune.

Go to **Devices** -> **macOS** -> **Shell Scripts** -> **Add Script**.
On the **Basics** tab, give it the name you want, then confirm with **Next**.

{{< image src="/img/blogpost2/04.png" position="center" style="border-radius: 8px;" >}}

And then let's move on, to the Scripts Settings, as there we will have some important choices to make.
You will be prompted to upload your script, but make sure to scroll down, as there will be some other settings to configure too!

{{< image src="/img/blogpost2/05.png" position="center" style="border-radius: 8px;" >}}

{{< image src="/img/blogpost2/06.png" position="center" style="border-radius: 8px;" >}}

You can choose whether or not you want the script to run as the logged-on user or as the root one.
Note: If you run it as root, users without admin rights will be unable to remove the app.

In my case, I have set it to run as the root user, to hide script notifications on devices, and to retry 3 times if the script fails.
I have not configured any frequency, as I only want it to run once on the device. Chrome will auto-update. In some cases, you might want it to be re-run on a regular basis.

Once these options are configured, you only need to set the scope tags, assignments, and create the script.

Now you can go to the Company Portal on your client, and sync the changes. As soon as the script will run, you should see the log file under */Library/Logs/Microsoft/IntuneScripts/installChrome*.

## Troubleshooting

What if the script fails? How can I tell what the cause is? Well, you'll be able to collect the logs remotely using Intune too!

If you go to **Overview**, and then go on **Device Status**, select the **Device** you want to collect the logs from, and then hit the **Collect Logs** button.

{{< image src="/img/blogpost2/07.png" position="center" style="border-radius: 8px;" >}}

You will have to specify a log file, so enter the path to it. You can specify multiple files if you want to.
The **IntuneMDMDaemon** log file does not need to be specified, as that will be included by default.

{{< image src="/img/blogpost2/08.png" position="center" style="border-radius: 8px;" >}}

Confirm with **OK** and then a request to collect the logs will be sent. 

If the device is online, it's a matter of minutes. After the logs are collected, from the same screen, the Download Logs option will become available.

Happy troubleshooting!

{{< image src="/img/blogpost2/09.png" position="center" style="border-radius: 8px;" >}}