+++
title = "Edge settings don't apply on MacOS devices - Intune"
description = ""
type = ["posts","post"]
tags = [
    "Intune",
    "macOS",
    "Edge"
]
date = "2022-08-28"
categories = [
    "Intune",
    "macOS"
]
[ author ]
  name = "Liviu"
+++

### UPDATE: As of January 2023, the favorites can be applied using a list, but not with the settings catalog.

Over the past few days, I've been trying to find a good topic to write about for my first article. After some intense procrastination… I found it!

While working on managing MacOS devices with Intune, I have reached the point where I needed to configure Edge - this is a requirement, as we also have Windows devices, and we want our users to be able to sync their browser between their devices.

For configuring Edge, I have followed the documentation Microsoft provides. Therefore, after assigning the built-in Intune Edge app to my Macbook test device, I started working on the configuration profile for it.
Next, I have used the Settings Catalog and have chosen just 2 settings - to start off slowly and to make sure the profile applies correctly.

{{< image src="/img/blogpost1/01.png" alt="Edge Configuration profile macOS Intune" position="center" style="border-radius: 8px;" >}}

After running a sync on the device, the profile was applied. This could be seen in the **Profiles**:

{{< image src="/img/blogpost1/02Macbook_Setting_Applied.png" alt="macOS Settings applied" position="center" style="border-radius: 8px;" >}}
{{< image src="/img/blogpost1/03_SettingsAppliedIntune.png" position="center" style="border-radius: 8px;" >}}

However, the settings didn't apply in Edge. Then I remembered that with Apple Devices, the identifier is important. We can see that Intune automatically applied the identifier for Edge as com.microsoft.Edge.
If we use the below-listed command to check the identified of Edge, it's clearly another one. We can see that the identifier is com.microsoft.edgemac.
This is not the case with only the Intune built-in Edge app, as I manually uninstalled that, and manually installed the latest version from the MS website, it was the same identifier for Edge, for a Macbook device with an Apple chip.

```
codesign -dv --entitlements - PathToApp
```

{{< image src="/img/blogpost1/04_ActualEdgeIdentifier.png" alt="Edge Identifier" position="center" style="border-radius: 8px;" >}}

As a next step, I decided to use a preference file (plist) to apply my settings. Microsoft provides details guides on [how to create and deploy your plist](https://learn.microsoft.com/en-us/deployedge/configure-microsoft-edge-on-mac).

More details about Information Property Files can be found [here](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/AboutInformationPropertyListFiles.html) and [here](https://learn.microsoft.com/en-us/mem/intune/configuration/preference-file-settings-macos).
A list of supported policies and their preference key names can be found here.

After creating the plist from Terminal, and converted it from binary to plain text format, I made my customizations.

{{< image src="/img/blogpost1/05_Plist_ConfigProfile.png" position="center" style="border-radius: 8px;" >}}

The Configuration Profile with the plist applied successfully:

{{< image src="/img/blogpost1/06_PlistAppliedSuccess.png" position="center" style="border-radius: 8px;" >}}

But Edge still remained unconfigured, because Intune couldn't find the app to configure.

At this moment, I am stuck. This is most likely related to the identifier, however, I cannot determine whether Intel chip devices are impacted as well, as no one in the community ever mentioned this issue before, applying custom settings to Edge always worked for everyone.

All that remains right now, is to reach out to people who know people. I will update this article once a fix will be found.