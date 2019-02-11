本文简单的介绍了Gradle以及Gradle的基本用法。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### Gradle 简述

* 百度百科

> Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化构建工具。它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，抛弃了基于XML的各种繁琐配置。

### Gradle 基本概念

##### Project

* Project是Gradle构建初始化过程中生成的，Project与build.gradle脚本是一一对应的。每个参与构建的项目都会生成一个Project，Project本质上是Task的容器。

* Project构建过程中通常会需要很多依赖，Project也会生成可被其他Project使用的Artifact。

##### Task

* Task是Gradle构建过程中的一个原子操作。Task是由Action objects按顺序组成的。Task执行实际是Action按顺序运行。Groovy可用来写Action。

* Task之间是有依赖关系的，Task执行的先后关系会受依赖关系的影响。

### Gradle 构建流程

* Initialization（初始化）

初始化阶段，所有参与构建的项目都会生成与其一一对应Project。

* Configuration（配置）

配置阶段中，会对生成的Project进行配置。

* Execution（执行）

根据配置执行tasks。

### Gradle 基本配置

* settings.gradle

```
// 配置子项目 项目路径需用:替换/
include "subproject01"
include "subproject02"
include "folder:subproject03"

include "subproject01", "subproject02", "folder:subproject03"

// 配置同级项目
includeFlat "project02"
```

* build.gradle

```
// 插件配置
apply plugin: 'java'
apply from: 'url'

// 仓库配置
repositories {
    mavenCentral()
}

// 任务
task hello << {
    println "Hello"
}
task world << {
    println "World"
}

// 任务关系
// world任务依赖hello任务
world.dependsOn hello

// 依赖管理
dependencies {    
    compile ('xxx.xxxx.xxx:xxx:x.xx')
    testCompile ('xxx.xxxx.xxx:xxx:x.xx')
}

// 不可跨gradle脚本获取值
def i = "Hello World"
println i
println "i的值为：${i}"

// 可跨gradle脚本获取值
// extra property
ext.j = "Hello World"
println j
println "j的值为：${j}"
println hasProperty("j") ? j : "Hello World"

ext {
    k = "Hello"
    l = "World"
}
println "${k} ${l}"

// 配置gradle自身
buildscript {}

// 配置所有项目（自身和子项目）
allprojects {}

// 配置子项目（不包含自身）
subprojects {}
```

### Gradle 常用命令

* 新建Gradle项目

```
gradle init
```

* 查看项目结构

```
gradle projects
```

* 查看tasks

```
gradle tasks

gradle tasks --all
```

* 执行任务

```
gradle task-name
```

* 检查

```
gradle check
```

* 检查构建

```
// 如果已检查通过则不会进行检查
gradle build
```

* 运行

```
// 需要配置
// apply plugin: 'application'
// mainClassName = 'xxx.xxx.Xxx'
gradle run
```

* 清除构建

```
gradle clean
```

* 查看依赖关系

```
gradle dependencies
```

### Gradle Wrapper

##### Gradle Wrapper理解

Gradle Wrapper是将Gradle和项目绑定在了一起，方便没有安装Gradle的人使用，同时，也统一了项目所使用的Gradle。

##### Gradle Wrapper添加

```
gradle wrapper
```

##### Gradle Wrapper文件结构

添加Gradle Wrapper后，项目目录下会增加如下文件

|文件名|相对路径|作用|
|:----:|:----:|:----:|
|gradle-wrapper.jar|gradle/wrapper/gradle-wrapper.jar|Gradle Wrapper所需要的jar包依赖|
|gradle-wrapper.properties|gradle/wrapper/gradle-wrapper.properties|Gradle Wrapper配置文件|
|gradlew|gradlew|shell脚本|
|gradlew.bat|gradlew.bat|Windows批处理脚本|

##### Gradle Wrapper基本配置

gradle-wrapper.properties文件

```
// gradle下载解压后位置
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
// zip文件存放路径
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
// Gradle下载路径
distributionUrl=https\://services.gradle.org/distributions/gradle-4.0-bin.zip
```

##### Gradle Wrapper使用

将Gradle命令中的gradle替换为gradlew后就可以使用，功能对应是一致的。
