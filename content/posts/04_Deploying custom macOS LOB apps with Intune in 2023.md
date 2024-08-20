+++
title = "Deploying custom macOS LOB apps with Intune in 2023"
description = ""
type = ["posts","post"]
tags = [
    "Intune",
    "macOS",
    "apps"
]
date = "2023-01-10"
categories = [
    "Intune",
    "macOS"
]
[ author ]
  name = "Liviu"
+++

{{< image src="/img/blogpost4/01.jpg" position="center" style="border-radius: 8px;" >}}

## Introduction

If you're managing macOS with Intune in 2023, you'll likely have to package some apps. Google Chrome, Spotify, and a lot more apps are not available in the macOS store. Some of them you'll have to customize

Microsoft has made it a lot easier to add .pkg and .dmg installers, as [you no longer need to convert them to .intunemac](https://learn.microsoft.com/en-us/mem/intune/apps/lob-apps-macos#select-the-app-type) files before uploading them. .pkg and .dmg files are now supported to be uploaded directly. You can also [automatically install](http://localhost:1313/posts/02_automatically-deploy-newer-versions-of-an-app-intunemac/) an app on your macOS devices every time there is a newer version using scripts.

However, in some cases, you might need to repackage those apps if you want to apply some customization, such as pre/post-install scripts, clean-up entries before install, and so on.
This guide will go over what's needed to repackage a pkg or dmg. As an example, we use Spotify. The installer provided by the vendor is a dmg, which, upon execution, goes online to download and install the app.

## Requirements

Assuming you already manage your mac devices with Intune, this is what you will need in order to repackage a pkg/dmg file and upload it as a LOB app in Intune (other than the Intune license).

### 1. Apple Developer Certificate

The first thing you will need is an [Apple Developer ID Certificate](https://developer.apple.com/support/developer-id/). This will be used to sign your re-packaged .pkg file. It is also mandatory, [Microsoft](https://learn.microsoft.com/en-us/mem/intune/apps/lob-apps-macos#select-the-app-type) set this as a requirement.
If the pkg file isn't signed, you will still be able to create the LOB app in Intune, but it will simply fail to install.

If you don't have a certificate yet, make sure to enroll in the Apple Developer Program to get a certificate. This process may take from a few days to a couple of weeks.
Once you will have the certificate, download and install it to KeyChain.

### 2. Packages app

The second thing you will need is the [Packages](http://s.sudre.free.fr/Software/Packages/about.html) app. This simple app will allow you to define what applications, bundles, documents, scripts should be part of the payload of your install packages. You can also easily define the installation directory, and the permissions as well.

### 3. mac Device

The final thing we will need is a mac. On this device, we will install the certificate, and the Packages app, then repackage our application.

## Repackaging using the Packages app

Download and install the packages app mentioned in the Requirements step. I also downloaded and installed Spotify.

1. Open the Packages app. Create a new **Distribution** project. This type of project will allow you to include multiple packages and add custom details. These custom details are only relevant for a manual install. We won't install Spotify manually, so these details don't matter.
2. Select your project. In my case, I just went on **Spotify** on the left-side ribbon.

{{< image src="/img/blogpost4/02.png" position="center" style="border-radius: 8px;" >}}

3. On the Project tab, you don't need to do anything. However, on the **Settings** tab, I always enter the identifier of the actual app, and its version too.

- By default, the Packages app will enter an automatically generated identifier and version.

{{< image src="/img/blogpost4/03.png" position="center" style="border-radius: 8px;" >}}

- The identifier will matter when it comes to the detection of the app. Intune will use the apps identifier as detection criteria. If you need to get the identifier of an app, you can do it by running this command in Terminal:

```
codesign -dv --entitlements - /Path/To/App.app
```

{{< image src="/img/blogpost4/04.png" position="center" style="border-radius: 8px;" >}}

4. On the Payload tab, you need to specify, well, the payload. You can simply drag and drop the application in the folder where it should be installed.
Another approach is to right-click on the target folder and add the files.\
\
You can also configure the permissions for this app in the bottom-right corner. I left the default.

{{< image src="/img/blogpost4/05.png" position="center" style="border-radius: 8px;" >}}
{{< image src="/img/blogpost4/06.png" position="center" style="border-radius: 8px;" >}}

5. If you need to add any pre- or post-installation shell scripts, you can do so from the Scripts tab.
We didn't add any in our example.

{{< image src="/img/blogpost4/07.png" position="center" style="border-radius: 8px;" >}}

6. Now click on **Project** in the left-side ribbon. Go to the **Project** Menu and click on **Set Certificate**. Here you can select your Developer Certificate. Once you set it in the project, we can just build the pkg file. It will then be signed automatically for us.

> **NOTE:** If you want to manually sign the pkg file later on, then skip this step.

{{< image src="/img/blogpost4/08.png" position="center" style="border-radius: 8px;" >}}

7. Once the certificate is set, simply go on the **Build** menu and hit the **Build** option (or press Command + B)

{{< image src="/img/blogpost4/09.png" position="center" style="border-radius: 8px;" >}}

8. The new PKG file will then be generated.

{{< image src="/img/blogpost4/10.png" position="center" style="border-radius: 8px;" >}}
{{< image src="/img/blogpost4/11.png" position="center" style="border-radius: 8px;" >}}

- If you didn't sign the .pkg using the Packages app, you can do so manually from Terminal using this command:

```
productsign --sign "Developer ID Installer: companyName (certID)" /pathTo/unsigned.pkg /pathTo/signed.pkg
```

9. Before we upload the pkg in Intune, let's ensure the file was signed properly. There are 2 ways to verify the signature.

- You can just run the pkg and click on the padlock in the top-right corner. It should show you the certificate details. If the padlock is missing, it means it's not signed properly.

{{< image src="/img/blogpost4/12.png" position="center" style="border-radius: 8px;" >}}

- Another option to verify the signature is to simply run this command:

```
pkgutil --check-signature /PathTo/YourApp.pkg
```
{{< image src="/img/blogpost4/13.png" position="center" style="border-radius: 8px;" >}}

13. Test the installation and ensure it works as intended.

## Creating the Line-of-Business app in Intune

1. At this point, we are ready to create the LOB in Intune. Go to Intune -> then on Apps -> macOS -> add a new Line-of-business app.

{{< image src="/img/blogpost4/14.png" position="center" style="border-radius: 8px;" >}}

2. Upload the pkg file, and add the details you want to have in your app. In my case, I ignore the app version, as the app will likely auto-update, so I want it to still be detected as installed based on the identifier we set.

{{< image src="/img/blogpost4/15.png" position="center" style="border-radius: 8px;" >}}

3. All that remains to do is to assign the app.

## Troubleshooting

1. In some cases, you might run into some issues. If the pkg file is not signed, then the installation will fail. Ensure that the .pkg is signed.

2. Another issue you might run into is detection. I learned this the hard way. I still had the Spotify application present on my MacBook, but not under Applications. It was residing on the Desktop. When I ran the install from the Company Portal, it would simply be detected.\
It took me a bit to realize that if the identifier is present on the device, Intune will just mark it as already installed.

3. If the macOS LOB apps are deployed, but they never install, and Intune will not show any error messages, follow the steps from this article.