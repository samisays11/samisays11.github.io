---
title: How to Drag & Re-order CollectionView Cells
author: Osaretin
date: 2022-07-19 03:08:00 +0800
categories: [iOS Tutorials]
tags:
  [
    swift,
    ios,
    uikit
  ]
image:
  path: https://i.stack.imgur.com/CJbiO.png
  width: 1000 # in pixels
  height: 400 # in pixels
  alt: image
---

Reordering Collection View Cells is a feature that was introduced in iOS 9. It is a surprisingly easy interaction to build, both in collectionviews and tableviews. By leveraging the <mark>UILongPressGestureRecognizer</mark> class we can easily implement a drag and reorder interaction of collectionview cells. It's simply matter of updating the collectionview's data model to match the re-order. 



## Overview

There are two things that needs to be done for us to implement the drag to reorder interaction:
1. Setup long press gesture recognizer to handle interactive drag movement.

2. Override collectionview's two delegate methods that's responsible for reordering cells.



## Getting Started

Initialize and add a long press gesture recognizer to your collectionview:

```swift
let longGestureGesture = UILongPressGestureRecognizer(target: self, action: #selector(handleLongPressGesture))
collectionView.addGestureRecognizer(longGestureGesture)

```
Don't forget to create a target selector for your gesture recognizer:


```swift
 @objc fileprivate func handleLongPressGesture(gesture: UILongPressGestureRecognizer) {
        switch gesture.state {
        case .began:
            guard let targetIndexPath = collectionView.indexPathForItem(at: gesture.location(in: collectionView)) else {return}
            collectionView.beginInteractiveMovementForItem(at: targetIndexPath)
        case .changed:
            collectionView.updateInteractiveMovementTargetPosition(gesture.location(in: collectionView))
        case .ended:
            collectionView.endInteractiveMovement()
         default:
            collectionView.cancelInteractiveMovement()
        }
    }
```
There is only 3 gesture states we are concerned with: the **.began**, **.changed**, and **.ended** states.

The **.began** case logic:  
Once the user long press interaction begins, we want to locate the cell they are long pressing, then we want to start the movement of that cell, so that when the user drags the cell, the cell will move accordingly with the finger. To accomplish this we leverage the collectionview's indexPathForItem method to grab the cell's indexPath and then we signal to the collectionview to begin interactive movement through it's beginInteractiveMovementForItem method.

> **Note:** The <mark>targetIndexPath</mark> constant is unwrapped because the **indexPathForItem** method is not guaranteed to return an indexPath. Since the user might be long pressing between cells or the collectionview's padded area, might not necessarily have a location, hence the unwrapping.
<!-- {: .prompt-tip } -->



The **.changed**, **.ended** and **default** cases are pretty straightforward and self-explanatory.

## Updating CollectionView's Data Model

With the long press gesture recognizer setup to handle collectionview interaction, we have to update the data models

```swift
// Responsible for collectionView Re-order
//1
    override func collectionView(_ collectionView: UICollectionView, canMoveItemAt indexPath: IndexPath) -> Bool {
        return true
    }
   
    //2
    override func collectionView(_ collectionView: UICollectionView, moveItemAt sourceIndexPath: IndexPath,
                                 to destinationIndexPath: IndexPath) {
        
        // first grab and remove item at sourceIndex
        let photo = photosArray.remove(at: sourceIndexPath.item)
        photosArray.insert(item, at: destinationIndexPath.item)

        
    }

```

1. This method enables us to specify which cells we want to allow the user to move. This can be pretty useful when dragging and reordering items between multiple sections. It can be used to enabled or disable reordering interaction for a specific cell or even an entire collectionview section. Here we return true since we want all our cells to be movable in this scenario.

2. This method is responsible for updating our data model. Making sure the cells order matches the data model's. When this method is not implemented the cells will essentially return to their old positions when the collectionview is scrolled.
When a cell is moved to a new position, the original index position in the array is removed and the new index position Is inserted into the array.

 > **Note:** The <mark>sourceIndexPath</mark> is the Initial index path where the long press gesture begins. While the <mark>destinationIndexPath</mark> is the final index path the user drags the cell.

## Done 🙌✊🥳🎉👏

That's all. Build and Run the project, long-press a cell and reorder it.

## Resources

[**Download the source code**](https://github.com/samisays11/Drag-Drop-CollectionView-Cells-Demo). The project contains a collectionview that has two sections, each cell cell contains a letter from the alphabet. Drag to reorder the cells to make words.




<!-- 
The Collection View contains only one section

26 items will be created in the Collection View

Each cell will contain a letter from the alphabet. -->



<!-- 
selecting a cell with a long press enables the user to reorder the cells. All that's needed is to update the order of the item in the  data model. In this tutorial some cells containing alphabet letters will be displayed, these cells can be easily reordered. This tutorial is made with Xcode 10 and built for iOS 12.



How to leverage long gesture recognizer to implement a drag and reorder collectionview cells feature
Once the user starts long pressing, we want to find what cell they are long pressing on, and then we want to start the movement of that cell so when they drag around the cell moves with their fingers

    guard let indexPath = collectionView.indexPathForItem(at: gesture.location(in: collectionView)) else {return} the reason is because the user might be long pressing between cells or in the padded area so we might not necessarily have a location -->
