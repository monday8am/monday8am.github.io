---
layout: post
title:  "Canon EOSLib. Update"
date:   2014-06-07 13:46:00 +0100
categories: blog
---

**CanonEOSLib** was developed for a very specific project and to interact only with the Canon EOS1100D. After that, we tried to update the code and communicate with more cameras but due to the differences between them, some functionalities were affected and they didn't work as expected.

The more problematic functionality is related with the **EVF** (electronic viewfinder) capture. Using the libraries provided by the camera manufacturer, the extension executes commands that are passed from the **Adobe AIR runtime**. The commands are executed in a separate thread to avoid overloading the main thread and to keep a good user experience when the camera is working.

To understand a bit better how the **EVF** captures work, check the follow graphic:

![Screenshot]({{ "/assets/img/graphic_ane_getEVF.png" | absolute_url }})


In order to download a live preview image from the camera, process it and save in a global native object we can use this method:

{% highlight c++ %}
_model->setEvfBitmap(targetBitmap);
{% endhighlight %}

If a new image is ready to capture, the native extension dispatch an event to notify runtime Adobe AIR.

{% highlight c++ %}
FREDispatchStatusEventAsync(_context, 
                            (uint8_t*) "downloadEVF",
                            (const uint8_t*) event.c_str());
{% endhighlight %}

and finally, the bytes are copied from the native bitmap:

{% highlight c++ %}
FREObject getEVF(FREContext context, 
                 void* funcData, 
                 uint32_t argc, 
                 FREObject argv[])
{
    BOOL r = 0;
    FREObject result;
    FREBitmapData2 bitmap_descriptor;
    FREObject freBitmap = argv[0];

    Bitmap* targetBitmap = _model->getEvfBitmap();

    if( targetBitmap == NULL ) {
        FRENewObjectFromBool( 0, &result ); 
        return result;
    }

    // Get AS3 bitmap content.
    FREAcquireBitmapData2(freBitmap, &bitmap_descriptor);
    uint32_t* input = bitmap_descriptor.bits32;

    if (bitmap_descriptor.isInvertedY == 1) 
        targetBitmap->RotateFlip( RotateNoneFlipY); 

    int pixelSize = 4;
    Rect rect(0, 0, bitmap_descriptor.width, 
                bitmap_descriptor.height);
    BitmapData* pBmData = new BitmapData;
    targetBitmap->LockBits (&rect, 
                            ImageLockModeRead, 
                            PixelFormat32bppARGB,
                            pBmData);

    for (int y = 0; y < targetBitmap->GetHeight(); y++) {
        //get bytes rows from the original image
        byte* oRow = (byte*) pBmData->Scan0 + 
                            (y * pBmData->Stride );	

        //get byte rows  from the new image	
        byte* nRow = (byte*) bitmap_descriptor.bits32 + 
                    (y * bitmap_descriptor.lineStride32 * 4); 

        for (int x=0;x<targetBitmap->GetWidth(); x++)
        {
            //set the new image's pixel to the grayscale
            nRow[x*pixelSize]=oRow[x*pixelSize]; //B
            nRow[x*pixelSize1]=oRow[x*pixelSize+1]; //G
            nRow[x*pixelSize+2]=oRow[x*pixelSize+2]; //R
        }
    }

    targetBitmap->UnlockBits(pBmData);
    _model->setEvfBitmap(NULL);
        
    // Free resources
    delete pBmData;
    delete targetBitmap;

    FREInvalidateBitmapDataRect(freBitmap, 0, 0, 
                                bitmap_descriptor.width,
                                bitmap_descriptor.height);
    FREReleaseBitmapData(freBitmap);	
	FRENewObjectFromBool(1, &result); 
	return result;
}
{% endhighlight %}

To transfer the native bitmap to the bitmap passed by the runtime, the ANE tries to do this in a "byte per byte' copy operation. It's is very important to keep the same dimensions (width and height) in both sides.

Currently the project is unsupported.


### Related links:
[Github CanonEOS CPP][github_code]

[Canon EosLib description post][canoneos_lib]

[github_code]: https://github.com/monday8am/CanonEOS_CPP
[canoneos_lib]: http://www.monday8am.com/2012/05/29/canoneos_lib_extension/