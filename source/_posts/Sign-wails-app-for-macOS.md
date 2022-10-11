---
title: "Sign wails app for macOS"
typora-root-url: ../../source
date: 2022-10-10 22:30:11
tags: [golang, wails, macOS]
---

## Problem

Once you built the [wails](https://wails.io) application, if you want other mackbooks to open it, you need to sign it. Otherwise, in other macbook, it will directly show the app cannot be opened with only one "Move to trash" option



## Solution

You can use [gon](https://github.com/mitchellh/gon) to sign it.

Steps:

### 1, Install gon

```bash
brew install mitchellh/gon/gon
```

### 2, Check if you have apple cert for applcation signing:

```bash
security find-identity -v -p codesigning
```

if you found any item that listed as "Developer ID Application", grab the string before it as its ID, like `88422442...52767439C5`.
If you have multiple like mine here,  you can just use any of them. Now you can go to step 4. If you didn't see any item marked as "Developer ID Application", go to step 3

![screenshot](https://cdn.jsdelivr.net/gh/bofeng/drive@main/uPic/2022-1665455630288-2JNRnL.png)

### 3, Create cert for "Developer ID Application"

You can do this with Xcode:

1. Open xcode, go to "Preference > Account"

2. Click "Manage certificates" in the bottom right.

3. In the pop-up modal, click "+" on the bottom-left, then choose "Developer ID Application". (Please notice, you have to sign in to your Apple ID's root/admin account, otherwise, you won't see the "Developer ID Application" option)

   ![screenshot](https://cdn.jsdelivr.net/gh/bofeng/drive@main/uPic/2022-1665455277842-JAgFGl.png)

Once done, run the above `security find-identity -v -p codesigning` command again, you should be able to see an item marked with "Developer ID Applicaiton", grab its string ID.

### 4, Create gon configuration file

Go to your wails application's `build/darwin` folder, create 3 files:

1, gon-sign.json

```json
{
  "source": ["./build/bin/Example.app"],
  "bundle_id": "app.example.appname",
  "apple_id": {
    "username": "your apple id",
    "password": "@env:application-passwrd"
  },
  "sign": {
    "application_identity": "string ID you grabbed in step 2"
  }
}
```

* `source` is the application path

* `bundle_id`: you can use your subdomain as the app's bundle id

* `apple_id`: username is the your apple id; password, you need to get an application specific password. Go to https://appleid.apple.com/ , sign in, then click "App-Specific Password". The password is like xxxx-xxxx-xxxx-xxxx. Copy and paste it as the password field, "@env:xxxx-xxxx-xxxx-xxxx"

  ![screenshot](https://cdn.jsdelivr.net/gh/bofeng/drive@main/uPic/2022-1665456160744-mrVUhu.png)

* `sign.application_identity`: the string ID you grabbed with the `security find-identity -v -p codesigning` command.



2, entitlements.plist

In this file you configure the entitlements you need for you app, e.g. camera permissions if your app uses the camera. Read more about entitlements [here](https://developer.apple.com/documentation/bundleresources/entitlements).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.app-sandbox</key>
  <true/>
  <key>com.apple.security.network.client</key>
  <true/>
  <key>com.apple.security.network.server</key>
  <true/>
  <key>com.apple.security.files.user-selected.read-write</key>
  <true/>
  <key>com.apple.security.files.downloads.read-write</key>
  <true/>
</dict>
</plist>
```

3, Open the `Info.plist` file in this folder (build/darwin), change the `CFBundleIdentifier`, make sure its value is the same as the "bundle_id" field in your `gon-sign.json`:

```xml
<key>CFBundleIdentifier</key><string>app.example.appname</string>
```



### 5, Sign the app

Now go to your wails project root folder, run:

```bash
gon -log-level=info ./build/darwin/gon-sign.json
```

If success, it will sign the app under your `build/bin` folder. Now you can send this app to other macbook. When other macbooks try to open it, the user can go to "System Preferences & Privacy" > "Open it anyway".



## Reference

* https://wails.io/docs/guides/signing
* https://github.com/mitchellh/gon
