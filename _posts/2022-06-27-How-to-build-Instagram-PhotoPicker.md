---
title: How To Build Instagram Stories Photos Picker Using PhotoKit
author: Samisays11
date: 2022-06-27 11:33:00 +0800
categories: [iOS Swift Tutorials]
tags: [Swift, PhotoKit, UIKit, AVKit]
---



## What Are We Building ?
In this tutorial, i will walk you through how to build a clone of **Instagram's Story Photo Picker Feature**.\
If you are not familiar with what that looks like there is a gif demo of it below. For clarity we will be focusing exclusively on the custom Photo Picker functionality since the gesture animations, UI design and transition animations are beyond the scope of this tutorial. Here is the link to the [Starter Project](https://github.com/samisays11/InstaPhotoPicker), it contains all of the afromentioned UI, animations and gesture related code.


 ![Desktop View](/assets/img/demo.gif){: .shadow }
 _Project Demo_



## What You'll Learn
To accomplish our goal we will have to use Apple's PhotoKit, and AVKit APIs respectively.
So at the end of this tutorial you'll know how to: 
- [ ] Setup and Ask for User's Photo Library Permission
- [ ] Fetch Smart Albums and User Collections
- [ ] Fetch High Res Versions of each Photo
- [ ] Fetch Video Assets and their thumbnails
- [ ] Use the Data from the PHAssetCollections to power our UI


## Getting Started
Like many iOS APIs, the PhotoKit requires us to ask for user permission before we can access the user's Camera Roll. The user is presented with a dialog  asking for permission to allow the app to access the Photo Library. To get this permission, we use the PHPhotoLibrary, a shared object that manages access to user's photo library.

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
Open your ViewController.swift file in the starter project and add the **getPhotoPermission** method below the //MARK: - Photokit section
 
```swift
//MARK: - Photokit
 fileprivate func getPhotoPermission(completionHandler: @escaping(Bool) -> Void) {
        
        // 1 If already previously granted proceed to return true
        guard PHPhotoLibrary.authorizationStatus() != .authorized else {
            completionHandler(true)
            return
        }
        
        
        PHPhotoLibrary.requestAuthorization(for: .readWrite) { status in
            switch status {
            case .notDetermined:
                // The user hasn't determined this app's access.
                print("notDetermined")
                //                completionHandler(false)
                
            case .restricted:
                // The system restricted this app's access.
                print("restricted")
                completionHandler(false)
                
            case .denied:
                // The user explicitly denied this app's access.
                print("denied")
                completionHandler(false)
                
            case .authorized:
                // The user authorized this app to access Photos data.
                print("authorized")
                completionHandler(true)
                
                
            case .limited:
                // The user authorized this app for limited Photos access.
                print("limited")
                completionHandler(true)
                
            @unknown default:
                fatalError()
            }
        }
    }
    
```




<span style="color:blue">some blue text</span>.

