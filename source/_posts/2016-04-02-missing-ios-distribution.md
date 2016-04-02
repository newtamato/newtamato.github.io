---
layout: post
title: "Xcode 7 error: “Missing iOS Distribution signing identity for …”"
date: 2016-04-02 14:25:05 +0800
comments: true
categories: xcode WWDR
---

最近准备重新打包一次我们的游戏，提交苹果，但是却遇到如下的问题。
![Alt text](/images/error.png)
这是个怎么回事？难道我的证书出问题了。
<!--more-->
于是到开发者member center去检查证书的过期时间。没问题啊。并且在xcode里面证书和签名文件都是能够对应的上的。keychain中文件也都是存在的。
这就奇怪了？

然后赶紧Google一下。没想到手气好到爆，在stack overflow上立马知道了答案。
![Alt text](/images/answer.png)
附上[地址 ](http://stackoverflow.com/questions/32821189/xcode-7-error-missing-ios-distribution-signing-identity-for)，请笑纳。
翻译一下就是WWDR这个中间第三方的证书文件已经过期了，我们要做的就是把它下载下来，然后重新安装，他会自动放到keychain中。但是原先keychain中的文件需要删掉。怎么删掉呢？就是打开keychain->System, 在View里面选择Show Expired Certificates,找到那个已经过期的证书，删掉它即可。

这样之后，我们就需要重新编译ipa。不能使用已经编译过的。

那么这个WWDR是做什么用的呢？
因为我们在创建自己开发者证书的时候，一般需要先上传一个自己的Cetefication。这个证书的有效性是谁保证的呢？就是这个WWDR来做的。所以说他相当于验证了我们的身份。因此非常的重要。

