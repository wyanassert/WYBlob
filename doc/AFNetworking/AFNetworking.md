# [AFNetworking3.1.0](https://github.com/AFNetworking/AFNetworking.git) 源码解析

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
*Tips: [NSURL path] 指的是url中主机名后面的部分, 比如https://baidu.com/test/123, baidu.com是主机名(hostname), /test/123 则是路径(path), 经过上面的预处理, 会变成https://baidu.com/test/123, 避免在后面加路径时候出错, 关于`NSURL`的各个属性, 可以从*[这个问题](https://stackoverflow.com/questions/3692947/get-parts-of-a-nsurl-in-objective-c)*直观的看出各个属性的区别.*

#### e. 使用GET请求完成一次数据传输任务


#### f. 使用HEAD请求完成一次数据传输任务

#### g. 使用POST请求完成一次数据传输任务

#### h.  使用POST请求完成一次数据传输任务, 并且可以对Http请求的body参数进行处理, 然后再进行网络请求

#### i. 使用PUT请求完成一次数据传输任务

#### j. 使用PATCH请求完成一次数据传输任务

#### k. 使用DELETE请求完成一次数据传输任务

#### l. 各个任务实际完成者(在.m文件中定义)

### 3. 小结


---

## `AFURLSessionManager`