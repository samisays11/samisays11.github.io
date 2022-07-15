---
title: Let's Build a Custom Image Picker Just like Instagram Stories
author: Samisays11
date: 2022-06-27 11:33:00 +0800
categories: [iOS Swift Tutorials]
tags: [swift, ios, photokit, phassetcollections, phfetchresult, phassets, custom photo picker, uikit]
---


## What Are We Building ?
In this tutorial, i will walk you through how to build a clone of **Instagram's Story Photo Picker Feature**.\
If you are not familiar with what that looks like there is a gif demo of it below. For clarity we will be focusing exclusively on the custom Photo Picker functionality since the gesture animations, UI design and transition animations are beyond the scope of this tutorial. Here is the link to the [**Starter Project**](https://github.com/samisays11/InstaPhotoPicker), it contains all of the afromentioned UI, animations and gesture related code.


 ![Desktop View](/assets/img/demo.gif){: .shadow }
 _Project Demo_



## What You'll Learn
To accomplish our goal we will have to use Apple's PhotoKit, and AVKit APIs respectively.
So at the end of this tutorial you'll know how to: 
- [ ] How to Setup and Ask for User's Photo Library Permission.
- [ ] How to Fetch Smart Albums and User-Created Albums.
- [ ] How to Fetch all Assets in a Specific Album.
- [ ] How to Retrieve Images from Assets.
- [ ] Use to use the retrieved PHAssets and PHAssetCollections to power our UI
- [ ] How to use PHPickerViewController to search and select specific assets.
- [ ] How to use the PHPhotoLibraryChangeObserver protocol to keep our assets up to date.


## Getting Started
Like many privacy-centric iOS APIs, the PhotoKit requires us to ask for user permission before we can access the user's Photo Library. The user is presented with a dialog  asking for permission to allow the app to access the Photo Library. To get this permission, we use the PHPhotoLibrary, a shared object that manages access to user's photo library.

### Modifying Info.plist Before Requesting Authorization
Download the [**Starter Project**](https://github.com/samisays11/InstaPhotoPicker) and open the **starter** folder. Double click on the <span style="background-color: #FFFF00">**InstaPhotoPicker.xcodeproj**</span> file and navigate to the **info.plist** file in the root directory. The first step to  Requesting Authorization is to add a key to the **Info.plist** file that explains why you need permission to access the user's photo library.
To do this you will need to:
1. Open the **Info.plist** file.  
2. Right-click **Information Property List** and select **Add Row**. A new line will be displayed. 
3. Start typing <span style="background-color: #FFFF00">**Privacy - Photo Library Usage Description**</span> and press Enter. 
4. In the Value column, type in whatever usage description you want the alert dialog to show the user the first time iOS request for permission. For this tutorial we will just go with <span style="background-color: #FFFF00">**"Access to Photo Library Allows you to upload media from your Camera Roll to InstaPhotoPicker"**</span> as our permission request description.
 
Your **Info.plist** should look like this:
 <img src="/assets/img/info.plist.sc.png">  
 


### Requesting Authorization to Access User's Photo Library
Open your ViewController.swift file in the starter project and add the following code inside your **getPhotoPermission** method in the Photokit section:
 
```swift
//MARK: - Photokit
 fileprivate func getPhotoPermission(completionHandler: @escaping(Bool) -> Void) {
        
        // 1 
        guard PHPhotoLibrary.authorizationStatus() != .authorized else {
            completionHandler(true)
            return
        }
        
        // 2 
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

2. If authorization was not previously granted, we request it. As stated earlier When requesting authorization, iOS displays an alert dialog box that asks for user's permission, upon user's interaction with the alert box it passes back the status of our user's selection as a PHAuthorizationStatus object in its PHPhotoLibrary.requestAuthorization completion handler method.\
We then call our completion handler and return true if the status value is .authorized, or .limited, otherwise we return false.



> **Note:** Prior to iOS 14 PHAuthorizationStatus enum, did not contain a .limited, or .restricted status options. This is something to keep in mind if you're also building for iOS 13 and below. 
{: .prompt-info }

### Finalizing Authorization

Run the project and click on the **Enable photo access** button. On tap iOS will ask for permission to let InstaPhotoPicker access the photo library. If you are using iOS 13 tap **OK** or on iOS 14 tap **Allow Access to All Photos**.

 <img src="/assets/img/alertDialogImage.png">  

Great job! and just like that we are done with:
- [x] How to Setup and Ask for User's Photo Library Permission.



## Understanding Photokit's Main Objects

Before we proceed to actually fetching photos and videos. It's important you get a quick overview of the main objects we will be working with. When working with PhotoKit, you'll be dealing alot with these 3 objects:
 
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
and their various methods to update and retrieve photos, videos, and albums along with their metadatas.

**PHAssets** \
First let's make something absolutely clear, the **PHAsset** object is not an actual video or photo object. Don't confuse it for a UIImage object. It's a metadata that represents the image, LivePhoto, or video as it resides in the user's Photo library. It's an immutable object that provide us the information we need to get the actual image or video and tons of other informations about them like their creation and modification dates, location data, favorite and hidden status etc.

**PHAssetCollections** \
Sometimes you need to retrieve a group of assets, like in the case of an album in the user's photo library. These are usually returned as a PHAssetCollection object. In essence it is PhotosKit's representation of an Album or Moment in the photo library.

**PHFetchResult** \
A simplified way of thinking of PHFetchResult is to consider it an array, which it is, in essence. It contains all the same methods and conventions of arrays, such as count() and index(of:). Plus, it intelligently handles fetching data, caching it and re-fetching it as needed. You’ll be fine if you think of PHFetchResult as an intelligent array of assets or collections. And for the afromentioned reasons we use it over regular arrays for storing our assets and collections.



## Asset Data Models
Now that you got the gist of the main objects we will be using to fetch our photos and albums it's time to set up our data models. Still in our ViewController.swift at the top of the file, add the following under the declaration for the Properties sections:

```swift
//MARK: - Properties
 // 1
 fileprivate var allPhotosInCurrentAlbum = PHFetchResult<PHAsset>()
 // 2
 fileprivate var smartAlbums = [PHAssetCollection]()
 // 3
 fileprivate var userCreatedAlbums = PHFetchResult<PHAssetCollection>()
 // 4
 fileprivate let listOfsmartAlbumSubtypesToBeFetched: [PHAssetCollectionSubtype] = [.smartAlbumUserLibrary, .smartAlbumFavorites, .smartAlbumVideos, .smartAlbumScreenshots]

```
For simplicity, i will be referring to PHFetchResult as a smart-array. 
1. In essence, think of it as Initializing an empty smart-array we will use in housing all the photos and videos we intend to fetch from the user's photo library.
2. Initializing an empty smart-array to house the smartAlbums we will be fetching from the user's photo library.
3. Initializing an empty smart-array to house the user-created albums we will be fetching from the user's photo library.
4. We will only be fetching four distinct smart Albums from the user's photo library. This is a list we will use to filter for that goal.  
You might be thinking, “Hey, what is this **PHAssetCollectionSubtype** thing ?” Well, PHAssetCollectionSubtype is simply an enumerable value that describes the particular subtype of a PHAssetCollection. Basically a way for us to specify which exact smart albums we want to fetch.


<br/><br/>



## Fetching The Assets {#Fetching-The-Assets}
  
Still in the ViewController.swift file go to the **fetchPhotoLibraryAssets** method below the **getPhotoPermission** method in the marked Photokit section and add the following code &darr;
```swift
//MARK: - PhotoKits
 fileprivate func fetchPhotoLibraryAssets() {
        
        // 1
        let authStatus = PHPhotoLibrary.authorizationStatus()
        guard authStatus ==  .authorized || authStatus == .limited else {return}
        
        // 2
        DispatchQueue.main.async {
            self.askPhotoPermissionView.updateTexts()
        }

        // 3
        for collectionSubType in  listOfsmartAlbumSubtypesToBeFetched {
            if let smartAlbum = PHAssetCollection.fetchAssetCollections(with: .smartAlbum, subtype: collectionSubType, options: nil).firstObject {
                smartAlbums.append(smartAlbum)
            }
        }
        
        // 4
        let userCreatedAlbumsOptions = PHFetchOptions()
        userCreatedAlbumsOptions.predicate = NSPredicate(format: "estimatedAssetCount > 0")
        userCreatedAlbums = PHAssetCollection.fetchAssetCollections(with: .album, subtype: .albumRegular, options: userCreatedAlbumsOptions)

        // 5
        let fetchOptions = PHFetchOptions()
        let sortDescriptor = NSSortDescriptor(key: "creationDate", ascending: false)
        fetchOptions.sortDescriptors = [sortDescriptor]
        allPhotosInCurrentAlbum = PHAsset.fetchAssets(with: fetchOptions)

        // 6
        DispatchQueue.main.async {
            self.mediaPickerView.bindDataFromPhotosLibrary(fetchedAssets: self.allPhotosInCurrentAlbum, albumTitle: "Recents")
        }
    }
    
```

1. This gets us the current authorization status from PHPhotoLibrary. If we’ve not been granted authorization, we return out of the method.

2. This updates a UILabel's text in the askPhotoPermissionView to "Swipe Up to View Photos". Since the PhotoKit's methods automatically runs in the background thread, it's important to update UI related stuff on the main thread.

3. When retrieving assets from the photo library with PHFetchResult, we can use a PHAssetCollectionSubtype object to retrieve a specific type of smart album. Here, we loop through our **listOfsmartAlbumSubtypesToBeFetched** array using the PHAssetCollection's fetchAssetCollection method to fetch the recent, favorites, videos, and screenshot smart albums. We then append them to the **smartAlbums** array we declared earlier. 

4. When retrieving user created albums or smart albums, we can use a PHFetchOptions object to apply a set of sorting paramters to indicate how we would like the retrieved assets to be sorted. Here, we create a PHFetchOptions object and use a predicate to specify that we only want to fetch user created albums that contains at least one photo or video.

5. When retrieving assets from the photo library with PHFetchResult, a set of sorting paramters to indicate how we would like the retrieved assets to be sorted can also be applied to the PHFetchOptions object. Here, we create a sort descriptor that sorts assets by creation date from newest to oldest, and then we use the PHAsset's fetchAssetsfetch method to fetch all assets in the user's photo library.

6. PhotoKit's methods automatically runs in the background thread, so we jump to the main thread to pass asset data to our to mediaPickerView to update the UI.


## Displaying All Photos In The User's Photo Library
Now that we are done fetching data from the Photo Library's Data Store let's display them in our UI.  
Navigate to **MediaPickerView.swift** file in the Views folder and replace the collectionView's **cellforItem**, **numberOfItemsInSection** and **didSelectItem** methods at the end of the class with these:


```swift
// 1
 func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: PhotoCell.cellReuseIdentifier, for: indexPath) as! PhotoCell
        cell.bind(asset: allPhotosInSelectedAlbum[indexPath.item])
        return cell
  }

// 2
 func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return allPhotosInSelectedAlbum.count
  }


// 3
 func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
      guard let cell = collectionView.cellForItem(at: indexPath) as? PhotoCell else {return}
      let asset = allPhotosInSelectedAlbum[indexPath.item]
      let image = getAssetThumbnail(asset: asset, size: PHImageManagerMaximumSize)
      cell.thumbnailImageView.image = image
      delegate?.handleTransitionToStoriesEditorVC(with: cell.thumbnailImageView)
    }

```

1. Here we are binding our array of assets to each cell's UI.
2. Here is a good example of how you treat PHFetchResult as an array. We are Returning the number of assets in the PHFetchResult as the number of items in the collectionView. 
3. Here we are getting the selected cell's imageView and asset and then calling the **getAssetThumbnail** helper method to fetch the asset's highest quality image. Then a delegate to handle the transition animation to StoriesEditorVC.  

> **Note:** To request the maximum possible size for a PHAsset's image we use the **PHImageManagerMaximumSize** object.
{: .prompt-info } 

## How We Get UIImage from PHAsset Object
The **getAssetThumbnail** method in the Constants.swift file inside the Misc folder is responsible for requesting a PHAsset's image from the PHImageManager. In essence, this is how we convert a PHAsset object to a UIImage object. I suggest you click on the method and take a look at the code:
```swift
public func getAssetThumbnail(asset: PHAsset, size: CGSize) -> UIImage? {
    // 1
    let manager = PHImageManager.default()
    let options = PHImageRequestOptions()
    options.isSynchronous = true
    options.isNetworkAccessAllowed = true
    
    var thumbnail: UIImage?
    // 2
    manager.requestImage(for: asset, targetSize: size, contentMode: .aspectFill, options: options) {(imageReturned, info) in
        guard let thumbnailUnwrapped = imageReturned else {return}
        thumbnail = thumbnailUnwrapped
    }
    return thumbnail
}
```
1. We instantiate PHImageManager with PHImageRequestOptions. 
2. We are requesting the image from the image manager. Providing the asset, size, content mode, and options. Size is the size at which you would like the image returned. contentMode is how you would like the image to fit within the aspect ratio of the size. The default value is aspectFill. And then finally we return the requested image in the result handler.  

Alright now that we are done discussing how we fetch images from assets let's build and run the project. In the app swipe up to view all photos in the photos library. Hooray 🙌✊🥳🎉👏 your collectionview now has images and video thumbnails!  


 ![Desktop View](/assets/img/projectDemo2.gif){: .shadow }
 _Hooray 🙌✊🥳🎉👏 your collectionview now has images and video thumbnails!_


## Passing Assets to AlbumVC
Navigate back to the **ViewController.swift** file and find the **handleOpenAlbumVC** method in the marked MediaPickerViewDelegate section. Replace the code in the **handleOpenAlbumVC** with the one below:

```swift
//MARK: - MediaPickerViewDelegate
func handleOpenAlbumVC() {
    // 1
     let albumVC = AlbumVC(smartAlbums: smartAlbums, userCreatedAlbums: userCreatedAlbums)
     albumVC.modalPresentationStyle = .custom
     albumVC.transitioningDelegate = self
     albumVC.delegate = self
     present(albumVC, animated: true, completion: nil)
}

```

1. Here we are injecting the smart albums and user created albums we fetched into the albumVC initializer right before the albumVC is modally presented.  

Now with the AlbumVC injected with the album assets, let's use the assets to display the albums.  
Displaying the cover image for an album is simply a matter of requesting each album's cover image from PHImageManager. The AlbumVC is almost set up to display albums, let us complete the setup process and get it working.


## Displaying Albums
Head to the **AlbumVC.swift** file in the Controllers folder and add this below the marked properties section:
```swift
// 1
fileprivate lazy var smartAlbumSection = [SmartAlbumItem(albumName: "Search", imageName: "magnifyingglass"),
                                          SmartAlbumItem(albumName: "Recents", imageName: "clock", collection: smartAlbums[0]),
                                          SmartAlbumItem(albumName: "Favorites", imageName: "heart", collection: smartAlbums[1]),
                                          SmartAlbumItem(albumName: "Videos", imageName: "play.circle", collection: smartAlbums[2]),
                                          SmartAlbumItem(albumName: "Screenshots", imageName: "iphone", collection: smartAlbums[3])
    ]
```

1. The smartAlbumSection is an array of SmartAlbumItems. This is a simple struct to power our tableView's first section.
Each item holds a albumName, imageName, and an optional PHAssetCollection(the actual smart-album). The PHAssetCollection property of the SmartAlbumItems struct is optional because we know the first item (the Search item) in the smartAlbumSection array will not a have a PHAssetCollection property.

Now scroll to the marked **TableView Protocols** section and let's update the tableView's cellForRowAt, didSelectRowAt, and numberOfRowsInSection methods to display our album's UI:  



Add the **cellForRowAt** method
```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let sectionType = albumSections[indexPath.section]
        switch sectionType {
        case .smartAlbums:
            let smartAlbumCell = dequeSmartAlbumCell(for: indexPath)
            return smartAlbumCell
        case  .userCreatedAlbums:
            let userCreatedAlbumCell = dequeUserCreatedAlbumCell(for: indexPath)
            return userCreatedAlbumCell
        }
    }
    
```
Here we deque each cell based on the album section type. Where section 0 is a smartAlbumCell and 1 is a userCreatedAlbumCell.  


Above the cellForRowAt method add this **dequeSmartAlbumCell** method:
```swift
fileprivate func dequeSmartAlbumCell(for indexPath: IndexPath) -> UITableViewCell {
        //1 cell dequeing
         let smartAlbumCell = tableView.dequeueReusableCell(withIdentifier: smartAlbumCellIdentifier, for: indexPath)
         smartAlbumCell.backgroundColor = .clear
         smartAlbumCell.selectionStyle = .none
        
        //2 configuring SmartAlbumCell's UI and data binding
        let album = smartAlbumSection[indexPath.row]
        var contentConfig = smartAlbumCell.defaultContentConfiguration()
        contentConfig.text = album.albumName
        
        let config = UIImage.SymbolConfiguration(pointSize: 15, weight: .semibold, scale: .large)
        let image = UIImage(systemName: album.imageName, withConfiguration:
                                config)?.withRenderingMode(.alwaysTemplate)

        contentConfig.image = image
        contentConfig.imageProperties.tintColor = .white
        contentConfig.textProperties.color = .white
        contentConfig.imageToTextPadding = 12
        smartAlbumCell.contentConfiguration = contentConfig
        
         return smartAlbumCell
     }
```
1. Here we are dequeing each smartAlbumCell.
2. Configuring each smartAlbumCell with the new iOS 14.0+ tableview cell's UI configurations. Then binding the smartAlbumSection data to display each smartAlbumCell's title and icon.  


Right below the dequeSmartAlbumCell method add this **dequeUserCreatedAlbumCell** method:
```swift
fileprivate func dequeUserCreatedAlbumCell(for indexPath: IndexPath) -> AlbumCell {
         
         // 1
         let userCreatedAlbumCell = tableView.dequeueReusableCell(withIdentifier: userCreatedAlbumCellIdentifier, for: indexPath) as! AlbumCell
         userCreatedAlbumCell.backgroundColor = .clear
         userCreatedAlbumCell.selectionStyle = .none
         
         // 2
         var coverAsset: PHAsset?
         let aUserCreatedAlbum = userCreatedAlbums[indexPath.item]

         // 3
         let fetchOptions = PHFetchOptions()
         fetchOptions.fetchLimit = 1
         let sortDescriptor = NSSortDescriptor(key: "creationDate", ascending: false)
         fetchOptions.sortDescriptors = [sortDescriptor]
         
         // 4
         let fetchedAssets = PHAsset.fetchAssets(in: aUserCreatedAlbum, options: fetchOptions)
         coverAsset = fetchedAssets.firstObject
         guard let asset = coverAsset else { return userCreatedAlbumCell }
         
         // 5
         let coverImage = getAssetThumbnail(asset: asset, size: userCreatedAlbumCell.bounds.size)
         userCreatedAlbumCell.bindData(albumTitle: aUserCreatedAlbum.localizedTitle ?? "", albumCoverImage: coverImage)
         
         return userCreatedAlbumCell
     }
```
1. Here we are dequeing each userCreatedAlbumCell.

2. Here we create variables to hold an asset, which will be used as each album's cover image. And another variable to get each user-created album.

3. Since we only want to fetch the most recent image in each user-created album, we use PHFetchOptions's fetchLimit and sortDescriptor to limit our fetch result to the most recently added asset in the album.

4. We retrieve the album's first asset using the PHAsset's fetchAssets method (which is a PHFetchResult object) and set it as the cover asset.

5. We grab the cover asset's image from PhotoKit's PHImageManager using our getAssetThumbnail method and then bind the title and image data to power our cell's UI.


Add the **numberOfRowsInSection** method
```swift
 func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        switch albumSections[section] {
        case .smartAlbums: return smartAlbumSection.count
        case .userCreatedAlbums: return userCreatedAlbums.count
        }
    }
    
```
Here we return the number of items in each section based on the album section type, so the tableView knows how many items to display in each section. 


Add the **didSelectRowAt** method
```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        switch albumSections[indexPath.section] {
        //1
        case .smartAlbums:
            
            if let smartAlbum = smartAlbumSection[indexPath.row].collection {
                delegate?.handleDidSelect(smartAlbum: smartAlbum)
            } else {
                delegate?.handlePresentPHPickerViewController()
            }
            
            dismiss(animated: true)
        // 2    
        case .userCreatedAlbums:
            delegate?.handleDidSelect(smartAlbum: userCreatedAlbums[indexPath.row])
            dismiss(animated: true)
        }
    }
    
```


1. Here if the user selects an item in the smart-album section we simply delegate the action to the parent ViewController.swift file, which is responsible for fetching the assets in the selected album and then we dismiss the AlbumVC. And if the user selects the search item we simply present a PHPickerViewController in the  ViewController.swift file via delegation.

2. On selection of an item in the user created-albums section we delegate the action to the ViewController.swift file and dismiss the AlbumVC.

Run the project and open the albums. You should see albums with their names and cover images being displayed!
Hooray 🙌✊🥳🎉👏 we are done displaying albums, all that's left is changing our mediaPickerView's assets to show the selected album's assets when didSelectRow method is Triggered from the AlbumVC.  

 <!-- I NEED A OPEN ALBUMVC GIF HERE -->


![Desktop View](/assets/img/projectDemo2.geffefif){: .shadow }
 _Hooray 🙌✊🥳🎉👏 we are done displaying albums!_

## Fetching Selected Album's Assets
To display the selected album's photos in our mediaPickerView's UI, navigate back to the **ViewController.swift** file. Locate the **handleDidSelect** method below the marked **AlbumVCDelegate** section and add the following code to it:

```swift

  func handleDidSelect(smartAlbum: PHAssetCollection) {
        let fetchOptions = PHFetchOptions()
        let sortDescriptor = NSSortDescriptor(key: "creationDate", ascending: false)
        fetchOptions.sortDescriptors = [sortDescriptor]
        let fetchedAssets = PHAsset.fetchAssets(in: smartAlbum, options: fetchOptions)
        allPhotosInCurrentAlbum = fetchedAssets
        mediaPickerView.bindDataFromPhotosLibrary(fetchedAssets: allPhotosInCurrentAlbum, albumTitle: smartAlbum.localizedTitle ?? "")
    }
    
```



Pretty self-explanatory from our [Fetching The Assets](#Fetching-The-Assets) section. Here we are simply fetching the selected album and passing it to our mediaPickerView to update our UI.  
Now let's run the project, open the albums and select any album, you should see the mediaPickerView's assets update to display those from the selected album.

 <!-- I NEED A OPEN ALBUMVC GIF AND SELECTING ALBUM THAT CHANGES MEDIAPICKER VIEWS PHOTO GIF HERE -->

![Desktop View](/assets/img/projectDemo2.gsdffeif){: .shadow }
 _Hooray 🙌✊🥳🎉👏 we are done displaying albums!_


## Searching Photos Library Using PHPickerViewController
The first item in our smart album section is a search item. To mimic the instagram app's design of this UX flow we will be using the PHPickerViewController. A view controller that provides the user interface to search and pick photos from the photo library.  
The PHPickerViewController uses the traditional delegate model that will alert you upon user interation completion.  

Still in the **ViewController.swift** file, locate the **handlePresentPHPickerViewController** method which is below the marked **AlbumVCDelegate** section and update it content to this:

```swift
func handlePresentPHPickerViewController() {
        // 1
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.2) {
            // 2
            var configuration = PHPickerConfiguration(photoLibrary: PHPhotoLibrary.shared())
            configuration.selectionLimit = 10
            // 3
            let picker = PHPickerViewController(configuration: configuration)
            picker.delegate = self
            self.present(picker, animated: true, completion: nil)
        }
    }
```

1. We are using the DispatchQueue's async delay method to ensure that the AlbumVC was dismissed before we attempt to present the PHPickerViewController.

2. The PHPickerConfiguration allows us to set a multi-selection limit of 10.

3. We instantiate the PHPickerViewController and set it's delegate before presenting it.

> **Note:** We can also filter the type of media that is presented to the user via PHPickerViewController by using **configuration.filter = .images** or **configuration.filter = .any(of: [.livePhotos, .images])**.  
{: .prompt-info }


#### PHPickerViewControllerDelegate
Just below the **handlePresentPHPickerViewController** method we have the marked **PHPickerViewControllerDelegate** section. Update the **picker()** method to this: 
```swift
func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
        // 1
        dismiss(animated: true)
        let identifiers = results.compactMap(\.assetIdentifier)
        let fetchResult = PHAsset.fetchAssets(withLocalIdentifiers: identifiers, options: nil)
        // 2
        DispatchQueue.main.async {
            self.mediaPickerView.bindDataFromPhotosLibrary(fetchedAssets: fetchResult, albumTitle: "Search Result")
        }
    }
```
1. First we dismiss the PHPickerViewController upon completion of photo selection. Then we retrieve the assets for the selected photos and videos using their assetIdentifier.
2. We pass the retrieved assets to be displayed by the mediaPickerView's UI.

Now let's run the project, open the album and tap the search item. The PHPickerViewController will present a familiar UI of your photos and albums. Search and select photos and then tap the add button to dismiss. You should have something similar to this happening:

 <!-- I NEED A SELECT SEARCH ITEM IN ALBUM AND SHOW  PHPickerViewController  ux flow gif-->

![Desktop View](/assets/img/projectDemo2.gsdffeif){: .shadow }
 _Hooray 🙌✊🥳🎉👏 we are done displaying albums!_


## PHPhotoLibraryChangeObserver
Now that our app is running nicely let's address how to handle changes in the photos library.  
We listen to changes in the photos library such as when a photo is deleted, added or edited by subscribing to the PHPhotoLibraryChangeObserver.  
According to Apple's own documentation: The PHPhotoLibraryChangeObserver protocol notifies you of changes that occur in the photo library, regardless of whether those changes are made by your app, by a user in the Photos app, or by another app that uses the Photos framework.

### Listening For Changes in ViewController
To register for updates in the **ViewController.swift** file navigate to the **fetchPhotoLibraryAssets** method. Add this registration code anywhere below the **PHPhotoLibrary.authorizationStatus()** logic:

```swift
fileprivate func fetchPhotoLibraryAssets() {
            
  // 1
 let authStatus = PHPhotoLibrary.authorizationStatus()
 guard authStatus ==  .authorized || authStatus == .limited else {return}
         
    // ADD THIS LINE OF CODE BELOW 1.
    // Please note We are only subscribing to PHPhotoLibraryChangeObserver if we have authorization to access photo library.
   PHPhotoLibrary.shared().register(self)

.
.
.
.
  }
  
```

With our view controller registered to receive updates. Add this to deregister the updates in the **deinit** method just below the **viewDidLoad** method:

```swift
deinit {
  PHPhotoLibrary.shared().unregisterChangeObserver(self)
}
  
```


Scroll down to the end of the ViewController.swift file and conform to the PHPhotoLibraryChangeObserver protocol with the following code:

```swift
//MARK: - PHPhotoLibraryChangeObserver
extension ViewController: PHPhotoLibraryChangeObserver  {
    // 1.
    func photoLibraryDidChange(_ changeInstance: PHChange) {
        DispatchQueue.main.async {
            // 2.
            if let changeDetails = changeInstance.changeDetails(for: self.allPhotosInCurrentAlbum) {
                self.allPhotosInCurrentAlbum = changeDetails.fetchResultAfterChanges
                // 3.
                self.mediaPickerView.bindDataFromPhotosLibrary(fetchedAssets: self.allPhotosInCurrentAlbum, albumTitle: self.mediaPickerView.getCurrentAlbumTitle())
            }
        }
    }
}
  
```
1. The change observer has only one method: photoLibraryDidChange(:). Every time the library changes, it triggers this method.

2. You need to check if the update affects our allPhotosInCurrentAlbum assets. Use changeInstance, a property that describes the library changes, by calling its changeDetails(for:) and pass in your asset. It returns nil if your asset is not affected by the changes. Otherwise, you retrieve the updated version of the asset by calling fetchResultAfterChanges.

3. Refreshing the mediaPickerView's UI to reflect the changes.


### Listening For Changes in AlbumVC
Similar to what we just did in the ViewController.swift file, let's go to the AlbumVC.swift file to register for changes in album assets.
Find the viewDidLoad method in the albumVC and modify it to this:

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        setUpTableView()
        view.backgroundColor = .clear
        // ADD THIS TO VIEWDIDLOAD
        let authStatus = PHPhotoLibrary.authorizationStatus()
        if authStatus ==  .authorized || authStatus == .limited {
            PHPhotoLibrary.shared().register(self)
        }
    }
```
Similarly lets unregister from the ChangeObserver in the deinit method. Add this anywhere in the marked Init section of your AlbumVC:
```swift
deinit {
        PHPhotoLibrary.shared().unregisterChangeObserver(self)
    }
```

Scroll down to the end of the AlbumVC.swift file and conform to the PHPhotoLibraryChangeObserver protocol with the following code:

```swift
//MARK: - PHPhotoLibraryChangeObserver
extension AlbumVC: PHPhotoLibraryChangeObserver  {
    
    func photoLibraryDidChange(_ changeInstance: PHChange) {
        DispatchQueue.main.async {
            //1. 
            if let changeDetails = changeInstance.changeDetails(for: self.userCreatedAlbums) {
                self.userCreatedAlbums = changeDetails.fetchResultAfterChanges
            }
            self.tableView.reloadData()
        }
    }
    
}
```

1. Since only the user-created album section has cover images, that's the only assets we should be listening to for changes.
We check if the update affects our userCreatedAlbums assets. Use changeInstance with it's changeDetails(for:) method to retrieve the updated versions of the userCreatedAlbums assets by calling fetchResultAfterChanges.

And just liked that both our AlbumVC.swift and ViewController.swift files are now listening for any changes that occurs in the photos library!

## Congratulations!
You did it! 🙌✊🥳🎉👏 you just replicated Instagram story's custom photo picker. That was a lot to digest in a short time, hopefully you got a good introduction to the power of apple's photokit framework. Remember to [**download the source code**](https://github.com/samisays11/InstaPhotoPicker).  

So far we've covered: 
- [x] How to Setup and Ask for User's Photo Library Permission.
- [x] How to Fetch Smart Albums and User-Created Albums.
- [x] How to Fetch all Assets in a Specific Album.
- [x] How to Retrieve Images from Assets.
- [x] Use to use the retrieved PHAssets and PHAssetCollections to power our UI
- [x] How to use PHPickerViewController to search and select specific assets.
- [x] How to use the PHPhotoLibraryChangeObserver protocol to keep our assets up to date.

### What's Next?
The PhotoKit is a robust framework with much to offer. There is still so much we can do with the impressive library that's beyond the scope of this tutorial. There are cool stuff like LivePhoto, video and the photo editing functionalities etc. Check out the [**Apple’s PhotoKit Documentation**](https://developer.apple.com/documentation/photokit) for more information.  

That's all peeps, I hope you liked the tutorial and with that sami says bye 👋.
