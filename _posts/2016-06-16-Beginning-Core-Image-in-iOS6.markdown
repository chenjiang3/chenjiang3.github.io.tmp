---
layout: post
title: core image
date: 2016-06-16 15:32:24.000000000 +09:00
---

core image是一个强大的图像处理框架，使用core image可以方便的实现滤镜，比如可以修改图像的形变，色彩和曝光。它使用GPU（或者是CPU）来处理图像数据，所以特别快，可以实时处理视频的每一帧。

core image的多个滤镜可以合在一起，同一时间处理视频帧或者图像。当多个滤镜合在一起使用的时候效率更高，因为合在一起的时候，只需要创建一个合成的滤镜来处理图像，如果分开的话，每一个滤镜都要创建。

每一个滤镜都自己的参数，这些参数可以在代码中获取。也可以查询系统可以提供的各种滤镜。iOS平台上可使用的滤镜比Mac上的要少，是Mac上的一个子集。

在这篇文章中，我会手把手的教你使用core image。会教你使用各种滤镜，你会发现对图片实现一个很cool的效果是很简单的。

#core image概述
开始之前，先简单介绍一下core image framework里面重要的几个类。<br>
· CIContext：core image的所有处理都是由CIContext来完成的，这有点类似于Core Graphics和openGL的context。<br>
· CIImage: 这个类持有图片的数据，可以用UIImage来创建，也可以使用图片文件或者像素数据来创建。<br>
· CIFliter：这个类表示一个滤镜，它有许多属性，在内部由一个dictionary来维护。滤镜的种类有很多，比如是图片形变的滤镜，可以改变图片色彩的滤镜，可以裁剪图片的滤镜，等等<br>

接下来在你的工程中会使用到这些类。

#开始
打开Xcode，创建一个新的single view application工程，工程名为CoreImageFun，选择Device为iPhone。

创建好工程的第一件事就是把Core Image framework加入到工程中，在Mac平台，coreimage framework是QuartzCore framework的一部分，在iOS平台上它是独立的framework。怎么给工程添加framework，这里就不细说了，不会的请查看相关文档。

接下来，下载项目中会用到的[资源](https://raw.githubusercontent.com/chenjiang3/blog-resources/master/resource/CIResources.zip),资源下下来之后加入到你的工程中。

接下来，在viewdidload里面新建一个UIImageView，代码如下

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.view.backgroundColor = [UIColor whiteColor];
    
    _imageView = [[UIImageView alloc] init];
    [self.view addSubview:_imageView];
    _imageView.bounds = CGRectMake(0, 0, self.view.width, 300);
    _imageView.centerX = self.view.width / 2.0f;
    _imageView.top = 0;
    //.......
}
```
注：项目用到的定位

#基本的滤镜效果
接下来实现一个很简单的滤镜效果，显示在imageview上。<br>
每次实现滤镜的效果，基本上都会包含一下四个步骤：
#####1、创建CIImage对象：CIImage有很多初始化方法，比如， imageWithURL:, imageWithData:, imageWithCVPixelBuffer:, 以及 imageWithBitmapData:bytesPerRow:size:format:colorSpace:，大多数情况下，使用imageWithURL: 就可以了。<br>
#####2、创建CIContext对象：一个CIContext对象可以使用CPU，也可以使用GPU。CIContext可以重用，所以，不需要重复的创建CIContext对象。<br>
#####3、创建CIFilter对象：当创建一个CIFliter对象的时候，可以根据你需要实现的滤镜效果给它设置一系列的属性。<br>
#####4、获取filter的输出对象：CIFilter会根据你设置的属性输出一个符合你预期效果的一个图片，这个图片是CIImage类型的对象，可以用CIContext转换成UIImage.<br>

请看代码是如何实现的。在ViewController.m文件的viewDidLoad:方法里面添加以下代码：

```
// 1
NSString *filePath =
  [[NSBundle mainBundle] pathForResource:@"image" ofType:@"png"];
NSURL *fileNameAndPath = [NSURL fileURLWithPath:filePath];
 
// 2
CIImage *beginImage =
  [CIImage imageWithContentsOfURL:fileNameAndPath];
 
// 3
CIFilter *filter = [CIFilter filterWithName:@"CISepiaTone"
                              keysAndValues: kCIInputImageKey, beginImage,
                    @"inputIntensity", @0.8, nil];
CIImage *outputImage = [filter outputImage];
 
// 4
UIImage *newImage = [UIImage imageWithCIImage:outputImage];
self.imageView.image = newImage;
```
上面代码的意思是：<br>
1、前两行代码创建一个NSURL对象，用来表示image的路径。<br>
2、接下来用imageWithContentsOfURL方法创建一个CIImage对象。<br>
3、创建一个CIFilter对象，这个对象的构造方法里面传了两个参数，第一个参数是滤镜的名字，第二个参数是一个dictionary，表示滤镜的一些参数。

CISepiaTone只需要两个参数，KCIInputImageKey和@”inputIntensity”，KCIInputImageKey表示一个图片。inputIntensity是一个float类型的值，用nsnumber包装，它的范围是0-1，这里设置的是0.8。大多数滤镜都有默认值，如果没有手动设置的话就用默认值。除了CIImage，它没有默认值。获取使用滤镜之后的图片很简单，只需要获取outputImage属性就行了。
<br>
4、一旦你获取了output CIImage，就可以把它转换成一个UIImage。在iOS6下可以使用UIImage的方法 +imageWithCIImage: 来完成这个步骤，这个方法使用一个CIImage对象来创建一个UIImage。一旦你创建了一个UIImage对象，就可以把它显示在image view 上了。<br>

编译运行，你会看到一个经过滤镜处理后的图片，恭喜你，现在你已经学会使用CIImage和CIFilter了！！！！，效果如下：<br>
![][1]
[1]:http://cdn3.raywenderlich.com/wp-content/uploads/2012/09/HelloCoreImage.png

#使用Context
在继续教程之前，你需要知道一个小小的优化。

之前提到，CIContext是用来处理CIFliter的，但是在上面的例子中却没有用到CIContext。这是因为上面的例子直接使用了UIImage的imageWithCIImage方法，在imageWithCIImage内部已经把所有的工作都做了，它在内部创建了一个CIContext对象，然后用这个对象获取了output Image。

这样做有一个缺点-每次都会创建一个CIContext对象，其实CIContext对象是可以重复使用的。如果使用一个slide控件更新filter的值，如果用上面的代码的话，每次都会创建一个CIContext对象，这样的话效率非常低。

接下来对上面的代码进行改进，删除上面的代码，用下面的代码替换。

```
CIImage *beginImage =
  [CIImage imageWithContentsOfURL:fileNameAndPath];
 
// 1
CIContext *context = [CIContext contextWithOptions:nil];
 
CIFilter *filter = [CIFilter filterWithName:@"CISepiaTone"
                              keysAndValues: kCIInputImageKey, beginImage,
                    @"inputIntensity", @0.8, nil];
CIImage *outputImage = [filter outputImage];
 
// 2
CGImageRef cgimg =
  [context createCGImage:outputImage fromRect:[outputImage extent]];
 
// 3
UIImage *newImage = [UIImage imageWithCGImage:cgimg];
self.imageView.image = newImage;
 
// 4
CGImageRelease(cgimg);
```
下面解释下上面的代码：

1、这里创建了一个CIContext对象，它的构造方法里面传了一个dictionary，这个dictionary可以执行颜色以及是否使用cpu或者gpu，在这里，我们使用默认的属性就行了，所以传一个nil即可。

2、使用context对象来创建CGImage，调用createCGImage:fromRect:即可，这样就创建了一个CGImageRef。

3、接下来使用UIImage +imageWithCGImage把上一步创建的CGImage转换为UIImage。

4、最后，释放CGImageRef，CGImage是C API的，需要手动管理内存，即使是在ARC模式下。

编译运行，确保结果和之前的一样。

在这里例子中，单独创建一个CIContext对象和之前的方法差别不是很大，但是，接下来的例子里，你会发现差别会很大。

#改变filter的值
接下来我们添加一个slide控件，用这个控件实时改变filter的值。

界面如下：
![][1]
[1]:http://cdn3.raywenderlich.com/wp-content/uploads/2012/09/AddingSlider.png

创建的代码如下：

```
1、创建slider控件
_amountSlider = [[UISlider alloc] init];
    [self.view addSubview:_amountSlider];
    _amountSlider.bounds = CGRectMake(0, 0, self.view.width - 40, 44);
    _amountSlider.top = _imageView.bottom + 20.0f;
    _amountSlider.centerX = self.view.width / 2.0f;
    [_amountSlider addTarget:self action:@selector(amountSliderValueChanged:) forControlEvents:UIControlEventValueChanged];
2、回调方法
- (void)amountSliderValueChanged:(UISlider *)slider {
    NSLog(@"amountSliderValueChanged...");
    float slideValue = slider.value;
    [filter setValue:@(slideValue) forKey:@"inputIntensity"];
    CIImage *outputImage = [filter outputImage];
    CGImageRef cgimg = [context createCGImage:outputImage fromRect:[outputImage extent]];
    UIImage *newImage = [UIImage imageWithCGImage:cgimg];
    _imageView.image = newImage;
    CGImageRelease(cgimg);
}
```
每一次改变slider的值，你需要重新对图片进行滤镜的效果。但是，你不需要每次都做全部的事情，这样会效率很低下。你只需要改变一点点东西，所以一些需要改为全局变量。

最重要的事情就是重用CIContext对象，如果每次都去创建，效率太低了。另外就是把cifilter和初始的ciimage对象放到全局变量中，这些对象是可以重用的。

所以需要把这三个变量当作viewcontroller的成员变量，代码如下：

```
@implementation ViewController {
    CIContext *context;
    CIFilter *filter;
    CIImage *beginImage;
}
```

viewdidload里面的初始化方法也要相应的改变。

```
beginImage = [CIImage imageWithContentsOfURL:fileNameAndPath];
context = [CIContext contextWithOptions:nil];
 
filter = [CIFilter filterWithName:@"CISepiaTone" 
  keysAndValues:kCIInputImageKey, beginImage, @"inputIntensity", 
  @0.8, nil];
```
现在需要实现changevalue的方法，在这个方法里需要改变CIFilter的dictionary里的@”inputIntensity”对应的值，一旦改变了这个值，接下来就需要执行以下三步:

1、获取CIFilter的output image。

2、把ciimage转为CGImageRef。

3、把CGImageRef转为UIImage，并且显示在image view上。

所以在amountSliderValueChanged里的代码如下：

```
- (void)amountSliderValueChanged:(UISlider *)slider {
    NSLog(@"amountSliderValueChanged...");
    float slideValue = slider.value;
    [filter setValue:@(slideValue) forKey:@"inputIntensity"];
    CIImage *outputImage = [filter outputImage];
    CGImageRef cgimg = [context createCGImage:outputImage fromRect:[outputImage extent]];
    UIImage *newImage = [UIImage imageWithCGImage:cgimg];
    _imageView.image = newImage;
    CGImageRelease(cgimg);
}
```
这段代码从slider获取float value，slide的value从0-1，刚好和CIFilter里的值的范围一样，这样很方便，可以直接拿来用。

编译运行，你移动slide，你会发现不同的效果，效果如下：

![][1]
[1]:http://cdn4.raywenderlich.com/wp-content/uploads/2012/09/SliderFilter.png

#从相册里获取照片
现在可以动态的改变filter的value，这看起来很有意思。但是仅仅是对这个固定的图片进行滤镜很没劲，下面使用UIImagePickerController获取相册的图片，并惊醒滤镜处理。

创建一个按钮，用来打开相册。界面如下：
![][1]
[1]:http://cdn3.raywenderlich.com/wp-content/uploads/2012/09/AddingButton.png

实现这个按钮的点击方法loadPhoto，代码如下：

```
- (IBAction)loadPhoto:(id)sender {
    UIImagePickerController *pickerC = 
      [[UIImagePickerController alloc] init];
    pickerC.delegate = self;
    [self presentViewController:pickerC animated:YES completion:nil];
}
```
第一行实例化一个UIImagePickerController，并把self设为它的delegate。

这里会有一个警告，你需要实现UIImagePickerControllerDelegate 和UINaviationControllerDelegate协议。代码如下：

```
@interface ViewController () <UIImagePickerControllerDelegate, UINavigationBarDelegate>
@end

- (void)imagePickerController:(UIImagePickerController *)picker 
  didFinishPickingMediaWithInfo:(NSDictionary *)info {
    [self dismissViewControllerAnimated:YES completion:nil];
    NSLog(@"%@", info);
}
 
- (void)imagePickerControllerDidCancel:
  (UIImagePickerController *)picker {
    [self dismissViewControllerAnimated:YES completion:nil];
}
```
在这两个方法中，都是dimiss UIPickerController。

第一个方法还没有完全实现，下面是完整的实现：

```
- (void)imagePickerController:(UIImagePickerController *)picker
  didFinishPickingMediaWithInfo:(NSDictionary *)info {
    [self dismissViewControllerAnimated:YES completion:nil];
    UIImage *gotImage =
      [info objectForKey:UIImagePickerControllerOriginalImage];
    beginImage = [CIImage imageWithCGImage:gotImage.CGImage];
    [filter setValue:beginImage forKey:kCIInputImageKey];
    [self amountSliderValueChanged:self.amountSlider];
}
```
完整的例子在[这里](https://raw.githubusercontent.com/chenjiang3/blog-resources/master/resource/CoreImageFun.zip)

参考文献：[http://www.raywenderlich.com/22167/beginning-core-image-in-ios-6](http://www.raywenderlich.com/22167/beginning-core-image-in-ios-6)





























