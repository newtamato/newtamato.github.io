---
layout: post
title: "certificate与provision profiles 2"
date: 2015-11-29 20:10:15 +0800
comments: true
categories: provision profiles
---

我们在创建Certificate的时候，要求先自己生成一个CSR文件。
![Alt text](/images/1448076369836.png)
<!--more-->
生成的这个CSR文件，用sublime打开之后，里面的样子如下：

```
-----BEGIN CERTIFICATE REQUEST-----
MIIByjCCATMCAQAwgYkxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlh
MRYwFAYDVQQHEw1Nb3VudGFpbiBWaWV3MRMwEQYDVQQKEwpHb29nbGUgSW5jMR8w
HQYDVQQLExZJbmZvcm1hdGlvbiBUZWNobm9sb2d5MRcwFQYDVQQDEw53d3cuZ29v
Z2xlLmNvbTCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkCgYEApZtYJCHJ4VpVXHfV
IlstQTlO4qC03hjX+ZkPyvdYd1Q4+qbAeTwXmCUKYHThVRd5aXSqlPzyIBwieMZr
WFlRQddZ1IzXAlVRDWwAo60KecqeAXnnUK+5fXoTI/UgWshre8tJ+x/TMHaQKR/J
cIWPhqaQhsJuzZbvAdGA80BLxdMCAwEAAaAAMA0GCSqGSIb3DQEBBQUAA4GBAIhl
4PvFq+e7ipARgI5ZM+GZx6mpCz44DTo0JkwfRDf+BtrsaC0q68eTf2XhYOsq4fkH
Q0uA0aVog3f5iJxCa3Hp5gxbJQ6zV6kJ0TEsuaaOhEko9sdpCoPOnRBm2i/XRD2D
6iNh8f8z0ShGsFqjDgFHyF3o+lUyj+UC6H1QW7bn
-----END CERTIFICATE REQUEST-----
```

这里面包含了什么信息呢 ？
![Alt text](/images/1448076787997.png)
[来源](https://www.sslshopper.com/what-is-a-csr-certificate-signing-request.html)

从上面的表格，我们可以看到这里面包含了我们生成它的时候填写的信息还有一个关键的就是public key。
也就是说我们在生成CSR文件的时候，还声称了一个public key。因此我们可以在keychain中检查以下是否真的有这个public key。
![Alt text](/images/1448076932966.png)
果然，在keychain中有一个钥匙对，私钥和公钥。
CSR自动包含了公钥，然后我们将这个CSR文件提交给苹果的member center。
![Alt text](/images/1448077003733.png)
此时会生成.cer文件。将此文件下载下来，双击安装。此时在keychain当中就会将我们的私钥和这个certificate绑定起来。
![Alt text](/images/1448077535385.png)


这个certificate就是验证开发者身份。

接下来的mobileprovision则是验证开发者权限的。
我们将自己生成的开发版mobileprovision打开看看。

```
<key>Entitlements</key>
  <dict>
    <key>keychain-access-groups</key>
      <array>
          <string>L23FMABA6X.*</string>
      </array>
      <key>get-task-allow</key>
      <true/>
      <key>application-identifier</key>
      <string>L23FMABA6X.*</string>
      <key>com.apple.developer.team-identifier</key>
      <string>L23FMABA6X</string>
      <key>aps-environment</key>
      <string>development</string>
  
  </dict>
	
	...

  <key>ProvisionedDevices</key>
  <array>
      <string>11f30c7340e54187e93e1d085f85574570206cd9</string>
      <string>0545f62295c9f117e2d6c19131df55444b0b8444</string>
      <string>8bcbf9be5476b09969fa597bf38f3949ed9b6e7d</string>
      <string>57a453b5e5f39618c4e09d6399efff2e2d0b59a4</string>
      <string>eff3c07516897e04c5d2aefaf91c2cf05e13a933</string>
      <string>5256d0acb2218cef2748641582c4993c0f193ed9</string>
      <string>030662def7e5fc03311f20891f0944f2b7f73251</string>
      <string>a3216d9f5e611dd1f76e45858432d9c5e945d73c</string>
      <string>cdd59a0a353c66a75a86a8419598a83a4db98e7c</string>
      <string>7001583678ac38d05b18928d049098329531d3e3</string>
      <string>8bd9c88be11675b7ef71de1d6122ba95292a3472</string>
      <string>bd63797509b861368e5a293717537b909d58c846</string>
      <string>89f4b347703111b0067b79d7d4226684d6efe6a3</string>
      <string>01798a0796437ae7eff5dddb3a442912ae57d50e</string>
      <string>6a7cf9c50f068ddcca529f589e3d70141c8f85de</string>
      <string>3c6e42454f9c551abc1738836ebf7a9513a90afa</string>
      <string>cad8d51523e6fdb4d96fc33df65376545a9a683e</string>
      <string>7b3a694a86e1d54e942ae1cbe6498eba4b2e68c6</string>
      <string>8da8d3dd98ed8a0f928003baaa79e8e0aaf432ec</string>
      <string>a262e990b8eec70ebe6bfe94e57b7a4f13ef8e9c</string>
  </array>
```

里面明确的告诉了我们aps-environment和相关的设备序列号。以及keychain-access-groups，在这里他告诉我们要使用keychain当中一个L23FMABA6X的文件。这是什么呢？可以打开keychain到Certificate当中检查一下，这就是我们刚才上面生成的! 如此一来mobileprovision就和我们的Certificate绑定在一起了。

因此一个开发版的mobileprovision包含的内容有，appID, 开发设备，以及Certificate等内容
如果我们生成的是Production版本的证书又该是什么样子呢？这个大家可以去试试看。

最后，[iDev blog](http://0xc010d.net/mobileprovision-files-structure-and-reading/)的作者提供了一个工具，用于检测证书的类型。

```
mobileprovisionParser -f <fileName> [-o <option>]
```

例：

```
mobileprovisionParser -f ~/Library/MobileDevice/Provisioning\ Profiles/a7f9bef8-f6b1-46e1-a463-2690b5781a61.mobileprovision

debug
```

```
mobileprovisionParser -f ~/Library/MobileDevice/Provisioning\ Profiles/a7f9bef8-f6b1-46e1-a463-2690b5781a61.mobileprovision -o devices

11f30c7340e54187e93e1d085f85574570206cd9
0545f62295c9f117e2d6c19131df55444b0b8444
...
8da8d3dd98ed8a0f928003baaa79e8e0aaf432ec
a262e990b8eec70ebe6bfe94e57b7a4f13ef8e9c
```

这样也能够方便的知晓证书的基本信息。

