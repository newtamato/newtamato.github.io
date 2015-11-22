---
layout: post
title: "cocos2dx如果实现骨骼动画的换装"
date: 2015-11-22 17:33:52 +0800
comments: true
categories: cocos2dx骨骼动画换装
---
我的工作环境是:
quick cocos2dx 2.2.6
flash IDE dragbone官方的插件

首先我们要知道如何创建一个骨骼动画，主要涉及到两个部分，一部分是做加载资源，一部分是根据刚刚加载的资源创建骨骼动画。

<!--more-->
```
#加载资源的函数
    void addArmatureFileInfo(const char *configFilePath);

    /**
     *	@brief	Add ArmatureFileInfo, it is managed by CCArmatureDataManager.
     *			It will load data in a new thread
     */
    void addArmatureFileInfoAsync(const char *configFilePath, CCObject *target, SEL_SCHEDULE selector);

    /**
     *	@brief	Add ArmatureFileInfo, it is managed by CCArmatureDataManager.
     */
    void addArmatureFileInfo(const char *imagePath, const char *plistPath, const char *configFilePath);

    /**
     *	@brief	Add ArmatureFileInfo, it is managed by CCArmatureDataManager.
     *			It will load data in a new thread
     */
    void addArmatureFileInfoAsync(const char *imagePath, const char *plistPath, const char *configFilePath, CCObject *target, SEL_SCHEDULE selector);

```

而创建骨骼动画就很简单了。

```
bool CCArmature::init(const char *name)
```

我们看到按照加载资源的函数要求，我们一般要提供三个文件，png，plist和一个config文件。这三个文件刚好是dragonbone插件为我们生成的。
![Alt text](/images/1448177716730.png)

然后就是创建，传入一个name参数，这个那么是什么呢？这就要我们打开生成的xml文件看一下它的结构了。

```
<skeleton name="battles_effect" frameRate="24" version="2.2">
  <armatures>
    <armature name="battles_effect">
      <b name="fsa" x="17" y="-52.95" kX="0" kY="0" cX="1" cY="1" pX="150" pY="120" z="0">
        <d name="battles_effect0001" pX="0" pY="0"/>
        <d name="battles_effect0002" pX="0" pY="0"/>
      </b>
    </armature>
  </armatures>
  <animations>
    <animation name="battles_effect">
      <mov name="walk" dr="4" to="6" drTW="4" lp="0" twE="NaN">
        <b name="fsa" sc="1" dl="0">
          <f x="17" y="-52.95" cocos2d_x="-133" cocos2d_y="-172.95" kX="0" kY="0" cX="1" cY="1" pX="150" pY="120" z="0" dI="0" dr="1"/>
          <f x="17" y="-52.95" cocos2d_x="-133" cocos2d_y="-172.95" kX="0" kY="0" cX="1" cY="1" pX="150" pY="120" z="0" dI="1" dr="1"/>
        </b>
      </mov>
    </animation>
  </animations>
  <TextureAtlas name="battles_effect" width="1024" height="512">
    <SubTexture name="battles_effect0004" width="300" height="240" cocos2d_pX="0" cocos2d_pY="0" x="604" y="0"/>
    <SubTexture name="battles_effect0003" width="300" height="240" cocos2d_pX="0" cocos2d_pY="0" x="0" y="242"/>
  </TextureAtlas>
</skeleton>
```

这是一个简单地骨骼动画文件。它分为三个部分，第一个部分是<armatures/>，表示有几个骨骼小人，他的名字就是我们要创建骨骼小人是用到的参数,第二个部分就是<animations/>他表示上面对应骨骼小人的动画，位置，缩放，关键帧事件等信息，它里面每一个mov表示小人能做的动作，比如walk，表示小人能播放走路的动画。第三个部分<TextureAtlas/>表示对应的骨骼小人用到的图片资源。

然后，我们将生成的素材资源放到xcode的工程的Resource目录下面，通过上面提供的函数和方法就可以创建出来一个简单的骨骼小人了。

在cocoschina的官方论坛上关于此部分有详细的讲解[直达论坛](http://www.cocoachina.com/bbs/read.php?tid=158607)

我们今天要讲的就是如果让小人换装。但是保持的动作不变。也就是上面提到了三个文件，只需要变化一个文件即可！
既然要变装，那就是改变显示的样子，只要变化png那个文件即可。这样从应用的size角度考虑，我们能够节省不少的资源。

也就是说动画信息文件不变，也就是里面要求的SubTexture也不会变化，那么怎么做到换装呢？能够想到的就是在解析这个xml文件的时候对其进行修改，到指定的png中去获取对应的图片。而我们在传入参数的时候是不是只要传入新的png地址就可以了呢？理论上是这样的，但是实际做起来又不是这样子。
我们先看加载资源的时候cocos2dx做了什么事情？

```
void CCArmatureDataManager::addArmatureFileInfo(const char *imagePath, const char *plistPath, const char *configFilePath)
{
    addRelativeData(configFilePath);

    m_bAutoLoadSpriteFile = false;
    CCDataReaderHelper::sharedDataReaderHelper()->addDataFromFile(configFilePath);

    addSpriteFrameFromFile(plistPath, imagePath, configFilePath);
}
```

addRelativeData是以configFilePath为key去检查是否有缓存一个CCRelativeData的数据，如果没有则创建一个。然后CCDataReaderHelper去加载configFilePath文件。
addSpriteFrameFromFile则是根据plist中的配置去切割png图片。

然后再CCDataReaderHelper中我们继续往下看，发现她使用如下命令`CCDataReaderHelper::addDataFromCache(load_str.c_str(), &dataInfo);`会处理xml文件。
而在这个函数里面则按照我们上面说得讲xml文件分成了三部分进行解析。
并且将解析出来对应的CCArmatureData，CCAnimationData，CCTextureData都存到了以上面提到的CCRelativeData数据中。

而addSpriteFrameFromFile最终调用的就是

```
CCSpriteFrameCache::sharedSpriteFrameCache()->addSpriteFramesWithFile(plistPath, imagePath);
```
这样一来我们加载的图片资源都要存到了CCSpriteFrameCache的一个大的表`m_pSpriteFrames`中。
问题来了，`m_pSpriteFrames`里是图片的名字为查找索引的，而我们没有改变plist文件，那么也就是两张图片最终会再这里被重叠覆盖。因此也就达不到换装的目的。
那么关键的一步就是让`m_pSpriteFrames`里面的图片名字发生变化，即我们读到了plist，但不完全plist里面记录的名字为准，还需要我们自己的变化。
由于这里面名字发生了变化，也就是上面那个xml里面SubTexture对应的名字也要发生变化，armature下面d
标签对应的名字也要发生变化。
这样我们只需要在`CCDataReaderHelper::addDataFromCache(load_str.c_str(), &dataInfo);`函数里面对上面提到的两处进行处理即可。参数dataInfo就是下面提到的DataInfo结构体。

我们只需要需要传入一个参数，用于区分是否是加载换装资源。

CCSpriteFrameCache里面函数

```
void addSpriteFrameFromFile(const char *plistPath, const char *imagePath, const char *configFilePath = "",const char *prefix = "");
```

CCArmatureDataManager里面函数处理

```
void addArmatureFileInfo(const char *imagePath, const char *plistPath, const char *configFilePath, const char* prefix= "");
```

DataInfo结构体增加一个prefix

```
typedef struct _DataInfo
{
    AsyncStruct *asyncStruct;
    std::queue<std::string>      configFileQueue;
    float contentScale;
    std::string    filename;
    std::string    baseFilePath;
    std::string    prefix;
    float flashToolVersion;
    float cocoStudioVersion;
} DataInfo;
```

讲到这里，大家是否心里已经对换装有所了解了。上面的方式是从根本上直接解决问题。当然还有另外一种方法。
比如我们只需要换一个头部资源。可以通过对骨骼小人的骨头规则命名拿到头部骨头数据CCBone。

伪代码如下：

```
local headDisplayData = CCDisplayData:create()
headDisplayData.displayType = CS_DISPLAY_SPRITE
headDisplayData.name = "new_head.png"
armature:getBone("head")->addDisplay(headDisplayData,0); 
```

但是这种方法有个问题就是如果头部骨头，在做动画中又切换了新的图片，你就会非常痛苦，图片又换回去了！
因此需要保证头部骨头不要发生变化。

大家可以根据具体的需求，比较看看那个更符合自己的要求。
