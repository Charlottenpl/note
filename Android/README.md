## ZEUS SDK

​	ZEUS SDK 是一套针对海外发行业务定制的SDK项目，通过提供账号，支付，客服，推送，第三方平台和数据分析等服务，帮助项目快速实现相应的功能。





---

[TOC]



## 项目结构

项目主要分为4部分

 - _continuous_integration_delivery

   ci/cd脚本目录

 - Android

   ZeusSDK的Android端实现以及demo工程

 - iOS

   ZeusSDK的iOS端实现以及demo工程

 - unity

   ZeusSDK的unity demo工程，除了和Android和iOS通信的代码，本身没有功能实现，接口分别由Android端和iOS端实现。

 - README.md

   当前文件



## 开发环境搭建

### Android

#### 0. 参考开发环境

Android studio：Android Studio Bumblebee | 2021.1.1 Patch 2

测试机：

	- 建议使用真机测试，使用虚拟机可能会
	- 安装谷歌三件套
	- HarmonyOS目前暂时没有测试出问题，不排除以后会更新出问题。

网络：能够访问外网

#### 1. clone项目到本地

​	Git clone https://git.youle.game/TC/TAG/frontend/ZeusSDK.git

#### 2. 通过Android studio运行项目

​	使用数据线连接电脑或者Android11以上设备可以在开发者选项中使用Wi-Fi调试进行连接。

​	等待项目编译结束，运行。

​	运行成功。

#### 3. 不能被混淆的文件

​	有些项目使用QuickSDK进行集成，需要将一些文件从混淆文件中排除，以便QuickSKD通过反射获取到这个文件。

<details>
<summary>以下文件不能被混淆</summary>
  <pre>
	MCConstant
	MCFlagControl
	MCOrderInfoBean
	MCShareInfoBean
	MCCallbackBean
	ZeusSDK
	ZeusSDKAnalytics
	MCLogUtil
	MCToastUtil
	callback.** 所有回调类型
	result.**
	com.adjust.sdk.**{ *; }# 如果您的发布目标非 Google Play 商店，adjust相关
  </pre>
</details>



### iOS

#### 0. 参考开发环境

​	XCode：Version 13.3 (13E113)

​	测试机：一台ios设备或者使用虚拟机

#### 1. 使用XCode运行项目

​	在上一步，项目已从gitlab克隆到本地。使用xcode直接打开项目。

​	配置钥匙串。

​	连接手机，运行项目。

​	运行成功。



### unity

#### 0. 参考开发环境

unity hub：版本3.2.0-c2 (3.2.0-c2)

unity：2019.4.32f1c1

visual studio

#### 1. 运行unityDemo

​	创建个人许可证

​	打开项目

​	配置player setting

​	导出工程

​	运行

### CICD流程

新功能开发完成之后需要集成到测试项目中进行测试，这个时候可以使用gitlab新建tag触发Jenkins任务。进行自动打包发布。

首先将需要测试的分支提交到gitlab，然后创建一个tag，将本次更新的内容以markdown的形式记录到message中，并提交tag。

之后等待Jenkins任务结束，产出的内容就会自动上传到release中。

### 第三方开发者账号

#### 1. Facebook开发者后台

https://developers.facebook.com/

#### 2. Twitter开发者后台

https://developer.twitter.com/

#### 3. Firebase

https://console.firebase.google.com/

#### 4. Adjust

https://dash.adjust.com/#/

#### 5. FreshDesk

https://freshdesk.com/cn/self-service/

#### 6. Google play

https://play.google.com/console/u/0/developers





### 其他问题

#### sdk的提供形式

目前zeus提供三种方式进行集成，分别是：aar用于Android，framework用于ios集成，unitypackage用于unity集成。

































