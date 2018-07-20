---
layout: post
title: "Remove background with OpenCV"
date:   2018-06-26
categories: opencv
---

Recently I need to work on a task to remove the background of certain images. I
have some experience with OpenCV in my university, so I decide to try it on this
task.

## [ruby-opencv](https://github.com/ruby-opencv/ruby-opencv)

My favorite language is Ruby, so I try with ruby first. `ruby-opencv` seems like
a good candidate: it's only stale for about a year, which is not bad because
opencv isn't upgraded very often.

However, after install opencv using brew, I realized that newest version is 3.x
while `ruby-opencv` only supports upto opencv `2.4.13`. There is a branch which
attempts to upgrade to opencv `3.2`, but I see that it is still too unstable.

It does not discourage me much, though. A little search shows that opencv version
`2.4.13` still has very good documentation, and I don't need very advance feature anyway.
I don't know that the nightmare waiting in front of me, though.

Everything goes smoothy at first, I got this piece of code running after the
first hour:

```ruby
original_image = CvMat.load(path, 1) # Read the file.
gray = original_image.BGR2GRAY
image, _ = gray.threshold(100, 255, CV_THRESH_BINARY_INV, true)
contours = image.find_contours

mask = image.create_mask
mask.draw_contours!(contours, 0, 255, 1)

dst_image = original_image.and(original_image, mask)
dst_image.save_image('result.png')
```

It results in an image slightly different from original image, with correct
grayscale and mask created.

(I do struggle a bit with `find_contours` method: the document says that I can
 pass in options such as `mode: :tree`, but in really, I must use `mode:
 CV_RETR_TREE` instead.)

For the rest of the day, I hopelessly fiddle with the code to make it work: I
cannot choose the max contour to get the whole object, `and` bitwise operation
work incorrectly compared to the document (hint: it does nothing).

Finally, I give up. I decide that it is much easier to use opencv directly
instead of relying on wrapper.

## Python opencv

With a few minutes I can convert the ruby code to python

```python
import cv2
import numpy as np

## Read
originalImage = cv2.imread("sample.png")
grayscaleImage = cv2.cvtColor(originalImage, cv2.COLOR_BGR2GRAY)

## Threshold
_, threshedImage = cv2.threshold(grayscaleImage, 127, 255, cv2.THRESH_BINARY_INV|cv2.THRESH_OTSU)

## Find the contours with biggest area
_, contours, _ = cv2.findContours(threshedImage, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
maxContour = max(contours, key=cv2.contourArea)

## Create mask
mask = np.zeros(originalImage.shape[:2], np.uint8)
cv2.drawContours(mask, [maxContour], -1, 255, -1)

## Copy the croped image to a white canvas
destinationImage = np.zeros(originalImage.shape[:3], np.uint8)
height = originalImage.shape[:2][1]
destinationImage[:, 0:height] = (255, 255, 255)

locations = np.where(mask != 0)
destinationImage[locations[0], locations[1]] = originalImage[locations[0], locations[1]]

## Save
cv2.imwrite("result.png", destinationImage)
```

The only part that different is the part about copying the croped image to the
white canvas. Because python opencv does not support `copyTo` like C++ lib, I
have to reimplement it with the help of [stackoverflow](https://stackoverflow.com/a/41573727).

And finally, it's done :D
