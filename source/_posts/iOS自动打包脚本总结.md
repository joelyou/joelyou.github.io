---
title: iOS自动打包脚本总结
date: 2018-03-29 18:11:29
tags: 自动打包
---

# iOS自动打包脚本总结
今天着重看了下ios自动打包的东西，主要是用shell打包，这方面的资料也不少，但是那都是别人的，今天对自己的项目用了下，成功了,算是熟悉了下流程了；

> 首先得有点基本知识

> * 得知道一些基本的linux命令和shell语法；
> * 得知道xcodebuild命令 
> * PlistBuddy的知识

## 归档
用xcodebuild archive命令来进行归档，生成一个.xcarchive文件

```
xcodebuild archive -workspace ${workspace_name}.xcworkspace GCC_PREPROCESSOR_DEFINITIONS="${GCC_PREPROCESSOR_DEFINITIONS}" -scheme ${scheme_name} -configuration Release -archivePath ${archive_path}
```
GCC_PREPROCESSOR_DEFINITIONS是指的编译环境，这里会弄一个变量来控制是正式服还是测试服；

## 导出
用xcodebuild -exportArchive 命令将上面生成的.xcarchive文件导出ipa

```
xcodebuild -exportArchive -archivePath ${archive_path} -exportOptionsPlist ${export_options_plist_path}/AppDevExportOptions.plist -exportPath ${ipa_temp_path}
```
为了区分包的不同，最后还会用mv命令移出来重命名
`mv ${ipa_temp_path}/${ipa_temp_name} ${ipa_dev_path}`

## 上传到svn服务器和fir
命令很简单，主要是配置好svn地址,fir我暂时没去弄了
`svn import ${ipa_dev_path} ${svn_ipa_path} -m "${ipa_name}"
fir publish ${ipa_dev_path} -c "测试服 PetSaasClient_dev_${ipa_name}"
`
## 流程
整理出我自己的流程如下：

1. 我桌面上有个放脚本和配置文件的文件夹
![](/img/15223149870398.jpg)
2. 桌面上建个project文件夹，然后里面再建一个PetSaasClient文件夹,再在里面放了一个exportOptionsPlist文件夹和ipa文件夹
![](/img/15223151877379.jpg)

exportOptionsPlist里的文件是需要我们配置的，最主要的有个证书的teamID，这里我们有dev，hoc，dis 三个plist文件
![](/img/15223152773664.jpg)
dev文件的内容如下：
![](/img/15223153705959.jpg)
teamID怎么知道呢？ 我这里是通过xcode去找到的，在build setting里查看：
![](/img/15223154694601.jpg)
然后在ipa文件夹里新建了四个文件夹，
![](/img/15223155384901.jpg)
目的看脚本就能知道，每次打包到ipa_temp目下，然后通过mv命令移动到具体哪种包里，这样以后包多了好管理；（这里只要建后面三个文件夹就行，因为脚本里打包时会自动生成ipa_temp文件夹）

脚本执行下来：
![](/img/15223157715906.jpg)

打包成功了，然后也上传到了svn，但是fir没配置，所以fir没成功。
![](/img/15223160969415.jpg)
最下面那个是我上传上去的，哈，成功了！

最后，脚本里有段代码：

```
if test $production_environment != "release"
then
production_environment="debug"
GCC_PREPROCESSOR_DEFINITIONS="COCOAPODS=1 SD_WEBP=1"

build_num_key="build_debug_num"
build_date_key="build_debug_date"

else
GCC_PREPROCESSOR_DEFINITIONS="SAAS_PRODUCTION_ENVIRONMENT=1 COCOAPODS=1 SD_WEBP=1"

build_num_key="build_num"
build_date_key="build_date"

fi
```
这里的SAAS_PRODUCTION_ENVIRONMENT主要用来控制是否是正式服，这样脚本打包的时候不用去改代码了，通过我们脚本传的参数就可以自动判断是否正式服
但是代码里得先做条件编译的宏判断
![](/img/15223164259125.jpg)

这样，当你脚本有SAAS_PRODUCTION_ENVIRONMENT时，然后编译代码时，我们就会知道是正式服还是测试服了；
