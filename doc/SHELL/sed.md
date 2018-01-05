在mac上使用sed和grep的坑 替换一个文件夹下文档的特定字符串, 如下:

```
grep -lr 'str1' . | xargs sed -i "" 's/str1/str2/g'
```

参考[StackOverflow](https://stackoverflow.com/questions/11942306/how-to-replace-a-string-in-multiple-files-using-grep-and-sed-when-the-i-argumen)

除此之外, 还会有sed的编码问题, 执行以下命令

```
export LC_CTYPE=C
export LANG=C
```
参考[RE error: illegal byte sequence on Mac OS X
](https://stackoverflow.com/questions/19242275/re-error-illegal-byte-sequence-on-mac-os-x)
