title: "Gradle User Guide"
date: 2016-02-06 23:19:45
categories:
 - 移动开发
 - Gradle
tags: 
 - Gradle
author: mzeht
avatar: /images/favicon.png
---
参考资料：gradle 2.11 官方文档<https://docs.gradle.org/current/userguide/userguide.html>

Gradle User Guide
Version 2.10

Copyright © 2007-2015 Hans Dockter, Adam Murdoch
Copies of this document may be made for your own use and for distribution to others, provided that you do not charge any fee for such copies and further provided that each copy contains this Copyright Notice, whether distributed in print or electronically.

#第一章：关于gradle
##1.介绍
很高兴能向大家介绍 Gradle, 这是一个构建系统, 我们认为它是 java ( JVM ) 世界中构建技术的一个飞跃.
Gradle 提供了:

- 一个像 Ant 一样的非常灵活的通用构建工具
- 一种可切换的, 像 maven 一样的基于合约构建的框架
- 支持强大的多工程构建
- 支持强大的依赖管理(基于 ApacheIvy )
- 很好的支持你已有的 maven 和 ivy 仓库
- 支持传递性依赖管理, 而不需要远程仓库或者 pom.xml 或者 ivy 配置文件
- 优先支持 Ant 式的任务和构建
- 基于 `groovy` 的构建脚本
- 有丰富的领域模型来描述你的构建


### 1.1. 关于用户指南
这份用户指南，就像Gradle自身一样，还在积极的发展之中，Gradle的一些功能没有被完全展示出来，现在的一些内容也不是很清晰，如果你更加了解Gradle，我们需要你的帮助来完善Gradle文档，你可以在[Gradle Web Site](http://www.gradle.org/contribute)了解如何为文档贡献力量

通过这本指南, 你将会看到一些代表 Gradle 任务之间依赖关系的图表. 类似于 UML 依赖关系
的表示方法, 从一个任务 A 指向另一个任务 B 的箭头代表A依赖于B.

## 2.概述
### 2.1. 特点
1. `声明式构建和合约构建`
Gradle 的核心是基于 Groovy 的 领域特定语言 (DSL), 具有十分优秀的扩展性. Gradle 通过提
供可以随意集成的声明式语言元素将声明性构建推到了一个新的高度. 这些元素也为 Java,
Groovy, OSGi, Web 和Scala 等项目提供基于合约构建的支持. 而且, 这种声明式语言是可扩展
的. 你可以添加自己的语言元素或加强现有的语言元素, 从而提供简洁, 易于维护和易于理解的
构建.

2. `基于依赖的编程语言`声明式语言位于通用任务图 ( general purpose task graph ) 的顶端，它可以被充分利用在你
的构建中. 它具有强大的灵活性, 可以满足使用者对 Gradle 的一些特别的需求.
3. `让构建结构化`
Gradle 的易适应性和丰富性可让你在构建里直接套用通用的设计原则. 例如, 你可以非常容易
地使用一些可重用的组件来构成你的构建. 但是不必要的间接内联内容是不合适的. 不要强行
拆分已经结合在一起的部分 (例如, 在你的项目层次结构中). 避免使构建难以维护. 总之, 你可
以创建一个结构良好，易于维护和易于理解的构建.
4. `API深化`
你会非常乐意在整个构建执行的生命周期中使用 Gradle, 因为Gradle 允许你管理和定制它的
配置和执行行为.
5. `Gradle 扩展`
Gradle 扩展得非常好. 不管是简单的独立项目还是大型的多项目构建, 它都能显著的提高效率.
这是真正的结构构建. 顶尖水平的构建功能，还可以解决许多大公司碰到的构建 性能低下的问
题.
6. `多项目构建`
Gradle 对多项目的支持是非常出色的. 项目依赖是很重要的部分. 它允许你模拟在多项目构建
中项目的关系，这正是你所要关注的地方. Gradle 遵从你的布局, 反过来就不是.
Gradle 提供了局部构建的功能. 如果你构建一个单独的子项目, Gradle 会构建这个子项目依赖
的所有子项目. 你也可以选择依赖于另一个特别的子项目重新构建这些子项目. 这样在一些大
型项目里就可以节省非常多的时间.
7. `多种方式来管理你的依赖`
不同的团队有不同的管理外部依赖的方法. Gradle 对于任何管理策略都提供了合适的支持. 从
远程 Maven 和 Ivy 库的依赖管理到本地文件系统的 jars 或者 dirs.
8. `Gradle 是第一个构建整合工具`
Ant 的 tasks是 Gradle 中很重要的部分, 更有趣是 Ant 的 projects 也是十分重要的部分.
Gradle 可以直接引入Ant 项目, 并在运行时直接将 Ant targets 转换成 Gradle tasks. 你可以从
Gradle 中依赖它们, 并增强它们的功能, 甚至可以在 build.xml 文件中声明 Gradle tasks 的依
赖. 并且properties, paths 等也可以通过同样的方法集成进来.
Gradle 完全支持你已有的 Maven 或者 lvy 仓库来构造发布或者提取依赖. Gradle 也提供了一
个转化器, 用来将 maven 的 pom.xml 文件转换成 Gradle 脚本. 在运行时引入 Maven 项目也
会在稍后推出.
9. `易于迁移`
Gradle 可以兼容任何结构. 因此你可以直接在你的产品构建的分支上开发你的 Gradle 构建,
并且二者可以并行. 我们通常建议编写一些测试代码来确保它们的功能是相同的. 通过这种方
式, 在迁移的时候就不会显得那么混乱和不可靠, 这是通过婴儿学步的方式来获得最佳的实践.
10. `Groovy`
Gradle 的构建脚本是通过 Groovy 编写的而不是 XML. 但是并不像其他方式, 这并不是为了简
单的展示用动态语言编写的原始脚本有多么强大. 不然的话, 只会导致维护构建变得非常困难.
Gradle 的整个设计是朝着一种语言的方向开发的, 并不是一种死板的框架. Groovy 就像胶水一
样, 把你像实现的构想和抽象的 Gradle 粘在一起. Gradle提供了一些标准的构想, 但是他们并
不享有任何形式的特权. 相比于其他声明式构建系统，对我们来说这是一个比较突出的特点.
11. `Gradle 包装器`
Gradle 包装器允许你在没有安装 Gradle 的机器上运行 Gradle 构建. 在一些持续集成的服务
器上, 这个功能将非常有用. 它同样也能降低使用一个开源项目的门槛, 也就是说构建它将会非
常简单. 这个包装器对于公司来说也是很有吸引力的. 它并不需要为客户机提供相应的管理防
范. 这种方式同样也能强制某一个版本 Gradle 的使用从而最小化某些支持问题.
12. `免费和开源`
Gradle 是一个开源项目, 遵循 ASL 许可.

### 2.2. 为什么选择Groovy
我们认为在构建脚本时，使用内部DSL［DSL：Domain Specific Languages,特定领域语言］(基于动态语言)，和XML相比，有很大优势，DSL有很多，为什么选择了Gradle？答案就在Gradle的使用背景中，虽然Gradle是通用的核心构建工具，但是主要运用在java项目上，在这些项目中，团队成员非常熟悉java，我们认为构建应该对团队全体成员尽量清晰明了

在这种情况下，你可能争论为什么不直接用java作为脚本构建语言，这的确是个合理的疑问，对于你的团队来说，java拥有最好的清晰度和最低的学习成本，但是由于java语言的限制，java作为脚本构建语言，并不是那么强大和出色[^hello].像Python，Groovy和Ruby这些语言表现更好，我们选择了Groovy，他最适合java开发人员，他的基本语法，类系统，包结构和其他一些特征，都和java相同，在和java的共同基础上，Groovy能做的更多.

对于熟悉Ruby,Python或者有志于学习它们的java开发者，以上依据都不成立了，Gradle的设计同时也适合用Ruby，Python来设计脚本构建引擎，只是现在不是我们的当务之急，我们乐于支持任何团体来开发更多的脚本构建引擎.



[^hello]: 在 <http://www.defmacro.org/ramblings/lisp.html> 你可以看到一篇比较Ant，XML，Java和Lisp的有趣文章， 如果java的语法和Groovy的语法一样的话，那将非常有趣

#使用现有的构建
##3.安装Gradle
###3.1前提
Gradle 需要安装一个 Java JDK 或者 JRE. 而且 Java 版本必须至少是 6 以上. (to check, use java -version). Gradle 有自带 Groovy 库, 所以没必要安装 Groovy. 任何已经安装的 Groovy 会被 Gradle 忽略.

Gradle 使用任何已经存在在你的路径中的 JDK (可以通过 java -version 检查, 如果有就说明系统已经安装了 Java 环境). 或者, 你也可以设置 JAVA_HOME 环境参数来指定希望使用的JDK的安装目录.

###3.2 下载
You can download one of the Gradle distributions from the [Gradle web site](http://www.gradle.org/downloads).

###3.3 解压
The Gradle distribution comes packaged as a ZIP. The full distribution contains:

- The Gradle binaries.
- The user guide (HTML and PDF).
- The DSL reference guide.
- The API documentation (Javadoc and Groovydoc).
- Extensive samples, including the examples referenced in the user guide, along with some complete and more complex builds you can use as a starting point for your own build.
- The binary sources. This is for reference only. If you want to build Gradle you need to download the source distribution or checkout the sources from the source repository. See the[Gradle web site](http://gradle.org/development/) for details.

###3.4 环境变量
For running Gradle, add GRADLE_HOME/bin to your PATH environment variable. Usually, this is sufficient to run Gradle.

###3.5 运行和测试安装
ou run Gradle via the gradle command. To check if Gradle is properly installed just type gradle -v. The output shows the Gradle version and also the local environment configuration (Groovy, JVM version, OS, etc.). The displayed Gradle version should match the distribution you have downloaded.

###3.6 JVM 设置
JVM 选项可以通过设置环境变量来更改. 您可以使用 GRADLE_OPTS 或者 JAVA_OPTS.
JAVA_OPTS 是一个用于 JAVA 应用的环境变量. 一个典型的用例是在 JAVA_OPTS 里设置HTTP代理服务器(proxy),
GRADLE_OPTS 是内存选项. 这些变量可以在 gradle 的一开始就设置或者通过 gradlew 脚本来设置.

注意，目前Gradle不能通过命令行设置jvm.







