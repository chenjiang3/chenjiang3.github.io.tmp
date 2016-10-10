---
layout: post
title: AVFoundation编程指南2-用AVPlayer播放视频
date: 2016-10-10 21:51:24.000000000 +09:00
---
控制assets的播放，你可以使用AVPlayer对象。在播放的过程中，你可以使用AVPlayerItem对象来管理asset的呈现，AVPlayerItemTrack来管理track。要显示视频，需要使用AVPlayerLayer。

#播放Assets
一个播放器就是控制asset播放的对象，比如开始和结束，seek到指定的时间。可以使用AVPlayer来播放单个asset，用AVQueuePlayer来播放多个连续的asset。
一个player向你提供播放的信息，如果需要，你通过player的状态同步显示到界面上。你也可以直接把player的输出显示笑傲指定的动画层（AVPlayerLayer或者AVSynchronizedLayer），想知道更多关于layer的信息，请查看[Core Animation Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514)

```
多个layer的情况：你可以创建多个AVPlayerLayer对象，但是只有最近创建的layer才会显示视频画面。
```
虽然是播放asset，但是不能直接把asset传给AVPlayer对象，你应该提供AVPlayerItem对象给AVPlayer。一个player item管理着和它相关的asset。一个player item包括player item tracks-（AVPlayerItemTrack对象，表示asset中的tracks）。他们之间的关系如下图：
![][1]
[1]:https://developer.apple.com/library/prerelease/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Art/avplayerLayer_2x.png
这表明你可以同时用不同的player播放同一个asset，如下图显示，两个不同的player播放同一个asset。
![][1]
[1]:https://developer.apple.com/library/prerelease/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Art/playerObjects_2x.png
你可以用一个存在asset直接初始化player，或者直接用URL初始化。和AVAsset一样，简单的初始化一个player并不表示可以马上进行播放，你需要观察它的status（通过kvo）来决定是否可以播放。

#处理不同类型的asset
配置asset的方式由需要播放的asset的类型决定的。概括的说，有两种方式：基于文件的asset，基于流式的（http live streaming format）
####加载基于文件的asset，有如下几步：
· 使用AVURLAsset创建一个asset。<p>
· 使用创建的asset来创建一个AVPlayerItem对象item<p>
· item和AVPlayer关联<p>
· 等待item的状态，知道可以播放。
####创建基于HTTP live stream的播放器。
用url初始化一个AVPlayerItem对象。（http live stream的情况下不能直接创建AVAsset对象）

```
NSURL *url = [NSURL URLWithString:@"<#Live stream URL#>];
// You may find a test stream at <http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8>.
self.playerItem = [AVPlayerItem playerItemWithURL:url];
[playerItem addObserver:self forKeyPath:@"status" options:0 context:&ItemStatusContext];
self.player = [AVPlayer playerWithPlayerItem:playerItem];
```
当你关联一个player item到player的时候，这个播放器开始准备播放。当它可以播放的时候，player item会创建AVAsset和AVAssetTrack对象，这些对象可以用来检查live stream的内容。
为了获取stream的时间，可以通过kvo的方式观察player item的duration的属性。当可以播放的时候，这个属性被设置为正确的值，这时就可以获取时间。

```
注意:只能在iOS4.3之后使用player item的duration属性。下面这种获取duration的方法适用于所有的iOS系统版本：当player item的状态变为AVPlayerItemStatusReadyToPlay时，duration可以通过下面代码获取
[[[[[playerItem tracks] objectAtIndex:0] assetTrack] asset] duration];
```
如果仅仅是想播放一个live stream，可以直接用下面的简短代码实现：

```
self.player = [AVPlayer playerWithURL:<#Live stream URL#>];
[player addObserver:self forKeyPath:@"status" options:0 context:&PlayerStatusContext];
```
正如assets和items一样，初始化一个player之后并不表明可以马上播放，你需要观察player的status属性，当status变为AVPlayerStatusReadyToPlay时表示可以播放了，你也需要观察curretItem属性来访问player item。

如果你不能确定你用的url是什么类型，可以用下面的方法检测：
1、尝试用url初始化AVURLAsset，然后load它的tracks key，如果tracks load 成功，表明你可以用这个asset创建player item。
2、如果第一步失败，直接用url创建AVPlayerItem，观察status属性，看是否有可播放的状态。

#播放一个item
如果想要播放，你可以想player发送play消息，如下代码：

```
- (IBAction)play:sender {
    [player play];
}
```
除了播放之外，还可以管理player的各种信息，比如rate和播放头，你也可以监控player的状态，这很有用，比如说你需要根据播放的状态来更新界面。

##改变播放的rate
可以改变播放的rate，代码如下：

```
aPlayer.rate = 0.5;
aPlayer.rate = 2.0;
```
rate=1.0表示正常的播放。0.0表示暂停。
player item支持逆向播放，当rate设置为负数的时候就是逆向播放.playeritem的 canPlayReverse 表示rate为-1.0，canPlaySlowReverse表示rate的范围是-0.0到-1.0，canPlayFastReverse表示rate小于-1.0f。

##seeking-重定位播放头
可以使用seekToTime：重定位播放头到指定的时间，如下代码：

```
CMTime fiveSecondsIn = CMTimeMake(5, 1);
[player seekToTime:fiveSecondsIn];
```
seekTime:不能精确定位，如果需要精确定位，可以使用seekToTime:toleranceBefore:toleranceAfter:，代码如下：

```
CMTime fiveSecondsIn = CMTimeMake(5, 1);
[player seekToTime:fiveSecondsIn toleranceBefore:kCMTimeZero toleranceAfter:kCMTimeZero];
```
当tolerance＝0的时候，framework需要进行大量解码工作，比较耗性能，所以，只有当你必须使用的时候才用这个方法，比如开发一个复杂的多媒体编辑应用，这需要精确的控制。

当播放结束后，播放头移动到playerItem的末尾，如果此时调用play方法是没有效果的，应该先把播放头移到player item起始位置。如果需要实现循环播放的功能，可以监听通知AVPlayerItemDidPlayToEndTimeNotification，当收到这个通知的时候，调用seekToTime：把播放头移动到起始位置，代码如下：

```
// Register with the notification center after creating the player item.
    [[NSNotificationCenter defaultCenter]
        addObserver:self
        selector:@selector(playerItemDidReachEnd:)
        name:AVPlayerItemDidPlayToEndTimeNotification
        object:<#The player item#>];
 
- (void)playerItemDidReachEnd:(NSNotification *)notification {
    [player seekToTime:kCMTimeZero];
}
```

##播放多个items
可以使用AVQueuePlayer播放多个items，AVQueuePlayer是AVPlayer的子类，可以用一个数组来初始化一个AVQueuePlayer对象。代码如下：

```
NSArray *items = <#An array of player items#>;
AVQueuePlayer *queuePlayer = [[AVQueuePlayer alloc] initWithItems:items];
```
和AVPlayer一样，直接调用play方法来播放，queue player顺序播放队列中的item，如果想要跳过一个item，播放下一个item，可以调用方法advanceToNextItem。

可以对队列进行插入和删除操作，调用方法insertItem:afterItem:, removeItem:, 和 removeAllItems。正常情况下当插入一个item之前，应该检查是否可以插入，通过使用canInsertItem:afterItem:方法，第二个参数传nil，代码如下：

```
AVPlayerItem *anItem = <#Get a player item#>;
if ([queuePlayer canInsertItem:anItem afterItem:nil]) {
    [queuePlayer insertItem:anItem afterItem:nil];
}
```

##监测播放状态
可以监测player和player item的状态，这个非常有用。比如：<br>
· 如果用户切换到其他应用程序，则需要把player的rate设为0.0 <br>
· 如果播放的是远程媒体，当收到更多的数据的时候，player的loadedTimeRange和seekableTimeRange属性将会不断改变。<br>
· 当player播放的是http live stream的时候，player的currentItem会不断改变。<br>
· 播放http live stream的时候，player item的tracks属性也不断改变。这会发生在player改变编码方式的时候。<br>
· 当播放失败的时候，player或者player item的status属性也会改变。<br>

可以使用kvo来监测上述改变。

```
注意：只能在主线程注册和取消kvo
```
###status改变后的处理方式
当player或者player item的状态status改变，系统会发送一个kvo的notification，如果一个对象由于一些原因不能播放，stauts会变成AVPlayerStatusFailed 或者 AVPlayerItemStatusFailed ，在这种情况下，这个对象的error属性会被附上一个error类型的对象，这个error对象描述了失败的原因。

AV Foundation不会指定这个notification是由哪个线程发出的，所以，如果你要更新UI，就必须确保更新的代码在主线程中调用，下面的代码表示收到status更新的处理方式：

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object
                        change:(NSDictionary *)change context:(void *)context {
 
    if (context == <#Player status context#>) {
        AVPlayer *thePlayer = (AVPlayer *)object;
        if ([thePlayer status] == AVPlayerStatusFailed) {
            NSError *error = [<#The AVPlayer object#> error];
            // Respond to error: for example, display an alert sheet.
            return;
        }
        // Deal with other status change if appropriate.
    }
    // Deal with other change notifications if appropriate.
    [super observeValueForKeyPath:keyPath ofObject:object
           change:change context:context];
    return;
}
```
###监听视频准备播放的状态
可以监听AVPlayerLayer的readyForDisplay属性，当layer有可显示的内容时，会发送一个notification。

###跟踪时间
可以使用addPeriodicTimeObserverForInterval:queue:usingBlock: 或者 addBoundaryTimeObserverForTimes:queue:usingBlock:来跟踪播放的进度，根据这个进度，你可以更新UI，比如播放了多少时间，还剩多少时间，或者其他的UI状态。<br>
· addPeriodicTimeObserverForInterval:queue:usingBlock:，这个方法传入一个CMTime结构的时间区间，每隔这个时间段的时候，block会回调一次，开始和结束播放的时候block也会回调一次。<br>
· addBoundaryTimeObserverForTimes:queue:usingBlock:,这个放传入一个CMTime结构的数组，当播放到数组里面的时间点的时候，block会回调。<br>
这两个方法都返回一个id类型的对象，这个对象必须一直被持有。可以使用removeTimeObserver:取消这个观察者。

对于这两个方法，AVFoundation不会保证每次时间点到了的时候都会回调block，如果前面回调的block没有执行完的时候，下一次就不会回调。所以，必须保证在block里面的逻辑不能太耗时。下面是使用的例子：

```
// Assume a property: @property (strong) id playerObserver;
 
Float64 durationSeconds = CMTimeGetSeconds([<#An asset#> duration]);
CMTime firstThird = CMTimeMakeWithSeconds(durationSeconds/3.0, 1);
CMTime secondThird = CMTimeMakeWithSeconds(durationSeconds*2.0/3.0, 1);
NSArray *times = @[[NSValue valueWithCMTime:firstThird], [NSValue valueWithCMTime:secondThird]];
 
self.playerObserver = [<#A player#> addBoundaryTimeObserverForTimes:times queue:NULL usingBlock:^{
 
    NSString *timeDescription = (NSString *)
        CFBridgingRelease(CMTimeCopyDescription(NULL, [self.player currentTime]));
    NSLog(@"Passed a boundary at %@", timeDescription);
}];
```

###播放结束
可以向通知中心注册 AVPlayerItemDidPlayToEndTimeNotification 通知，当播放结束的时候可以收到一个结束的通知。代码：

```
[[NSNotificationCenter defaultCenter] addObserver:<#The observer, typically self#>
                                         selector:@selector(<#The selector name#>)
                                             name:AVPlayerItemDidPlayToEndTimeNotification
                                           object:<#A player item#>];
```

#完整的例子：用AVPlayerLayer播放一个video
下面的代码向你展示如何使用AVPLayer播放一个video文件，步骤如下：<br>
1、用AVPlayerLayer配置一个view	<br>
2、创建一个AVPlayer	<br>
3、用video文件创建一个AVPlayerItem对象，并且用kvo观察他的status	<br>
4、	当收到item的状态变成可播放的时候，播放按钮启用<br>
5、	播放，结束之后把播放头设置到起始位置<br>
##player view
为了播放一个视频，需要个view，这个view的layer是AVPlayerLayer对象。可以创建一个子类实现这个需求。代码：

```
#import <UIKit/UIKit.h>
#import <AVFoundation/AVFoundation.h>
 
@interface PlayerView : UIView
@property (nonatomic) AVPlayer *player;
@end
 
@implementation PlayerView
+ (Class)layerClass {
    return [AVPlayerLayer class];
}
- (AVPlayer*)player {
    return [(AVPlayerLayer *)[self layer] player];
}
- (void)setPlayer:(AVPlayer *)player {
    [(AVPlayerLayer *)[self layer] setPlayer:player];
}
@end
```
##View controller
同时你需要一个view controller，代码如下：

```
@class PlayerView;
@interface PlayerViewController : UIViewController
 
@property (nonatomic) AVPlayer *player;
@property (nonatomic) AVPlayerItem *playerItem;
@property (nonatomic, weak) IBOutlet PlayerView *playerView;
@property (nonatomic, weak) IBOutlet UIButton *playButton;
- (IBAction)loadAssetFromFile:sender;
- (IBAction)play:sender;
- (void)syncUI;
@end
```
syncUI方法的作用是根据player的status来更新播放按钮，实现如下：

```
- (void)syncUI {
    if ((self.player.currentItem != nil) &&
        ([self.player.currentItem status] == AVPlayerItemStatusReadyToPlay)) {
        self.playButton.enabled = YES;
    }
    else {
        self.playButton.enabled = NO;
    }
}
```
在viewdidload里面调用syncUI，确保刚开始显示的页面的时候按钮是不可用的。代码如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    [self syncUI];
}
```
其他的属性和方法在接下来的文字说明。

##创建一个asset
通过URL创建一个AVURLAsset对象。代码如下：

```
- (IBAction)loadAssetFromFile:sender {
 
    NSURL *fileURL = [[NSBundle mainBundle]
        URLForResource:<#@"VideoFileName"#> withExtension:<#@"extension"#>];
 
    AVURLAsset *asset = [AVURLAsset URLAssetWithURL:fileURL options:nil];
    NSString *tracksKey = @"tracks";
 
    [asset loadValuesAsynchronouslyForKeys:@[tracksKey] completionHandler:
     ^{
         // The completion block goes here.
     }];
}
```
在完成的block里面，用asset创建一个AVPlayerItem对象和一个AVPlayer对象，并且把player设为player view的属性。和创建一个asset一样，简单的创建一个player item并不意味着可以马上使用，可以用kvo观察player item 的status来判断是否可以开始播放，这个kvo的设置应该在player item和player关联之前设置。代码如下：

```
// Define this constant for the key-value observation context.
static const NSString *ItemStatusContext;
 
// Completion handler block.
         dispatch_async(dispatch_get_main_queue(),
            ^{
                NSError *error;
                AVKeyValueStatus status = [asset statusOfValueForKey:tracksKey error:&error];
 
                if (status == AVKeyValueStatusLoaded) {
                    self.playerItem = [AVPlayerItem playerItemWithAsset:asset];
                     // ensure that this is done before the playerItem is associated with the player
                    [self.playerItem addObserver:self forKeyPath:@"status"
                                options:NSKeyValueObservingOptionInitial context:&ItemStatusContext];
                    [[NSNotificationCenter defaultCenter] addObserver:self
                                                              selector:@selector(playerItemDidReachEnd:)
                                                                  name:AVPlayerItemDidPlayToEndTimeNotification
                                                                object:self.playerItem];
                    self.player = [AVPlayer playerWithPlayerItem:self.playerItem];
                    [self.playerView setPlayer:self.player];
                }
                else {
                    // You should deal with the error appropriately.
                    NSLog(@"The asset's tracks were not loaded:\n%@", [error localizedDescription]);
                }
            });
```

###player item的status改变时的处理
当player item的status改变时，view controller会收到一个通知消息，AVFoundation并不指定这个消息是由哪个线程发出的。如果你要更新ui，必须确保更新ui的代码在主线程中。下面的代码显示更新ui的逻辑：

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object
                        change:(NSDictionary *)change context:(void *)context {
 
    if (context == &ItemStatusContext) {
        dispatch_async(dispatch_get_main_queue(),
                       ^{
                           [self syncUI];
                       });
        return;
    }
    [super observeValueForKeyPath:keyPath ofObject:object
           change:change context:context];
    return;
}
```

###播放
播放很简单，直接向player发送play消息即可。代码如下：

```
- (IBAction)play:sender {
    [player play];
}
```
这样的情况只能播放一次，当播放结束的时候，再调用play方法是没有效果的，如果你要重新播放，需要向通知中心注册AVPlayerItemDidPlayToEndTimeNotification消息，当收到播放结束的消息的时候，调用seekToTime：把播放头移动到起始位置，这样再调用play的时候就可以重新播放了。代码如下：

```
// Register with the notification center after creating the player item.
    [[NSNotificationCenter defaultCenter]
        addObserver:self
        selector:@selector(playerItemDidReachEnd:)
        name:AVPlayerItemDidPlayToEndTimeNotification
        object:[self.player currentItem]];
 
- (void)playerItemDidReachEnd:(NSNotification *)notification {
    [self.player seekToTime:kCMTimeZero];
}
```

######参考文献：
[https://developer.apple.com/library/prerelease/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/02_Playback.html#//apple_ref/doc/uid/TP40010188-CH3-SW8](https://developer.apple.com/library/prerelease/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/02_Playback.html#//apple_ref/doc/uid/TP40010188-CH3-SW8)


