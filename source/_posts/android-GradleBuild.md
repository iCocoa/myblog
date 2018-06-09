---
title: android-GradleBuild
date: 2018-06-09 12:36:16
tags: Android
---

最近为了制作 Android 应用打包脚本，学习了一下 gradle。Gradle 构建系统语法简洁、功能强大、配置灵活，笔者只是把它当作一个构建工具来使用，基于它所提供的便利制作可以修改版本号、编译号、id 及导入证书的脚本。

对于一个项目或者一个工程，Gradle 可以定义多个构建任务，debug 和 release 是常见的两个构建任务，用户还可以根据需要自定义自己的构建任务，如测试构建任务和预发布构建任务，甚至是针对不同发布渠道的构建任务。这里只用到 debug 任务。

gradle 命令行支持传入自定义参数，并在编译过程注入这些参数。

## 修改 appid 及 版本号

* 修改 build.gradle 文件

```
android {
    compileSdkVersion 21
    buildToolsVersion '26.0.2'
    defaultConfig {
        applicationId project.hasProperty('applicationId') ? applicationId : "com.domain.myApp"
        minSdkVersion 14
        targetSdkVersion 21
        versionCode project.hasProperty('versionCode') ? versionCode.toInteger() : 100
        versionName project.hasProperty('versionName') ? versionName : "1.0.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
```

* 命令行中传入对应 key 的参数

```
gradle assembleDebug -PversionCode="200" -PversionName="2.0.0" -PapplicationId="com.domain.myApp.debug"
```

修改应用 id 的最好同时修改包名，不然会有包名冲突，修改包名需要修改 `AndroidManifest.xml` 文件，先在 `build.gradle` 文件中使用 `manifestPlaceholders ` 属性定义一个键：

```
// 获取应用 id
def getApplicationId = { ->
    def appId = project.hasProperty('applicationId') ? applicationId : "com.domain.myApp"
    return appId
}

manifestPlaceholders = [
                PACKAGE_NAME: "${getApplicationId()}"
                ]
```
然后在 `AndroidManifest.xml` 文件中以 `${PACKAGE_NAME}` 的方式引用，gradle 会在构建过程中把这个值给替换掉([Inject build variables into the manifest](https://developer.android.com/studio/build/manifest-build-variables))。应用名称的修改也是按照这种方式。

## 导入 keystore 文件及密码

也可以使用类似上面的方式导入签名文件及密码：

```
signingConfigs {
           debug {
               storeFile file('keystores/debug.keystore')
           }
           release {
               keyAlias project.hasProperty('keyAlias') ? keyAlias : 'mykey'
               keyPassword project.hasProperty('keyPassword') ? keyPassword : '123456'
               storeFile file(project.hasProperty('storeFilePath') ? storeFilePath : 'keystores/debug.keystore')
               storePassword project.hasProperty('storePassword') ? storePassword : '123456'
           }
       }


    buildTypes {
        debug {
            signingConfig signingConfigs.debug
            minifyEnabled false
        }

        release {
            signingConfig signingConfigs.release
```

命令行输入：
```
gradle assembleDebug -PkeyAlias="mykey" -PkeyPassword="123456" -PstoreFilePath="Users/hack/Documents/keys/mykey.keystore" -PstorePassword="654321"
```

另外，导入 `keystore` 文件，还可以使用官方定义的的属性：

```
android.injected.signing.store.file
android.injected.signing.store.password
android.injected.signing.key.alias
android.injected.signing.key.password
```

命令行输入：
```
gradle assembleDebug -Pandroid.injected.signing.store.file=$KEYFILE -Pandroid.injected.signing.store.password=$STORE_PASSWORD -Pandroid.injected.signing.key.alias=$KEY_ALIAS -Pandroid.injected.signing.key.password=$KEY_PASSWORD
```

