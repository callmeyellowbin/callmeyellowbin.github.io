今天就开始我的OC学习之旅啦！首先HelloWorld是必须要掌握的！

可以看到OC的NSLog这个函数挺有意思的，参数还要加个@，哈哈。
```
#import <Foundation/Foundation.h>
int main(int argc, char * argv[]) {
    NSLog(@"Hello World!");
    NSLog(@"This is my first code!");
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
好了，可以看到我的OC之旅就这样启程啦！但是XCode自带的**@autoreleasepool**是什么意思呢，经过查阅资料，我发现了小秘密，（资料来自http://tutuge.me/2015/03/17/what-is-autoreleasepool/)
***
1.**MRC 与 ARC**
MRC（Mannul Reference Counting）和ARC(Automatic Reference Counting)，分别对应着手动引用计数和自动引用计数。
对！是计数，不是“GC、垃圾回收”什么的，就是说，在Objective-C的开发中，ARC不代表像Java那样有GC做垃圾回收，所以本质上还是要“手动”管理内存的。也就是说，我们在ARC环境下写的代码，不用自己手动插入“retain、release这些消息”，ARC会在编译时为我们在合适的位置插入，释放不必要的内存。
而@autoreleasepool就跟对象的release密切相关。

2.**@autoreleasepool 干了啥**
在ARC时代，我们不用手动发送autorelease消息，ARC会自动帮我们加。而这个时候，@autoreleasepool做的事情，跟NSAutoreleasePool就一模一样了。

3.**什么时候用@autoreleasepool**
1)写基于命令行的的程序时，就是没有UI框架，如AppKit等Cocoa框架时。
2)写循环，循环里面包含了大量临时创建的对象。（本文的例子）
3)创建了新的线程。（非Cocoa程序创建线程时才需要）
4)长时间在后台运行的任务。

4.**利用@autoreleasepool优化循环**
利用@autoreleasepool优化循环的内存占用，我觉得最有用的一点，下面就说说这个点。
如下面的循环，次数非常多，而且循环体里面的对象都是临时创建使用的，就可以用@autoreleasepool包起来，让每次循环结束时，可以及时的释放临时对象的内存。
```
//来自Apple文档，见参考
NSArray *urls = <# An array of file URLs #>;
for (NSURL *url in urls) {

    @autoreleasepool {
        NSError *error;
        NSString *fileContents = [NSString stringWithContentsOfURL:url
                                        encoding:NSUTF8StringEncoding error:&error];
        /* Process the string, creating and autoreleasing more objects. */
    }
}
```
***
好的，了解了最基本的main函数的代码之后，我们来看看怎么配置GitHub吧（说实话第一次用苹果系统，坑了我有点久..）

1.先打开GitHub，看到Setting那里，配置一下你的SSH Key。
![image.png](https://upload-images.jianshu.io/upload_images/8407639-c9f96387324bff96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.创建SSH KEY
参考文章：https://www.cnblogs.com/blogzhangwei/p/5944975.html
ssh-keygen -t rsa -C "your_email@example.com"（注意替换成自己的邮箱）
```
user@yourmac:~/.ssh$ ssh-keygen -t rsa -b 4096 -C "youremail@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/yourname/.ssh/id_rsa):
/Users/yourname/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/yourname/.ssh/id_rsa.
Your public key has been saved in /Users/yourname/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:m3Hu1iJ9s3ddNR3ffFgfRIXuzF1ACGl9e/6aTdee45Ku6ffKiLVE youremail@gmail.com
The key's randomart image is:
+---[RSA 4096]----+
|         . ..=o*.|
|          + o 2B o|
|         . . o1 =.|
|             E1..*|
|        S ...3 ..B|
|         *o....==|
|        oo.+.。.++ |
|        ..=.=o.o |
|         o.*=+=. |
+----[SHA256]-----+
user@yourmac:~/.ssh$ cat id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCmjA6rcFaSl6FemRODfD+oaHC4xU79odYySHi53XzYXjPYK1LcvlgZjylpgbwerQEqfobVOa4VhKPO91pa43Ll3iMyCxqkrfycvaPNdZFRxtuc59jsdfsdfXHLnt2uYMp3Twqewqh3Wsdfsdf44+wUS/dph5jOm6ItQYc9aBhxjBLSld42W+lveZnqGyaaq6Qzmsa89/J1KJlIdYZbMx+6DQ5u99JAevK7JpTdsDZu1yvpvqcAVQlgGGfgfghIliVTfKeDVR26MuHOdMwXnv+PhPpsdfsfJsfdsfZQvs7+K+dozcOjMnaew+UOGifp5oy6ySrk3Sw+okpc2c2DszJ015OJAVy+VtpjQ6DMsjMv/gHP9Hh89ExgZqHRVBKxhumGOCwuYaiK3hKKjn9+ynJF5JdVVY/aqqXWzBXwNN57cphRqxs1vlZFo5CYUsdfsdfsdfk8SMdJXsijEPL+HGkYyxGbXGfsfsdfSi564Z4wqIQ== youremail@gmail.com
```
接着又会提示你输入两次密码（该密码是你push文件的时候要输入的密码，而不是github管理者的密码），

当然，你也可以不输入密码，直接按回车。那么push的时候就不需要输入密码，直接提交到github上了，如：
```
Enter passphrase (empty for no passphrase): 
# Enter same passphrase again:
```
接下来，就会显示如下代码提示，如：
```
Your identification has been saved in /c/Users/you/.ssh/id_rsa.
# Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
# The key fingerprint is:
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```

4.添加你的 SSH key 到 github上面去
首先你需要拷贝 id_rsa.pub 文件的内容，你可以用编辑器打开文件复制，也可以用git命令复制该文件的内容，如：
```
$ clip < ~/.ssh/id_rsa.pub
```
还记得刚才那幅图吗，我们把SSH key加到上面去：
![image.png](https://upload-images.jianshu.io/upload_images/8407639-e86502b0e63012ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.测试SSH有没有成功
```
$ ssh -T git@github.com (替换成你的)
```
可以看到警告代码
```
The authenticity of host 'github.com (207.97.227.239)' can't be established.
# RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
# Are you sure you want to continue connecting (yes/no)?
```
不管他，直接yes。
```
Hi username! You've successfully authenticated, but GitHub does not
# provide shell access.
```
看到上面的文字就说明你成功啦！

6.在XCode上面配置我的GitHub。
![image.png](https://upload-images.jianshu.io/upload_images/8407639-c6cf6b3ca04cbb9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

添加自己的账户信息和Repository之后，就可以创建我们的GitHub仓库啦！

我们commit一下
![image.png](https://upload-images.jianshu.io/upload_images/8407639-e6d4bb10f2230099.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再push一下
![image.png](https://upload-images.jianshu.io/upload_images/8407639-8e8153649114660b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大功告成！（昨天的坑算是补了，溜了，今天开始我的新征程）