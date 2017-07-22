# [AFNetworking3](https://github.com/AFNetworking/AFNetworking.git) 阅读源码笔记

** Version : 3.1.0**

---


## 结构导航(由表及里)
#### 直接使用的模块:[AFHTTPSessionManager]()


---

## 直接使用的模块--`AFHTTPSessionManager`
阅读头文件可以知道这个模块究竟做了什么, 特别是在此作为`AFNetWorking`暴露给外面的头文件. 但是有个小坑就是`AFHTTPSessionManager`的基类也做了很多事情, 所以仅仅看`AFHTTPSessionManager `是完全不够的, 还要看`AFURLSessionManager`这个文件才能较为全面的了解`AFNetWorking`所提供的功能.

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
`AFHTTPSessionManager`作为最上层的接口, 一如既往地将绝大多数事情交给了其他类来完成, 请求头的序列化交给了[AFURLSessionManager]()完成, 绑定请求的各种回调交给了基类[AFURLSessionManager]()来完成. 着这些基础之上, 顾及到了一些系统"feature"和不同iOS版本的不一致, 有一些亮点值得关注, 在`AFNetWorking`的源码以及文中有相关描述解释以及链接, 值得一看.

---

## `AFURLSessionManager`

---

## `AFURLSessionManagerTaskDelegate `

___