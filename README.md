# WYBlob
学习笔记, 主要会是一些源码的流程分析和 Tips 吧;
## 10/05/2017 更新 - [AFNetworking源码分析](https://github.com/wyanassert/WYBlob/blob/master/doc/AFNetworking/AFNetworking.md)
#### 1.与外部HTTP请求交互的Manager -- [AFHTTPSessionManager](https://github.com/wyanassert/WYBlob/blob/master/doc/AFNetworking/AFNetworking.md#与外部http请求交互的manager--afhttpsessionmanager)
#### 2.外部会话的核心(基类) -- [AFURLSessionManager](https://github.com/wyanassert/WYBlob/blob/master/doc/AFNetworking/AFNetworking.md#外部会话的核心基类----afurlsessionmanager-1)
#### 3.回调处理 -- [AFURLSessionManagerTaskDelegate](https://github.com/wyanassert/WYBlob/blob/master/doc/AFNetworking/AFNetworking.md#回调处理----afurlsessionmanagertaskdelegate-1)
#### 4.请求报文序列化 -- [AFHTTPRequestSerializer](https://github.com/wyanassert/WYBlob/blob/master/doc/AFNetworking/AFNetworking.md#请求报文序列化----afhttprequestserializer-1)


___
## 07/16/2017 更新 - [SDWebImage源码分析](https://github.com/wyanassert/WYBlob/blob/master/doc/SDWebImage/Analyze.md)

### 按照模块分析 SDWebImage
#### 1. UI交互的基类 [UIView+WebCache](https://github.com/wyanassert/WYBlob/blob/master/doc/SDWebImage/Analyze.md#uikit-交互1----uiviewwebcache)
#### 2. SDWebImage 的主要管理者 [SDWebImageManager](https://github.com/wyanassert/WYBlob/blob/master/doc/SDWebImage/Analyze.md#sdwebimage幕后管理者----sdwebimagemanager)
#### 3. 缓存模块 [SDImageCache](https://github.com/wyanassert/WYBlob/blob/master/doc/SDWebImage/Analyze.md#sdwebimage缓存模块----sdimagecache)
#### 4. 下载模块 [SDWebImageDownloader](https://github.com/wyanassert/WYBlob/blob/master/doc/SDWebImage/Analyze.md#sdwebimage下载模块----sdwebimagedownloader)
#### 5. 下载的执行者 [SDWebImageDownloaderOperation](https://github.com/wyanassert/WYBlob/blob/master/doc/SDWebImage/Analyze.md#sdwebimage下载的执行者----sdwebimagedownloaderoperation)
#### 6. 预加载 [SDWebImagePrefetcher](https://github.com/wyanassert/WYBlob/blob/master/doc/SDWebImage/Analyze.md#sdwebimage-预加载----sdwebimageprefetcher)
#### 7. GIF子模块 [FLAnimatedImage](https://github.com/wyanassert/WYBlob/blob/master/doc/SDWebImage/Analyze.md#sdwebimage-子模块-gif-----flanimatedimage)
