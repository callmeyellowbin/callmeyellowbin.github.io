1. 首先安装Ruby，https://ruby-china.org/wiki/install_ruby_guide/
2. 然后安装Cocoapods，https://www.jianshu.com/p/1e7ab521000b
3. 然后配置Podfile，https://blog.csdn.net/turkeyteo/article/details/43232083
```
platform :ios, '11.2'
target 'TryMasonry' do
pod 'Masonry'
end
```
4. 我遇到的问题：
  pod install 失败：[!] Unable to satisfy the following requirements:-\`Masonry\` required by \`Podfile\`Specs satisfying the \`Masonry\` dependency were found, but they required a higher minimum deployment target.
  解决：podfile platform :ios,’8.0’ 版本号不能少
  参考的是https://www.jianshu.com/p/afe6d349bdbf
  5.最坑的地方来了！教程没说！
  如果直接打开.xcodeproj文件的话，就会崩溃！
  我们可以看到，其实是生成一个.xcworkspace文件的，要打开的是这个！
  ![](https://upload-images.jianshu.io/upload_images/8407639-87629068201e524d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  坑了我很久。。分享给大家。。

基本就是这样啦，✿✿ヽ(°▽°)ノ✿