# MKNetworkKit
####MKNetworkKit 的中文说明版本
####所有文件均来自原作者,原框架地址:https://github.com/MugunthKumar/MKNetworkKit

-----------------------------------------------------------------------------
###以下中文说明文档也来自于作者,只用于开发参考

####假设有一个网络框架，它能自动为你缓存 respones,能在你离线时自动记忆你的操作，你觉得怎样？当你离线时，你可以收藏某个 tweet 页或者标记某个 feed 为已读，当你再次上线时，网络框架会自动执行你的这些操作，这一切都不需要你额外编写代码。请看我对于MKNetworkKit 框架的介绍。
###什么是MKNetworkKit?
####MKNetworkKit是一个 O-C 编写的网络框架，支持块，ARC 且用法简单。MKNetworkKit 集 ASIHTTPRequest 和 AFNetworking 两个框架于一体。在集成二者的优秀特性之外，还增加了一堆新的功能。尤其是，相比起其它框架，它能让你更轻松地编写代码。它让你彻底远离那些恶心的网络代码。

###特点

* 超轻量级框架
     整个框架只有 2 个类和一些类别方法。因此，它的使用极其简单。
     在整个程序中只有一个全局队列
     高度依赖互联网连接的 app 应该优先考虑网络线程的并发数。不幸的是，没有任何网络框架在这方面做得够好。
    因此，一旦你在程序中没有控制好网络线程的并发数，就极易导致出错。

假设你要上传一堆图片到服务器上。绝大多数移动网络（3G）不会允许你对同一个IP 地址的 HTTP 并发连接数超过 2 个。换句话说，在设备上，你不能从 3G 网络中获得 2 个以上的 HTTP 并发连接。对于 Edge 则更糟，大多数情况不能超过1 个。相比较家用宽带网络（Wifi），则这个限制要宽得多（6 个）。但是，你不可能总是使用 wifi，你必须也考虑到有限网络（窄带）的连通性。更多的时候，iDevice设备几乎都能连接到 3G 网络，因此，你同时只能上传 2 张图片。但是，真正的问题不是缓慢的上传速度，而是另一种情况。在你打开一个 view 试图加载缩略图（不同的view）时，上传线程被运行到后台。如果你没有控制好上传队列中的线程数，你的缩略图会加载超时。这是不正常的。正确的方式是优化缩略图加载线程，或者让线程等待直到上传完成再加载缩略图。这需要你在整个程序中只拥有一个queue 队列。MKNetworkKit 在它的每个实例中使用单例来保证这一点。并不是说MKNetworkKit 是单例的，而是说它的共享队列是单例的。

* 正确显示网络状态指示

    许多第 3 方框架都通过一个“网络连接数增加/减少”的方法回调来显示网络状态，MKNetworkKit则由于使用了单
    例的共享队列，能自动显示网络状态。在共享队列中有一个线程通过 KVO 方式会随时观察 operationCount 属
    性。因此对于开发者，一般情况下根本不需要操心网络状态的显示。

~~~~objc
if (object == _sharedNetworkQueue && [keyPath isEqualToString:@"operationCount"]) {
     [UIApplication sharedApplication].networkActivityIndicatorVisible 
                                = ([_sharedNetworkQueue.operations count] < 0);


     }
~~~~

* 自动改变队列大小
    如前所述，绝大部分移动网络不允许 2 个以上的并发连接，因此你的队列大小在3G 网络下应当设置为 2。
    MKNetworkKit 会自动为你处理好这个。当网络出于3G/EDGE/GPRS 时，它会将并发数调整到 2。当网络处
    于 Wifi 网络时，则自动调整到 6。当你通过 3G 网络中从远程服务器加载缩略图时，这种调整能带来极大
    的好处。

* 自动缓存
    MKNetworkKit 能自动缓存你所有的 GET 请求。当你再次发起同样的请求时，MKNetworkKit 随即就能调用
    response缓存（如果可用的话）传递给 handler 进行处理。当然，它同时也向服务器发出请求。一旦获得服
    务器数据，handler 被再次要求处理新获取的数据。也就是说，你不用手动缓存。你只需要使用：
~~~~objc
 [[MKNetworkEngine sharedEngine] useCache];
~~~~
    当然，你可以覆盖这个方法（子类化），定制你的缓存路径和缓存占用的内存开销。

* 冻结网络操作
    MKNetworkKit 能够“冻结”网络操作。在一个网络操作被“冻结”的情况下，一旦网络连断开，它们将自动序列
    化并在设备再次连线时自动被提交一次。类似 twitter 客户端的“drafts”。当你提交一篇 tweet 时，如果
    网络被标记为“可冻结”，MKNetworkKit 会自动执行冻结并储存这些请求。因此会在将来推迟发送这篇tweet。
    整个过程不需要你写一行代码。这个特性你可以用于其他操作，诸如收藏一篇 tweet 或者从 Goolge reader
    客户端共享一个帖子，加一个链接到Instapaper 中，等等。

* 类似的请求只执行一个操作
    当你加载缩略图（针对 twitter stream）时，你最终得为每个实际的图片创建一个新的请求。实际上你所
    进行的多个请求都是同一个URL。MKNetworkKit 对于队列中的每个 GET 请求都只会执行一次。它还不能
    到缓存 POST 请求。

* 图片缓存
    MKNetworkKit 内置了缩略图缓存。只要覆盖几个方法，就可以设置内存中最大能缓存的图片数量，以及缓
    存要保存到目录。当然，你也可以不覆盖这些方法。

* 性能
    即速度。MKNetworkKit 缓存是内置的，就如 NSCache,当发现有内存警告，缓存到内存中的数据将被写入
    缓存目录。
 
* 完全支持 ARC
    一般你只会在新项目中使用新的网络框架。MKNetworkKit并不意味着要放弃已有的框架（当然你也可以放弃，
    这会是个乏味的工作）。对于新的项目，你总是想使用 ARC。当你看到本文的时候，很可能 MKNetworkKit
    会是仅有的完全支持 ARC 的网络框架。ARC 通常比非 ARC 代码更快。
 
### 用法
    Ok，我就不“自卖自夸”了。让我们立即了解如果使用这个框架。

###添加MKNetworkKit
    1.将 MKNetworkKit 目录拖到项目中
    2.添加下列框架： CFNetwork.Framework,SystemConfiguration.framework,
                    Security.framework and ImageIO.Framework.
    3.将 MKNetworkKit.h 头文件包含到 PCH 文件中
    4.对于 iOS，删除 NSAlert+MKNetworkKitAdditions.h
    5.对于 Mac，删除 UIAlertView+MKNetworkKitAdditions.h
        总共只需要 5 个核心文件，真是一个强大的网络开发包

###MKNetworkKit 的类

      MKNetworkOperation
      MKNetworkEngine
      一些工具类 (Apple 的 Reachability) 以及类别

    我喜欢简单。苹果已经写了最基本最核心的网络代码。第 3 方框架需要的是提供一个优雅的网络队列最多再
    加上缓存。我认为第3方框架不应该超过 10 个类（无论它是网络的还是 UIKit 还是别的什么）。超过这个
    数就太臃肿了。Three20 就是一个例子。现在 ShareKit 又是这样。尽管它们是优秀的，但仍然是庞大和
    臃肿的。ASIHttpRequest or AFNetworking 比 RESTKit 更轻，JSONKit比TouchJSON (或者任何
    TouchCode 库)更轻。这只是我自己的看法，但当一个第三方库的代码超过程序源代码1/3，我就不会使用它。

    框架臃肿带来的问题是很难理解它的内部工作机制，以及很难根据自己的需求定制它（当你需要时）。我曾经
    写过的一些框架（例如MKStoreKit ，用于应用程序内购的 ）总是易于使用，我认为MKNetworkKit 也应
    该是这样。对于 MKNetworkKit ，你所需要了解的就是暴露在两个类MKNetworkOperation 和
    MKNetworkEngine 中的方法。MKNetworkOperation 就好比ASIHttpRequest类。它是一个NSOperation
    子类，封装了你的 request 和 response 类。对于每个网络操作，你需要创建一个MKNetworkOperation。
    
    MKNetworkEngine 是一个伪单例类，管理程序中的网络队列。它是伪单例的，也就是说，对于简单请求，
    你可以直接用MKNetworkEngine 中的方法。要进行深度的定制，你应该进行子类化。每个 MKNetworkEngine
    子类有它自己的Reachability 对象，用于通知它来自服务器的reachability 通知。对于不同的 REST
    服务器，你可以考虑创建单独的 MKNetworkEngine子类。

    它是伪单例，它的子类的每个请求都共用唯一的一个队列。你可以在应用程序委托中retain 这个
    MKNetworkEngine ，就像CoreData 的 managedObjectContext 类一样。在使用MKNetworkKit
    时，创建一个 MKNetworkEngine 子类将你的网络请求进行逻辑上的分组。例如，将所有关于 Yahoo 
    的方法放在一个类，所有 Facebook 有关的方法放进另一个类。来看 3 个实际使用的例子。

###例1:创建一个  “YahooEngine” 从 Yahoo 财经服务器抓取货币汇率。
*  步骤 1:创建YahooEngine 类继承于MKNetworkEngine。
MKNetworkEngine 使用主机名和指定的头（如果有的话）进行初始化。头信息可以是nil。如果你是在自己的 REST 服务器上，你可以考虑加一个客户端 app 的版本或者其他信息（比如客户端的标识）。
~~~~objc
    NSMutableDictionary *headerFields = [NSMutableDictionary dictionary]; 
    [headerFields setValue:@"iOS" forKey:@"x-client-identifier"];
    self.engine = [[YahooEngine alloc] initWithHostName:@"download.finance.yahoo.com" customHeaderFields:headerFields];
~~~~

    * 注意，yahoo 并不识别你在头中发送x-client-identifier 给它，这个示例仅仅是演示这个特性而由于使用了 ARC 代码，
    作为开发者你需要拥有（强引用）Engine对象。

    * 一旦你创建了一个 MKNetworkEngine子类, Reachability 即自动实现。当你的服务器由于某些情况挂了，主机名不可访问，
    你的请求会自动被冻结。关于“冻结”，请参考后面的“冻结操作”小节。

*  步骤 2:设计Engine 类 (关注分离)
现在，开始编写 Yahoo Engine 中的方法，以抓取汇率。这些方法将在ViewController 中被调用。良好的设计体验是确保不要将 engine 类中的 URL/HTTPHeaders 暴露给调用者。你的视图不应该知道URL 或者相关的参数。也就是，只需要向 engine 方法传递货币种类和货币单位就可以了。方法的返回值可能是 double，即汇率，以及获取汇率的时间。由于是异步操作，你应当在块中返回这些值。例如：
~~~~objc 
-(MKNetworkOperation*) currencyRateFor:(NSString*) sourceCurrency                   
            inCurrency:(NSString*) targetCurrency    
        onCompletion:(CurrencyResponseBlock) completion
        onError:(ErrorBlock) error;
~~~~

在父类 MKNetworkEngine 中，定义了3 个块类型：
~~~~objc
typedef void (^ProgressBlock)(double progress);
typedef void (^ResponseBlock)(MKNetworkOperation* operation);
typedef void (^ErrorBlock)(NSError* error);
~~~~
在 YahooEngine中，我们使用了一个新的块类型：CurrencyResponseBlock，用以返回汇率。其定义如下：
~~~~objc
typedef void (^CurrencyResponseBlock)(double rate);
~~~~

    在其他正式的 app 中，你应该定义自己的块类似于CurrencyResponseBlock ，用以向 ViewController 返回数据。

*  步骤 3:处理数据
处理数据，包括将从服务器抓来的数据（例如 JSON/XML/plists）进行数据类型转换。这应当在 Engine 中完成。注意，不要在控制器中完成。你的 Engine 应当将数据以适当的模型对象或模型对象的数组返回。在engine 中转换 JSON/XML 为模型——注意，适当保持关注分离，view controller 不应当知道任何用于访问 JSON 节点的 key。这种思想主导了engine 的设计。许多网络框架并不强制要求你服从关注分离，我们这样做，是因为我们为你考虑到了。

*  步骤 4:实现方法
现在，我们来讨论方法实现细节。要从 Yahoo 获得汇率信息，最简单的是发起一个 GET 请求。下列宏用一对指定的货币格式化 URL 字串：We will now discuss the implementationdetails of the method that calculates your currency exchange.Getting currency information from Yahoo,is as simple as making a GET request.I wrote a macro to format this URL for a given currency pair.
~~~~objc
#define YAHOO_URL(__C1__, __C2__)  [NSString stringWithFormat:
                @"d/quotes.csv?e=.csv&amp;f=sl1d1t1&amp;s=%@%@=X", __C1__, __C2__]
~~~~

按如下顺序编写 engine类方法：
    1.根据参数准备 URL
    2.创建一个 MKNetworkOperation 对象
    3.设置方法参数
    4.设置 operation 的 completion 块和 error 块（在 completation 块中处理 response 并转换为模型）
    5.可选地，添加一个 progress 块（或者在 view controller 中做这个）
    6.如果 operation 是下载，设置下载流（通常是文件）。这步也是可选的
    7.当 operation 完成，处理结果并调用方法块，并将数据返回给调用者。

示例代码如下：
~~~~objc
MKNetworkOperation *op = [self operationWithPath:YAHOO_URL(sourceCurrency, targetCurrency)
params:nil  httpMethod:@"GET"];

[op onCompletion:^(MKNetworkOperation*completedOperation)
     {
        DLog(@"%@", [completedOperation responseString]);
 //do your processing here
         completionBlock(5.0f);
     }onError:^(NSError* error) {
         errorBlock(error);
     }];
    [self enqueueOperation:op];
    return op; 
~~~~

上述代码格式化 URL 并创建了 MKNetworkOperation。设置完 completion 和 error 块之后，将 operation 加入到队列（通过父类的 enqueueOperation 方法），然后返回一个 operation 的引用。因此，如果你在 viewDidAppear 中调用这个方法，则在 viewWillDisappear 方法中取消operation。取消 operation 将释放 operation 以便执行 queue 中用于其他view 的 operation（牢记，在移动网络中只有2 个 operation 能被同时进行，当 operation 不再需要时取消它们能提升 app 的性能和速度）。

在 viewcontroller 中也可以添加一个 progress 块用以刷新UI。例如：

~~~~objc
    [self.uploadOperation onUploadProgressChanged:^(double progress) {   
        DLog(@"%.2f", progress*100.0);              
        self.uploadProgessBar.progress = progress;    
     }];
~~~~
MKNetworkEngine 也有一个只用 URL 创建 operation 的有用方法。因此第1行代码也可以写成：
~~~~objc 
MKNetworkOperation *op = [self operationWithPath:YAHOO_URL(sourceCurrency, targetCurrency)];
~~~~

    注意，请求的 URL将自动添加上主机名（在 engine 实例化时指定的）。

像这样的实用方法 MKNetworkEngine还有许多，你可以查看头文件。

###例2:上传图片到服务器 (例如 TwitPic)。

现在让我们看一个上传图片到服务器的例子。要上传图片，显然要 operation 能编码 multi-part 表单数据。MKNetworkKit 使用类似 ASIHttpRequest 的方式。你可以非常简单地通过MKNetworkOperation 的 addFile:forKey:方法将一个文件作为请求中的 multi-part 表单数据提交。MKNetworkOperation 也有一个方法，可以将图片以 NSData 的方式提交。即 addData:forKey: 方法，它可以将图片以NSData 的方法上传到服务器。 (例如直接从相机中捕获的图片)。

###例3:下载文件到本地目录 (缓存)

使用MKNetworkKit 从服务器下载文件并保存到 iPhone 的本地目录非常简单。只需要设置 MKNetworkOperation的outputStream。
~~~~objc 
    [operation setDownloadStream:[NSOutputStream outputStreamToFileAtPath:
        @"/Users/mugunth/Desktop/DownloadedFile.pdf" append:YES]];
~~~~

    你可以设置多个outputStream 到一个 operation，将同一文件保存到几个地方（例如其中一个是你的缓存
    目录，另一个用做你的工作目录）。

###例4:缓存图片的缩略图

对于下载图片，你可能需要提供一个绝对 URL 地址而不是一个路径。MKNetworkEngine 的operationWithURLString:params:httpMethod: 方法根据绝对 URL地址来创建网络线程。

MKNetworkEngine 相当聪明。它会将同一个 URL 的多次 GET 请求合并成一个，当 operation 完成时它会通知所有的块。这显著提升了抓取图片 URL 以渲染缩略图的速度.

子类化 MKNetworkEngine然后覆盖图片的缓存目录及缓存的大小。如果你不想定制这二者，你可以直接调用MKNetworkEngine中的方法来下载图片。这是我极力推荐的。

*  缓存operation
    MKNetworkKit 默认会缓存所有请求。你所需要的仅仅是在你自己的 engine 中打开它。当执行一个 GET 
    请求时，如果上次的 response 已缓存，相应的 completion 块将用缓存的response 进行调用（瞬间）。
    要想知道 response 是否缓存，可以调用 isCachedResponse 方法，如下所示：
~~~~objc 
[op onCompletion:^(MKNetworkOperation *completedOperation) {
          if([completedOperation isCachedResponse]) {
              DLog(@"Data from cache");
          }else {
              DLog(@"Data from server");
          }
            DLog(@"%@", [completedOperation responseString]);
      }
onError:^(NSError* error) {
            errorBlock(error);
}];
~~~~

*  冻结operation
    MKNetworkKit 的一个最有趣的特性是它内置的冻结 operation 特性。你只需要设置 operation 的 freeesable 
    属性就可以。几乎什么也不用做！
~~~~objc
[op setFreezable:YES];
~~~~

    *  冻结是指 operation 在网络被断开时自动序列化并在网络恢复后自动执行。例如当你离线时也能够进行收藏tweet 的操作，
    *  然后在你再次上线时 operation 自动恢复执行。

    在应用程序进入后台时，冻结的 operation 也会被持久化到磁盘。然后在应用程序回到前台后自动恢复执行。
    MKNetworkOperation 中的有用方法

###如下所示，MKNetworkOperation 公开了一些有用的方法，你可从中获取各种格式的 response 数据：
        responseData
        responseString
        responseJSON (Only on iOS 5)
        responseImage
        responseXML
        error

    当 operation 执行完时，这些方法被用于获取响应数据。如果格式不正确，方法会返回nil。例如，响
    应的数据明明是一个 HTML 格式，你用 responseImage 方法只会得到 nil。只有 responseData 能
    保证无论什么格式都返回正确，而其他方法你必须确保和相应的repsone 类型匹配。

####有用的宏
    DLog 和 ALog 宏被无耻地从 Stackoverflow 剽窃来了，我找不到源作者。如果是你写的，请告诉我。

