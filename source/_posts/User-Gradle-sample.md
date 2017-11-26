title: "User Gradle sample"
date: 2016-02-10 01:19:32
categories:
 - 移动开发
 - Gradle
tags:
- Gradle
author: mzeht
avatar: /images/favicon.png
---
#Example 4.1. Executing multiple tasks
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

Output of gradle dist test

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

#Example 4.2. Excluding tasks
Output of gradle dist -x test

你能通过使用-x 命令来排除某些任务的执行，我们来使用之前的脚本来演示

```
> gradle dist -x test
:compile
compiling source
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs

```


