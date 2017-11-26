title: "Gradle.4 Using the Gradle Command-Line"
date: 2016-02-10 01:12:31
author: mzeht
avatar: /images/favicon.png
categories:
 - 移动开发
 - Gradle
tags: 
 - Gradle
---

#4.通过命令行使用gradle
这张将介绍使用gradle命令行的基础知识，就像你在前几章看到的那样
##4.1 执行多重任务
你能使用命令通过一个脚本来执行多个任务，例如这个命令`gradle compile test`,他会执行compite和test两个任务，gradle会按照命令行中的任务顺序来有序执行，也会运行其依赖项目，每个任务仅仅只执行一次，无论它在脚本中如何被引用，是直接指定还是作为其他任务的依赖，或者两者都是，让我们来看一个例子.

以下四个任务，`dist`和`test`都依赖于`complie`，执行`gradle dist test`，结果`complie`任务只会运行一次.

图4.1 task依赖图
![](http://7xqtsx.com1.z0.glb.clouddn.com/commandLineTutorialTasks.png)
`官方演示图错了，test不是指向complie，而是complie指向test`

例4.1 执行多任务脚本

build.gradle

```
task compile << {
    println 'compiling source'
}

task compileTest(dependsOn: compile) << {
    println 'compiling unit tests'
}

task test(dependsOn: [compile, compileTest]) << {
    println 'running unit tests'
}

task dist(dependsOn: [compile, test]) << {
    println 'building the distribution'
}
```

gradle dist test的输出

```
> gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
```

每个任务只执行了一次，所以gradle test test 和 gradle test 的结果是一样的

这是本人测试的，第一次速度明显很慢
参考<https://docs.gradle.org/2.11/userguide/gradle_daemon.html>

windows:

```bash
(if not exist "%USERPROFILE%/.gradle" mkdir "%USERPROFILE%/.gradle") && (echo org.gradle.daemon=true >> "%USERPROFILE%/.gradle/gradle.properties")
```


UNIX:

```
>touch ~/.gradle/gradle.properties && echo "org.gradle.daemon=true" >> ~/.gradle/gradle.properties
```



```
➜  Desktop  gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 4.446 secs

This build could be faster, please consider using the Gradle Daemon: https://docs.gradle.org/2.8/userguide/gradle_daemon.html
➜  Desktop  gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1.768 secs
```


##4.2执行任务
你能通过使用-x 命令来排除某些任务的执行，我们来使用之前的脚本来演示

例4.2 执行任务
gradle dist -x test的输出

```
> gradle dist -x test
:compile
compiling source
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
```
![](http://7xqtsx.com1.z0.glb.clouddn.com/commandLineTutorialTasks.png)

`官方演示图错了，test不是指向complie，而是complie指向test`

从这个示例的输出,可以看到test任务没有执行,即使这是一个dist的依赖任务。你还会注意到依赖于test的任务中,compileTest没有执行。但是test依赖的另一个任务,如complie,仍会被执行。

##4.3如何在构建失败的时候继续构建

默认情况下，gradle将会在任何任务失败时中止执行。这使得构建更快地完成，但隐藏了其他错误。为了在一个单一的构建尽可能多的发现可能失败的任务，你可以使用--continue选项。

当--continue执行，gradle将执行每项任务，而不是只要遇到第一个故障就停止。遇到的每一个故障将在构建结束时报告。

如果一个任务失败，依赖于它的任何后续任务将不会被执行，因为这样做是不安全的。例如，如果complie的代码发生错误，test就不会被执行，因为test任务依赖于complie任务（直接或间接地） 。


##4.4 任务名缩写
当你命令行指定任务时，不需要提供全名，只要能足够辨认出该任务，在上面同样的脚本中，我们会通过执行`gradle -d `来执行dist任务

Example 4.3. Abbreviated task name

Output of gradle di

```
> gradle di
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
```

你也可以缩写驼峰命名的每个词，例如，可以通过`gradle compTest `甚至`gradle cT`来执行 compileTest 任务

Example 4.4. Abbreviated camel case task name
Output of gradle cT

```
> gradle cT
:compile
compiling source
:compileTest
compiling unit tests

BUILD SUCCESSFUL

Total time: 1 secs
```

在使用缩写的同时也可以使用 －x 命令

##4.5选择执行的脚本

当运行gradle命令时，会从当前路径查找脚本文件，你也可以使用 －b 命令来选择其他文件，如果使用－b命令，setting.gradle 文件就不会被使用，例如：

Example 4.5. Selecting the project using a build file

subdir/myproject.gradle

```
task hello << {
    println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
}
```

Output of gradle -q -b subdir/myproject.gradle hello

```
> gradle -q -b subdir/myproject.gradle hello
using build file 'myproject.gradle' in 'subdir'.
```

或者，你可以使用 －p 命令来指定文件路径，多个工程中，你应该使用－p命令 而不是 －b 命令

Example 4.6. Selecting the project using project directory

Output of gradle -q -p subdir hello

```
> gradle -q -p subdir hello
using build file 'build.gradle' in 'subdir'.
```

##4.6获得构建信息





