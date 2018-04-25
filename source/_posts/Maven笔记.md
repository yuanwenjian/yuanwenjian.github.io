---
title: Maven笔记
description:  maven问题记录汇总
date: 2018-04-20 13:25:00
comments: true
tags: 
    - Maven
    - tool
categories:
    - Tools
---

# Maven记录汇总

## maven 生命周期
maven生命周期分为三个，分别为clean,default,site,每个生命周期包含多个阶段(phase),这些阶段是有顺序的，后面的依赖前面的

### clean包含阶段
- **pre-clean**  执行实际项目清理前所需的流程
- **clean**  删除以前版本生成的所有文件
- **post-clean** 执行完成项目清理所需的过程
### default包含阶段
- **validate**	验证项目是否正确，并提供所有必要的信息。
- **initialize**	初始化构建状态，例如设置属性或创建目录。
- **generate-sources**	生成包含在编译中的所有源代码。
- **process-sources**	处理源代码，例如过滤任何值。（filter any value）。
- **generate-resources**	生成包含在包中的资源。
- **process-resources**	将资源复制并处理到目标目录中，准备打包。
- **compile**	编译工程源源码。
- **process-classes**	从编译后处理生成的文件，例如在Java类上进行字节码增强。
- **generate-test-sources**	生成包含在编译中的任何测试源代码。
- **process-test-sources**	处理测试源代码，例如过滤任何值。
- **test-compile**	将测试源代码编译到测试目标目录中
- **process-test-classes**	从测试编译后处理生成的文件，例如在Java类上进行字节码增强。对于Maven 2.0.5及以上版本。
- **test**	使用合适的单元测试框架运行测试。这些测试不应该要求打包或部署代码。
- **prepare-package**	在实际包装之前执行任何必要的准备包装的操作。这通常会导致软件包的解压缩，处理版本。 （Maven 2.1及以上）。
- **package**	接受编译的代码并将其打包为可分发的格式，例如JAR
- **pre-integration-test**	在集成测试执行之前执行所需的操作。这可能涉及诸如设置所需环境等事情。
- **integration-test**	如果需要，可将程序包处理并部署到可运行集成测试的环境中。
- **post-integration-test**	执行集成测试后执行所需的操作。这可能包括清理环境。
- **verify**	运行任何检查来验证包是否有效并且符合质量标准。
- **install**	将软件包安装到本地存储库中，作为本地其他项目的依赖项。
- **deploy**	在集成或发行版环境中完成，将最终包复制到远程存储库，以便与其他开发人员和项目共享。

### site包含阶段
- **pre-site** 执行实际项目站点生成之前所需的流程 
- **site** 生成项目的网站文档
- **post-site** 执行完成网站生成所需的流程，并为网站部署做好准备
- **site-deploy** 将生成的网站文档部署到指定的Web服务器
### 源码
[phase阶段][phase]

## maven 插件
插件本身为了复用代码，往往完成了多个任务。例如maven-dependency-plugin插件就能够分析项目依赖、列出依赖树、分析依赖来源、列出所有已解析依赖等等。这些功能里面有很多代码其实是复用的，所以为了复用，他们集成为了一个插件。每个插件的<font color='red'>目标</font>就是一个插件的功能。例如dependency:analyze的目标就是analyze，功能就是分析依赖。dependency:tree的目标tree的功能就是列出依赖树。

## maven 插件内置绑定
ejb / ejb3 / jar / par / rar / war 类型通用绑定

| 阶段|插件及目标|
|-|-|
|process-resources	|resources:resources|
|compile	|compiler:compile|
|process-test-resources|	resources:testResources|
|test-compile	|compiler:testCompile|
|test|	surefire:test|
|package|	ejb:ejb or ejb3:ejb3 or jar:jar or par:par or rar:rar or war:war|
|install|	install:install |
|deploy	|deploy:deploy|

ear 类型

|阶段|插件及目标|
|-|-|
|generate-resources|	ear:generate-application-xml|
|process-resources	|resources:resources|
|package|	ear:ear|
|install|	install:install|
|deploy	|deploy:deploy|

maven-plugin 类型

|阶段|插件及目标|
|-|-|
|generate-resources	|plugin:descriptor|
|process-resources |	resources:resources|
|compile|	compiler:compile|
|process-test-resources|	resources:testResources|
|test-compile	|compiler:testCompile|
|test	|surefire:test|
|package|	jar:jar and plugin:addPluginArtifactMetadata|
|install|	install:install|
|deploy	|deploy:deploy|

pom类型

|阶段|插件及目标|
|-|-|
|package	|site:attach-descriptor|
|install	|install:install|
|deploy	|deploy:deploy|

所有类型通用

|阶段|插件及目标|
|-|-|
|clean|clean:clean|
|site	|site:site|
|site-deploy|	site:deploy|

### 源码
[插件绑定源码][plugin]


## maven依赖解决方案
1. 第一原则：路径最短者优先。
假设 X 是 A 的传递性依赖：A->B->C->X(1.0) 依赖路径长度为 3，A->D->X(2.0) 长度为 2，则 X(2.0) 会先被解析使用。

2. 第二原则：第一声明者优先。
在依赖路径长度相等的前提下，在 POM 中，依赖声明的顺序决定了谁会先被解析使用。

## maven scope详解
指定当前包的依赖范围和依赖的传递性,可选值为以下
1. complie 默认值 在编译、运行、测试时均有效
2. provided 在编译、测试时有效，但是在运行时无效，如servlet-api，运行时由容器提供，注意，idea有一个bug,使用idea启动SpringBoot，如果没有serlet-api会报错
3. runtime 在运行、测试时有效
4. test 只在测试时有效，如JUnit
5. system 类似provided,在编译，测试时有效,但是不会在repository中查找，需要手动指定
可以使用如下配置
```xml
<dependency>
    <groupId>javax.sql</groupId>
    <artifactId>jdbc-stdext</artifactId>
    <version>2.0</version>
    <scope>system</scope>
    <systemPath>${java.home}/lib/rt.jar</systemPath>
</dependency>
```

### scope 传递关系
![scope][scopeType]

项目B依赖于项目A
```xml
<!-- 项目B配置 scope对应上图第一列-->
<dependency>
    <groupId>com.yuanwj</groupId>
    <artifactId>project-A</artifactId>
    <version>2.0</version>
    <scope>complie</scope>
</dependency>

<!-- 项目A配置 scope对应上图第一行-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>*.4</version>
</dependency>

<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>*.4</version>
    <scope>runtime</scope>
</dependency>

当mysql传递到项目B中，对应上图可知对应的scope为complie
commons-codec传递项目B中，对应上图可知scope为runtime
```
[官网传递](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope)

## maven 打包跳过测试编译
```bash
mvn clean install -Dmaven.test.skip=true
```


## 参考
[maven官网][maven]
[maven源码][maven_github]

[scopeType]: ../images/scope.jpg
[phase]:https://github.com/apache/maven/blob/master/maven-core/src/main/resources/META-INF/plexus/components.xml
[plugin]:https://github.com/apache/maven/blob/master/maven-core/src/main/resources/META-INF/plexus/default-bindings.xml
[maven]:https://maven.apache.org/
[maven_github]:https://github.com/apache/maven
