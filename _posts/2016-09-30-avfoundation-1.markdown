---
layout: post
title: AVFoundation编程指南1-使用 Assets
date: 2016-10-10 21:47:24.000000000 +09:00
---
##创建assert对象
为了创建一个由URL标识的代表任何资源的assert对象，可以使用AVURLAssert，最简单的是从文件里创建一个assert对象：

```
NSURL *url = <#A URL that identifies an audiovisual asset such as a movie file#>;
AVURLAsset *anAsset = [[AVURLAsset alloc] initWithURL:url options:nil];
```
###Asset 初始化的可选操作
AVURLAsset初始化方法的第二个参数使用一个dictionary，这个dictionary里的唯一一个key是 AVURLAssetPreferPreciseDurationAndTimingKey，它的value是一个boolean类型（用NSValue包装的对象），这个值表示asset是否提供一个精确的duration。
获取asset精确的duration需要很多处理时间，使用一个预估的duration效率比较高并且对播放来说足够。因此：
· 如果你想要播放asset，初始化方法传nil就行了，而不是一个dictionry，或者传一个以AVURLAssetPreferPreciseDurationAndTimingKeydictionary为key，值为NO的一个dictionary。
· 如果你想把asset加到一个composition中，你需要一个精确的访问权限，这时你可以传一个dictionary，这个dictionary的一组键值对为

```
AVURLAssetPreferPreciseDurationAndTimingKey和YES。
NSURL *url = <#A URL that identifies an audiovisual asset such as a movie file#>;
NSDictionary *options = @{ AVURLAssetPreferPreciseDurationAndTimingKey : @YES };
AVURLAsset *anAssetToUseInAComposition = [[AVURLAsset alloc] initWithURL:url options:options];
```

###访问用户的asset
为了访问 iPod library 和相册里asset，你需要获取asset的URL。
· 为了访问ipod Library，需要创建MPMediaQuery对象来找到你想要对象，然后通过MPMediaItemPropertyAssetURL获取它的URL。要想获取更多关于Media Library，查看Multimedia Programming Guide.
· 为了访问相册，可以使用ALAssetsLibrary.
下面的例子用来获取相册里面的第一个视频:
ALAssetsLibrary *library = [[ALAssetsLibrary alloc] init];
    
    // Enumerate just the photos and videos group by using ALAssetsGroupSavedPhotos.
    [library enumerateGroupsWithTypes:ALAssetsGroupSavedPhotos
                           usingBlock:^(ALAssetsGroup *group, BOOL *stop) {
                               
                               // Within the group enumeration block, filter to enumerate just videos.
                               [group setAssetsFilter:[ALAssetsFilter allVideos]];
                               
                               // For this example, we're only interested in the first item.
                               [group enumerateAssetsAtIndexes:[NSIndexSet indexSetWithIndex:0]
                                                       options:0
                                                    usingBlock:^(ALAsset *alAsset, NSUInteger index, BOOL *innerStop) {
                                                        
                                                        // The end of the enumeration is signaled by asset == nil.
                                                        if (alAsset) {
                                                            ALAssetRepresentation *representation = [alAsset defaultRepresentation];
                                                            NSURL *url = [representation url];
                                                            AVAsset *avAsset = [AVURLAsset URLAssetWithURL:url options:nil];
                                                            // Do something interesting with the AV asset.
                                                        }
                                                    }];
                           }
                         failureBlock: ^(NSError *error) {
                             // Typically you should handle an error more gracefully than this.
                             NSLog(@"No groups");
                         }];
##准备使用Asset
初始化一个asset（或者track）并不是表示asset里面所有的信息都是马上可用的。它需要一些时间去计算，即使是durtation（比如没有摘要信息的mp3文件），你应该使用AVAsynchronousKeyValueLoading协议获取这些值，通过- loadValuesAsynchronouslyForKeys:completionHandler:在handler里面获取你要的值。
你可以用statusOfValueForKey:error:测试一个属性的value是否成功获取，当一个assert第一次被加载，大多数属性的值是AVKeyValueStatusUnknown状态，为了获取一个或多个属性的值，你要调用loadValuesAsynchronouslyForKeys:completionHandler:，在comletiton handler里面，你可以根据属性的状态做任何合适的处理。你要处理加载没有成功的情况，可能因为一些原因比如网络不可连接，又或者loading被取消。

```
NSURL *url = <#A URL that identifies an audiovisual asset such as a movie file#>;
AVURLAsset *anAsset = [[AVURLAsset alloc] initWithURL:url options:nil];
NSArray *keys = @[@"duration"];
 
[asset loadValuesAsynchronouslyForKeys:keys completionHandler:^() {
 
    NSError *error = nil;
    AVKeyValueStatus tracksStatus = [asset statusOfValueForKey:@"duration" error:&error];
    switch (tracksStatus) {
        case AVKeyValueStatusLoaded:
            [self updateUserInterfaceForDuration];
            break;
        case AVKeyValueStatusFailed:
            [self reportError:error forAsset:asset];
            break;
        case AVKeyValueStatusCancelled:
            // Do whatever is appropriate for cancelation.
            break;
   }
}];
```
如果你想要播放asset，你应该加载他的tracks属性。

##从video中获取静态图片
从asset中获取静态图片（比如说缩略图），你可以用AVAssetImageGenerator对象。可以用asset初始化一个AVAssetImageGenerator对象.即使asset在初始化的时候没有可见的track也能成功，所以你应该检测asset是否有track，使用tracksWithMediaCharacteristic:

```
AVAsset anAsset = <#Get an asset#>;
if ([[anAsset tracksWithMediaType:AVMediaTypeVideo] count] > 0) {
    AVAssetImageGenerator *imageGenerator =
        [AVAssetImageGenerator assetImageGeneratorWithAsset:anAsset];
    // Implementation continues...
}
```
你还可以设置imagegenerator的其他属性，比如，你可以指定生成图片的最大的分辨率，你可以生成指定时间的一张图片，或者一系列图片。你必须一直持有generator的引用，直到生成所有的图片。
###生成单个图片
你可以使用copyCGImageAtTime:actualTime:error:生成一张指定时间点的图片。AVFoundation不一定能精确的生成一张你所指定时间的图片，所以你可以在第二个参数传一个CMTime的指针，用来获取所生成图片的精确时间。

```
AVAsset *myAsset = <#An asset#>];
AVAssetImageGenerator *imageGenerator = [[AVAssetImageGenerator alloc] initWithAsset:myAsset];
 
Float64 durationSeconds = CMTimeGetSeconds([myAsset duration]);
CMTime midpoint = CMTimeMakeWithSeconds(durationSeconds/2.0, 600);
NSError *error;
CMTime actualTime;
 
CGImageRef halfWayImage = [imageGenerator copyCGImageAtTime:midpoint actualTime:&actualTime error:&error];
 
if (halfWayImage != NULL) {
 
    NSString *actualTimeString = (NSString *)CMTimeCopyDescription(NULL, actualTime);
    NSString *requestedTimeString = (NSString *)CMTimeCopyDescription(NULL, midpoint);
    NSLog(@"Got halfWayImage: Asked for %@, got %@", requestedTimeString, actualTimeString);
 
    // Do something interesting with the image.
    CGImageRelease(halfWayImage);
}
```
###生成一系列图片
为了生成一系列图片，你可以调用generateCGImagesAsynchronouslyForTimes:completionHandler:，第一个参数是一个包含NSValue类型的数组，数组里每一个对象都是CMTime结构体，表示你想要生成的图片在视频中的时间点，第二个参数是一个block，每生成一张图片都会回调这个block，这个block提供一个result的参数告诉你图片是否成功生成或者图片生成操作是否取消。

在你的block实现中，需要检查result，判断image是否成功生成，另外，确保你持有image generator直到生成图片的操作结束。

```
AVAsset *myAsset = <#An asset#>];
// Assume: @property (strong) AVAssetImageGenerator *imageGenerator;
self.imageGenerator = [AVAssetImageGenerator assetImageGeneratorWithAsset:myAsset];
 
Float64 durationSeconds = CMTimeGetSeconds([myAsset duration]);
CMTime firstThird = CMTimeMakeWithSeconds(durationSeconds/3.0, 600);
CMTime secondThird = CMTimeMakeWithSeconds(durationSeconds*2.0/3.0, 600);
CMTime end = CMTimeMakeWithSeconds(durationSeconds, 600);
NSArray *times = @[NSValue valueWithCMTime:kCMTimeZero],
                  [NSValue valueWithCMTime:firstThird], [NSValue valueWithCMTime:secondThird],
                  [NSValue valueWithCMTime:end]];
 
[imageGenerator generateCGImagesAsynchronouslyForTimes:times
                completionHandler:^(CMTime requestedTime, CGImageRef image, CMTime actualTime,
                                    AVAssetImageGeneratorResult result, NSError *error) {
 
                NSString *requestedTimeString = (NSString *)
                    CFBridgingRelease(CMTimeCopyDescription(NULL, requestedTime));
                NSString *actualTimeString = (NSString *)
                    CFBridgingRelease(CMTimeCopyDescription(NULL, actualTime));
                NSLog(@"Requested: %@; actual %@", requestedTimeString, actualTimeString);
 
                if (result == AVAssetImageGeneratorSucceeded) {
                    // Do something interesting with the image.
                }
 
                if (result == AVAssetImageGeneratorFailed) {
                    NSLog(@"Failed with error: %@", [error localizedDescription]);
                }
                if (result == AVAssetImageGeneratorCancelled) {
                    NSLog(@"Canceled");
                }
  }];
```
你也可以取消生成图片的操作，通过想generator发送cancelAllCGImageGeneration的消息。

##裁剪视频和对视频转码
你可以对视频进行转码、裁剪，通过使用AVAssetExportSession对象。这个流程如下图所示，
![][1]
[1]: https://developer.apple.com/library/prerelease/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Art/export_2x.png
一个export session是一个控制对象，可以异步的生成一个asset。可以用你需要生成的asset和presetName来初始化一个session，presetName指明你要生成的asset的属性。接下来你可以配置export session，比如可以指定输出的URL和文件类型，以及其他的设置，比如metadata等等。
你可以先检测设置的preset是否可用，通过使用exportPresetsCompatibleWithAsset:方法。

```
AVAsset *anAsset = <#Get an asset#>;
NSArray *compatiblePresets = [AVAssetExportSession exportPresetsCompatibleWithAsset:anAsset];
if ([compatiblePresets containsObject:AVAssetExportPresetLowQuality]) {
    AVAssetExportSession *exportSession = [[AVAssetExportSession alloc]
        initWithAsset:anAsset presetName:AVAssetExportPresetLowQuality];
    // Implementation continues.
}
```
你可以配置session的输出的url（这个url必须是文件url），AVAssetExportSession可以推断通过url的扩展名出输出文件的类型。当然，你可以直接设置文件类型，使用outputFileType。你还可以指定其他属性，比如time range，输出文件的长度等等，下面是列子：

```
exportSession.outputURL = <#A file URL#>;
    exportSession.outputFileType = AVFileTypeQuickTimeMovie;
 
    CMTime start = CMTimeMakeWithSeconds(1.0, 600);
    CMTime duration = CMTimeMakeWithSeconds(3.0, 600);
    CMTimeRange range = CMTimeRangeMake(start, duration);
    exportSession.timeRange = range;
```
生成一个新的asset，可以调用exportAsynchronouslyWithCompletionHandler:，当生成操作结束后会回调block，在这个block中你需要通过检查session的status来判断是否成功，如下：

```
  [exportSession exportAsynchronouslyWithCompletionHandler:^{
 
        switch ([exportSession status]) {
            case AVAssetExportSessionStatusFailed:
                NSLog(@"Export failed: %@", [[exportSession error] localizedDescription]);
                break;
            case AVAssetExportSessionStatusCancelled:
                NSLog(@"Export canceled");
                break;
            default:
                break;
        }
    }];
```
你可以取消这个生成操作，通过给session发送 cancelExport 消息。
如果导出的文件存在，或者导出的url在沙盒之外，这个导出操作会失败。还有两种情况也可能导致失败：
· 来了一个电话
· 你的程序在后台运行并且其他的应用开始播放。
这种情况下，你应该通知用户export失败，并且重新export。

#参考：
[https://developer.apple.com/library/prerelease/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/01_UsingAssets.html](https://developer.apple.com/library/prerelease/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/01_UsingAssets.html)

