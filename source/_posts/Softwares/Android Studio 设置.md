---
title: Android Studio 设置
date: 2018-09-25 23:51:53
category: Softwares
---

    版本：Android Studio 3.0

# 修改 SDK 镜像地址

打开 SDK manager，添加下面的链接

```
http://mirrors.neusoft.edu.cn/android/repository/addon.xml
http://mirrors.neusoft.edu.cn/android/repository/addons_list-2.xml
http://mirrors.neusoft.edu.cn/android/repository/repository-12.xml
http://mirrors.neusoft.edu.cn/android/repository/extras/intel/addon.xml
http://mirrors.neusoft.edu.cn/android/repository/sys-img/android-tv/sys-img.xml
http://mirrors.neusoft.edu.cn/android/repository/sys-img/android-wear/sys-img.xml
http://mirrors.neusoft.edu.cn/android/repository/sys-img/android/sys-img.xml
http://mirrors.neusoft.edu.cn/android/repository/sys-img/google_apis/sys-img.xml
http://mirrors.neusoft.edu.cn/android/repository/sys-img/google_apis_playstore/sys-img.xml
```

# 解决 Gradle 下载卡主失败的问题

首次打开工程时，会下载 Gradle，但是太慢了，如果当前网络太慢了，可以手动下载 Gradle 包，步骤如下：

- 初次打开工程时会自动下载，并生成一个目录:`/.gradle/wrapper/dists/gradle-4.4-all/9br9xq1tocpiv8o6njlyu5op1`，其中`gradle-4.4-all`里面说明了要下载的 gradle 版本号，后面的`9br9xq1tocpiv8o6njlyu5op1`是随机字符串
- 我们在[https://services.gradle.org/distributions/](https://services.gradle.org/distributions/) 里面找到上面对应版本的 zip 包下载，比如`gradle-4.4-all.zip`
- 下载完成后，清空目录里面的缓存文件，将上面下载的 zip 包放在这里，不需要解压。
- 再次打开工程或者 sync 一下

# 解决 Gradle 下载 maven 库太慢的问题

使用阿里云的镜像。

对单个项目，修改 build.gradle：

```
buildscript {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
}
```

对所有项目生效，在`~/.gradle/`下创建`init.gradle`文件：

```
allprojects{
    repositories {
        def ALIYUN_REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public'
        def ALIYUN_JCENTER_URL = 'http://maven.aliyun.com/nexus/content/repositories/jcenter'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                    remove repo
                }
                if (url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL."
                    remove repo
                }
            }
        }
        maven {
            url ALIYUN_REPOSITORY_URL
            url ALIYUN_JCENTER_URL
        }
    }
}
```

# 模拟器安装 HAXM

到[Intel 官网](https://software.intel.com/en-us/articles/intel-hardware-accelerated-execution-manager-intel-haxm)下载 zip 包，解压之后，打开 silent_install.sh，搜索`10.12`，在后面添加`10.13`，然后运行命令`sudo ./silent_install.sh`即可。

# 单独启动模拟器

使用到了`emulator`命令，但是这个命令在`$ANDROID_HOME/tools`也有一个同名的，实际上要用到`$ANDROID_HOME/emulator`里面的。

- 增加`export PATH=$PATH:$ANDROID_HOME/emulator`到`.zshrc`里面
- 修改`$ANDROID_HOME/tools`目录下的`emulator`文件名为`emulator-bak`
- 使用`avdmanager list avd`列出存在的模拟器：
  ```
  The following Android Virtual Devices could not be loaded:
      Name: Pixel_2_XL_API_26
      Path: /Users/zhangwen/.android/avd/Pixel_2_XL_API_26.avd
     Error: Google pixel_2_xl no longer exists as a device
  ```
- 使用命令`emulator -netdelay none -netspeed full -avd Pixel_2_XL_API_26`启动模拟器，`Pixel_2_XL_API_26`是上面的`Name`字段
