---
title: Gradle构建基础
date: 2017-07-24 15:49:12
tags: [Gradle , Android]
---
对于`Android`开发的同学来说，`Gradle`肯定不陌生，它是一种依赖管理工具，基于Groovy语言，面向Java应用为主。
<!--more-->
## 命令
### 常用命令
`gradle`命令格式一般是`./gradlew +参数`，`gradlew`代表`gradle wrapper`，意思是`gradle`的一层包装。`gradlew`脚本的路径是项目的根目录，和`settings.gradle`文件同级。

* `./gradlew build` 检查依赖并编译打包
* `./gradlew clean` 清除app目录下的build文件夹
* `./gradlew -v`    查看当前gradle版本号

编译打包除了使用`./gradlew build`，还可以用`assemble`命令
* `./gradlew assembleRelease` 编译并打Release的包
* `./gradlew assembleDebug`   编译并打Debug包

`./gradlew build`会同时将Debug和Release环境包一同打出来，而`assemble`则是取其所需。

* `./gradlew installRelease` 打Release包并安装
* `./gradlew uninstallRelease` 卸载Release包
同样的，针对Debug包也有效。

### 进阶命令
#### assemble与Build Variants结合
`assemble`还可以与`Product Flavor`（这个稍后介绍）结合创建新的任务，实际上`assemble`是和`Build Variants`一起结合使用的，而`Build Variants` = `Product Flavor` + `Build Type`，例如：
* 如果我们想打包X渠道的Release版本，执行如下命令就好了：
`./gradlew assembleXRelease`
假设渠道是Wandoujia，那么对应的命令就是
`./gradlew assembleWandoujiaRelease`

* 如果我们只打X渠道版本，则：
`./gradlew assembleX`
此命令会生成X渠道的Release和Debug版本  
继续拿Wandoujia举例子，命令对应为
`./gradlew assembleWandoujia`

* 如果现在有多个渠道，需要同时打Release包，只需执行
`./gradlew assembleRelease`
这条命令会把`Product Flavor`下的所有渠道的Release版本全部打出来

总结一下，`assemble`与Build Variants搭配起来的用法有三种：
* `./gradlew assemble<Variant Name>`：允许直接构建一个Variant版本，  
例如`./gradlew assembleFlavor1Debug`。

* `./gradlew assemble<Product Flavor Name>`：允许直接构建一个名字叫‘Variant Name’的Release版和Debug版，例如`./gradlew assembleFlavor1`。

* `./gradlew assemble<Build Type Name>`：允许构建指定Build Type的所有APK，  
例如`./gradlew  assembleDebug`将会构建Flavor1Debug和Flavor2Debug两个Variant版本。

#### 依赖检查
运行命令`./gradlew <projectname>:dependencies --configuration compile`，projectname指的是即将查询依赖的模块名字，一般新建的项目默认是`app`，具体根据个人实际情况变动。
这条命令会把依赖树会打印出来，依赖树显示了你build 脚本声明的顶级依赖和它们的传递依赖：
```shell
compile - Classpath for compiling the main sources.
+--- com.android.support:appcompat-v7:25.1.0
|    +--- com.android.support:support-annotations:25.1.0
|    +--- com.android.support:support-v4:25.1.0
|    |    +--- com.android.support:support-compat:25.1.0
|    |    |    \--- com.android.support:support-annotations:25.1.0
|    |    +--- com.android.support:support-media-compat:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    +--- com.android.support:support-core-utils:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    +--- com.android.support:support-core-ui:25.1.0
|    |    |    +--- com.android.support:support-annotations:25.1.0
|    |    |    \--- com.android.support:support-compat:25.1.0 (*)
|    |    \--- com.android.support:support-fragment:25.1.0
|    |         +--- com.android.support:support-compat:25.1.0 (*)
|    |         +--- com.android.support:support-media-compat:25.1.0 (*)
|    |         +--- com.android.support:support-core-ui:25.1.0 (*)
|    |         \--- com.android.support:support-core-utils:25.1.0 (*)
|    +--- com.android.support:support-vector-drawable:25.1.0
|    |    +--- com.android.support:support-annotations:25.1.0
|    |    \--- com.android.support:support-compat:25.1.0 (*)
|    \--- com.android.support:animated-vector-drawable:25.1.0
|         \--- com.android.support:support-vector-drawable:25.1.0 (*)
\--- project :ScheduleView
     +--- com.android.support:appcompat-v7:25.1.0 (*)
     +--- com.android.support:gridlayout-v7:25.1.0
     |    +--- com.android.support:support-compat:25.1.0 (*)
     |    \--- com.android.support:support-core-ui:25.1.0 (*)
     \--- com.android.support:support-v4:25.1.0 (*)

(*) - dependencies omitted (listed previously)
```
`(*)`表示这个依赖被忽略了，这是因为其他顶级依赖中也依赖了这个传递的依赖，Gradle会自动分析下载最合适的依赖。

## 配置
### build.gradle
这是gradle的构建脚本，所有的运行环境配置，依赖配置，构建类型配置都写在里面。
关于`build.gradle`文件，一个项目里有两种，一种是这个项目的，另一种是每个模块的。
#### 整个项目的gradle
整个项目的gradle文件一般包含以下内容
```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.3'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
可以看到，项目的build.gradle文件声明了仓库的源和Android gradle plugin的版本。
* `buildscript{}`设置脚本的运行环境。
* `repositories{}`java依赖库管理源，用于项目依赖。
* `dependencies{}`声明依赖包。支持maven/ivy，远程，本地库，也支持单文件。

#### 每个模块的gradle
每个模块的gradle文件一般包含以下内容
```groovy
apply plugin: 'com.android.application'

android {
    // 编译SDK的版本
    compileSdkVersion 26
    // build tools的版本
    buildToolsVersion "25.0.2"
    //aapt配置
    aaptOptions {
        //不用压缩的文件
        noCompress 'pak', 'dat', 'bin', 'notice'
        //打包时候要忽略的文件
        ignoreAssetsPattern "!.svn:!.git"
        //分包
        multiDexEnabled true
        //--extra-packages是为资源文件设置别名：意思是通过该应用包名+R，com.android.test1.R和com.android.test2.R都可以访问到资源
        additionalParameters '--extra-packages', 'com.android.test1','--extra-packages','com.android.test2'
    }
    defaultConfig {
        //应用的包名
        applicationId "com.xxx.xxx.xxx"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    //编译配置
    compileOptions {
    // java版本
    sourceCompatibility JavaVersion.VERSION_1_7
    targetCompatibility JavaVersion.VERSION_1_7
    }
    //源文件目录设置
    sourceSets {
        main {
             //jni lib的位置
             jniLibs.srcDirs = jniLibs.srcDirs << 'src/jniLibs'
             //定义多个资源文件夹,这种情况下，两个资源文件夹具有相同优先级，即如果一个资源在两个文件夹都声明了，合并会报错。
             res.srcDirs = ['src/main/res', 'src/main/res2']
             //指定多个源文件目录
             java.srcDirs = ['src/main/java', 'src/main/aidl']
        }
    }
    //签名配置
    signingConfigs {
        debug {
            keyAlias 'androiddebugkey'
            keyPassword 'android'
            storeFile file('keystore/debug.keystore')
            storePassword 'android'
        }
    }
    //多渠道打包
    productFlavors {
        _360 {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "_360"]
        }
        baidu {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu"]
        }
        wandoujia {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
        }
    }
    buildTypes {
        //release版本配置
        release {
            debuggable false
            // 是否进行混淆
            minifyEnabled true
            //去除没有用到的资源文件，要求minifyEnabled为true才生效
            shrinkResources true
            // 混淆文件的位置
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
            signingConfig signingConfigs.debug
            //ndk的一些相关配置，也可以放到defaultConfig里面。
            //指定要ndk需要兼容的架构(这样其他依赖包里mips,x86,arm-v8之类的so会被过滤掉)
            ndk {
                abiFilter "armeabi"
            }
        }
        //debug版本配置
        debug {
            debuggable true
            minifyEnabled false
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
            signingConfig signingConfigs.debug
            ndk {
                abiFilter "armeabi"
            }
        }
}
// lint配置
lintOptions {
  //移除lint检查的error
  abortOnError false
  //禁止掉某些lint检查
  disable 'NewApi'
}

dependencies {
    //单文件依赖
    compile files('libs/android-support-v4.jar')
    // 编译extras目录下的ShimmerAndroid模块
    // 使用transitive属性设置为false来排除所有的传递依赖，默认为true
    compile project(':extras:ShimmerAndroid'){
        transitive = false
    }
    // 编译CommonSDK模块，但是去掉此模块中对com.android.support的依赖，防止重复依赖报错
    compile (project(':CommonSDK')) { exclude group: "com.android.support" }
    //某个文件夹下面全部依赖
    provided fileTree(dir: 'src/android5/libs', include: ['*.jar'])
    //应用格式: packageName:artifactId:version
    provided 'com.android.support:support-v4:21.0.3'
    provided project(':main-host')
    //通用使用exclude排除support-compat模块的依赖
    compile ('com.jakewharton:butterknife:8.5.1'){
        exclude module: 'support-compat'
    }
    //动态版本声明
    compile "org.codehaus.cargo:cargo-ant:1.+"
}

```
第一行`apply plugin`，声明的是构建的项目类型  
如果构建的对象是Android程序，声明为`apply plugin: 'com.android.application'`  
如果构建的对象是库，声明为`apply plugin: 'com.android.library'`

关于代码混淆的问题，`buildTypes`的`proguardFiles`指定了混淆规则的文件，它分为两块，一块是`getDefaultProguardFile('proguard-android.txt')`，它是Android默认的混淆文件，这个文件的目录在 sdk目录`/tools/proguard/proguard-android.txt`，它包含一些基础的混淆规则；第二块是后面跟着的`proguard-rules.txt`，目录就在`app/proguard-rules.txt`，在这里可以自定义一些混淆规则，多用于第三方依赖，最终的混淆是要结合两个文件，共同完成混淆任务。

最下面的`dependencies`里声明的是模块的依赖，有几个关键字需要注意
* `compile` 表示编译时提供并打包进apk
* `provided` 表示只在编译时提供，不打包进apk
* `exclude` 设置不编译指定的模块，排除指定模块的依赖，防止重复依赖报错
* `transitive` 默认为true，gradle自动添加子依赖项。设置为false排除所有的传递依赖

### gradle.properties
gradle.properties文件中可以配置一些变量，这些变量在这个工程下的所有module的build.gradle文件里都可以使用。这样就可以把一些共用的变量放到这里，这样后面修改的时候就可以只修改这个变量。
例如把SDK版本信息放进去：
```groovy
MIN_SDK_VERSION=21
TARGET_SDK_VERSION=22
VERSION_CODE=200100
VERSION_NAME=2.1.0
```
在 `build.gradle`文件中，通过`project`引入
```groovy
minSdkVersion project.MIN_SDK_VERSION as int
//效果等同于minSdkVersion Integer.parseInt(project.MIN_SDK_VERSION)
targetSdkVersion project.TARGET_SDK_VERSION as int
versionCode project.VERSION_CODE as int
versionName project.VERSION_NAME
```

### settings.gradle
这个文件是全局的项目配置文件，里面主要声明一些需要加入gradle的module。
一般在setting.gradle中主要是调用include方法，导入工程下的各个子模块。
```groovy
include ':AndroidDemo'

include ':CommonSDK'
project(':CommonSDK').projectDir = new File(rootProject.projectDir, './SDK/CommonSDK/')
//
```
这里解释一下`CommonSDK`的引入流程。首先通过`include`生成了`CommonSDK`对象，接下来就是对对象的操作。如果模块在项目根目录，就不需要自己指定目录，如果不在根目录，就需要`projectDir`方法来指明待引用模块的路径，`rootProject.projectDir`指的是项目根目录，后一项参数是待引用模块的相对路径。
