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
Like many privacy-centric iOS APIs, the PhotoKit requires us to ask for user permission before we can access the user's Camera Roll. The user is presented with a dialog  asking for permission to allow the app to access the Photo Library. To get this permission, we use the PHPhotoLibrary, a shared object that manages access to user's photo library.

### Modifying Info.plist Before Requesting Authorization
The first step is to add a key to the **Info.plist** file that explains why you need permission to access the user's photo library.
To do this you will need to:
1. Open the **Info.plist** file.  
2. Right-click **Information Property List** and select **Add Row**. A new line will be displayed. 
3. Start typing <span style="background-color: #FFFF00">**Privacy - Photo Library Usage Description**</span> and press Enter. 
4. In the Value column, type in whatever usage description you want the alert dialog to show the user the first time iOS request for permission. For this tutorial we will just go with <span style="background-color: #FFFF00">**"Access to Photo Library Allows you to upload media from your Camera Roll to InstaPhotoPicker"**</span>
 
Your **Info.plist** should look like this:
 <img src="/assets/img/info.plist.sc.png">  


### Requesting Authorization to Access User's Photo Library
Open your ViewController.swift file in the starter project and add the **getPhotoPermission** method below the //MARK: - Photokit section
 
```swift
//MARK: - Photokit
 fileprivate func getPhotoPermission(completionHandler: @escaping(Bool) -> Void) {
        
        // 1 If already previously granted proceed to return true
        guard PHPhotoLibrary.authorizationStatus() != .authorized else {
            completionHandler(true)
            return
        }
        
        // 2 Request Authorization
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


1. This gets us the current authorization status from PHPhotoLibrary. If we've already been granted authorization from prior prompts, we call the completion handler with a value of true and return out of the method.

2. If authorization was not previously granted, we request it. As stated earlier When requesting authorization, iOS displays an alert dialog box that asks for user's permission, upon user's interaction with the alert box it passes back the status of our user's selection as a PHAuthorizationStatus object in its **PHPhotoLibrary.requestAuthorization** completion handler method.\
We then call our completion handler and return true if the status value is .authorized, or .limited, otherwise we return false.



> **Note:** Prior to iOS 14 PHAuthorizationStatus enum, did not contain a .limited, or .restricted status options. This is something to keep in mind if you're also building for iOS 13 and below. 
{: .prompt-info }


In the **viewDidLoad** method call the getPhotoPermission method and run the project. iOS will ask for permission to access the photo library when InstaPhotoPicker launches. If you are using, iOS 13 tap **OK** or on iOS 14, tap **Allow Access to All Photos**.
Your viewDidLoad method in your ViewController.swift file should look something like this:
```swift


//MARK: - View's LifeCycle
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .black
        setUpViews()
        setUpGestureRecognizers()
        
        // call the getPhotoPermission method
        getPhotoPermission { [weak self] granted  in
            if granted {
                self?.fetchPhotos()
            }
        }
        
    }

```
- [x] Setup and Ask for User's Photo Library Permission was Completed.


## Understanding Photokits Main Objects

When working with PhotoKit, you'll be dealing alot with these 3 objects:
 
<details>
  <summary> PHAssets </summary>
  TL;DR &rarr;
  Basically a metadata that represents an image, video, or Live Photo in the Photos library.
</details>

<details>
  <summary> PHAssetCollections </summary>
  TL;DR &rarr;
 A group of PHAssets. Basically a user-created album, or an ios smart album.
</details>

<details>
  <summary> PHFetchResults </summary>
  TL;DR &rarr;
  Basically PhotosKit's smart array that fetches and auto caches objects like PHAssets or PHAssetCollection.
</details>

\
and their various methods to update and retrieve photos, videos, and albums along with their metadatas.\
Okay Great! So what are these objects ?

**PHAssets** \
First let's make something absolutely clear, the **PHAsset** object is not an actual video or photo object. Don't confuse it for a UIImage object. It's a metadata that represents the image, LivePhoto, or video as it resides in the user's Photo library. It's an immutable object that provide us the information we need to get the actual image or video and tons of other informations about them like their creation and modification dates, location data, favorite and hidden status etc.

**PHAssetCollections** \
Sometimes you need to retrieve a group of assets, like in the case of an album in the user's photo library. These are usually returned as a PHAssetCollection object. In essence it is PhotosKit's representation of an Album or Moment in the photo library.

**PHFetchResult** \
A simplified way of thinking of PHFetchResult is to consider it an array, which it is, in essence. It contains all the same methods and conventions of arrays, such as count() and index(of:). Plus, it intelligently handles fetching data, caching it and re-fetching it as needed. You’ll be fine if you think of PHFetchResult as an intelligent array of assets or collections. And for the afromentioned reasons we use it over regular arrays for storing our assets and collections.


## Fetching PHAssets and PHAssetCollections
