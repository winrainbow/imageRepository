# Gradle4Android

> 谷歌推广Gradle和AndroidStudio ,他们立下目标：让代码复用、构建variant、配置和定制构建过程变的简章。更重要的：让构建系统不在依赖IDE，并使得能过命令行或者持续集成服务器运行Gradle生成的apk文件和在AndroidStudio中执行构建生成的结果一致

最常用的应用场景：在持续集成服务器上，不需要安装AndroidStudio 

## Gradle基础

> 约定优于配置原则（为设置和属性提供默认值）

> Gradle构建脚本没有使用xml,而是基于Groovy的领域专用语言（DSL）

> DSL:domain-specific language: 指的是专注于某个应用程序领域的计算机语言[领域特定语言]
>
> xml 中的配置文件是通过高级语言解析而来，我们也可以用高级语言解析 DSL 文件，执行相关的逻辑。但是与 xml 配置文件不同的是，xml 配置文件需要预先定义规范。而 DSL 不需要定义规范，其语法受到现有语言的约束，不用使用任何解析器，而是**巧妙的讲语法映射到底层语言中的方法和属性**，这样使得用户在使用起来可能意识不到自己正在使用的是一门更广泛的语言的语法。

### 项目和任务

1. 每一次构建都包括至少一个项目，每一个项目又包含一个或多个任务
2. 每个build.gradle文件都代表一个项目
3. 任务定义在构建脚本里
4. 在初始化构建过程时，Gradle会基于build文件组装项目和任务对象，
5. 一个任务对象包含一系列的动作对象，这些动作对象之后会按顺序执行
6. 一个单独的动作对象就是一个待执行的代码块，和java中的方法类似

### 生命周期

执行一个Gradle构建的最简单的形式是只执行**任务**中的**动作**,但这些任务又依赖于其他任务。为了简化构建过程，构建工具会新建一个动态的模型流，叫作**Driected Acyclic Graph(DAG)** 意味着所有的任务都会顺序执行，任务不可循环执行

**没有依赖的任务**通常会被优先执行，在**构建的配置阶段**生成依赖关系图

Gradle构建三个阶段：

1. 初始化：项目实例会在该阶段被创建，如果有多个模块，并且每个模块都有对应的build.gradle,就会创建多个项目实例
2. 配置：在该阶段，构建脚本被执行，并为每个项目实例创建和配置任务
3. 执行：在该阶段，Gradle将决定哪个任务被执行。具体哪些任务被执行取决于开始构建的的参数配置和该Gradle文件的当前目录

### 构建配置文件

每个基于Gradle构建的项目，至少有一个build.gradle文件，Android的构建文件中，有些元素是必须的

```groovy
buildscript{
    repositories {
        jcenter() // 整个构建过程中的依赖仓库
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}
```

每个Android项目中都应该申请插件：

```groovy
// 如果是一个应用
apply plugin: 'com.android.application'
// 如果是一个库
apply plugin: 'com.android.library'
```

```groovy
// 使用android插件时，可以生成只用于Android的任务
android{
    compileSdkVersion 22
    buildToolsVersion "22.0.1"
}
```

### 项目目录

Gradle有一个源集的概念，*一个源集就是一组源文件，它们会一起执行和编译*，如main目录是一个源集

## GradleWrapper

> Gradle是一个不断发展的工具，新版本可能会打破向后兼容，而使用GradleWrapper可以避免这个问题，并能确保构建是可重复的

GradleWrapper为windows提供一个batch文件，为其他系统提供了shell角本

先安装Gradle才可以获取gradleWrapper

```groovy
task wrapper (type:Wrapper){
    gradleVersion = '2.4'
}
//执行下面命令：
gradle wrapper --gradle-version 2.4

```

# 自定义构建（基础）

## settings.gradle

在初始化阶段执行，并且定义了哪些模块应该包含在构建内

单模块的项目，并不一定需要setttings文件，则多模块的必须有settings.gradle文件，不然Gradle不知道哪个模块应包含在构建内

## 顶层build.gradle文件，

```groovy
buildscript{
    repositories{
        jcenter()
    }
    dependencies{
        classpath 'com.android.tools.build:gradle:2.3.3'
    }
}
// 声明那些需要被用于所有模块的属性，
allprojects{
    repositories{
        jcenter()
    }
}
```

## 模块内的build.gradle文件

可以覆盖顶层的build.gradle文件的任何属性

```groovy
// 插件： 由google团队维护，提供构建、测试、打包android应用及所依赖项目的所有任务
apply plugin:'com.android.application'
android{
    // 下面的两个属性，必有
	compileSdkVersion 22
    buildToolsVersion "22.0.1"
    // defaultConfig用于配置应用的核心属性
    defaultConfig{
        applicationId "com.gradleforandroid.gettingstarted"
        minSdkVersion 14
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes{
        release{
            minifyEnable false
            progruardFiles getDefaultProgruardFile
            ('proguard-android.txt'),'proguard-rules.pro'
        }
    }
}
//依赖包
dependencies {
    compile fileTree(dir:'libs',include:['*.jar'])
    compile 'com.android.support:appcompat-v7:22.2.0'
}
```

compile:

apk:

provided

testCompile

androidTestCompile