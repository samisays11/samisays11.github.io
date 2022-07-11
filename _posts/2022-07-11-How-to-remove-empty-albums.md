---
title: How To Remove Empty PHAssetCollections from a PHFetchResult
author: Osaretin
date: 2022-07-11 03:08:00 +0800
categories: [iOS Tutorials]
tags: [photokit, phassetcollections, phfetchresult, phassets, custom photo picker]
---


```swift
fileprivate var smartAlbums = PHFetchResult<PHAssetCollection>()
fileprivate var userCreatedAlbums = PHFetchResult<PHAssetCollection>()

fileprivate func fetchPhotoAssets(){

/* fetches all smart albums like 
favorites, 
selfies, 
screenshots, 
livephotos, 
panaramas etc in the user's photo lobrary and stores it in a variable named "smartAlbums"
*/
smartAlbums = PHAssetCollection.fetchAssetCollections(with: .smartAlbum, subtype: .albumRegular, options: nil)

// fetches all of the user-created albums in the user's photo library and stores it in a variable named "userCreatedAlbums"
userCreatedAlbums = PHAssetCollection.fetchAssetCollections(with: .album, subtype: .albumRegular, options: nil)
}
```

If you've worked with the PhotoKit framework in the past to build a custom image picker, I'm sure you've had to write a similar code to the one above to fetch user's smart albums and user-created albums.  
The problem is users probably have a handful of empty albums in their photos library. To prevent us from cluttering the UI with empty PHAssetCollections there is a way for us to fetch only PHAssetCollections that contains at least one PHAsset (photo or video).  
We accomplish that by applying predicate on a PHFetchOptions object. See below:

```swift
fileprivate func fetchPhotoAssets(){
// fetches all of the non-empty user-created albums in the user's photo library and stores it in a variable named "userCreatedAlbums"
 let userCreatedAlbumsOptions = PHFetchOptions()
 userCreatedAlbumsOptions.predicate = NSPredicate(format: "estimatedAssetCount > 0")
 userCreatedAlbums = PHAssetCollection.fetchAssetCollections(with: .album, subtype: .albumRegular, options: userCreatedAlbumsOptions)
}
```


<!-- (How to Fetch only PHAssetCollections containing at least one photo) -->