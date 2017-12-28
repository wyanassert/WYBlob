###MWPhotoBrowser 与 SDWebImage的依赖冲突

需要用到[MWPhotoBrowser](https://github.com/mwaterfall/MWPhotoBrowser), 但是直接pod,  `MWPhotoBrowser`依赖的`SDWebImage`版本跟我们平时用的不一样, 参考这篇文章[MWPhotoBrowser 更新其依赖的第三方库](http://ablackcrow.com/2017/05/10/MWPhotoBrowserUpdate3rd/), fork了一个`MWPhotoBrowser`到自己的仓库, 用如下方式引用即可.

```
pod 'MWPhotoBrowser', :git => 'https://github.com/wyanassert/MWPhotoBrowser.git', :commit => '8b1ab836ce17642a8eea6fb18e4c1ea3a64e21dc'
```
