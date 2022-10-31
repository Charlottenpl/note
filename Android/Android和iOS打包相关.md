[TOC]

### Android打包

Android打包提供给unity使用需要的打包格式是aar，替换的时候不能只替换aar文件，还需要同时??还需要同时啥？？

* android打包需要使用的命令为gradle assembleRelease
* 如果提示gradle版本低于最小版本需要升级
* 首先打开终端，进入项目对应的目录下，运行gradle assembleRelease命令行
* 执行成功之后到build/output/aar目录中即可找到对应的aar



### iOS打包

1. 首先打开Xcode进行product->build，ios build之后保存的路径并不在项目里，而是在Xcode配置中指定的一个目录中。

   ![image-20220321103457914](/Users/topjoy/Library/Application Support/typora-user-images/image-20220321103457914.png)

   从这个目录中找到release目录，下面的Basic.framework文件就是unity项目需要的

   ![image-20220321103532924](/Users/topjoy/Library/Application Support/typora-user-images/image-20220321103532924.png)

   > 打包时会选择对应的设备，会根据设备的指令集导出，可以用`lipo - info 目录`查看支持那些指令集，一般会打包多个平台的framework，然后用命令合并成一个使用，具体命令忘记了，，

   将上面的目录中的文件复制到unity对应目录下进行替换。替换前删掉所有旧的文件不然会不生效。替换目录：`/git/basicsdk/Unity/BasicSDK/Assets/Plugins/ios`

   

### unity导出IOS项目

1. File->BuildSetting

2. 把导出类型改成ios，后面带有unity小图标的表示当前导出的项目类型，打开ios选项，点switch Platform切换到ios

   ![image-20220321104839507](/Users/topjoy/Library/Application Support/typora-user-images/image-20220321104839507.png)

3. 切换完成之后选build，选择导出目录，然后到对应目录用Xcode打开并运行到手机上。