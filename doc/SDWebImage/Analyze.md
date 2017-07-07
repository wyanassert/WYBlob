# [SDWebImage](https://github.com/rs/SDWebImage) 分析

#### Version 4.0.0
---

## UIKit 交互1 -- `UIView+WebCache`
### 1. 接口定义

#### a. 下载图片


```
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
```

#### b. 下载已经缓存过的图片

```
- (void)sd_setImageWithPreviousCachedImageWithURL:(nullable NSURL *)url
                                 placeholderImage:(nullable UIImage *)placeholder
                                          options:(SDWebImageOptions)options
                                         progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                        completed:(nullable SDExternalCompletionBlock)completedBlock;
```

#### c. 下载动图

```
- (void)sd_setAnimationImagesWithURLs:(nonnull NSArray<NSURL *> *)arrayOfURLs;
```

#### d. 取消下载动图
```
- (void)sd_cancelCurrentAnimationImagesLoad;
```

***

### 2. 分析

#### a. 下载图片
*下载图片有数个方法定义, 见`UIView+WebCache.h`, 最终都调用了`UIView+WebCache`中的 `sd_internalSetImageWithURL` 这个方法.*

sd_internalSetImageWithURL 方法做的事情：

* 首先取消了当前 View 所绑定的一切请求.
* 设置placeholder.
* 调用`SDWebImageManager`下载图片, 并将方法返回的 operation 与当前 View 绑定.
* 下载图片回调处理.


*Tips*  
`dispatch_main_async_safe ` 定义了在主线程进行UI操作的宏:

```
#ifndef dispatch_main_async_safe
#define dispatch_main_async_safe(block)\
    if (strcmp(dispatch_queue_get_label(DISPATCH_CURRENT_QUEUE_LABEL), dispatch_queue_get_label(dispatch_get_main_queue())) == 0) {\
        block();\
    } else {\
        dispatch_async(dispatch_get_main_queue(), block);\
    }
#endif
```

`weakSelf` 为 nil 时候直接结束避免崩溃或者其他错误.

#### b. 下载已经缓存过的图片

* 首先调用`SDImageCache`从缓存中读取Image.
* 再直接执行上一步下载图片的代码.

```
NSString *key = [[SDWebImageManager sharedManager] cacheKeyForURL:url];
UIImage *lastPreviousCachedImage = [[SDImageCache sharedImageCache] imageFromCacheForKey:key];

[self sd_setImageWithURL:url placeholderImage:lastPreviousCachedImage ?: placeholder options:options progress:progressBlock completed:completedBlock]; 
```

*Tips* : 在执行下载的过程中, 如果找到了缓存, 就忽略placeholder, 避免一次无效操作.


#### c. 下载动图
* 取消当前View 绑定的动图下载操作.
* 遍历传入的URL数组, 对每个URL调用`SDWebImageManager`下载图片, 并将方法返回的 operation 装入数组,再将这个数组与当前 View 绑定.

PS. 这也解释了 `- (void)sd_cancelImageLoadOperationWithKey:(nullable NSString *)key` 为什么会有如下看起来很奇怪的代码, operation的类型并不是固定的.

```
SDOperationsDictionary *operationDictionary = [self operationDictionary];
id operations = operationDictionary[key];
if (operations) {
    if ([operations isKindOfClass:[NSArray class]]) {
        for (id <SDWebImageOperation> operation in operations) {
            if (operation) {
                [operation cancel];
            }
        }
    } else if ([operations conformsToProtocol:@protocol(SDWebImageOperation)]){
        [(id<SDWebImageOperation>) operations cancel];
    }
    [operationDictionary removeObjectForKey:key];
}
```

*Tips* : `[self operationDictionary]` 使用 Runtime 为实例增加了变量.

*Question* : 如何保证下载的顺序?

#### d. 取消下载动图

* 直接调用取消方法 `- (void)sd_cancelImageLoadOperationWithKey:(nullable NSString *)key`

*Tips* : 关于`id <SDWebImageOperation> operation`解释:
	每个 operaton 都有实现一个 `- (void)cancel;` 方法, 这个是在`SDWebImageOperation`协议中定义, 无论是什么类型实例, 只要实现了该协议, 都可以统一调用,详细解释可以搜索`iOS`+`面向接口编程`.
	
### 3. 小结
在 `UIView+WebCache` 模块中, 只做了一些简单的操作, 定义好了与 `UIKit` 交互接口, 下载与取消交给了 `SDWebImageManager` 处理, 缓存交给了 `SDImageCache` 处理.


---
## SDWebImage幕后管理者 -- `SDWebImageManager `

### 1. 接口定义

#### a. 缓存模块
```
@property (strong, nonatomic, readonly, nullable) SDImageCache *imageCache;
```

#### b. 下载模块
```
@property (strong, nonatomic, readonly, nullable) SDWebImageDownloader *imageDownloader;
```

#### c. 下载图片
```
- (nullable id <SDWebImageOperation>)loadImageWithURL:(nullable NSURL *)url
                                              options:(SDWebImageOptions)options
                                             progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                            completed:(nullable SDInternalCompletionBlock)completedBlock;
```

#### d. 手动设置图片缓存
```
- (void)saveImageToCache:(nullable UIImage *)image forURL:(nullable NSURL *)url;
```

#### e. 取消所有的操作
```
- (void)cancelAll;
```

#### f. 当前是否有操作在运行
```
- (BOOL)isRunning;
```

#### g. 异步检查图片是否已经被缓存
```
- (void)cachedImageExistsForURL:(nullable NSURL *)url
                     completion:(nullable SDWebImageCheckCacheCompletionBlock)completionBlock;
```

#### h. 异步检查图片是否已经被缓存在了磁盘上
```
- (void)diskImageExistsForURL:(nullable NSURL *)url
                   completion:(nullable SDWebImageCheckCacheCompletionBlock)completionBlock;
```

#### i. 获取URL缓存索引的Key
```
- (nullable NSString *)cacheKeyForURL:(nullable NSURL *)url;
```


### 2. 分析
#### a. 缓存模块
* 通过 `[SDImageCache sharedImageCache]` 引用到了 SDImageCache 的单例.

#### b. 下载模块
* 通过 `[SDWebImageDownloader sharedImageCache]` 引用到了 SDWebImageDownloader 的单例.

#### c. 下载图片
* 判断输入错误处理等, 并生成一个 `SDWebImageCombinedOperation` 实例, 也就是上文提到过的一个operation, 然后将改operation加入`self.runningOperations`方便管理;
* 使用 `[SDImageCache queryCacheOperationForKey: done:]` 生成了一个NSOperation实例 并赋值给了上一步所使用的operation的cacheOperation属性, 方便执行cancel方法.
* 上一步中, 在 `SDImageCache` 中先在内存中找图片的缓存,找到直接执行回调,若没找到则在硬盘上找缓存,若找到切可以在内存做缓存,则在内存中做缓存, 然后执行回调.
* 在`queryCacheOperationForKey `方法的回调中,如果发现当前Operation被取消了,则将该operation从`self.runningOperations`移除,并使用`@synchronized`保证线程安全.
* 如果在缓存找到了图片,执行回调,并移除operation
* 如果没有找到, 则调用`[SDWebImageDownloader downloadImageWithURL:options:progress:completed:]`方法下载图片, 并将该方法换回的token标志绑定到operation的cancelBlock中, 方便取消下载请求;
* 下载完成之后之后, 执行`imageManager:transformDownloadedImage:withURL:`方便使用者在下载完成时候立刻对图片做一些自定义处理,再将该图片进行缓存. 最后执行回调并移除改operation.

#### d. 手动设置图片缓存
* 直接调用`[SDImageCache storeImage:forKey:toDisk:completion:]`

#### e. 取消所有的操作
* 因为涉及到 `self.runningOperations` 的读写, 因此用 `@synchronized` 保证线程安全.
* copy 了一份`self.runningOperations`, 并使用 `[NSArray makeObjectsPerformSelector:]` 方法取消队列中所有的操作.
* 将复制的队列中的所有元素从`self.runningOperations`移除.

#### f. 当前是否有操作在运行
* 为保证原子性,使用` @synchronized `访问`self.runningOperations`

#### g. 异步检查图片是否已经被缓存
* 分别调用`[SDImageCache imageFromMemoryCacheForKey:]`和 `[SDImageCache diskImageExistsWithKey:completion:]` 方法检查是否在内存和硬盘上缓存.
* 由于检查硬盘是否缓存要用到专门的IO线程(在SDImageCache中定义), 调用者不可能去等待IO线程,因此此方法被设计为异步方法.

#### h. 异步检查图片是否已经被缓存在了磁盘上
* 调用`[SDImageCache diskImageExistsWithKey:completion:]` 方法检查是是否在硬盘上缓存.

#### i. 获取URL缓存索引的Key
* 如果定义了`self.cacheKeyFilter`自定义存储Key,则使用该回调获取用于缓存索引的Key

### 3. 小结
从`SDWebImage `可以看出作者考虑到了很多一般开发者不会去考虑的事情, 简单的如线程安全, 更细致的如`imageManager:transformDownloadedImage:withURL:`方法, 方便使用SDWebImage的人在使用之前先处理, 再缓存, 一个个简单的应用场景是用户想对一张网络图片进行模糊处理, 一般的步骤是先用SDWebImage下载,然后自行模糊处理,再展示. 但如果有大量图片要处理, 又涉及到tableView的复用问题, 为了提高性能, 使用者要自己对模糊之后的图片做缓存, 优化缓存策略和IO潜在地问题等等. 实际上SDWebImage 已经可以处理这个问题而不需要使用者再去考虑.

---
## SDWebImage缓存模块 -- `SDImageCache`

### 1. 接口定义
#### a. 缓存图片
```
- (void)storeImage:(nullable UIImage *)image
         imageData:(nullable NSData *)imageData
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock;
```

#### b. 缓存图片到硬盘上(只能从IO Queue 调用)
```
- (void)storeImageDataToDisk:(nullable NSData *)imageData forKey:(nullable NSString *)key;
```

#### c. 检查图片是否在硬盘上缓存(只检查, 不会把图片加到内存)
```
- (void)diskImageExistsWithKey:(nullable NSString *)key completion:(nullable SDWebImageCheckCacheCompletionBlock)completionBlock;
```

#### d. 检查是否有缓存
```
- (nullable NSOperation *)queryCacheOperationForKey:(nullable NSString *)key done:(nullable SDCacheQueryCompletedBlock)doneBlock;
```

#### e. 从内存中取缓存的图片(同步)
```
- (nullable UIImage *)imageFromMemoryCacheForKey:(nullable NSString *)key;
```

#### f. 从硬盘取缓存的图片(同步)
```
- (nullable UIImage *)imageFromDiskCacheForKey:(nullable NSString *)key;
```

#### g. 从缓存中取图片(同步)
```
- (nullable UIImage *)imageFromCacheForKey:(nullable NSString *)key;
```

#### h. 从内存和硬盘删除图片缓存(异步)
```
- (void)removeImageForKey:(nullable NSString *)key withCompletion:(nullable SDWebImageNoParamsBlock)completion;
```

#### i. 从内存移除缓存, 选择是否从硬盘移除
```
- (void)removeImageForKey:(nullable NSString *)key fromDisk:(BOOL)fromDisk withCompletion:(nullable SDWebImageNoParamsBlock)completion;
```

#### j. 清除内存缓存
```
- (void)clearMemory;
```

#### k. 异步清除磁盘缓存
```
- (void)clearDiskOnCompletion:(nullable SDWebImageNoParamsBlock)completion;
```

#### l. 异步清除硬盘上已经过期的缓存
```
- (void)deleteOldFilesWithCompletionBlock:(nullable SDWebImageNoParamsBlock)completionBlock;
```

### 2. 分析
#### 初始化方法
```
- (nonnull instancetype)initWithNamespace:(nonnull NSString *)ns
                       diskCacheDirectory:(nonnull NSString *)directory;
```
初始化各个属性:
```
@property (strong, nonatomic, nullable) dispatch_queue_t ioQueue;

_ioQueue = dispatch_queue_create("com.hackemist.SDWebImageCache", DISPATCH_QUEUE_SERIAL);
```

*生成的一个串行队列,专用于IO操作. 不再使用的时候应该使用`dispatch_release`释放队列.*


`@property (strong, nonatomic, nonnull) NSCache *memCache;` 

*`NSCache `是iOS系统提供的缓存类,通过键值对对需要缓存的对象作强引用来达到缓存的目的.*

```
NSFileManager *_fileManager;
dispatch_sync(_ioQueue, ^{
            _fileManager = [NSFileManager new];
        });
``` 
*注意,生成_fileManager在ioQueue中,并且是是一个同步操作, 之后_fileManager都要在ioQueue中进行.*


#### a. 缓存图片
* 首先根据`SDImageCacheConfig`判断是否在在内存中缓存,使用 [NSCache setObject:forKey:cost:]方法缓存
* 根据`toDisk`参数判断是否在磁盘中缓存,在ioQueue中调用`[self storeImageDataToDisk:data forKey:key];`缓存到硬盘,使用`@autoreleasepool`释放临时变量.
* 最后在组线程执行回调.

*Tips* 
*NSCache 有最大缓存容积的设置`totalCostLimit`, 但是这个设置只有在设置缓存的时候指定要缓存对象占用的字节数(cost)才能生效. 但是对象的内存占用计算十分复杂, SDWebImage只是给出了一个大致值`image.size.height * image.size.width * image.scale * image.scale;`.*

#### b. 缓存图片到硬盘上(只能从IO Queue 调用)
*此方法只能在ioQueue中调用,奇怪的是`SDImageCache`并没有暴露ioQueue访问, 因此, 将此方法暴露在.h文件是没有意义的.*

* 首先检查是不是在ioQueue
* 使用key的16位MD5编码+文件后缀作为缓存文件名生成存储路径, 如果目标路径不存在,则创建该路径
* 使用`[NSfileManager createFileAtPath:contents:attributes:]`将下载下来的图片的原始二进制数据写磁盘

#### c. 检查图片是否在硬盘上缓存(只检查, 不会把图片加到内存)
* 先后用key参数的md5编码组合路径找是都存在文件,在用key参数的md5编码+文件后缀寻找是否存在
* 在主线程中回调

*在某个版本之前, 硬盘缓存没有文件后缀名, 为了兼容, 要做两次查找*

#### d. 检查是否有缓存
* 先调用`imageFromMemoryCacheForKey`, 并用结果执行回调, 注意这一步是同步操作, 因此不需要 NSOperation 来取消操作, 故返回nil.
* 再调用`diskImageDataBySearchingAllPathsForKey`在硬盘找缓存, 如果找到并且条件允许, 在内存缓存该图片. 这一步要在ioQueue中异步执行, 可以利用 NSOperation 取消这一步操作. 因此在执行回调后返回该NSOperation.

#### e. 从内存中取缓存的图片(同步)
* 直接在`self.memCache`读取, 可能情况是没有缓存-返回nil, 有缓存但是缓存已经被释放-返回nil, 或是寻找缓存命中-返回目标图片.

#### f. 从硬盘取缓存的图片(同步)
* 在硬盘上找目标二进制文件,找到后调用`UIImage *image = [UIImage sd_imageWithData:data];image = [self scaledImageForKey:key image:image];`生成目标图片,若允许,在内存中缓存该图片.

*理论上来说, 这句话放在ioQueue中执行会好一些, 猜测可能是需要同步执行* 

#### g. 从缓存中取图片(同步)
* 直接调用上面两个方法.

#### h. 从内存和硬盘删除图片缓存(异步)
* 直接调用下方的方法.

#### i. 从内存移除缓存, 选择是否从硬盘移除
* 直接从内存缓存中移除, (NSCache是线程安全的)
* 在ioQueue中移除目标文件, 并在主线程回调

#### j. 清除内存缓存
* 直接移除`self.memCache`所有对象

#### k. 异步清除磁盘缓存
* 调用下方方法

#### l. 异步清除硬盘上已经过期的缓存
* 遍历缓存目录下每一个文件, 获取其属性, 若获取失败或是该文件是文件夹则跳过
* 若文件的修改时间早于设定时间,则将文件地址加入待删除列表.
* 遍历结束, 从硬盘移除待删除列表中的文件.
* 计算未删除的文件的总大小, 若仍大于目标大小, 进一步删除最早改动的文件.

*Tips* : `[[modificationDate laterDate:expirationDate] isEqualToDate:expirationDate]`这个比较挺有意思.

### 3. 小结
这一个模块中,有内存与硬盘两级缓存, NSCache 在系统级别保证了线程安全,相对来说处理容易. 但是IO操作本身较为耗时, 单独创建一个队列作为ioQueue来进行IO操作, 达到在硬盘上缓存的目的.

## SDWebImage下载模块 -- `SDWebImageDownloader`

### 1. 接口定义
#### a. 初始化
```
- (nonnull instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)sessionConfiguration NS_DESIGNATED_INITIALIZER;
```

#### b. 设置请求的Header
```
- (void)setValue:(nullable NSString *)value forHTTPHeaderField:(nullable NSString *)field;
```

#### c. 下载图片
```
- (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageDownloaderOptions)options
                                                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                 completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock;
```

#### d. 取消下载
```
- (void)cancel:(nullable SDWebImageDownloadToken *)token;
```

#### e. 暂停下载
```
- (void)setSuspended:(BOOL)suspended;
```

#### f. 取消所有的下载
```
- (void)cancelAllDownloads;
```

### 2. 分析
#### a. 初始化
* `_downloadQueue` 是一个NSOperationQueue实例, 每个URL的请依赖这个queue进行管理.
* `_URLOperations` 用于存储所有的operation实例, 每个URL对应一个operation.
* `_barrierQueue` 是一个并行队列, operation的创建于取消都在这个队列中完成.
* `self.session` 是用于网络请求的NSURLSession组件, 所有的operation对这个session保持了弱引用.

#### b. 设置请求的Header

#### c. 下载图片
* 直接调用并返回了[SDWebImageDownloader addProgressCallback:completedBlock:forURL:createCallback:]方法, 以下分析`addProgressCallback`.
* 首先使用`dispatch_barrier_sync`方法, 这是一个同步方法, 但是参数`self.barrierQueue`是一个并发队列, 因此当前线程会等待bolck中执行完(由于使用的是`dispatch_barrier_sync`, 而不是`dispatch_sync`,所以当前block也会等待`self.barrierQueue`中已经添加的任务执行完).
* 如果 执行`addProgressCallback`最后一个参数`createCallback()`, 并返回一个operation, 注意,执行`createCallback `又回到了上一层方法`downloadImageWithURL`方法中.
* 在`downloadImageWithURL `方法, 首先组装好了一个request, 然后生成了一个`SDWebImageDownloaderOperation`或者其子类的的实例, 并将这个operaion加入了`self.downloadQueue`队列中. 如果这个队列严格采用LIFO(是栈不是队), 那么上一个加入的operation要依赖于这个operation, 用`[sself.lastAddedOperation addDependency:operation];`达成目的. 最后返回这个operation. 然后又回到了`addProgressCallback `这个方法. ~~吐下槽, 思路有点纠结.~~
* 回到`addProgressCallback`方法后,执行`[operation addHandlersForProgress:progressBlock completed:completedBlock]`将两个Block绑定到operation中, 复制使用的`[NSBlock copy]`方法, 避免不必要的引用.  *注意这儿同一个url可能被请求多次, 因此一个url绑定一个operation, 一个operation绑定多个执行回调*
* 返回cancelToken 下载方法执行结束.

*Tips* : 怎么开始下载的? `SDWebImageDownloaderOperation`继承了`NSOperation `, 并重写了`start()`方法, 并在`start()`方法中调用了`[self.dataTask resume];`开始下载.

#### d. 取消下载
* 放在 `self.barrierQueue` 异步执行.
* 执行`[SDWebImageDownloaderOperation cancel:]`方法.

*Tips* : `[SDWebImageDownloaderOperation cancel:]`首先将token对应的callback移除掉. 当所有的callbacl都移除掉之后, 会调用父类`NSOperation`的`cancel`方法, 这会将`isCancelled`属性置为YES, 在start方法调用的时候就不会真正执行. 最后调用`[self.dataTask cancel];`关闭数据传输.

*Question*: 手动调cancel方法后, 就不会执行失败的block了吗?

#### e. 暂停下载
* 直接调用`(self.downloadQueue).suspended = suspended;`, 这儿利用了`NSOperationQueue`的功能.

#### f. 取消所有的下载
* `[self.downloadQueue cancelAllOperations];`
* 自动调用`SDWebImageDownloaderOperation`的`cancel`方法.

### 3. 小结
这一个模块开始进行图片下载相关代码的执行, 然而真正的下载代码还是被放在了`SDWebImageDownloaderOperation`中, 'SDWebImageDownloader'模块的分析只是对`SDWebImageDownloaderOperation`做了简单的描述, 主要还是重点分析本模块所做的事情--管理所有的下载行为. 此外, `self.downloadQueue`保证了对`self.URLOperations`操作能并发, 但又不相互干扰(同时保证异步和并发, 但实际上并没有并发).

## SDWebImage下载的执行者 -- `SDWebImageDownloaderOperation `
---
### 1. 接口定义
#### a. 初始化
```
- (nonnull instancetype)initWithRequest:(nullable NSURLRequest *)request
                              inSession:(nullable NSURLSession *)session
                                options:(SDWebImageDownloaderOptions)options;
```

#### b. 存储回调Block
```
- (nullable id)addHandlersForProgress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                            completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock;
```

#### c. 开始(继承父类)
```
- (void)start;
```

#### d. 取消(继承父类)
```
- (void)cancel;
```

#### e. 是否在执行(继承父类)
```
- (void)setFinished:(BOOL)finished;
```

#### f. 是否已结束(继承父类)
```
- (void)setExecuting:(BOOL)executing;
```

#### g. 取消单个操作
```
- (BOOL)cancel:(nullable id)token;
```
### 2. 分析
#### a. 初始化
* 首先将参数中的`request`复制了一份, 注意, `NSURLRequest`实现了`NSCopying`协议.
* 初始化了存储block回调的数组`_callbackBlocks`.
* 将参数`session`复制给了`_unownedSession`属性, 注意这儿是弱引用, 避免不必要的引用.
* 生成了一个并发队列`_barrierQueue`,用于`_callbackBlocks`的增删操作,保证线程安全.

#### b. 存储回调Block
* 在这儿将`progressBlock`和`completedBlock`两个block都复制了一份,再存储到`_callbackBlocks`中.
* 在这儿也使用了`dispatch_barrier_async`方法, 这是个异步操作

#### c. 开始(继承父类)
* 注意, 这儿虽然覆盖了父类的`start`方法, 但是不能调用[super start];
* 在`SDWebImage`中, Operation被加到`SDWebImageDownloader`的`downloadQueue`中后会被自动执行, (自动调用`operation`的`start`方法)
* 首先判断自己是否被取消了
* 再判断`self.unownedSession`是否还在, 一般情况下是还在的, 因为默认的`SDWebImageDownloader`是个单例不会被释放, 但如果开发者自己初始化一个`SDWebImageDownloader`就会存在`self.unownedSession`不再引用一个session的情况.
* 根据`session`和`request`生成一个`dataTask`, 并将自己标记为正在执行.
* 开始下载, 并触发第一次'progressBlock'.

#### d. 取消(继承父类)
* 若已经完成, 直接返回.
* 调用[super cancel], 会将`isFinished`标记为YES.
* 取消下载操作.

#### e. 是否在执行(继承父类)
* 用KVO通知值改变.

#### f. 是否已结束(继承父类)
* 同上.

#### g. 取消单个操作
* 根据token将对应的回调从``删除, 在这儿使用了`[NSArray removeObjectIdenticalTo:]`方法, 利用"本体性"而不是"相等性"去移除对应的回调, 个人猜测是为了提高查找的速度. 具体可以参考[Equality](http://nshipster.cn/equality/)这篇文章.
* 在这儿使用了`dispatch_barrier_sync`, 注意这儿是一个同步方法, 后面根据移除后`_callbackBlocks`是否为空判断是否要停止当前的下载.

*Tips*: `start`与`cancel`用`@synchronized`保证的线程安全, 对`_callbackBlocks`的操作使用一个队列保障线程安全. 此外, operation持有两个`session`, 一个是`unownedSession`, 这个由`SDWebImageDownloader`持有, operation对它保持弱引用, 还有一个是`ownedSession`, 当初始化的`session`被释放时候, 使用自己生成的session, 并用`ownedSession`保持引用, 并在`[self reset]`中释放这个`session`.

### 3. 小结
这个Operation完成了SDWebImage最重要的下载功能. 将一个URL的下载下载封装成一个`NSOperation`, 特别是在线程安全上做了一些优化, 和使用异步或是同步, 哪些操作需要保证线程安全, 哪些元素需要复制, 值得思考. 在`SDWebImage`的issue中有很多于此模块有关的, 值得细看.


## SDWebImage 预加载 -- `SDWebImagePrefetcher`
---
**Todo**
