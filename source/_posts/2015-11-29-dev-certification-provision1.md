---
layout: post
title: "Certificate与provision profiles"
date: 2015-11-29 20:04:52 +0800
comments: true
categories: provision profiles
---
作为需要将应用上传到App Store的开发者，我们肯定会遇到一个有意思的问题，那就是申请AppId和开发者证书。

开发者证书主要有两种类型。Developer Program 和Enterprise Program 。前一种就是99美金的开发者证书，而后一种就是299美金的开发者证书，具体的区别请参看下图：
![Alt text](/images/iOS.png)
<!--more-->

其中有一个AdHoc的发布方式，认为可以将应用发布到100个设备上面，但是必须将这100个设备的UUID也放到证书当中，否则设备是无法运行应用的。
而In House的发布方式，则非常的简单，可以不需要uuid就可以装在设备上。
申请开发者证书 步骤如下：
1. 通过自己的开发者账号进入到个人后台，前提是你已经购买了99美金或者299美金的开发者资格。
2. 进入申请证书界面。
3. 选择Certificates
![Alt text](/images/1447855210829.png)。
下面会要求你上传一个签名，此签名作用非常的大，后面我们在说。生成签名的方式也很简单，因为只要你用苹果电脑开发，就会在keychain中申请一个签名。
4. 提交了自己生产的CSR( Certificate Signing Request )文件。就会生成一个p12文件。需要将其下载下来。
5. 接下来生成mobileprovision文件。
![Alt text](/images/1447987845690.png)
上面我们看到有identifier和Devices，暂时可以不用设置，我们生成mobileprovision的时候会要求我们设置的。
按照苹果的要求填写，最后会生成一个mobileprovision文件。同样也要下载下来。

接下来就要将其和我们的xcode结合在一起。
1. 双击p12文件安装。他会自动安装到mac电脑的keychain中。用于验证我们的身份。
2. 双击mobileprovision文件。他会自动的安装到~/Library/MobileDevice/Provisioning\ Profiles下面。
也可以通过xcode的账户同步功能，将开发者账号下面的mobileprovision同步下来。
![Alt text](/images/1447988460200.png)

这样做了之后，我们点击自己工程的的build setting，在其下面的code signing就可以看到mobileprovision
![Alt text](/images/1447988655673.png)
点击选择我们自己的mobileprovision即可！
