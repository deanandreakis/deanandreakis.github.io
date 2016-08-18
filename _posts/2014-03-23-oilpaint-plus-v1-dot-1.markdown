---
layout: post
title: "OilPaint+ v1.1"
date: 2014-03-23 19:58:05 -0500
comments: false
categories: apps
---

Version 1.1 of my iOS app [OilPaint+](https://itunes.apple.com/us/app/oilpaint+/id827491007?mt=8) hit the app store today. This is a bug fix release that addresses a problem with with crashing on iPhone 5C and iPhone 5 models. The app uses Brad Larson's excellent [GPUImage library](https://github.com/BradLarson/GPUImage) to provide an oil paint effect to any image. The crashes were occuring on these devices with large image sizes such as 2448x3264. The specific filter from GPUImage that is used is the GPUImageKuwaharaFilter which is described as being "extremely computationally expensive". I use [Crashlytics](https://www.crashlytics.com) to provide crash reports from the app and I have always been very happy with the service. It was pretty easy to see the issue from the crash reports I was receiving. The real lesson I learned was to write tests for your app that test the main functions. I ended up writing a test in Xcode using the XCTest framework that tested the use of the GPUImageKuwaharaFilter in my app. I was able to recreate the crash with this test and then test out fixes until I was confident that I had a solution. Using GPUImage for this app was pretty straightforward. As an example of how easy it is to use GPUImage, here is the code from OilPaint+ that uses it:

``` objective-c GPUImage Code in OilPaint+
- (void)processImage
{
    dispatch_async(dispatch_get_global_queue( DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
        
        UIImage* newImage = imageToSet; //the image to add the effect to
        
        // Do the task in the background
        GPUImagePicture *stillImageSource;
        UIImage *currentFilteredImage;
        
        stillImageSource = [[GPUImagePicture alloc] initWithImage:newImage];
        
        GPUImageKuwaharaFilter *oilPaintingTransformFilter = [[GPUImageKuwaharaFilter alloc] init];
        oilPaintingTransformFilter.radius = 8.0;
        
        [stillImageSource addTarget:oilPaintingTransformFilter];
        [stillImageSource processImage];
        
        currentFilteredImage = [oilPaintingTransformFilter imageFromCurrentlyProcessedOutput];
        
    });   
}
```

