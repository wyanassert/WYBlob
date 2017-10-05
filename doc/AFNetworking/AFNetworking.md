# [AFNetworking3](https://github.com/AFNetworking/AFNetworking.git) 阅读源码笔记

**Version : 3.1.0**

---


## 结构导航(由表及里)
#### 与外部HTTP请求交互的Manager -- [AFHTTPSessionManager]()
#### 外部会话的核心(基类) -- [AFURLSessionManager]()
#### 回调处理 -- [AFURLSessionManagerTaskDelegate]()
#### 请求报文序列化 -- [AFHTTPRequestSerializer]()

---

## 与外部HTTP请求交互的Manager--`AFHTTPSessionManager`
阅读头文件可以知道这个模块究竟做了什么, 特别是在此作为`AFNetWorking`暴露给外面的头文件. 但是有个小坑就是`AFHTTPSessionManager`的基类也做了很多事情, 所以仅仅看`AFHTTPSessionManager.h `是完全不够的, 还要看`AFURLSessionManager.h`这个文件才能较为全面的了解`AFNetWorking`所提供的功能.

### 1. 接口定义
#### a. 继承`AFURLSessionManager`, 很多较为常用的功能实际在基类定义.

#### b. 请求报文序列化
```
AFHTTPRequestSerializer <AFURLRequestSerialization> * requestSerializer;
```

#### c. 响应报文序列化
```
AFHTTPResponseSerializer <AFURLResponseSerialization> * responseSerializer;
```
#### d. 初始化
```
- (instancetype)initWithBaseURL:(nullable NSURL *)url
           sessionConfiguration:(nullable NSURLSessionConfiguration *)configuration NS_DESIGNATED_INITIALIZER;
```

#### e. 使用GET请求完成一次数据传输任务
```
- (nullable NSURLSessionDataTask *)GET:(NSString *)URLString
                            parameters:(nullable id)parameters
                              progress:(nullable void (^)(NSProgress *downloadProgress))downloadProgress
                               success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                               failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

#### f. 使用HEAD请求完成一次数据传输任务
```
- (nullable NSURLSessionDataTask *)HEAD:(NSString *)URLString
                    parameters:(nullable id)parameters
                       success:(nullable void (^)(NSURLSessionDataTask *task))success
                       failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

#### g. 使用POST请求完成一次数据传输任务
```
- (nullable NSURLSessionDataTask *)POST:(NSString *)URLString
                             parameters:(nullable id)parameters
                               progress:(nullable void (^)(NSProgress *uploadProgress))uploadProgress
                                success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                                failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

#### h.  使用POST请求完成一次数据传输任务, 并且可以对Http请求的body参数进行处理, 然后再进行网络请求
```
- (nullable NSURLSessionDataTask *)POST:(NSString *)URLString
                             parameters:(nullable id)parameters
              constructingBodyWithBlock:(nullable void (^)(id <AFMultipartFormData> formData))block
                               progress:(nullable void (^)(NSProgress *uploadProgress))uploadProgress
                                success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                                failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

#### i. 使用PUT请求完成一次数据传输任务
```
- (nullable NSURLSessionDataTask *)PUT:(NSString *)URLString
                   parameters:(nullable id)parameters
                      success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                      failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

#### j. 使用PATCH请求完成一次数据传输任务
```
- (nullable NSURLSessionDataTask *)PATCH:(NSString *)URLString
                     parameters:(nullable id)parameters
                        success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                        failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

#### k. 使用DELETE请求完成一次数据传输任务
```
- (nullable NSURLSessionDataTask *)DELETE:(NSString *)URLString
                      parameters:(nullable id)parameters
                         success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                         failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

#### l. 各个任务实际完成者(在.m文件中定义)
```
- (NSURLSessionDataTask *)dataTaskWithHTTPMethod:(NSString *)method
                                       URLString:(NSString *)URLString
                                      parameters:(id)parameters
                                  uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgress
                                downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgress
                                         success:(void (^)(NSURLSessionDataTask *, id))success
                                         failure:(void (^)(NSURLSessionDataTask *, NSError *))failure
```

### 2. 分析
#### a. 继承`AFURLSessionManager`
* 之前也说了, 挺多较为常用的方法世界是在基类里面定义的, 由于本文更加倾向于按模块来分析功能, 这些功能都放在[AFURLSessionManager]()说明.

#### b. 请求报文序列化
* 使用 `[AFHTTPRequestSerializer serializer]`方法初始化一个`requestSerializer`, 注意, 在这儿, 无论是`AFHTTPSessionManager`还是`requestSerializer`或者`responseSerializer `都不是单例.
* 具体的会在[AFHTTPRequestSerializer]()分析讲解.

#### c. 响应报文序列化
* 使用`[AFJSONResponseSerializer serializer]`初始化一个实例.
* 具体的会在[AFJSONResponseSerializer]()分析讲解.

#### d. 初始化
* 使用`configuration`参数初始化基类.
* 对参数`url`进行预处理, 如果`[[url path] length]`大于0, 并且url没有后缀'/', 就会给url加上一个'/'.
* 初始化`requestSerializer`和`responseSerializer`
* 
*Tips: [NSURL path] 指的是url中主机名后面的部分, 比如URL:https://baidu.com/test/123, 其中 baidu.com是主机名(hostname), /test/123 则是路径(path), 经过上面的预处理, 会变成https://baidu.com/test/123/, 避免在后面加路径时候出错, 关于`NSURL`的各个属性, 可以从*[这个问题](https://stackoverflow.com/questions/3692947/get-parts-of-a-nsurl-in-objective-c)*直观的看出各个属性的区别.*

#### e. 使用GET请求完成一次数据传输任务
* 实际是调用[[AFHTTPSessionManager dataTaskWithHTTPMethod: URLString:parameters: uploadProgress: downloadProgress: success: failure:]]()方法完成.
* 使用`[dataTask resume];`启动任务.

#### f. 使用HEAD请求完成一次数据传输任务
* 同上. 把GET请求换成了HEAD请求.

#### g. 使用POST请求完成一次数据传输任务
* 同上. 把GET请求换成了POST请求.

#### h.  使用POST请求完成一次数据传输任务, 并且可以对Http请求的body参数进行处理, 然后再进行网络请求
* 使用`self.requestSerializer`构造一个`NSMutableURLRequest`实例, body参数进行处理这一回调也是在这一步的最后调用的.
* 如果构造失败, 执行`failure`回调.
* 然后使用上面的request 生成了一个`dataTask`, 在这儿使用如下代码, 目的是为了解决iOS8以下在并发队列创建dataTask造成回调错位的bug, [参考这儿](https://github.com/AFNetworking/AFNetworking/issues/2093).

	```
	__block NSURLSessionDataTask *dataTask = nil;
   url_session_manager_create_task_safely(^{
       dataTask = [self.session dataTaskWithRequest:request];
   });
	```
* 调用基类`AFHTTPSessionManager`的方法(虽然我习惯按模块来分析, 但是这而为了保不打断分析的节奏, 就继续写了, 在[AFHTTPSessionManager]()可能会重新分析这一块)`addDelegateForDataTask:uploadProgress:downloadProgress:completionHandler:`存储回调, 并绑定dataTask. 详细来说, 先使用[AFURLSessionManagerTaskDelegate]()创建一个实例delegate, 将`completionHandler`, `uploadProgressBlock` ,`downloadProgressBlock`三个参数绑定到delegate中, 同时delegate弱引用当前的manager, 最后使用`self.lock`加锁, 将delegate放到当前manager的`mutableTaskDelegatesKeyedByTaskIdentifier`数组中, 添加`resume`与`suspend`的通知, 做完之后进行解锁.  **在这个地方有个trick的写法, 构建了一个`_AFURLSessionTaskSwizzling`, 并在它的`load`方法中, 获取了一个`NSURLSessionDataTask`实例, 并一直沿着继承关系向父类遍历, 当该类实现了`resume`方法, 但是实现与该类的父类实现不一样, 也同`af_resume`不一样时候就替换方法(实际替换了resume 和 suspend), 然后再判断这个类的父类, 具体情况见[AFNetWorking](https://github.com/AFNetworking/AFNetworking/pull/2702), 或者[【原】AFNetworking源码阅读（四）](http://www.cnblogs.com/polobymulberry/p/5160946.html#_label3).**简单的理解就是监听了NSURLSessionTask的`resume`和`suspend`方法, 然后达到改变indicator状态之类的目的.


*Tips:由于`self.completionQueue`属性是可以在外部设置的, 因此, 这儿采用了`dispatch_async(self.completionQueue ?: dispatch_get_main_queue(), ^{ });`这样一种写法, `?:`是一个C语法, [GNU extension (Conditional with Omitted Operand)](http://gcc.gnu.org/onlinedocs/gcc/Conditionals.html)中规定了`x ? : y`等同于`x ? x : y`, 好处是写法更简洁同时能避免`x`操作符执行两次.*

#### i. 使用PUT请求完成一次数据传输任务
* 实际是调用[[AFHTTPSessionManager dataTaskWithHTTPMethod: URLString:parameters: uploadProgress: downloadProgress: success: failure:]]()方法完成.
* 使用`[dataTask resume];`启动任务.

#### j. 使用PATCH请求完成一次数据传输任务
* 同上. 把GET请求换成了PATCH请求.

#### k. 使用DELETE请求完成一次数据传输任务
* 同上. 把GET请求换成了DELETE请求.

#### l. 各个任务实际完成者(在.m文件中定义)
* 各种HTTP的Method实际都是调用这个方法完成.
* 使用`self.requestSerializer`构造一个`NSMutableURLRequest`实例.
* 调用基类的`dataTaskWithRequest: uploadProgress: downloadProgress: completionHandler:`方法. 这之后实际在前面[h.  使用POST请求完成一次数据传输任务]()也有说明.

### 3. 小结
`AFHTTPSessionManager`作为最上层的接口, 一如既往地将绝大多数事情交给了其他类来完成, 请求头的序列化交给了[AFHTTPRequestSerializer]()完成, 绑定请求的各种回调交给了基类[AFURLSessionManager]()来完成. 着这些基础之上, 顾及到了一些系统"feature"和不同iOS版本的不一致, 有一些亮点值得关注, 在`AFNetWorking`的源码以及文中有相关描述解释以及链接, 值得一看.

---

## 外部会话的核心(基类) -- `AFURLSessionManager`
`AFURLSessionManager`是`AFHTTPSessionManager`的基类, AFNetWorking中常用的下载方法`downloadTaskWithRequest: progress: destination: completionHandler:`方法实际是定义在基类里面的. 此外, 子类中的所有与Http交互的接口,最终都是调用基类中的`dataTaskWithRequest`方法实现的, 所以这个模块可以视为`AFNetWorking`隐藏在Manager下的核心模块.

### 1. 接口定义

#### a. 指定初始化方法.

```
- (instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)configuration;
```

#### b. 废弃当前会话, 并决定是否取消挂起的请求.

```
- (void)invalidateSessionCancelingTasks:(BOOL)cancelPendingTasks;
```

#### c. 生成一个数据传输任务.

```
- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request
                               uploadProgress:(nullable void (^)(NSProgress *uploadProgress))uploadProgressBlock
                             downloadProgress:(nullable void (^)(NSProgress *downloadProgress))downloadProgressBlock
                            completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject,  NSError * _Nullable error))completionHandler;
```

#### d. 根据特地的本地文件创建一个上传任务.

```
- (NSURLSessionUploadTask *)uploadTaskWithRequest:(NSURLRequest *)request
                                         fromFile:(NSURL *)fileURL
                                         progress:(nullable void (^)(NSProgress *uploadProgress))uploadProgressBlock
                                completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject, NSError  * _Nullable error))completionHandler;
```

#### e. 创建一个上传任务.

```
- (NSURLSessionUploadTask *)uploadTaskWithRequest:(NSURLRequest *)request
                                         fromData:(nullable NSData *)bodyData
                                         progress:(nullable void (^)(NSProgress *uploadProgress))uploadProgressBlock
                                completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject, NSError * _Nullable error))completionHandler;
```

#### f. 根据数据源上传(是这么翻译?)

```
- (NSURLSessionUploadTask *)uploadTaskWithStreamedRequest:(NSURLRequest *)request
                                                 progress:(nullable void (^)(NSProgress *uploadProgress))uploadProgressBlock
                                        completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject, NSError * _Nullable error))completionHandler;
```

#### g. 创建下载请求

```
- (NSURLSessionDownloadTask *)downloadTaskWithRequest:(NSURLRequest *)request
                                             progress:(nullable void (^)(NSProgress *downloadProgress))downloadProgressBlock
                                          destination:(nullable NSURL * (^)(NSURL *targetPath, NSURLResponse *response))destination
                                    completionHandler:(nullable void (^)(NSURLResponse *response, NSURL * _Nullable filePath, NSError * _Nullable error))completionHandler;
```

#### h. 继续未完成的下载

```
- (NSURLSessionDownloadTask *)downloadTaskWithResumeData:(NSData *)resumeData
                                                progress:(nullable void (^)(NSProgress *downloadProgress))downloadProgressBlock
                                             destination:(nullable NSURL * (^)(NSURL *targetPath, NSURLResponse *response))destination
                                       completionHandler:(nullable void (^)(NSURLResponse *response, NSURL * _Nullable filePath, NSError * _Nullable error))completionHandler;
```

### 2. 分析

#### a. 指定初始化方法.
* 设置`configuration`, 若为空, 使用`[NSURLSessionConfiguration defaultSessionConfiguration]`.
* 初始化`self.operationQueue`并将并发数设置为1.
* 初始化`self.session`-- 会话, `self.responseSerializer`--请求序列化, `self.securityPolicy`--安全相关设置, `self.reachabilityManager`, `self.mutableTaskDelegatesKeyedByTaskIdentifier`--存储delegate, `self.lock`--对`mutableTaskDelegatesKeyedByTaskIdentifier`进行操作时候上锁保证线程安全.
* 使用`getTasksWithCompletionHandler`方法对`self.session`所持的task进行一次遍历, 将已经存在的task加入管理.~~但是初始化的时候这样遍历应该是没有东西的吧?~~

#### b. 废弃当前会话, 并决定是否取消挂起的请求.
* 根据参数执行`[self.session invalidateAndCancel];`或者`[self.session finishTasksAndInvalidate];`

#### c. 生成一个数据传输任务.
* 使用`url_session_manager_create_task_safely`安全方法与`[self.session dataTaskWithRequest:request];`产生一个dataTask. 参数`fileURL`作为产生dataTask的参数使用, 实际是iOS8及以上什么都不做, iOS8以下专门创建一个串行队列用于产生dataTask.
* 调用`[self addDelegateForDataTask: uploadProgress: downloadProgress: completionHandler:];`方法将各个回调block加入管理, 在这儿使用了NSLock锁.
* 返回产生的dataTask, 注意, 可能为nil.

#### d. 根据特地的本地文件创建一个上传任务.
* 使用`url_session_manager_create_task_safely`安全方法产生一个uploadTask.
* 由于iOS7上`[self.session uploadTaskWithRequest:request fromFile:fileURL]`可能返回nil, 因此失败时候会重试三次.
* 调用`[self addDelegateForDataTask: uploadProgress: downloadProgress: completionHandler:];`方法将各个回调block加入管理.
* 返回产生的uploadTask, 注意, 可能为nil.

#### e. 创建一个上传任务.
* 使用`url_session_manager_create_task_safely`安全方法与`[self.session uploadTaskWithRequest:request fromData:bodyData]`产生一个uploadTask.
* 调用`[self addDelegateForDataTask: uploadProgress: downloadProgress: completionHandler:];`方法将各个回调block加入管理.
* 返回产生的uploadTask, 注意, 可能为nil.

#### f. 根据数据源上传(是这么翻译?)
* 使用`[self.session uploadTaskWithStreamedRequest:request]`产生一个uploadTask, 除此之外, 跟上面一个方法一致.

#### g. 创建下载请求
* 使用`[self.session downloadTaskWithRequest:request]`创建一个下载任务.
* 跟上面方法类似, 进行回调相关的管理.
* 如果`destination`参数有赋值, 在该回调中返回期望的下载地址(是具体文件的地址, 不能是文件夹, 不在乎文件明德情况下可以使用 文件夹路径/uuid , 如果在乎文件名可以使用 文件夹路径/[NSURL lastPathComponent]), 下载完成后, 文件会被移到该地址.

#### h. 继续未完成的下载
* 使用`[self.session downloadTaskWithResumeData:resumeData]`初始化下载任务, 其他同上.

### 3. 小结
这个模块只是分析了一些主要的功能, 还有很多譬如获取上下传进度,要求认证,重定向之类的方法, 基本都是直接调用系统方法, 不过多说明. 一些`NSURLSessionDownloadTask`, `NSURLSessionUploadTask`和 `NSURLSessionDataTask`的初始化都是直接调用的系统方法. 回调的绑定处理则是在[AFURLSessionManagerTaskDelegate]()完成.

---

## 回调处理 -- `AFURLSessionManagerTaskDelegate`
`AFURLSessionManagerTaskDelegate` 是构造的的一个专门处理各种回调, 声明实现了`NSURLSessionTaskDelegate`, `NSURLSessionDataDelegate`, `NSURLSessionDownloadDelegate`三个delegate, 具体是在`[self setDelegate:delegate forTask:downloadTask];`这个方法中完成的delegate绑定. 定义在`AFURLSessionManager.m`文件中, 即只有`AFURLSessionManager`可以使用这个delegate.

### 1. 接口定义
#### a. 初始化
```
- (instancetype)initWithTask:(NSURLSessionTask *)task;
```

#### b.`NSURLSessionTaskDelegate` -- 完成任务
```
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
                           didCompleteWithError:(nullable NSError *)error;
```

#### c. `NSURLSessionDataDelegate` -- 下载了部分数据
```
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
                                     didReceiveData:(NSData *)data;
```

#### d. `NSURLSessionDataDelegate` -- 已经上传了部分数据
```
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
                                didSendBodyData:(int64_t)bytesSent
                                 totalBytesSent:(int64_t)totalBytesSent
                       totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend;
```

#### e. `NSURLSessionDownloadDelegate` -- 已经写入部分数据
```
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask
                                           didWriteData:(int64_t)bytesWritten
                                      totalBytesWritten:(int64_t)totalBytesWritten
                              totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
```

#### f. `NSURLSessionDownloadDelegate` -- 开始恢复下载
```
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask
                                      didResumeAtOffset:(int64_t)fileOffset
                                     expectedTotalBytes:(int64_t)expectedTotalBytes;
```

#### g. `NSURLSessionDownloadDelegate` -- 下载完成
```
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask
                              didFinishDownloadingToURL:(NSURL *)location;
```

### 2. 分析
#### a. 初始化
* 初始化`mutableData`, `uploadProgress`, `downloadProgress`.
* 定义progress的`cancellationHandler`, `pausingHandler`, `resumingHandler`行为, 方便再将progress抛到外部后可以从外部控制task. **注意, `resumingHandler`是iOS9 之后才有的API**
* 使用`KVO`监听progress的改变, 当改变时, 执行对应的`downloadProgressBlock`或者是`uploadProgressBlock`. 而progress的改变则是在`NSURLSessionDataDelegate `和`NSURLSessionDownloadDelegate`两组协议中触发.

#### b. `NSURLSessionTaskDelegate` -- 完成任务
* 将`self.mutableData`转化为NSData并释放.
* 在专用的处理的一个并行队列(`url_session_manager_processing_queue()`)中完成结果的序列化
* 在这儿的结果回调使用了`dispatch_group_async`, 但是源码并没有使用相关的`dispatch_group_notify`和`dispatch_group_wait`, 应该是希望调用者将manager的`completionGroup`自己设置, 并自行配合使用`dispatch_group_notify `和 `dispatch_group_wait `方法.

#### c. `NSURLSessionDataDelegate` -- 下载了部分数据
* 改变downloadProgress的下载进度, 使用`[self.mutableData appendData:data];`拼接数据.

#### d. `NSURLSessionDataDelegate` -- 已经上传了部分数据
* 改变uploadProgress的进度

#### e. `NSURLSessionDownloadDelegate` -- 已经写入部分数据
* 改变downloadProgress的下载进度, 数据拼接是系统自动完成.

#### f. `NSURLSessionDownloadDelegate` -- 开始恢复下载
* 改变downloadProgress的下载进度, `expectedTotalBytes`是总共要下载的数据量, 'fileOffset'开始下载时的偏移量是恢复下载时已经下载好的数据量.

#### g. `NSURLSessionDownloadDelegate` -- 下载完成
* 根据`downloadTaskDidFinishDownloading`回调去获取目标下载地址, 然后将下载好的文件从/tmp目录移到目标地址.


### 3. 小结
`AFURLSessionManagerTaskDelegate`模块主要是暂时存储一次请求的各个回调, 以及持有dataTask并处理三种dataTask的回调, 并触发progress回调. 所有属性都是对外公开的, 因此在manager中对此模块的操作并没有在此模块中解释, 而是在manager中有所提及.


___

## 请求报文序列化 -- `AFHTTPRequestSerializer`

### 1. 接口定义

#### a. 默认实例
```
+ (instancetype)serializer;
```

#### b. 设置请求头参数
```
- (void)setValue:(nullable NSString *)value
forHTTPHeaderField:(NSString *)field;
```

#### c. 获取请求头参数
```
- (nullable NSString *)valueForHTTPHeaderField:(NSString *)field;
```

#### d. 构造请求
```
- (NSMutableURLRequest *)requestWithMethod:(NSString *)method
                                 URLString:(NSString *)URLString
                                parameters:(nullable id)parameters
                                     error:(NSError * _Nullable __autoreleasing *)error;
```

### 2. 分析

#### a. 默认实例
* `stringEncoding`属性默认设置为`NSUTF8StringEncoding`.
* 为`requestHeaderModificationQueue`创建一个并行队列, 对存储header的字典的操作都在这个队列中完成.
* 设置`Accept-Language`, 取`[NSLocale preferredLanguages]`中前六个, 并从前往后以此设置优先级.
* 设置`User-Agent`, 类似于`AppIdentifier/2.0.0 (iPhone; iOS 11.0; Scale/3.00)`, 包含设备的一些基本信息.
* 对`allowsCellularAccess`, `cachePolicy`, `HTTPShouldHandleCookies`, `HTTPShouldUsePipelining`, `networkServiceType`, `timeoutInterval`等几个属性使用KVO.

* **Tips: 用下面的方法定义了一个静态方法, 已达到定义一个静态数组的目的**

	```
	static NSArray * AFHTTPRequestSerializerObservedKeyPaths() {
	    static NSArray *_AFHTTPRequestSerializerObservedKeyPaths = nil;
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        _AFHTTPRequestSerializerObservedKeyPaths = @[NSStringFromSelector(@selector(allowsCellularAccess)), NSStringFromSelector(@selector(cachePolicy)), NSStringFromSelector(@selector(HTTPShouldHandleCookies)), NSStringFromSelector(@selector(HTTPShouldUsePipelining)), NSStringFromSelector(@selector(networkServiceType)), NSStringFromSelector(@selector(timeoutInterval))];
	    });
	
	    return _AFHTTPRequestSerializerObservedKeyPaths;
	}
	```
* 在KVO使用时, 为了避免在[XCTest中的崩溃](https://github.com/AFNetworking/AFNetworking/issues/2523), 在这几个属性主动改变时候, 应该关掉自动提醒[(参考KVO Compliance)](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOCompliance.html#//apple_ref/doc/uid/20002178-BAJEAIEE), 代码如下:

	```
	+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {
	    if ([AFHTTPRequestSerializerObservedKeyPaths() containsObject:key]) {
	        return NO;
	    }
	
	    return [super automaticallyNotifiesObserversForKey:key];
	}
	```
	

#### b. 设置请求头参数
* 在`self.requestHeaderModificationQueue`队列中使用 `dispatch_barrier_async`异步调用完成Set操作.

#### c. 获取请求头参数
* 在`self.requestHeaderModificationQueue`队列中使用 `dispatch_barrier_async`同步调用完成Get操作.

#### d. 构造请求
* 在`AFHTTPRequestSerializerObservedKeyPaths()`所说的若干个属性, 若是调用者有自己设置, 则会调用KVO, 在KVO的监控方法中, 若新值不为空, 则将改属性存到`self.mutableObservedChangedKeyPaths`中.
* 在构造请求第一步, 遍历`self.mutableObservedChangedKeyPaths`, 将值存入`NSMutableURLRequest`实例`mutableRequest`中.
* 调用`requestBySerializingRequest: withParameters: error:`方法, 对`mutableRequest`重新赋值, 注意的是, 该方法是`AFURLRequestSerialization`所定义的方法, 在`AFHTTPRequestSerializer`及子类`AFJSONRequestSerializer`和`AFPropertyListRequestSerializer`中都有实现. 接下来解析在`AFHTTPRequestSerializer `该方法的实现.
* 将参数`request`复制了一份并赋值给临时变量`mutableRequest`. 并确保header一致.
* 若有`parameter`参数, 将parameter转化为如"xxx=xxx&xxx=xxx"的字符串.(这一步可以通过设置`queryStringSerialization`完成自定义的参数序列化)
* 如果当前请求是"GET", "HEAD", "DELETE"三者之一, 则将刚刚得到的字符串接到`mutableRequest `的`absoluteString`属性后面, 若`mutableRequest`的`query`有值, 应该接到`query`后面.
* 如果当前请求不是"GET", "HEAD", "DELETE"三者之一, 将请求头`Content-Type`设置为`application/x-www-form-urlencoded`, 并将上一步所得到的字符串按照指定编码转化为NSData, 并设置为请求body.
* 返回`mutableRequest`, 完成一次请求的构造.


---