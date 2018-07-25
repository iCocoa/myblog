---
title: iOS 远程打包脚本制作
date: 2018-07-23 20:00:50
tags: [iOS,Python]
---

在 iOS 开发中，一般打发布包都是在本地打包，也就是工程师在自己开发电脑上使用 Xcode 编译并导出安装包来进行发布，为了提高效率可能会制作一些自动化打包脚本。本文聊的是远程打包的内容，通过资源拷贝及参数替换然后编译完成打包。

由于 HTML5 跨平台的特点，很多技术团队考虑到代码复用，在部分模块中会采用 h5 来描述界面。甚至有些不需要太复杂交互的 app，全部界面采用 h5 来编写，也就是一个 web 工程。对于大部分现有的 web 工程，能打包成 app 就已经满足了业务诉求。DCloud 团队开发的 HBuilder（IDE）工具中提供了云打包的功能，用起来很方便，简单的说，就是把 web 工程上传到云打包服务器，最后打包生成 app，点击下载即可安装使用。

<img src="http://pbt8li0dz.sabkt.gdipper.com/dcloud_pack_policy.png"  style="width:420px; height:350px; align:left"/>

由于笔者经手的业务比较敏感，领导不让把代码上传到云上去打包，所以，只能自己倒腾了。

按照 HBuilder 提供的云打包功能，先定一个初步的需求：

* 支持修改应用 id、版本号 、icon、启动图
* 支持导入签名文件

开工！！！

## 准备工作

首先，需要一台安装了 MacOS 的电脑（当做服务器使用）。

*笔者向领导申请购买的时候，领导问我知不知道黑苹果，我回答知道，然后领导就让我试试，从此踏上不归路，最终生产使用的是黑苹果。
（PS：不建议使用黑苹果，好多坑，当时弄到宁愿自己掏钱捐一台 Mac。如果领导不记得有黑苹果这东西，千万不要提。）如果不幸入了黑苹果的坑，欢迎留言交流。*

> 物理机 windows7，内存 4G；虚拟机 MacOS，内存 3G。

其次，在服务器上部署一个 web 服务，提供打包交互界面方便客户端上传资源文件及下载安装包。我们的界面只提供了一个 `www` zip 包的上传入口，所有应用资源及打包相关的配置文件都在里面。www 目录结构如下：

<img src="http://pbt8li0dz.sabkt.gdipper.com/www_directory.png"  style="width:300px; height:200px; align:left"/>

### appConfig.json 文件内容

```
{
	"id":"com.domain.pack",
	"appName":"我的应用",
	"debug":true,
	"launchPath": "index.html",
	"version": {
		"name": "1.0.0",
		"code": "100"
	},
	...
}
```

`launchPath` 对应 web 应用入口文件，iOS 工程使用这个文件路径作为 webview 的加载入口。

### secret.json 文件内容

```
{
	"ios" : {
		"p12Password" : "123456"
	},
	"android" : {
		"keyAlias" : "keyAlias",
		"keyPassword" : "123456",
		"storePassword" : "123456",
		"amapApiKey" : "",
		"jpushApiKey" : "",
		...
	}
}
```

除了交互界面外，打包服务还需要提供调起 Python 脚本的功能。

## Python 打包脚本

基本所有的功能都使用脚本实现，使用 Python 编写打包脚本是因为 Python 用起来方便，刚开始打算用 Shell 来编写，执行效果可能好一些，但是对这个不熟，只好将就用 Python。我们的 web 服务采用 Java 编写，Java 是可以调用 Python 脚本的 `ProcessBuilder pb = new ProcessBuilder(command.split(" ")); `。打包脚本事先准备好，放在 web 服务站点根目录下，在解压完 `www` zip 包之后，把脚本拷贝到与 www 目录同级目录中，然后执行脚本打包。打包脚本主要做以下几件事情：

* 下载 iOS 工程代码到指定目录
* 将客户端上传的 www 文件资源拷贝到 iOS 工程目录，应用图标、启动图等
* 修改 iOS 工程配置
* 导入证书到系统钥匙串
* 导入 mobileprovision 文件
* 编译工程
* 导出 ipa 安装包

> 打包脚本和客户端上传的 www 文件夹需要放在同一目录下。

实现难度不是很大，但是细节很多，需要反复实践尝试。脚本全部内容见文章末尾。

### 下载 iOS 工程代码到指定目录

```
svnChekoutCmd = 'svn co --username=%s --password=%s %s %s' %(SVN_USERNAME, SVN_PASSWORD, SVN_URL, checkoutPath())
p = subprocess.Popen(svnChekoutCmd, shell=True, stderr=subprocess.PIPE)
p.wait()
```

从 svn 仓库拉取 iOS 工程代码，使用 `svn checkout` 命令把代码拷贝到指定目录，后面会使用这个目录下的工程进行编译。

### 将客户端上传的 www 文件资源拷贝到 iOS 工程目录

```
sourceWWWDir = currentDir() + '/www'
projectWWWDir = '/packProject/www'
destinationWWWDir = checkoutPath() + projectWWWDir
copyFiles(sourceWWWDir, destinationWWWDir)
for file in os.listdir(destinationWWWDir):
    if file.startswith('secret.json') or file.endswith('.mobileprovision') or file.endswith('.p12'):
        os.remove(destinationWWWDir + '/' + file)
```

将客户端上传的 `www` 文件夹拷贝到 iOS 工程中的 www 目录下。

```
iconAssetDirectory = checkoutPath() + '/packProject/Assets.xcassets/AppIcon.appiconset'
iconSrcDirectory = projectWWWDir + '/Icons/ios'
items = os.listdir(iconSrcDirectory)
for filename in items:
    copyFile(iconSrcDirectory + '/' + filename, iconAssetDirectory + '/' + filename)
clearDir(iconSrcDirectory)
```

将 `www/Icons/ios` 文件夹中的各种尺寸的应用图标拷贝到 `Assets.xcassets/AppIcon.appiconset` 目录中。这个需要事先编写好 `AppIcon.appiconset` 中的 `Contents.json` 文件，为每种尺寸的 icon 指定文件名，这里的文件名与 `Icons/ios` 目录下的图片文件名一一对应，所以，`Icons/ios` 中的图片名称是固定不变的。`Contents.json` 文件部分内容：

```
{
  "images" : [
    {
      "idiom" : "iphone",
      "size" : "20x20",
      "filename" : "40x40.png",
      "scale" : "2x"
    },
    {
      "idiom" : "iphone",
      "size" : "20x20",
      "filename" : "60x60.png",
      "scale" : "3x"
    },
    {
      "idiom" : "iphone",
      "size" : "29x29",
      "filename" : "58x58.png",
      "scale" : "2x"
    },
    {
      "idiom" : "iphone",
      "size" : "29x29",
      "filename" : "87x87.png",
      "scale" : "3x"
    },
    {
      "idiom" : "iphone",
      "size" : "40x40",
      "filename" : "80x80.png",
      "scale" : "2x"
    },
}
```

启动图资源的拷贝跟应用图标的拷贝一样，需要事先编写好 `Contents.json` 文件，并且启动图的名称也是固定的。

### 修改 iOS 工程配置

需要根据客户端上传的配置文件 `appConfig.json` 来修改工程配置。

首先，读取配置文件的内容，包括应用 id 、名称、版本号、编译号、应用入口等。Python 读取 json 文件字符串类型的值默认会转为 unicode 编码表示，需要进行处理，笔者专门写了一个 `json_load_byteified` 函数来处理这个问题。

其次，使用从配置文件中获取到的内容来修改 `info.plist` 文件。这里需要使用 MacOS 系统自带的工具 `PlistBuddy` 来辅助修改。

### 导入证书到系统钥匙串

```
p12FilePath = findFileInDirectory('.p12', sourceWWWDir)
unlockKeychainCmd = 'security unlock-keychain -p %s' %MacOS_ADMIN_PASSWORD
p = subprocess.Popen(unlockKeychainCmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
p.wait()
if p.returncode != 0:
    print p.stderr.read()
    return
importCertCmd = 'security import %s -P %s -T /usr/bin/codesign' % (p12FilePath, p12Password)
p = subprocess.Popen(importCertCmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
p.wait()
if p.returncode != 0:
    print p.stderr.read()
```

使用系统 `security` 工具将 p12 文件导入到系统钥匙串中，先打开系统钥匙串并提供系统管理员密码，然后再导入。

> 证书和私钥需要客户端事先准备好，并导出为 p12 文件一并放入 www 文件夹中上传（如何导出 p12 文件请自行查看官方文档）。p12 文件的密码规定写在 `secret.json` 文件中。

### 导入 mobileprovision 文件

```
provisionFileExtension = '.mobileprovision'
provisionFilePath = findFileInDirectory(provisionFileExtension, sourceWWWDir)
if not len(provisionFilePath) > 0:
    print ("[packageFailed]: Not found \'%s\' file in \'www\' directory.") %(provisionFileExtension)
    return
teamIdentifier = getMobileProvisionItem(provisionFilePath, 'TeamIdentifier') #MNVGPV2SFZ
provisionUUID = getMobileProvisionItem(provisionFilePath, 'UUID')
provisionName = getMobileProvisionItem(provisionFilePath, 'Name')
# type – prints mobileprovision profile type (debug, ad-hoc, enterprise, appstore)
provisionType = getMobileProvisionItem(provisionFilePath, 'type')
teamName = getMobileProvisionItem(provisionFilePath, 'TeamName')
desProvisionFilePath = PROVISONING_PROFILE_DIRECTORY + provisionUUID + provisionFileExtension
copyFile(provisionFilePath, desProvisionFilePath)
```

读取 `.mobileprovision` 文件的信息，并将 uuid 作为它的文件名保存到 `/Users/%s/Library/MobileDevice/Provisioning Profiles/` 目录，完成导入。如果先前已经导入过该类文件（一般双击文件导入），打开这个目录可以看到，文件名都是 uuid。这里，除了 uuid 之外，还可以读取团队 id、名称以及文件类型（debug, ad-hoc, enterprise, appstore）等信息。

为了方便读取 `.mobileprovision` 文件信息，这里使用一个第三方命令行小工具。安装命令如下：

`curl https://raw.githubusercontent.com/0xc010d/mobileprovision-read/master/main.m | clang -framework Foundation -framework Security -o /usr/local/bin/mobileprovision-read -x objective-c - `

安装命令会使用 curl 工具下载源码，然后使用 clang 编译并将可执行文件输出到 `/usr/local/bin/` 目录，命名为 `mobileprovision-read`，用法：

`mobileprovision-read -f fileName [-o option]`

该工具实现比较简单，使用 `security` 库解析 `mobileprovision` 文件，然后根据命令行输入的 option 选择输出结果，因为笔者没有对源码进行修改，所以需要对输出结果中的控制字符 `\n` 进行处理（`removeControlChars` 函数的作用）。

### 编译工程

编译源码。以前在苹果线上开发者文档可以查看 `xcodebuild` 用法，不知道什么时候删掉了，现在只能使用 `man xcodebuild` 查看 `xcodebuild` 用法，这个不多说。需要注意的是，刚才只是导入了 `.mobileprovision` 文件，工程配置并没有修改，所以没有关联起来。在 `project.pbxproj` 文件中有以下几个字段需要进行替换，替换完之后才算完成整个工程编译变量的配置。

```
PRODUCT_BUNDLE_IDENTIFIER
PROVISIONING_PROFILE_SPECIFIER
PROVISIONING_PROFILE
```

可以在命令行传入这几个编译变量完成替换，命令行中传入的编译变量优先级最高。

*`project.pbxproj` 不是常见的文件格式，在不知道 xcodebuild 可以注入编译变量之前，找了一圈发现没有方便的工具可以用来编辑。有人建议先转成 json 然后再使用 json 编辑工具进行修改。笔者没有采纳，笔者想到用 sed，但 sed 只对简单的文本内容有效，这种嵌套层级太多的内容貌似匹配不了，所以，无法进行修改。awk 应该可以，但这个我没有尝试。*

### 导出 ipa 安装包

创建 `exportOptions.plist` 文件并导出 `.ipa ` 安装包。把生成的 `.ipa` 文件路径输出给 java 进程，java 进程将结果显示在界面上，方便客户端进行下载。

> 注意：
> Python 脚本没有执行权限，需要使用 [Chmod](https://zh.wikipedia.org/wiki/Chmod) 命令添加执行权限。

脚本全部内容如下：

```
#!/usr/bin/env python
# _*_ coding:utf-8 _*_

import subprocess
import os
import json
import re

SVN_USERNAME = 'Hansen'
SVN_PASSWORD = '123456'
SVN_URL = 'https://Hansen@svn.domain.com/svn/****/trunk/iOS/packProject'
CHECKOUT_FOLDER = 'ios_source_code'
MacOS_ADMIN_USER = 'packrobot'
MacOS_ADMIN_PASSWORD = '123456'
EXPORT_MAIN_DIRECTORY = "/Users/%s/Documents/ios_appArchive/" % MacOS_ADMIN_USER
PROVISONING_PROFILE_DIRECTORY = "/Users/%s/Library/MobileDevice/Provisioning Profiles/" % MacOS_ADMIN_USER

def json_load_byteified(file_handle):
    return _byteify(
        json.load(file_handle, object_hook=_byteify),
        ignore_dicts=True
    )

def json_loads_byteified(json_text):
    return _byteify(
        json.loads(json_text, object_hook=_byteify),
        ignore_dicts=True
    )

def _byteify(data, ignore_dicts = False):
    # if this is a unicode string, return its string representation
    if isinstance(data, unicode):
        return data.encode('utf-8')
    # if this is a list of values, return list of byteified values
    if isinstance(data, list):
        return [ _byteify(item, ignore_dicts=True) for item in data ]
    # if this is a dictionary, return dictionary of byteified keys and values
    # but only if we haven't already byteified it
    if isinstance(data, dict) and not ignore_dicts:
        return {
            _byteify(key, ignore_dicts=True): _byteify(value, ignore_dicts=True)
            for key, value in data.iteritems()
        }
    # if it's anything else, return it in its original form
    return data

def currentDir():
    return os.path.split(os.path.realpath(__file__))[0]

def checkoutPath():
    return currentDir() + '/' + CHECKOUT_FOLDER

def pullSvnSourceCode():
    svnChekoutCmd = 'svn co --username=%s --password=%s %s %s' %(SVN_USERNAME, SVN_PASSWORD, SVN_URL, checkoutPath())
    p = subprocess.Popen(svnChekoutCmd, shell=True, stderr=subprocess.PIPE)
    p.wait()
    if p.returncode != 0:
        print ('[packageFailed]: %s') %p.stderr.read()
    else:
        print ('Sucessfullly checkout source code at path: %s') %(checkoutPath())

def clearDir(Dir):
    cleanCmd = "rm -r %s" %(Dir)
    process = subprocess.Popen(cleanCmd, shell=True)
    (stdoutdata, stderrdata) = process.communicate()

def getAppConfig():
    projectWWWDir = 'packProject/www'
    destinationWWWDir = currentDir() + '/' + CHECKOUT_FOLDER + '/' + projectWWWDir;
    appConfigFilePath = destinationWWWDir + '/appConfig.json'
    if os.path.exists(appConfigFilePath):
        appConfigReader = open(appConfigFilePath, 'r')
        appConfig = json_load_byteified(appConfigReader)
        appConfigReader.close()
        return appConfig
    return None


def copyFiles(sourceDir, destinationDir):
    if not os.path.exists(sourceDir):
        print ('[packageFailed]: Copy file -- sourceDir doesn\'t exist ')
        pass

    clearDir(destinationDir)
    for file in os.listdir(sourceDir):
        sourceFile = os.path.join(sourceDir, file)
        destinationFile = os.path.join(destinationDir, file)
        if os.path.isfile(sourceFile):
            if not os.path.exists(destinationDir):
                os.makedirs(destinationDir)
            if not os.path.exists(destinationFile) or (os.path.exists(destinationFile) and (os.path.getsize(destinationFile) != os.path.getsize(sourceFile))):
                open(destinationFile, "wb").write(open(sourceFile, "rb").read())
        if os.path.isdir(sourceFile):
            copyFiles(sourceFile, destinationFile)
    print ('Copy assets success!')

def copyFile(srcFile, dstFile):
    srcReader = open(srcFile, "rb")
    desWriter = open(dstFile, "wb")
    desWriter.write(srcReader.read())
    srcReader.close()
    desWriter.close()

def cleanArchiveFile(archiveFile):
    cleanCmd = "rm -r %s" %(archiveFile)
    process = subprocess.Popen(cleanCmd, shell=True)
    (stdoutdata, stderrdata) = process.communicate()

def buildExportDirectory():
    dateCmd = 'date "+%Y-%m-%d_%H-%M-%S"'
    process = subprocess.Popen(dateCmd, stdout=subprocess.PIPE, shell=True)
    (stdoutdata, stderrdata) = process.communicate()
    exportDirectory = "%s%s" %(EXPORT_MAIN_DIRECTORY, stdoutdata.strip())
    return exportDirectory

def getMobileProvisionItem(filepath, key):
    cmd = 'mobileprovision-read -f %s -o %s' %(filepath ,key)
    p = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE)
    p.wait()
    return removeControlChars(p.stdout.read())

def updatePlistEntry(filePath, key, value):
    cmd = "/usr/libexec/PlistBuddy -c 'Set :%s %s' %s" % (key, value, filePath)
    p = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    p.wait()
    if p.returncode != 0:
        print p.stderr.read()

def deletePlistEntry(filePath, key):
    cmd = "/usr/libexec/PlistBuddy -c 'Delete :%s' %s" %(key, filePath)
    p = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    p.wait()
    if p.returncode != 0:
        print p.stderr.read()

def addPlistEntry(filePath, key, _type, value):
    cmd = "/usr/libexec/PlistBuddy -c 'Add :%s %s %s' %s" % (key, _type, value, filePath)
    p = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    p.wait()
    if p.returncode != 0:
        print p.stderr.read()

def findFileInDirectory(ext, dir):
    fileName = ''
    items = os.listdir(dir)
    for name in items:
        if name.endswith(ext):
            fileName = name
            break
    if not len(fileName) > 0:
        return ''
    return dir + '/' + fileName

def removeControlChars(s):
    control_chars = ''.join(map(unichr, range(0,32) + range(127,160)))
    control_char_re = re.compile('[%s]' % re.escape(control_chars))
    return control_char_re.sub('', s)

def main():

    # Pull ios project source code from svn.
    pullSvnSourceCode()

    # Copy 'www' files. 
    sourceWWWDir = currentDir() + '/www'
    projectWWWDir = '/packProject/www'
    destinationWWWDir = checkoutPath() + projectWWWDir
    copyFiles(sourceWWWDir, destinationWWWDir)
    for file in os.listdir(destinationWWWDir):
        if file.startswith('secret.json') or file.endswith('.mobileprovision') or file.endswith('.p12'):
            os.remove(destinationWWWDir + '/' + file)

    # Copy app icons.
    iconAssetDirectory = checkoutPath() + '/packProject/Assets.xcassets/AppIcon.appiconset'
    iconSrcDirectory = projectWWWDir + '/Icons/ios'
    items = os.listdir(iconSrcDirectory)
    for filename in items:
        copyFile(iconSrcDirectory + '/' + filename, iconAssetDirectory + '/' + filename)
    clearDir(iconSrcDirectory)

    # Copy launch images.
    launchImageAssetDirectory = checkoutPath() + '/packProject/Assets.xcassets/LaunchImage.launchimage'
    LaunchImageSrcDirectory = projectWWWDir + '/LaunchImages/ios'
    items = os.listdir(LaunchImageSrcDirectory)
    for filename in items:
        copyFile(LaunchImageSrcDirectory + '/' + filename, launchImageAssetDirectory + '/' + filename)
    clearDir(launchImageAssetDirectory)

    # Read 'appConfig.json' file.
    appConfig = getAppConfig()
    if appConfig is None:
        print ("[packageFailed]: Not found \'%s\' file in \'www\' directory.") % ('appConfig.json')
        return
    versionName = appConfig['version']['name']
    versionCode = int(appConfig['version']['code'])
    applicationId = appConfig['id']
    appName = appConfig['appName']
    mode = 'Debug' if appConfig['debug'] else 'Release'

    # Modify 'info.plist' file in project/workspace according to appconfig params those read from 'appConfig.json' file.
    infoPlistPath = checkoutPath() + '/packProject/' + 'info.plist'
    updatePlistEntry(infoPlistPath, 'CFBundleShortVersionString', versionName)
    updatePlistEntry(infoPlistPath, 'CFBundleVersion', versionCode)
    updatePlistEntry(infoPlistPath, 'CFBundleIdentifier', applicationId)
    updatePlistEntry(infoPlistPath, 'CFBundleDisplayName', appName)

    # Get p12 file's password.
    secretFilePath = sourceWWWDir + '/secret.json'
    if os.path.exists(secretFilePath):
        secretReader = open(secretFilePath, 'r')
        secretKeyDict = json_load_byteified(secretReader)
        secretReader.close()
    else:
        print ("[packageFailed]: Not found \'%s\' file in \'www\' directory.") % ('secret.json')
        return
    iosKeyDict = secretKeyDict['ios'] if 'ios' in secretKeyDict else None
    p12Password = iosKeyDict['p12Password'] if 'p12Password' in iosKeyDict else '123456'

    # Import p12 file into system keychain.
    p12FilePath = findFileInDirectory('.p12', sourceWWWDir)
    unlockKeychainCmd = 'security unlock-keychain -p %s' %MacOS_ADMIN_PASSWORD
    p = subprocess.Popen(unlockKeychainCmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    p.wait()
    if p.returncode != 0:
        print p.stderr.read()
        return
    importCertCmd = 'security import %s -P %s -T /usr/bin/codesign' % (p12FilePath, p12Password)
    p = subprocess.Popen(importCertCmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    p.wait()
    if p.returncode != 0:
        print p.stderr.read()

    # Read mobileprovision profile info.
    provisionFileExtension = '.mobileprovision'
    provisionFilePath = findFileInDirectory(provisionFileExtension, sourceWWWDir)
    if not len(provisionFilePath) > 0:
        print ("[packageFailed]: Not found \'%s\' file in \'www\' directory.") %(provisionFileExtension)
        return
    teamIdentifier = getMobileProvisionItem(provisionFilePath, 'TeamIdentifier') #MNxxxxx8
    provisionUUID = getMobileProvisionItem(provisionFilePath, 'UUID')
    provisionName = getMobileProvisionItem(provisionFilePath, 'Name')
    # type – prints mobileprovision profile type (debug, ad-hoc, enterprise, appstore)
    provisionType = getMobileProvisionItem(provisionFilePath, 'type')
    teamName = getMobileProvisionItem(provisionFilePath, 'TeamName')
    desProvisionFilePath = PROVISONING_PROFILE_DIRECTORY + provisionUUID + provisionFileExtension
    copyFile(provisionFilePath, desProvisionFilePath)

    # Build
    archiveName = "%s_%s.xcarchive" % (applicationId, versionName)
    archiveFilePath = currentDir() + '/' + archiveName
    xcworkspaceFilePath = findFileInDirectory('.xcworkspace', checkoutPath())
    projectSettingParams = 'PRODUCT_BUNDLE_IDENTIFIER=%s PROVISIONING_PROFILE_SPECIFIER=%s PROVISIONING_PROFILE=%s' %(applicationId, provisionName, provisionUUID)
    archiveCmd = 'xcodebuild -workspace %s -scheme %s -configuration %s archive -archivePath %s -destination generic/platform=iOS build %s' % (xcworkspaceFilePath, 'packProject', mode, archiveFilePath, projectSettingParams)
    p = subprocess.Popen(archiveCmd, shell=True, stderr=subprocess.PIPE)
    p.wait()
    if p.returncode != 0:
        print ("[packageFailed]: %s") %p.stderr.read()
        return

    # Create 'exportOptions.plist' file and export ipa.
    exportOptionsPlistFilePath = currentDir() + '/' + 'exportOptions.plist'
    addPlistEntry(exportOptionsPlistFilePath, 'provisioningProfiles', 'dict', '')
    addPlistEntry(exportOptionsPlistFilePath, 'provisioningProfiles:'+ applicationId, 'string', provisionUUID)
    addPlistEntry(exportOptionsPlistFilePath, 'teamID', 'string', teamIdentifier)
    # {app-store, ad-hoc, enterprise, development}
    method = 'development' if cmp(provisionType, 'debug') == 0 else provisionType
    method = 'app-store' if cmp(method, 'appstore') == 0 else method
    addPlistEntry(exportOptionsPlistFilePath, 'method', 'string', method)
    exportDirectory = buildExportDirectory()
    exportCmd = "xcodebuild -exportArchive -archivePath %s -exportPath %s -exportOptionsPlist %s" % (archiveFilePath, exportDirectory, exportOptionsPlistFilePath)
    p = subprocess.Popen(exportCmd, shell=True, stderr=subprocess.PIPE)
    p.wait()
    if p.returncode != 0:
        print ("[packageFailed]: %s") %p.stderr.read()
    else:
        ipaVersion = str(versionCode) if mode == 'Debug' else versionName
        ipaName = applicationId + '_' + ipaVersion + '.ipa'
        os.rename(exportDirectory + '/packProject.ipa', exportDirectory + '/' + ipaName)
        print("[packageName]: %s") % (ipaName)
        print("[packagePath]: %s") % (exportDirectory)

    cleanArchiveFile(archiveFilePath)

    p = subprocess.Popen('security lock-keychain', shell=True)
    p.wait()

if __name__ == '__main__':
    main()

```

*脚本并不限于将 web 工程打成 app，只是刚好笔者有这样的需求。欢迎留言交流。*


