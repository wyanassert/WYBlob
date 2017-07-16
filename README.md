# WYBlob
学习笔记, 主要会是一些源码的流程分析和 Tips 吧;

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
