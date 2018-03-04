---
layout: post
title: "搭建Scala项目——SBT篇"
author: "郝强"
categories: documentation
tags: [documentation,scala-project]
image: scala-project-sbt-0.jpeg
---

## 关于SBT

**「[SBT](https://www.scala-sbt.org)」**：

>  The interactive build tool。

**「SBT」**是一个“交互式的”（*interactive*）构建工具。在开发过程中，使用最多的是对引入的包进行管理、定义一些task以及打包。对于Scala的项目，SBT当然是首选的build工具，但是如果项目是Java和Scala的混合项目（虽然我不喜欢混合着用），还是用Maven好一些。顺带一提，SBT的交互模式和Play Console结合起来用。

 在Mac上安装SBT是非常方便的只需要用home brew安装就可以了。

```powershell
$ brew install sbt@1
```



## 创建项目

我在IDE中创建了一个名为**「mrnyanko-play-scala」**的SBT项目。当然也可以通过`sbt new`的命令来创建项目再import到IDE中也是没有问题的。

```
JDK：1.8
ScalaVersion：2.12.4
SBTVersion：1.0.3
```

则会生成如下项目：

```
mrnyanko-play-scala     → 项目的基目录
 └ project              → 元构建根项目的基目录
    └ project           → 元元构建的根项目的基目录；构建定义的构建定义工程
    └ target            → target
    └ build.properties  → 设置一些SBT的属性
    └ plugins.sbt       → 配置一些需要的插件
 └ target               → target
 └ .gitignore           → git忽略文件配置
 └ build.sbt            → 是构建定义项目的一部分。
 └ README.md            → 项目介绍
```

在 sbt 的术语里，“基础目录”是包含项目的目录。所以，如果你创建了一个和「mrnyanko-play-scala」一样的项目 `mrnyanko-play-scala` ，包含 `mrnyanko-play-scala/build.sbt `，`mrnyanko-play-scala` 就是基础目录。



## 拆分项目

我打算把项目拆分成三个部分：

1. **mrnyanko-web**主要提供API接口和业务逻辑处理
2. **mrnyanko-model**通过slick的codegen生成数据模型
3. **mrnyanko-common**公共组建，可单独发布。

依赖关系：**mrnyanko-web** →（**mrnyanko-model**，**mrnyanko-common**）

```
name := "mrnyanko-play-scala"

lazy val commonSetting = Seq(
    organization := "mr.nyanko",
    version := "1.0.0-SNAPSHOT",
    scalaVersion := "2.12.4"
)

lazy val `mrnyanko-play-scala` = project in file(".") settings (commonSetting: _*)

lazy val `mrnyanko-model` = project in file("mrnyanko-model") settings (commonSetting: _*)

lazy val `mrnyanko-common` = project in file("mrnyanko-common") settings(commonSetting: _*)

lazy val `mrnyanko-web` = project in file("mrnyanko-web") settings(commonSetting: _*) enablePlugins PlayScala dependsOn (`mrnyanko-model`,`mrnyanko-common`)

```

这样的话，如果业务有新扩展，而这种拓展不太适合放在现有的业务中那么我们就可以通过拓展新的module来满足业务需求。



## SBT常用命令


| 命令              | 描述                                       |
| --------------- | ---------------------------------------- |
| `clean`         | 删除所有生成的文件 （在 `target` 目录下）。              |
| `compile`       | 编译源文件（在 `src/main/scala` 和 `src/main/java` 目录下）。 |
| `test`          | 编译和运行所有测试。                               |
| `console`       | 进入到一个包含所有编译的文件和所有依赖的 classpath 的 Scala 解析器。输入 `:quit`， Ctrl+D （Unix），或者 Ctrl+Z （Windows） 返回到 sbt。 |
| `run <参数>*`     | 在和 sbt 所处的同一个虚拟机上执行项目的 main class。       |
| `package`       | 将 `src/main/resources` 下的文件和 `src/main/scala` 以及 `src/main/java`中编译出来的 class 文件打包成一个 jar 文件。 |
| `help <命令>`     | 显示指定的命令的详细帮助信息。如果没有指定命令，会显示所有命令的简介。      |
| `reload`        | 重新加载构建定义（`build.sbt`， `project/*.scala`， `project/*.sbt` 这些文件中定义的内容)。在修改了构建定义文件之后需要重新加载。 |
| `projects`      | 罗列项目中所有的project                          |
| `project <项目名>` | 在不同的项目中进行切换                              |
| `publish`       | 将项目打包发布至对应的仓库                            |



## 总结

在实际项目中，我就是按照上述的拆分方式对项目结构进行拆分的。每个module都可以单独run或者publish。这样的好处就是业务逻辑和数据模型进行了隔离，使开发人员只关注逻辑实现就可以了。有时候我也会将API单独抽离出来并创建一个名为**xxx-dal**的module。把API当作一个内部网关，可以单独部署到其他服务器上，web层与dal层通过common中提供的业务对象对传输的json进行序列化与反序列化的方式进行交互，当然也可以直接操作json。这时如果业务变化，需要我们提供一个可视化的操作入口，那么DAL的module可以几乎不动，只需要在新增一个view module或者在项目之外单独做一些静态页面就能满足业务需求了。

在两年前，微服务概念还没有现在成熟，公司中也没有这样的环境。虽然这样的纵向拆分方式当然也有他的局限性，不过也为未来的大方向的技术架构提供了思路。当然，作为一个规模不大的APP的server 端应该说是没有问题的。