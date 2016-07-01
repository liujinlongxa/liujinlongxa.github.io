---
layout: post
title: "【翻译】MVVM Tutorial with ReactiveCocoa Part 2"
categories: "iOS开发"
---

原文地址：[MVVM Tutorial with ReactiveCocoa: Part 2/2](https://www.raywenderlich.com/74131/mvvm-tutorial-with-reactivecocoa-part-2)

----

MVVM作为一种UI设计模式，正在成为一种非常流行的MVC模式的替代方案。

在MVVM系列教程的第一部分，你已经了解了ReactiveCocoa是如何将ViewModels和他们各自的的View绑定起来的。

![1](https://cdn5.raywenderlich.com/wp-content/uploads/2014/06/MVVMReactiveCocoa-700x121.png)

下面是你创建的应用，一个搜索Flickr的APP。

![2](https://cdn3.raywenderlich.com/wp-content/uploads/2014/06/FinishedApp-671x500.png)

在 MVVM系列教程的第二部分以及最后一部分，你讲学会如果从应用程序的ViewModel中控制ViewController的导航。

截止目前，你所开发的应用可以通过一个简单的字符串搜索Flickr，如果你需要当前的项目，可以从[这里](http://www.raywenderlich.com/wp-content/uploads/2014/06/FlickrSearchPart1Project1.zip)下载

使用ReactiveCocoa构建的模型层服务提供搜索结果，对应的ViewModel能够简单地打印出响应结果。

现在，是时候完成跳转到搜索结果界面的任务了。

## 实现ViewModel导航

搜索Flickr成功并返回正确的结果后，应用程序应该跳转到一个新的ViewController并显示这些搜索结果。

当前应该程序只有单一的一个ViewModel，即RWTFlickrSearchViewModel类。为了达到预期的效果，你应该再创建一个ViewModel来描述返回的搜索结果视图。

在ViewModel文件夹中一个名为RWTSearchResultsViewModel类，他继承自NSObject。代码如下：

```objectivec
@import Foundation;
#import "RWTViewModelServices.h"
#import "RWTFlickrSearchResults.h"

@interface RWTSearchResultsViewModel : NSObject

- (instancetype)initWithSearchResults:(RWTFlickrSearchResults *)results services:(id<RWTViewModelServices>)services;

@property (strong, nonatomic) NSString *title;
@property (strong, nonatomic) NSArray *searchResults;

@end
```

上面的代码中添加了一些属性用于描述搜索结果视图，并且声明了一个包含一个RWTFlickrSearchResults参数（返回模型层的服务）的构造方法。

打开RWTSearchResultsViewModel.m实现以下代码：

```objectivec
- (instancetype)initWithSearchResults:(RWTFlickrSearchResults *)results services:(id<RWTViewModelServices>)services {
  if (self = [super init]) {
    _title = results.searchString;
    _searchResults = results.photos;
  }
  return self;
}
```

这样RWTSearchResultsViewModel就完成了。

回忆第一部分的内容，ViewModel应该在对应的View创建之前就已经创建好了，下面就是要完成各个ViewModel对应了View了。

打开RWTSearchResultsViewController.h文件，导入对应的ViewModel头文件，并且添加一个初始化方法，代码如下：

```objectivec
#import "RWTSearchResultsViewModel.h"

@interface RWTSearchResultsViewController : UIViewController

- (instancetype)initWithViewModel:(RWTSearchResultsViewModel *)viewModel;

@end
```

打开RWTSearchResultsViewController.m文件，并且在顶部的类扩展中添加一个私有属性，如下：

```objectivec
@property (strong, nonatomic) RWTSearchResultsViewModel *viewModel;
```

进一步完成初始化方法的实现，如下：

```objectivec
- (instancetype)initWithViewModel:(RWTSearchResultsViewModel *)viewModel {
  if (self = [super init]) {
    _viewModel = viewModel;
  }
  return self;
}
```

这一步中，你将关注视图导航是如何工作的。

你的应用程序现在有两个ViewModel，但是这里你将面临一个难题！我该怎么在视图控制器切换的同时也把对应的ViewModel进行切换。

ViewModel不能直接持有View的引用，所以，有什么巧妙的方法可以实现这个呢？

答案已经在RWTViewModelServices协议中展现了。它当前被用来获得一个模型层的引用，你将用这个协议来实现ViewModel的切换。

打开RWTViewModelServices.h并且添加如下方法到协议中去。

```objectivec
- (void)pushViewModel:(id)viewModel;
```

从概念上讲，ViewModel层驱动着整个App，这一层的逻辑决定着界面上会显示什么，以及界面之间切换的方式和时机。

这个方法允许ViewModel也像UINavigationController那样可以从一个ViewModel以Push的方法切换到另一个ViewModel。

在实现这个协议之前，你应该先是这个方法在ViewModel层可以工作。

打开RWTFlickrSearchViewModel.m文件并导入新添加的ViewModel。

```objectivec
#import "RWTSearchResultsViewModel.h"
```

在同一文件更新executeSearchSignal方法的实现，代码如下：

```objectivec
- (RACSignal *)executeSearchSignal {
  return [[[self.services getFlickrSearchService]
    flickrSearchSignal:self.searchText]
    doNext:^(id result) {
      RWTSearchResultsViewModel *resultsViewModel =
        [[RWTSearchResultsViewModel alloc] initWithSearchResults:result services:self.services];
      [self.services pushViewModel:resultsViewModel];
    }];
}
```

上面的代码在搜索信号执行过程中添加了一个doNext操作，在doNext的block中创建了一个新的ViewModel来展示搜索结果，然后通过ViewModel的services的push操作切换到这个新的ViewModel上。

现在，该更新代码，实现这个协议并达到切换ViewModel同时切换ViewController的效果。为了实现这个效果，我们需要在代码中引用导航控制器。

打开RWTViewModelServicesImpl.h并且添加如下初始化方法：

```objectivec
- (instancetype)initWithNavigationController:(UINavigationController *)navigationController;
```

打开RWTViewModelServicesImpl.m并且添加下面的头文件：

```objectivec
#import "RWTSearchResultsViewController.h"
```

然后添加一个私有属性：

```objectivec
@property (weak, nonatomic) UINavigationController *navigationController;
```

然后在同一个文件完成方法的实现：

```objectivec
- (instancetype)initWithNavigationController:(UINavigationController *)navigationController {
  if (self = [super init]) {
    _searchService = [RWTFlickrSearchImpl new];
    _navigationController = navigationController;
  }
  return self;
}
```

上述代码用来把传过来的导航控制器引用记录下来。

最后，添加一个新的方法：

```objectivec
- (void)pushViewModel:(id)viewModel {
  id viewController;

  if ([viewModel isKindOfClass:RWTSearchResultsViewModel.class]) {
    viewController = [[RWTSearchResultsViewController alloc] initWithViewModel:viewModel];
  } else {
    NSLog(@"an unknown ViewModel was pushed!");
  }

  [self.navigationController pushViewController:viewController animated:YES];
}
```

上述代码会根据ViewModel的类型来判断该显示那个视图。

一般来说，View和ViewModel是一一对应的关系，但是你一定可以举出反例。

最后一步，打开RWTAppDelegate.m，将createInitialViewController方法中创建RWTViewModelServicesImpl的一行改为如下代码：

```objectivec
self.viewModelServices = [[RWTViewModelServicesImpl alloc] initWithNavigationController:self.navigationController];
```

编译并运行程序，输入一个搜索关键字然后点击Go，可以观察到应用程序将切换到一个新的View/ViewModel。

![3](https://cdn4.raywenderlich.com/wp-content/uploads/2014/06/BlankView-281x500.png)

界面是空白的，别着急，待会你将修复这个问题。

现在，可以庆祝一下，你已经实现了一个含有多个ViewModel并且可以通过ViewModel来切换界面的应用程序了。

> 小贴士：John Gossman是一名微软WPF团队的工程师，他创造了MVVM模式。他说测试MVVM的一种方法就是你的应用程序应该脱离UI也可以运行。

> 你的应用程序通过了这个测试。如果你不确信，可以通过单元测试来执行一个搜索或从一个ViewModel切换到另一个ViewModel。

现在，你已经有了一个非常纯净的解决方案，下面要开始绑定UI了。

## 呈现搜索结果列表

搜索结果视图控制器RWTSearchResultsViewController在nib中定义了一个UITableview，下一步就是讲ViewModel中的内容展现在这个列表中。

打开RWTSearchResultsViewController.m，添加一个类扩展，让他实现UITableViewDataSource协议，如下：

```objectivec
@interface RWTSearchResultsViewController () <UITableViewDataSource>
```

在同一文件中重写viewDidLoad方法，代码如下：

```objectivec
- (void)viewDidLoad {
  [super viewDidLoad];

  [self.searchResultsTable registerClass:UITableViewCell.class
                  forCellReuseIdentifier:@"cell"];
  self.searchResultsTable.dataSource = self;

  [self bindViewModel];
}
```

这些操作会在tableview初始化时执行一次，并绑定ViewModel。请忽略上面的Cell Identifier的硬编码，这个待会会删掉。

然后在同一文件中，添加bindViewModel方法。

```objectivec
- (void)bindViewModel {
  self.title = self.viewModel.title;
}
```

当前并没有添加太多代码。ViewModel有两个属性，一个是标题，另一个是将要展示在列表中的搜索结果的数组。

因此，你该如何把这些搜索结果绑定到Tableview上呢？

ReactiveCocoa可以绑定简单的属性到UIKit上，但是无法处理复杂的Tableview的数据绑定。

不用焦急，还有其他方法。可以使用传统的代理方法实现。

在同一文件中，添加如下两个dataSource的代理方法：

```objectivec
- (NSInteger)tableView:(UITableView *)tableView
 numberOfRowsInSection:(NSInteger)section {
  return self.viewModel.searchResults.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView
         cellForRowAtIndexPath:(NSIndexPath *)indexPath {
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell"];
  cell.textLabel.text = [self.viewModel.searchResults[indexPath.row] title];
  return cell;
}
```

第一个方法用来告诉Tableview搜索结果的个数，第二个方法根据ViewModel返回一个相应的Cell。这就够了，不是吗？

编译并运行程序，就可以看到数据了

![4](https://cdn2.raywenderlich.com/wp-content/uploads/2014/06/PopulatedTable-281x500.png)

## 更好的Tableview数据绑定方法

这种绑定Tableview数据的方法的缺点就是很快就会使视图控制器的代码变得很大。手动绑定数据的方法看起来也不是很优雅。

这个问题困扰着我，因此我着手解决这个问题。

从理论上讲，ViewModel的搜索结果数据中的每一项自身也是一个与Cell一一对应的ViewModel。在我的一篇最近的[博客](http://www.scottlogic.com/blog/2014/05/11/reactivecocoa-tableview-binding.html)中，我创建了一个通用的绑定辅助类，叫做CETableViewBindingHelper，这个类允许你使用每一个子ViewModel创建一个视图。这个类可以用来辅助实现datasource协议。

这个CETableViewBindingHelper类的构造方法如下：

```objectivec
+ (instancetype) bindingHelperForTableView:(UITableView *)tableView
                              sourceSignal:(RACSignal *)source
                          selectionCommand:(RACCommand *)selection
                              templateCell:(UINib *)templateCellNib;
```

你仅仅只需要创建一个辅助类的引用，就可以用来绑定一个View的数组。这个构造方法的参数如下：

1. 需要加载ViewModel数组的Tableview
2. 一个source的信号用来监控数据的变化
3. 一个在选中一个Cell后要执行的命令
4. cell视图的nib

nib文件定义的Cell必须实现CEReactiveView协议。

项目中已经包含了一个用于展示搜索结果的Cell视图。打开RWTSearchResultsTableViewCell.h导入协议头文件：

```objectivec
#import "CEReactiveView.h"
```

实现这个协议：

```objectivec
@interface RWTSearchResultsTableViewCell : UITableViewCell <CEReactiveView>
```

下一步就是实现协议方法，打开RWTSearchResultsTableViewCell.m，导入如下内容：

```objectivec
#import <SDWebImage/UIImageView+WebCache.h>
#import "RWTFlickrPhoto.h"
```

然后添加如下方法：

```objectivec
- (void)bindViewModel:(id)viewModel {
  RWTFlickrPhoto *photo = viewModel;
  self.titleLabel.text = photo.title;

  self.imageThumbnailView.contentMode = UIViewContentModeScaleToFill;

  [self.imageThumbnailView setImageWithURL:photo.url];
}
```

现在RWTSearchResultsViewModel的searchResults属性包含了一个RWTFlickrPhoto的引用的数组。

你刚刚添加的bindViewModel方法中用到了SDWebImage框架，这是一个非常有用的用来在后台加载图片的第三方框架。

setImageWithURL:是SDWebImage添加的一个UIImageView的分类方法。

最后一步是用绑定辅助类去展示列表。

打开RWTSearchResultsViewController.m导入辅助类头文件：

```objectivec
#import "CETableViewBindingHelper.h"
```

在同一文件中删除UITableDataSource协议以及实现的两个协议方法。

下一步，在类扩展中添加一个如下的属性：

```objectivec
@property (strong, nonatomic) CETableViewBindingHelper *bindingHelper;
```

在同一文件中，删除你之前添加的ViewDidload方法中的内容，然后添加如下代码到ViewDidLoad中：

```objectivec
- (void)viewDidLoad {
  [super viewDidLoad];
  [self bindViewModel];
}
```

最后，在bindViewModel方法的最后添加如下代码：

```objectivec
UINib *nib = [UINib nibWithNibName:@"RWTSearchResultsTableViewCell" bundle:nil];

self.bindingHelper =
  [CETableViewBindingHelper bindingHelperForTableView:self.searchResultsTable
                                         sourceSignal:RACObserve(self.viewModel, searchResults)
                                     selectionCommand:nil
                                         templateCell:nib];
```

上述代码从nib文件中创建了一个UINIb类型的引用，并且创建了一个绑定辅助类的对象。通过观察ViewModel中的searchResults属性来创建一个sourceSignal。

编译并运行，结果如下：

![5](http://www.raywenderlich.com/wp-content/uploads/2014/06/UsingTheBindingHelper.png)

这就是一种更加又要的绑定数组到Tableview的方法。

## 一些UI的个性化

到目前为止，这篇教程始终重点叫的都是如何根据MVVM模式构建你的应用程序。我想你已经忍不住想添加一些个性化的东西了。

自从iOS7发布后，已经过了1年了，[运动设计](http://www.beyondkinetic.com/motion-ui-design-principles)获得了更大的知名度，许多设计人员现在倾向于微妙的动画和流体行为。

这一步中，你将在照片上添加一个微妙的视差滚动效果，棒极了。

打开RWTSearchResultsTableViewCell.h添加如下方法：

```objectivec
- (void) setParallax:(CGFloat)value;
```

列表视图的每个Cell将使用这个方法来为每个Cell做视觉偏移。

打开 RWTSearchResultsTableViewCell.m 文件添加如下代码：

```objectivec
- (void)setParallax:(CGFloat)value {
  self.imageThumbnailView.transform = CGAffineTransformMakeTranslation(0, value);
}
```

打开RWTSearchResultsViewController.m导入如下头文件：

```objectivec
#import "RWTSearchResultsTableViewCell.h"
```

在同一个文件的类扩展中实现UITableViewDelegate的协议，如下：

```objectivec
@interface RWTSearchResultsViewController () <UITableViewDataSource, UITableViewDelegate>
```

你刚刚添加了一个绑定辅助类，并且把他设置为列表视图的代理，是他可以响应列表中行的选中动作。

在bindViewModel方法中设置绑定辅助类的代理：

```objectivec
self.bindingHelper.delegate = self;
```

在同一文件中添加scrollViewDidScroll方法的实现，如下：

```objectivec
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
  NSArray *cells = [self.searchResultsTable visibleCells];
  for (RWTSearchResultsTableViewCell *cell in cells) {
    CGFloat value = -40 + (cell.frame.origin.y - self.searchResultsTable.contentOffset.y) / 5;
    [cell setParallax:value];
  }
}
```

每当列表视图滚动时，就会调用这个方法。在这个方法中，遍历所有可见的Cell，然后计算出一个偏移量，然后给每一个Cell设置一个视觉偏移。实际的偏移量将根据每一个Cell在列表视图可见部分的位置而定。

编译并运行程序，就可以看到一个视差效果。

![5](http://www.raywenderlich.com/wp-content/uploads/2014/06/ParallaxAnimation.gif)

回到之前Views和ViewModels的工作。

## 根据评论和点赞数查询

界面上应该在每一张图片的底部右侧显示评论数和点赞数。但是，当前从nib创建的文件仅仅只是显示了一个123的假数据。

在你使用真实数据之前，你需要在模型层添加这个功能。

在Model文件夹中添加一个新的继承自NSObject的类RWTFlickrPhotoMetadata，打开RWTFlickrPhotoMetadata.h添加如下代码：

```objectivec
@property (nonatomic) NSUInteger favorites;
@property (nonatomic) NSUInteger comments;
```

打开 RWTFlickrPhotoMetadata.m实现description方法：

```objectivec
- (NSString *)description {
  return [NSString stringWithFormat:@"metadata: comments=%lU, faves=%lU",
          self.comments, self.favorites];
}
```

这个方法用于测试新添加的获取第一张照片的元数据的接口返回的数据是否正确。结果将被打印出来。

编译并运行，搜索一些照片，当结果显示时，你将看到如下log将被打印出来。

```objectivec
2014-06-04 07:27:26.813 RWTFlickrSearch[76828:70b] metadata: comments=120, faves=434
```

## 取得元数据赋给可见的Cell

你可以将当前的代码扩展成获取所有搜索结果的元数据。

但是，如果结果有100张图片，你就要调用200次API接口，每张图片两次。许多API都有一个调用频率限制，这个操作可能会导致API Key被锁定，最少也是暂时不能用。

其实，你只需要获取到当前在列表视图中显示的几张图片的元数据。因此，你该怎样实现这样的行为呢？你应该已经猜到了，你需要一个可以意识到他是否显示的一个ViewModel。

当前的RWTSearchResultsViewModel对外暴漏了一个展示在视图上的RWTFlickrPhoto数组的接口，这些模型层的对象会被保留给View。为了能够添加一个可见性的概念，你将把这些在ViewModels里的模型对象封装起来。

在ViewModel文件夹中添加一个NSObject的子类RWTSearchResultsItemViewModel，打开头文件，添加如下内容：

```objectivec
@import Foundation;
#import "RWTFlickrPhoto.h"
#import "RWTViewModelServices.h"

@interface RWTSearchResultsItemViewModel : NSObject

- (instancetype) initWithPhoto:(RWTFlickrPhoto *)photo services:(id<RWTViewModelServices>)services;

@property (nonatomic) BOOL isVisible;
@property (strong, nonatomic) NSString *title;
@property (strong, nonatomic) NSURL *url;
@property (strong, nonatomic) NSNumber *favorites;
@property (strong, nonatomic) NSNumber *comments;

@end
```

如你所见，这个ViewModel封装了RWTFlickrPhoto模型对象。

这个ViewModel的属性混合了如下内容：

* 对外的模型属性（title，url）
* 会动态更新的图像元数据模型（favorites，comments）
* isVisible用来指示ViewModel是否可见或不可见

打开RWTSearchResultsItemViewModel.m导入如下头文件：

```objectivec
#import <ReactiveCocoa/ReactiveCocoa.h>
#import <ReactiveCocoa/RACEXTScope.h>
#import "RWTFlickrPhotoMetadata.h"
```

然后在类扩展中添加如下私有属性：

```objectivec
@interface RWTSearchResultsItemViewModel ()

@property (weak, nonatomic) id<RWTViewModelServices> services;
@property (strong, nonatomic) RWTFlickrPhoto *photo;

@end
```

在同一文件中实现如下方法：

```objectivec
- (instancetype)initWithPhoto:(RWTFlickrPhoto *)photo services:(id<RWTViewModelServices>)services {
  self = [super init];
  if (self) {
    _title = photo.title;
    _url = photo.url;
    _services = services;
    _photo = photo;

    [self initialize];
  }
  return  self;
}
```

然后添加一个initialize方法，注意了，这是关键的地方：

```objectivec
- (void)initialize {
  RACSignal *fetchMetadata =
    [RACObserve(self, isVisible)
     filter:^BOOL(NSNumber *visible) {
       return [visible boolValue];
     }];

  @weakify(self)
  [fetchMetadata subscribeNext:^(id x) {
    @strongify(self)
    [[[self.services getFlickrSearchService] flickrImageMetadata:self.photo.identifier]
     subscribeNext:^(RWTFlickrPhotoMetadata *x) {
       self.favorites = @(x.favorites);
       self.comments = @(x.comments);
     }];
  }];
}
```

方法的第一部分创建了一个用于监听isVisible属性的信号fetchMetadate，并且根据isVisible的值进行了过滤。结果就是只有当isVisible为true时这个信号才会发出Next Value。

接下来一部分就是订阅这个信号以便开始请求flickrImageMetadata方法。当这个内嵌的信号触发Next时，将更新赞数和评论数。

总的来说，当isVisible为true时，将触发Flickr请求，然后在未来的某个点更新comments和favorites属性。

为了使这个新的ViewModel能够使用，打开RWTSearchResultsViewModel.m导入如下头文件：

```objectivec
#import <LinqToObjectiveC/NSArray+LinqExtensions.h>
#import "RWTSearchResultsItemViewModel.h"
```

在初始化方法中，删除当前的代码，然后设置_searchResults如下：

```objectivec
_searchResults =
  [results.photos linq_select:^id(RWTFlickrPhoto *photo) {
    return [[RWTSearchResultsItemViewModel alloc]
              initWithPhoto:photo services:services];
  }];
```

这段代码将每一个模型对象封装成了一个ViewModel。

最后一步就是根据视图设置isVisible属性，使得这些新的属性能够生效。

打开RWTSearchResultsTableViewCell.m添加如下头文件：

```objectivec
#import "RWTSearchResultsItemViewModel.h"
```

在同一文件中，改变bindViewModel方法的第一行代码，使用新添加的ViewModel。

```objectivec
RWTSearchResultsItemViewModel *photo = viewModel;
```

在同一方法中添加如下代码：

```objectivec
[RACObserve(photo, favorites) subscribeNext:^(NSNumber *x) {
  self.favouritesLabel.text = [x stringValue];
  self.favouritesIcon.hidden = (x == nil);
}];

[RACObserve(photo, comments) subscribeNext:^(NSNumber *x) {
  self.commentsLabel.text = [x stringValue];
  self.commentsIcon.hidden = (x == nil);
}];

photo.isVisible = YES;
```

这样当comments和favorites属性发生变化时，对应的lable以及image也会更新。

最后一步，设置ViewModel的isVisible为YES。

## 后记

终于翻译完了，这是我第一次尝试翻译英文技术文章，前后断断续续用了大概两周的时间，真的很累，说实话，最后完全是为了翻译而去翻译的，而不是为了学习技术而去翻译的。不过这个过程中还是学到了不少东西。以后我还会坚持找一些经典的文章拿来翻译，不过不会翻译这么长的文章了，太累了。

水平有限，很多地方翻译的不是很通顺，有些句子不知道怎么翻好，干脆就没有翻译，个中错误，以后有空会慢慢纠正。（2016-5-19）
