---
title: "4.1 - 开源插件构建教程"
published: 2025-07-21
description: "开源万岁~0721万岁~"
tags: [IDEA, Plugin, MC, Building]
category: "教程"
draft: false
series: "MC高版本腐竹指南"
---

> 挨多☆米那桑(◦˙▽˙◦)，今天是柚子社♡大贤☆宗☆师「绫地宁宁」的纪念日07.21喵(＊°∀°＊)✧＊。将这段文字转发到十个群（笑），将会在「图书馆」遇到一位正在打☆磨桌角的宁宁的说(=^▽^=)♡，我试过了，是假的喵(。・ˋω´・)，还会被骂柚♪子♪厨退群，苦亚西(◦ˋ~´◦)，得磨，今天真的是绫地宁宁的纪念日07.21……（ˋΔ´）ゞ，让我们一起为宁宁庆祝喵！Ciallo～(∠・ω< )⌒☆

# 开源插件的构建

我们已经了解过了 **GitHub** 网站，而部分插件作者把自己的插件开源但在 **SpigotMC** 等平台提供付费下载。对于只想研究该插件、没有充足资金的服主来说，学会自己构建它们也可以省下一笔支出，但需要注意的是，这么做以后作者是不会给你提供服务支持的，遇到问题、提交建议等一般都需要自己解决，除非作者声明了付费用户和自行构建用户不区别对待。

## IntelliJ IDEA 的下载与安装

点击 [此链接](https://www.jetbrains.com/idea/download/download-thanks.html?platform=windows\&code=IIC) 下载，安装与常规软件并无差异，在此不再赘述，实在不会的请自己查询。

## 从 GitHub 上克隆项目

打开软件后，点击“从VCS获取”，在打开的页面中输入 GitHub 的地址即可。

![](https://400373137-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FFlP4xP4pRQ4Bt9AMcMkX%2Fuploads%2FxGCXbpHQGXg03LA6JZXG%2Fimage.png?alt=media\&token=1a1d8928-c7c7-4b00-a278-d202b6c3cbbe)

## 进入项目

进入项目后，IntelliJ IDEA 会自动开始构建项目，如果并没有，点击图片所示的图标即可。构建项目中的主要过程就是下载依赖。

<figure><img src="https://400373137-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FFlP4xP4pRQ4Bt9AMcMkX%2Fuploads%2FWFUJ1DWEdVunjScQZnCq%2FInked%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-04-08%20183921.jpg?alt=media&#x26;token=235735c4-0634-4050-b47d-e7e91db96189" alt=""><figcaption></figcaption></figure>

如果这个过程中出现下载依赖方面出现问题，可以着手以下两个方法：

* 使用镜像节点。
* 使用代理。以 Clash 为例，你需要在常规选项卡开启 TUN 模式。

## 打包 jar

一般开源项目会自带构建、打包插件成 **jar** 文件的 **gradle** 或者 **maven** 文件。使用 **maven** 作为构建、打包工具的项目源代码会含有 **pom.xml** 文件，而 **gradle** 则是很好就可以辨别出来。

知道插件使用的构建、打包工具后就可以开始着手打包了：

* 在上图所指图标右侧有一个 **添加配置** 按钮，请点击它。
* 在打开的新页面中点击 **添加新的运行配置** 按钮。
* 在再次打开的新界面中找到对应的构建、打包工具，例如这里是 **gradle**。
* 在再次打开的新页面中的运行字段框中输入打包指令。**gradle** 输入 **clean build**，**maven** 输入 **clean package**。

<figure><img src="https://400373137-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FFlP4xP4pRQ4Bt9AMcMkX%2Fuploads%2FTWIOOdAHWmGlYp4rpwm9%2F%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-04-08%20185042.png?alt=media&#x26;token=5cda798d-2496-4b69-8758-5960a6cf6130" alt=""><figcaption></figcaption></figure>

* 打包完成。

<figure><img src="https://400373137-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FFlP4xP4pRQ4Bt9AMcMkX%2Fuploads%2F0w62rEOPvZDt0ewH5os6%2F%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-04-08%20185206.png?alt=media&#x26;token=80955d55-1bd9-4d9e-b0b2-c683342dc7d4" alt=""><figcaption></figcaption></figure>

## 添加本地库

本地库可以在 **Maven** 或者 **Gradle** 的构建文件添加。

:::note
请注意，如果添加的 jar 文件下又需要添加其他 jar 文件作为依赖，那么添加本地库需要将这些所有用到的 jar 文件全部打包。

例如，在下面这个例子中使用了 GamePoints 插件作为依赖打包，而 GamePoints 又用了其前置插件 NexEngine 作为依赖，因而我将两个插件都写进了这个构建文件中。
:::

对于 **Maven**，打开项目下的 `pom.xml` 文件，找到如下格式的内容：

```html no-wrap
    <dependencies>
        <dependency>
            <groupId>org.spigotmc</groupId>
            <artifactId>spigot-api</artifactId>
            <version>1.19.1-R0.1-SNAPSHOT</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>su.nightexpress.gamepoints</groupId>
            <artifactId>GamePoints</artifactId>
            <version>1.3.5</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/lib/GamePoints-1.3.5.jar</systemPath>
        </dependency>
        <dependency>
            <groupId>su.nexmedia</groupId>
            <artifactId>NexEngine</artifactId>
            <version>2.2.8</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/lib/NexEngine.jar</systemPath>
        </dependency>
    </dependencies>
```

然后将对应的 **jar** 文件放置到项目文件夹下的 **libs** 文件夹即可。

**Gradle** 就比较简单了，找到 `build.gradle`，在：

```
dependencies {
  ...（一系列内容）
}
```

中添加：

```html no-wrap
compileOnly fileTree(dir:'libs',includes:['*.jar'])
```
