---
title: Picking Photos in Swift 5 using PHPickerViewController
author: Osaretin
date: 2022-07-19 03:08:00 +0800
categories: [iOS Tutorials]
tags: [swift, ios, phpickerviewController, phassetcollections, phfetchresult, phassets, photo picker, uikit, phpicker, photosui]
image:
  path: https://www.creativeboom.com/uploads/articles/c6/c658f02157b50035414db2456d18d61ab872f653_810.jpeg
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: image
---


<!-- Using PHPickerViewController to Select a Photo on iOS 14 -->
Prior to iOS 14, the quickest and simplest way to fetch photos from the user's photo library was through the UIImagePickerController class. However, starting from iOS 14.0, Apple is providing us developers a new way to access the user's photo library using the new **PHPickerViewController** class. A reusable class built on top of the PhotosUI framework rather than UIKit. It makes photo and video selection a much modern experience for apps.

## Why Even Use PHPickerViewController?

Unless your app still supports anything below iOS 14, dumping the old guard of UIImagePickerController should be very easy when you consider some of the disadvantages that came with the old guy.  
The old UIImagePickerController was fairly basic and limited. It had no search or filtering functionality and did not support multi-item selection. Although the UIImagePickerController class is not currently deprecated, if you look at the header file you'll see the API marked with this:

```swift
API_DEPRECATED("Will be removed in a future release, use PHPicker.", ios(11, API_TO_BE_DEPRECATED));

```
So expect deprecation in future updates.

## What's the Buzz

Here's a list of why PHPickerViewController is awesome
1. **No more permission alerts**. Developer don't have to specify the Privacy - Photo Library Usage Description key in the info.plist file anymore, because there won’t be any need for alert views that's asking for permission.
2. **Runs in a separate process**. Although the PHPickerViewController might appear to be present inside the host app, it runs in a separate process. 
3. **Built-in privacy**. Since PHPickerViewController runs in a separate process, only the content selected by the user is shared with the Host App. Unlike the UIImagePickerViewController, you can't programmatically take a screenshot of this picker view or access any content that's not selected by the user.
4. **Built-in search**. Searching photos is a built-in feature in PHPickerViewController, and the UI is similar to that of the Photos app.
5. **Multi-selection**. PHPickerViewController comes equiped with multi-selection capabilty and also allows the developer to limit the number of photos a user can select by setting the **selectionLimit** property in PHPickerConfiguration.


## Implementing PHPickerViewController

Getting started with the new PHPickerViewController API is pretty straightforward. It requires a configuration and uses the traditional delegate model that alert's us when user selection is done. 

The <mark>PHPickerViewController</mark> takes an instance of <mark>PHPickerConfiguration</mark> as an initialiser argument. 


```swift
import PhotosUI


 fileprivate func setUpImagePicker() {
        var configuration = PHPickerConfiguration()
 }

```


### PHPickerConfiguration

The <mark>PHPickerConfiguration</mark> provides us with three properties we can use to customize the PHPickerViewController.


1. **selectionLimit** - This property specifies the number of items a user can select. The default value is 1, while the value 0 specifies unlimited selection. The below configuration limits the selection to 10 items.

2. **filter** - This property restricts the type of items that can be displayed (images, livePhotos or videos). Setting the value to nil will display all the supported items. The below configuration displays only the live photos and images.

3. **preferredAssetRepresentationMode** - This determines how an item provider should represent an asset. The default value is automatic. The below configuration sets the asset’s representation mode to automatic. 


> **Note:** By default the selectionLimit is set to 1, and there is no filter.  
{: .prompt-info }

```swift
import PhotosUI

 fileprivate func setUpImagePicker() {
        var configuration = PHPickerConfiguration()
        configuration.selectionLimit = 10
        configuration.filter = .any(of: [.images,.livePhotos])
        configuration.preferredAssetRepresentationMode = .automatic
        let picker = PHPickerViewController(configuration: configuration)
        picker.delegate = self
        present(picker, animated: true, completion: nil)
    }

```

### PHPickerViewControllerDelegate

Accessing selected items from the PHPickerViewController is simply a matter of implementing the <mark> PHPickerViewControllerDelegate </mark>, which has just one method:  

**picker's didFinishPicking method**
```swift
 func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
        
 }
```
The method comes with a result object which is an array of type PHPickerResult which contains an object of NSItemProvider. Now let's see how we can parse selected images from this item provider object.  


**Parsing Selected Images from ItemProvider**
```swift

func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {

       // it's the developer’s responsibility to dismiss the PHPickerViewController upon didFinishPicking completion
        dismiss(animated: true, completion: nil)
        guard !results.isEmpty else { return }
        
        
        for result in results {

            let itemProvider = result.itemProvider
            
            if itemProvider.canLoadObject(ofClass: UIImage.self) {

                itemProvider.loadObject(ofClass: UIImage.self) { (image, error) in

                    DispatchQueue.main.async {

                        if let image = image as? UIImage {

                            self.images.append(image)
                            self.collectionView.reloadData()
                        }
                    }
                }
            }
        }
    }

```
> **Note:** There is no support for automatic compression. The picker's itemProvider will always load the original video/image size and quality attributes. It's up to the developer to perform any necessary compressions or edition.
{: .prompt-info }


## Resources

* [**Download the source code**](https://github.com/samisays11/PHPickerViewController-Demo)
* [**Apple Video Resource**](https://developer.apple.com/videos/play/wwdc2020/10652/)



