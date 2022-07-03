---
title: Building Instagram Stories Photos Picker Using PhotoKit
author: Samisays11
date: 2022-06-27 11:33:00 +0800
categories: [iOS Swift Tutorials]
tags: [Swift, PhotoKit, UIKit, AVKit]
---

## What it will look like 
 <img src="/assets/img/finalimg.sc.png" width="400" height="400">  


## What You'll Learn
In this tutorial, i will walk you through how to build a clone of **Instagram's Story Photo Picker** using Apple's PhotoKit, and AVKit APIs respectively.

At the end of this tutorial you'll know how to: 
* Ask for User's Photo Library Permission
* Fetch Smart Albums and User Collections
* Fetch High Res Versions of each Photo 
* Fetch Video Assets and their thumbnails
* Use the Data from the PHAssetCollections to power our UI

## Getting Started
Like many iOS APIs, the PhotoKit requires us to ask for user permission before we can access their Camera Roll. The user is presented with a dialog  asking for permission to allow the app to access the Photo Library. To get this permission, we use the PHPhotoLibrary, a shared object that manages access to user's photo library.

### Asking for Permission to Access The User's Photo Library 
The first step is to add a key to the **Info.plist** file that explains why you need permission to access the user's photo library.
To do this you will need to:
1. Open the **Info.plist** file.  
2. Right-click **Information Property List** and select **Add Row**. A new line will be displayed. 
3. Start typing <span style="background-color: #FFFF00">**Privacy - Photo Library Usage Description**</span> and press Enter. 
4. In the Value column, type in whatever usage description you want the alert dialog to show the user the first time iOS request for permission. For this tutorial i'll just type in <span style="background-color: #FFFF00">**"Access to Photo Library Allows you to upload media from your Camera Roll to InstaPhotoPicker"**</span>
 
Your **Info.plist** should look like this:
 <img src="/assets/img/info.plist.sc.png">  


### Requesting Authorization
Navigate to the ViewController.swift file in our starter project and add the authorization code below in our handleSetUpPhotoPermission method

 
```swift
fileprivate func handleSetUpPhotoPermission() {
        // 1
        guard PHPhotoLibrary.authorizationStatus() != .authorized else {
//          completionHandler(true)
          return
        }
        // 2
        PHPhotoLibrary.requestAuthorization { status in
//          completionHandler(status == .authorized)
        }
    }
print s
```




<span style="color:blue">some blue text</span>.

