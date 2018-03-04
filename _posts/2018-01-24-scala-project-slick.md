---
layout: post
title: "搭建Scala项目——slick篇"
author: "郝强"
categories: documentation
tags: [documentation,scala-project]
image: scala-project-slick-0.jpeg
---

## 关于Slick

**「[Slick](http://slick.lightbend.com)」** (“Scala Language-Integrated Connection Kit”) is [Lightbend](https://lightbend.com/)’s Functional Relational Mapping (FRM) library for Scala that makes it easy to work with relational databases. It allows you to work with stored data almost as if you were using Scala collections while at the same time giving you full control over when database access happens and which data is transferred. You can also use SQL directly. Execution of database actions is done asynchronously, making Slick a perfect fit for your reactive applications based on [Play](https://playframework.com/) and [Akka](http://akka.io/).

简单来说**「Slick」**就是ORM，不过他提供的API是函数式的，所以是FRM。**「Slick」**对于Play这种响应式应用是完美适配的，这也是我选择**「Slick」**的原因。我们可以很方便的用Scala的语法来操作数据对象。

```scala
class Coffees(tag: Tag) extends Table[(String, Double)](tag, "COFFEES") {
  def name = column[String]("COF_NAME", O.PrimaryKey)
  def price = column[Double]("PRICE")
  def * = (name, price)
}
val coffees = TableQuery[Coffees]
```

```scala
// Query that only returns the "name" column
// Equivalent SQL: select NAME from COFFEES
coffees.map(_.name)

// Query that limits results by price < 10.0
// Equivalent SQL: select * from COFFEES where PRICE < 10.0
coffees.filter(_.price < 10.0)
```



## 用Slick codegen来生成数据映射

与**「Mybatis」**的codegen用来生成带SQL模版的XML文件和Entity类似，**「Slick」**也能通过codegen来生成Tables.scala文件。之前在「[搭建Scala项目——SBT篇]({{ site.url }}/documentation/scala-project-slick.html)」中已经创建了一个名叫**「mrnyanko-model」**的module。这个module就是用来执行codegen task和生成数据对象。目录结构如下：

```
mrnyanko-model      → 项目的基目录
 └ project          → 元构建根项目的基目录
 └ src              → src目录
    └ main          → mian目录
      └ java        → Java文件目录
      └ resources   → 资源配置文件目录
      └ scala       → Scala文件目录
    └ test          → test目录
      └ java        → Java文件目录
      └ resources   → 资源配置文件目录
      └ scala       → Scala文件目录
 └ target           → target目录
 └ .gitignore       → git忽略文件配置
 └ build.sbt        → 项目build配置
```

创建build.sbt并引入需要的依赖：

```scala
name := "mrnyanko-model"

val slickVersion = "3.2.1"

libraryDependencies ++= Seq(
    "com.typesafe" % "config" % "1.3.0",
    "com.typesafe.slick" %% "slick" % slickVersion,
    "com.typesafe.slick" %% "slick-codegen" % slickVersion,
    "org.slf4j" % "slf4j-nop" % "1.7.19",
    "mysql" % "mysql-connector-java" % "5.1.38"
)
```

之后在`resources`目录下创建`application.conf`和`database.conf`文件。

`application.conf`文件主要定义了log级别：

```properties
logger.scala.slick = DEBUG
db.default.logSql = true
```

而在`database.conf`文件中主要是用来定义一些数据库配置和生成的文件路径：

```properties
slick.driver = "slick.jdbc.MySQLProfile"
jdbc.driver = "com.mysql.jdbc.Driver"
db.url = "jdbc:mysql://localhost:3306/playscala"
output.folder = "mrnyanko-model/src/main/scala"
pkg.name = "mr.nyanko.playscala.models"
db.user.name = "XXXXX"
db.user.password = "*************"
```

在`src/main/scala`下创建`codegen` package。并在`codegen` package下创建`CodeGenerator` Object：

```scala
package codegen

import com.typesafe.config.ConfigFactory


/**
  * Created by Nyankosensei on 18/1/24.
  */
object CodeGenerator extends App {

    val dbConf = ConfigFactory.load("database")

    val slickDriver = dbConf.getString("slick.driver")

    val jdbcDriver = dbConf.getString("jdbc.driver")

    val url = dbConf.getString("db.url")

    val outputFolder = dbConf.getString("output.folder")

    val pkg = dbConf.getString("pkg.name")

    val user = dbConf.getString("db.user.name")

    val password = dbConf.getString("db.user.password")

    slick.codegen.SourceCodeGenerator.main(Array(slickDriver, jdbcDriver, url,
        outputFolder, pkg, user, password))
}

```

之后我们只需要在项目中执行以下sbt命令就可以生成Tables.scala文件了。

```powershell
➜  mrnyanko-play-scala git:(master) ✗ sbt                     
sbt:mrnyanko-play-scala> project mrnyanko-model
sbt:mrnyanko-model> run
```
目录结构如下：
```
scala                   → Scala文件目录
 └ codegen              → codegen package
    └ CodeGenerator$    → CodeGenerator Object
 └ models               → models package
    └ Tables            → 生成的数据映射文件
```



## 总结

一般情况下，数据库设计和架构设计都是并行的。因此大多数情况我都是先搭框架，等数据库设计完成后，再生成相应的PO。只需要通过依赖的方式把model模块依赖进来就可以了。

值得注意的是官方给的[slick-codegen-example](https://github.com/slick/slick-codegen-example)使用的sbt版本为0.13.9我使用的是1.0.3因此有些在build.sbt的参数可能存在差异。当然定义codegen task方式有很多，除了我使用的这种官方还提供了另一种方式[slick-codegen-customization-example](https://github.com/slick/slick-codegen-customization-example)。实际搭建中，应注意版本问题。之前我们项目中用的是：

>Scala版本：2.11.8
>
>SBT版本：0.13.8
>
>Slick版本：3.1.1

在这个项目中我用的是：

>Scala版本：2.12.4
>
>SBT版本：1.0.3
>
>Slick版本：3.2.1

版本不同会导致一些使用上的变化，这点应该是需要特别注意的。比如说之前一个配置：

```properties
## slick version 3.1.1
slick.driver = "slick.driver.MySQLDriver"
```

在新版本中变为：

```properties
## slick version 3.2.1
slick.driver = "slick.jdbc.MySQLProfile"
```
还有一个需要吐槽和注意的地方时，如果一个表的字断超过了22个，由于Scala Tuple最多支持22个元素，因此在生成对应的table时会生成一个Hlist的链表结构。目前我们还没有找到特别好的方法去解决这个问题。我的解决方法是，通过写一个隐式转换，将相应case calss转换成对应的PO中的链表结构的成员属性。

```scala
implicit class TContactsShiftImplicit(tContact: _root_.xxx.xxx.xxxxxxx.models.Tables.TContactsRow) {
        def shift: TContactsShiftRow = TContactsShiftRow(tContact)
    }

    implicit class TContactsShiftsImplicit(tContacts: Seq[_root_.xxx.xxx.xxxxxxx.models.Tables.TContactsRow]) {
        def shift = tContacts map (TContactsShiftRow(_)) toList
    }


    case class TContactsShiftRow(id: Long, uuid: String, customerName: String,
                                 customerIdNumber: String, name: String, gender: String,
                                 age: Option[Int] = None, idNumber: Option[String] = None, relationship: String,
                                 relationshipOthers: Option[String] = None, department: Option[String] = None,
                                 position: Option[String] = None, company: Option[String] = None,
                                 `type`: String, province: Option[String] = None,
                                 city: Option[String] = None, district: Option[String] = None, detail: Option[String] = None,
                                 evaluation: String, origin: String, remarks: Option[String] = None, initialOrigin: String, version: Long,
                                 effectiveTimestamp: java.sql.Timestamp, auditTimestamp: java.sql.Timestamp, isDeleted: Boolean = false)


    object TContactsShiftRow {
        def apply(tContact: _root_.xxx.xxx.xxxxxxx.models.Tables.TContactsRow): TContactsShiftRow = {
            tContact.toList match {
                case id :: uuid :: customerName ::
                    customerIdNumber :: name :: gender ::
                    age :: relationship :: relationshipOthers ::
                    department :: position :: company ::
                    typeType :: province :: city ::
                    district :: detail :: evaluation ::
                    origin :: idNumber :: remarks :: initialOrigin ::
                    version :: effectiveTimestamp :: auditTimestamp ::
                    isDeleted :: Nil => {
                    TContactsShiftRow(id.asInstanceOf[Long], uuid.asInstanceOf[String], customerName.asInstanceOf[String],
                        customerIdNumber.asInstanceOf[String], name.asInstanceOf[String], gender.asInstanceOf[String],
                        age.asInstanceOf[Option[Int]], idNumber.asInstanceOf[Option[String]], relationship.asInstanceOf[String],
                        relationshipOthers.asInstanceOf[Option[String]], department.asInstanceOf[Option[String]],
                        position.asInstanceOf[Option[String]], company.asInstanceOf[Option[String]],
                        typeType.asInstanceOf[String], province.asInstanceOf[Option[String]],
                        city.asInstanceOf[Option[String]], district.asInstanceOf[Option[String]], detail.asInstanceOf[Option[String]],
                        evaluation.asInstanceOf[String], origin.asInstanceOf[String], remarks.asInstanceOf[Option[String]], initialOrigin.asInstanceOf[String], version.asInstanceOf[Long],
                        effectiveTimestamp.asInstanceOf[java.sql.Timestamp], auditTimestamp.asInstanceOf[java.sql.Timestamp], isDeleted.asInstanceOf[Boolean])
                }

            }
        }
    }
```

