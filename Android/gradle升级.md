> 有时候进行android打包时会提示你gradle版本太低，需要升级到指定版本，就可以用以下方法进行升级



### 下载对应版本的安装包

网址：https://gradle.org/releases/

之后找到对应版本的安装包，下载下来

![image-20220522131822155](/Users/topjoy/Library/Application Support/typora-user-images/image-20220522131822155.png)

### 卸载之前版本的gradle

~/.gradle/wrapper/dists这个位置找到gradle版本目录，删掉相应的就可以

### 解压到指定目录

下载下来之后是个压缩包，解压到对应的位置

### 配置环境变量

在~/.bash_profile中配置环境变量

### 检查版本

gradle -v

