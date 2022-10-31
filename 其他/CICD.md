> 为luoyang项目添加cicd流程

### 名词简介

1. ci：持续产出资源
2. cd：持续整合资源后打包发布

### 操作步骤

----------

####  创建CI目录结构

目录中包含以下文件

1. Jenkinsfile：定义流水线流程
2. Makefile：对应jenkinsfile中的各个步骤
3. pipeline.py：实现复杂业务逻辑
4. requirements.txt：下载第三方资源库
5. init.py：初始化

----

#### 在gitlab中添加webhook

![image-20220613152323560](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613152323560.png)

填写url等信息，

url：jenkins中创建的项目的url

secret token：Jenkins中生成，依次点击构建触发器---高级----点击Generate按钮生成

![image-20220613170627627](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613170627627.png)

勾选**Tag push events**和**Enable SSL verification**

----------

#### 在jenkins中新增加项目

新建item

![image-20220613152447705](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613152447705.png)

填写名称，选择pipeline

![image-20220613152523199](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613152523199.png)

填写对应参数

![image-20220613152823574](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613152823574.png)

添加参数，两个String Parameter

1. token：<font color = "#458920">7oJRNgs83xAC5o7Rzeue</font>

   token获取：![image-20220613155449232](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613155449232.png)

2. url: https://git.youle.game/api/v4/projects/项目id

   ![image-20220613153529767](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613153529767.png)

勾选构建触发器

![image-20220613154150077](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613154150077.png)

设置流水线

![image-20220613155117486](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613155117486.png)

![image-20220613155147996](/Users/topjoy/Library/Application Support/typora-user-images/image-20220613155147996.png)

结束

----

### 补充代码

1. Init.py

   ```
   (空)
   ```

2. Jenkinsfile

   ```json
   //定义流水线流程，在Jenkinsfile中设置必要的环境变量，
   //调用Makefile中封装好的Shell脚本，避免跨平台适配，判断逻辑等会导致Jenkinsfile变得复杂的代码。 
   pipeline {
   
       //构建节点
       agent { label 'sdk-frontend-slave' }
   
       environment {
           PYTHONPATH = "${env.WORKSPACE}/${env.module}"
       }
   
       //构建阶段
       stages {
   
           stage('Setup') {
   
               steps {
                   sh '''
                       set +x
                       env
                   '''
                   script {
                       currentBuild.displayName = "#${BUILD_NUMBER} (Tag: ${env.gitlabTargetBranch})"
                       currentBuild.description = "#${BUILD_NUMBER} (任务执行在Slave: ${env.NODE_NAME})"
                   }
               }
           }
   
           stage('Setup Environment') {
               steps {
                   sh 'make -f _continuous_integration_delivery/Makefile setup'
               }
           }
   
           stage('Build') {
               steps {
                   sh 'make -f _continuous_integration_delivery/Makefile build'
               }
           }
       }
   
       //运行结束后的操作
       post {
           cleanup {
               sh 'make -f _continuous_integration_delivery/Makefile clean'
           }
       }
   }
   
   
   ```

3. Makefile

   ```
   # 对应Jenkinsfile中的各个步骤，封装直接调用Shell命令的逻辑代码。
   # 实现跨平台Shell逻辑，但是不实现和业务相关的逻辑，避免Makefile变得复杂，难以维护。
   
   .PHONY: help build uploadrc ding
   
   help:
   	@echo "help"
   	@echo "make setup		 -- setup执行环境"
   	@echo "make build  		 -- 执行代码构建"
   	@echo "make clean        -- 清理工作空间"
   
   
   # Setup Env
   setup:
   	( \
   		source ~/.bash_profile; \
   		virtualenv -p python3 venv; \
   		source venv/bin/activate; \
   		pip install -I --trusted-host nexus.youle.game -i http://nexus.youle.game/repository/pypi-public/simple -r _continuous_integration_delivery/requirements.txt; \
   	)
   
   
   # Build
   build:
   	( \
   		source venv/bin/activate; \
   		python _continuous_integration_delivery/pipeline.py -j build; \
   	)
   
   
   clean:
   	( \
   		git reset --hard; \
   		git clean -df; \
   		rm -rf venv; \
   	)
   ```

4. Pipeline.py

   ```
   # 实现复杂业务逻辑，跨平台业务细节等。
   # encoding: utf-8
   import os
   import argparse
   import logging
   import shutil
   import subprocess
   import zipfile
   import re
   import eve.frontend.utils as utils
   from xml.etree import ElementTree as ET
   from eve.base.command_toolkit import run_external_command
   import requests
   import json
   
   logging.basicConfig(level=logging.INFO)
   logger = logging.getLogger(__name__)
   
   
   # 资源相关目录地址
   WORKSPACE = os.getenv('WORKSPACE')
   CI_OUTPUT = os.path.join(WORKSPACE, 'ci_output')
   TEMP_PATH = os.path.join(CI_OUTPUT, 'temp')
   GRADLE_HOME = '/Users/jenkins/walle/gradle-6.7.1/bin/gradle'
   ANDROID_SDK_HOME = '/Users/jenkins/walle/android-sdk-as-macosx'
   UNITY_APP_PATH = "/Applications/Unity/Hub/Editor/2019.4.32f1c1/Unity.app/Contents/MacOS/Unity"
   ANDROID_NS = 'http://schemas.android.com/apk/res/android'
   
   
   def get_root_path():
       return os.path.dirname(os.path.dirname(os.path.realpath(__file__)))
   
   
   def get_tag_name():
       tag_temp_name = os.getenv('gitlabBranch')
       _, tag_name = tag_temp_name.rsplit('/', 1)
       return tag_name
   
   
   def get_token():
       return os.getenv('token')
   
   
   def get_url():
       return os.getenv('url')
   
   
   def get_tag_name_code():
       tag_temp_name = get_tag_name()
       tag_name = tag_temp_name[-9:]
       tag_name_code = re.sub("\\D", "", tag_name)
       return tag_name_code
   
   
   def get_android_basic_path(tag_name):
       return os.path.join(CI_OUTPUT, 'Android', 'LuoyangSDK_' + tag_name)
   
   
   def get_ios_basic_path(tag_name):
       return os.path.join(CI_OUTPUT, 'iOS', 'LuoyangSDK_' + tag_name)
   
   
   def get_unity_basic_path(tag_name):
       return os.path.join(CI_OUTPUT, 'Unity', 'LuoyangSDK_' + tag_name)
   
   
   def zipDir(dirpath, out_path, root_dir_name=''):
       zip_file = zipfile.ZipFile(out_path, "w", zipfile.ZIP_DEFLATED)
       for path, dirnames, filenames in os.walk(dirpath):
           fpath = path.replace(dirpath, root_dir_name)  # 去掉目标根路径，只对目标文件夹下边的文件及文件夹进行压缩
           for filename in filenames:
               zip_file.write(os.path.join(path, filename), os.path.join(fpath, filename))
       zip_file.close()
   
   
   def resetAndroidVersionCodeAndName(decompileDir, newVersionCode, newVersionName):
       manifestFile = os.path.join(decompileDir)
       ET.register_namespace('android', ANDROID_NS)
       tree = ET.parse(manifestFile)
       root = tree.getroot()
       versionCode = '{' + ANDROID_NS + '}versionCode'
       versionName = '{' + ANDROID_NS + '}versionName'
       root.attrib[versionCode] = newVersionCode
       root.attrib[versionName] = newVersionName
       tree.write(manifestFile, encoding="utf-8", xml_declaration=True)
   
   
   def resetIOSBundleVersionAndName(decompileDir, newBundleVersion, newBundleName):
       infoPlistPath = os.path.join(decompileDir, "ZeusSDK", "Info.plist")
       run_external_command("/usr/libexec/PlistBuddy -c 'Set :CFBundleVersion " + newBundleVersion + "' " + infoPlistPath)
       run_external_command("/usr/libexec/PlistBuddy -c 'Set :CFBundleName " + newBundleName + "' " + infoPlistPath)
   
   
   def uploadsFile(privateToken, urlPath, file_path):
       headers = {
           'PRIVATE-TOKEN': privateToken
       }
       files = {'file': open(file_path, 'rb')}
       url = urlPath + "/uploads"
       try:
           response = requests.post(url, headers=headers, files=files)
           logger.info("statistic: request to {}, status code {}, response is {}.".format(
               url, response.status_code, response.content))
           return response.json()['markdown']
       except Exception as e:
           logger.info("statistic: request to {} failed, exception: {} .".format(url, e))
   
   
   def sendRelease(token, url_path, markdown_lists, tag_name):
       headers = {
           'Content-Type': 'application/json',
           'PRIVATE-TOKEN': token
       }
       description = "Android: {}<br>iOS: {}<br>Unity: {}".format(markdown_lists[0], markdown_lists[1], markdown_lists[2])
       data_dict = {"name": "Release", "tag_name": tag_name, "description": description}
       data = json.dumps(data_dict)
       url = url_path + "/releases/"
       try:
           response = requests.post(url, headers=headers, data=data)
           logger.info("statistic: request to {}, status code {}, response is {}.".format(
               url, response.status_code, response.content))
       except Exception as e:
           logger.info("statistic: request to {} failed, exception: {} .".format(url, e))
   
   
   # 项目组自己的CI流程
   def build():
       utils.create_and_clean_path(CI_OUTPUT)
       tag_name = get_tag_name()
       tag_code = get_tag_name_code()
       android_basic_path = get_android_basic_path(tag_name)
       ios_basic_path = get_ios_basic_path(tag_name)
       unity_basic_path = get_unity_basic_path(tag_name)
       buildAndroid(android_basic_path, tag_code, tag_name)
       buildIOS(ios_basic_path, tag_code, tag_name)
       buildUnity(android_basic_path, ios_basic_path, unity_basic_path, tag_name, tag_code)
       uploadPackage(android_basic_path, ios_basic_path, unity_basic_path, tag_name)
   
   
   def buildAndroid(android_basic_path, tag_code, tag_name):
       logger.info("=== build Android ===")
       android_workspace = os.path.join(WORKSPACE, 'Android', 'ZeusSDK')
       android_workspace_main = os.path.join(android_workspace, 'ZeusSDK')
       resetAndroidVersionCodeAndName(os.path.join(android_workspace_main, 'AndroidManifest.xml'), tag_code, tag_name)
       resetAndroidVersionCodeAndName(os.path.join(android_workspace_main, 'AndroidManifest_empty.xml'), tag_code, tag_name)
       zeus_aar_build_path = os.path.join(android_workspace, "ZeusSDK", "build", "outputs", "aar")
       aar_path = os.path.join(android_basic_path, 'ZeusSDK', 'libs')
       strings_default_path = os.path.join(android_basic_path, 'ZeusSDK', 'res', 'values')
       utils.create_and_clean_path(aar_path)
       utils.create_and_clean_path(strings_default_path)
   
       # 创建local.properties文件
       if os.path.exists(os.path.join(android_workspace, 'local.properties')):
           os.remove(os.path.join(android_workspace, 'local.properties'))
       with open(os.path.join(android_workspace, 'local.properties'), 'a+') as fw:
           fw.write('sdk.dir=' + ANDROID_SDK_HOME)
   
       shutil.move(os.path.join(android_workspace_main, 'AndroidManifest.xml'), os.path.join(android_workspace_main, 'AndroidManifest_temp.xml'))
       shutil.move(os.path.join(android_workspace_main, 'AndroidManifest_empty.xml'), os.path.join(android_workspace_main, 'AndroidManifest.xml'))
       shutil.move(os.path.join(android_workspace_main, 'res', 'values', 'strings-default.xml'), os.path.join(strings_default_path, 'strings-default.xml'))  # 移动strings-default.xml
   
       # 导出aar包资源
       run_external_command(GRADLE_HOME + " clean --project-dir " + android_workspace)
       run_external_command(GRADLE_HOME + " assembleRelease --project-dir " + android_workspace
                            + " --parallel --max-workers 16 --rerun-tasks")
       shutil.copy(os.path.join(zeus_aar_build_path, 'ZeusSDK-release.aar'), os.path.join(aar_path, 'ZeusSDK.aar'))
   
       # 复制Android工程目录资源到Android产出目录
       shutil.move(os.path.join(android_workspace_main, 'AndroidManifest_temp.xml'), os.path.join(android_basic_path, 'ZeusSDK', 'AndroidManifest.xml'))  # 复制AndroidManifest.xml
       shutil.copy(os.path.join(android_workspace_main, 'build.gradle'), os.path.join(android_basic_path, 'ZeusSDK'))  # 复制build.gradle
       shutil.copy(os.path.join(android_workspace_main, 'proguard-rules.pro'), os.path.join(android_basic_path, 'ZeusSDK'))  # 复制proguard-rules.pro
   
   
   def buildIOS(ios_basic_path, tag_code, tag_name):
       logger.info("=== build iOS ===")
       ios_workspace = os.path.join(WORKSPACE, 'iOS', 'ZeusSDK')
       resetIOSBundleVersionAndName(ios_workspace, tag_code, tag_name)
       framework_path = os.path.join(ios_basic_path, 'ZeusSDK', 'ZeusSDK.framework')
       utils.create_and_clean_path(TEMP_PATH)
   
       # build真机framework
       framework_build_path = os.path.join(ios_workspace, 'build')
       iphone_framework_path = framework_build_path + '/Release-iphoneos/ZeusSDK.framework'
       iphone_path = iphone_framework_path + '/ZeusSDK'
   
       subprocess.check_call("xcodebuild -project " + ios_workspace + "/ZeusSDK.xcodeproj -configuration Release" +
                             " -arch arm64 -target ZeusSDK -sdk iphoneos  clean build", shell=True)
       temp_iphone_arm64_path = os.path.join(TEMP_PATH, 'iphone_arm64')
       utils.create_and_clean_path(temp_iphone_arm64_path)
       shutil.copy(iphone_path, temp_iphone_arm64_path)
       shutil.copytree(iphone_framework_path, framework_path)
   
       subprocess.check_call("xcodebuild -project " + ios_workspace + "/ZeusSDK.xcodeproj -configuration Release" +
                             " -arch armv7 -target ZeusSDK -sdk iphoneos clean build", shell=True)
       temp_iphone_armv7_path = os.path.join(TEMP_PATH, 'iphone_armv7')
       utils.create_and_clean_path(temp_iphone_armv7_path)
       shutil.copy(iphone_path, temp_iphone_armv7_path)
   
       # 产出真机framework
       iphoneos_arm64_path = TEMP_PATH + '/iphone_arm64/ZeusSDK '
       iphoneos_armv7_path = TEMP_PATH + '/iphone_armv7/ZeusSDK '
       subprocess.check_call("lipo -create " + iphoneos_arm64_path + iphoneos_armv7_path +
                             "-output " + framework_path + "/ZeusSDK", shell=True)
   
       # 复制工程目录资源文件
       shutil.copytree(os.path.join(ios_workspace, 'ZeusSDK', 'ThirdSDK'), os.path.join(ios_basic_path, 'ZeusSDK', 'ThirdSDK'))
       shutil.move(os.path.join(ios_basic_path, 'ZeusSDK', 'ZeusSDK.framework', 'ZeusSDK.bundle'), os.path.join(ios_basic_path, 'ZeusSDK', 'ZeusSDK.bundle'))  # 复制ZeusSDK.bundle
       shutil.move(os.path.join(ios_basic_path, 'ZeusSDK', 'ZeusSDK.framework', 'TwitterKitResources.bundle'), TEMP_PATH)
       shutil.move(os.path.join(ios_basic_path, 'ZeusSDK', 'ZeusSDK.framework', 'FCLocalization.bundle'), TEMP_PATH)
       shutil.move(os.path.join(ios_basic_path, 'ZeusSDK', 'ZeusSDK.framework', 'FCResources.bundle'), TEMP_PATH)
       shutil.move(os.path.join(ios_basic_path, 'ZeusSDK', 'ZeusSDK.framework', 'FreshchatModels.bundle'), TEMP_PATH)
   
   
   def buildUnity(android_basic_path, ios_basic_path, unity_basic_path, tag_name, tag_code):
       logger.info("=== build Unity ===")
       unity_workspace = os.path.join(WORKSPACE, 'Unity', 'ZeusSDK')
   
       # 将Android产出资源复制到Unity插件Android目录中
       unity_android_basicsdk_path = os.path.join(unity_workspace, 'Assets', 'Plugins', 'Android', 'ZeusSDK')
       shutil.copy(unity_android_basicsdk_path + "/build.gradle", TEMP_PATH)  # 备份build.gradle
       shutil.copy(unity_android_basicsdk_path + "/project.properties", TEMP_PATH)  # 备份project.properties
       shutil.copy(unity_android_basicsdk_path + "/res/values/strings-default.xml", TEMP_PATH)  # 备份strings-default.xml
       shutil.rmtree(unity_android_basicsdk_path)
       shutil.copytree(os.path.join(android_basic_path, 'ZeusSDK'), unity_android_basicsdk_path)  # 复制Android_ZeusSDK目录
       shutil.copy(TEMP_PATH + "/build.gradle", unity_android_basicsdk_path)  # 复制build.gradle
       shutil.copy(TEMP_PATH + "/project.properties", unity_android_basicsdk_path)  # 复制project.properties
       shutil.copy(TEMP_PATH + "/strings-default.xml", unity_android_basicsdk_path + "/res/values/")  # 复制strings-default.xml
   
       # 备份iOS工程目录资源
       unity_ios_basicsdk_path = os.path.join(unity_workspace, 'Assets', 'Plugins', 'iOS', 'SdkLib')
       utils.create_and_clean_path(unity_ios_basicsdk_path)
       shutil.copytree(os.path.join(ios_basic_path, 'ZeusSDK'), os.path.join(unity_ios_basicsdk_path, 'ZeusSDK'))  # 复制iOS_ZeusSDK目录
       shutil.move(os.path.join(unity_ios_basicsdk_path, 'ZeusSDK', 'ThirdSDK'), unity_ios_basicsdk_path)
   
       # 导出UnityPackage
       utils.create_and_clean_path(unity_basic_path)
       run_external_command(UNITY_APP_PATH + " -quit -batchmode -projectPath " + unity_workspace
                            + " -executeMethod ExportPackage.ExportAssetsPackage " + unity_basic_path)
   
   
   def uploadPackage(android_basic_path, ios_basic_path, unity_basic_path, tag_name):
       # 删除无用文件
       shutil.rmtree(TEMP_PATH)
   
       # 将工程资源目录打成压缩包
       zipDir(android_basic_path, android_basic_path + "_Android.zip", os.path.basename(android_basic_path) + '_Android')
       zipDir(ios_basic_path, ios_basic_path + "_iOS.zip", os.path.basename(ios_basic_path) + '_iOS')
       zipDir(unity_basic_path, unity_basic_path + "_Unity.zip", os.path.basename(unity_basic_path) + '_Unity')
   
       # 删除src源文件
       shutil.rmtree(android_basic_path)
       shutil.rmtree(ios_basic_path)
       shutil.rmtree(unity_basic_path)
   
       # 上传文件
       token = get_token()
       url_path = get_url()
       file_path_lists = [android_basic_path + "_Android.zip",
                          ios_basic_path + "_iOS.zip",
                          unity_basic_path + "_Unity.zip"]
       markdown_lists = []
       for i in file_path_lists:
           markdown_path = uploadsFile(token, url_path, i)
           markdown_lists.append(markdown_path)
       sendRelease(token, url_path, markdown_lists, tag_name)
       pass
   
   
   if __name__ == '__main__':
       parser = argparse.ArgumentParser(description="ZeusSDK CI pipeline -- script")
       parser.add_argument('-j', '--job', choices=['build'])
       args = parser.parse_args()
       if args.job == 'build':
           print('build job start')
           build()
   
   ```

   