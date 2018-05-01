---
title: OSGi博客笔记
date: 2018-02-02 23:45:09
tags: [笔记,OSGi,模块化,Java] 
categories: 模块化
---

本文是对killko一些博客的笔记，可以在[《不可错过的OSGi入门学习资源》](http://www.osgi.com.cn/article/7289520)中找到，包括：
* [《走近Java模块化系统OSGi》](https://www.tianmaying.com/tutorial/osgi-kickstart)
* [《创建OSGi Hello World工程》](https://www.tianmaying.com/tutorial/osgi-helloworld)
* [《OSGi中Bundle间的耦合：Export/Import Package与服务》](https://www.tianmaying.com/tutorial/osgi-bundle-coupling)
* [《动态的OSGi服务》](https://www.tianmaying.com/tutorial/osgi-service)
* [《初次接触OSGI Blueprint》](https://www.tianmaying.com/tutorial/osgi-blueprint)
* [《OSGi的配置管理：ConfigAdmin》](https://www.tianmaying.com/tutorial/osgi-configadmin-service)

此外，还参考了：
* [《osgi确实面临鸡肋之嫌》](http://blog.csdn.net/itd018/article/details/51035176)

阅读前建议先阅读
* [《OSGi入门教程》](https://course.tianmaying.com/osgi-toturial+osgi-concept#0)
* 或者[《OSGi入门教程》笔记](https://mindawei.github.io/2018/01/31/%E3%80%8AOSGi%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B%E3%80%8B%E7%AC%94%E8%AE%B0/)

<!-- more -->

# 走近Java模块化系统OSGi
## OSGi是什么
1. OSGi作为Java的模块化规范，其目标（也是软件设计的目标）是复用、内聚、耦合。
2. 被神化的“动态性”、“热插拔”的特性，是OSGi规范带来的一种可能，需要配合好的设计才能达到，并不是一定会有的。
3. OSGi是不是一个应用层面的框架（如spring、structs等），而是设计层面规范，类似面向对象的设计。所以，不要问“怎么将spring和OSGI集成？”这样的问题。
4. OSGi的目的是进行模块化，模块（bundle）的物理形式是Jar包。
5. OSGi的课程主要是让你去设计应用的，是形而上需要个人领悟的，不是去学一套应用框架那种。

## OSGi framework
OSGi规范定义了一个OSGi framework平台，它是运行在JVM上的应用，负责管理模块bundle。

### bundle生命周期
bundle需要关注它的生命周期。
1. install：框架通过classloader（类加载器）来装载bundle里的类和资源。
2. resolve：检查bundle依赖的package是否可用。
3. start：运行activator里的start方法，方法运行完毕后进入"ACTIVE"状态，至此bundle可用正式使用了。
![生命周期层状态转移](/images/00006/生命周期层状态转移.png)

### bundle的隔离
1. 模块以Jar包形式存在，Jar包的物理边界也是模块的物理边界。
2. 在OSGi规范下，得显式地说明模块之间的依赖关系。OSGi是利用JVM的classloader和它的父委托模型（PDM:Parent Delegation Mode）来实现这点的。
3. 每个bundle都分别用一个classloader来加载里面的类，所以不同bundle之间的类，在默认情况下，是互不可见的。
4. 如果没有额外处理，一个bundle里的类要访问另一个bundle里的类时，通常会出现ClassNotFound的异常。

### bundle之间的耦合
方式一：import/export package的机制
* OSGi通过import/export package的机制来控制bundle间有限地耦合。
* Export/Import package是通过bundle里的META-INF/manifest.mf文件里指定的。

方式二：OSGi service方式(更松散的耦合)
* OSGi framework实现服务的注册、查找和使用。该服务是实现某种接口的bean实例。
* 该服务是实现某种接口的bean实例，所以本质上osgi service就是一个bean。
* 通常会把接口定义在bundle A里，接口的实现则在bundle B里，并将接口实现实例化后注册成osgi service，bundle C可以引用这个service。bundle A需export接口定义所在的package，而bundle B和C则需import这个package。bundle B和C之间就不需用export/import package来耦合了，实现了B和C之间的解耦。
* OSGi应用中，会有大量的osgi service存在，可以说osgi service是osgi规范中最重要的机制，没有之一。

### 其它机制
OSGi规范还提供了Event、配置管理（ConfigAdmin）、声明式服务（Declarative  Service）、Service Tracker、Blueprint等等运行时机制，方便我们构建模块化的应用系统。

# 创建OSGi Hello World工程
本节内容参考了[《创建OSGi Hello World工程》](https://www.tianmaying.com/tutorial/osgi-helloworld)，需要注意的是，这篇文章中maven配置文件pom.xml中的`<name/>`标签需要进行修改，不能以冒号分隔，示例如下：
```
<name>osgi-demo</name>
   
```

## 模块构建
1.创建Maven项目。
2.在包中创建 OSGI Activator。

```
package mindw.osgi.demo;

import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;

public class MyActivator implements BundleActivator {

	@Override
	public void start(BundleContext context) throws Exception {
		 System.out.println("Hello world!");
	}

	@Override
	public void stop(BundleContext context) throws Exception {
		 System.out.println("Stop bundle!");
	}

}
```

3.配置pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:pom="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <groupId>mindw.osgi</groupId>
    <artifactId>demo</artifactId>
    <packaging>jar</packaging>
    <version>0.1</version>
    
    <name>osgi-demo</name>
   
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                        </manifest>
                        <manifestEntries>
                            <Class-Path>${project.build.finalName}.jar</Class-Path>
                            <Built-By>Ponder</Built-By>
                            <Bundle-ManifestVersion>2</Bundle-ManifestVersion>
                            <Bundle-Name>${project.groupId}.${project.ArtifactId}</Bundle-Name>
                            <Bundle-SymbolicName>${project.name}</Bundle-SymbolicName>
                            <Bundle-Version>${project.version}</Bundle-Version>
                            <Bundle-Vendor>${project.groupId}</Bundle-Vendor>
                            <Bundle-Activator>${project.groupId}.${project.ArtifactId}.MyActivator</Bundle-Activator>
                            <Export-Package>${project.groupId}.${project.ArtifactId};version="0.1"</Export-Package>
                            <Import-Package>org.osgi.framework</Import-Package>
                        </manifestEntries>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>org.osgi</groupId>
            <artifactId>org.osgi.core</artifactId>
            <version>4.2.0</version>
            <type>jar</type>
        </dependency>
    </dependencies>
</project>

```

4打包生成 demo-0.1.jar。

## servicemix安装
为了使用模块，需要一个支持模块化运行的环境，参考文章的作者使用了servicemix。
### servicemix介绍
* Apache ServiceMix是一个灵活、开源的集成容器。
* 它可以将Apache ActiveMQ、Camel、CXF和Karaf集成为一个强大的运行平台。
* 你可以通过它来建立自己的集成解决方案。
* 它提供了一个由OSGi支持的完整、企业级的ESB（企业服务总线）。

### 安装
* 去[servicemix官网](http://servicemix.apache.org/)下载zip压缩文件。（版本都可，5.4.0版本86M左右，7.0.1版本113M左右）。
* 解压压缩包即可。

## 模块使用（生命周期变化）
* 将demo-0.1.jar放入servicemix的deploy目录下，本人放在 apache-servicemix-5.4.0\deploy 中。
* 进入 apache-servicemix-5.4.0\bin 目录，运行 servicemix.bat。可以看到以下界面。"Hello world!"表示模块已经加载。
![模块启动](/images/00006/模块启动.png)
* 使用`list`命令，可以查看模块状态。可以看到模块ID为220,状态为 Active。
![模块状态查看](/images/00006/模块状态查看.png)
* 输入`stop 220`，可以看到"Stop bundle!",然后再输入`list`，可以看到当前模块处于Resolved状态。
![模块停止](/images/00006/模块停止.png)
* 输入`uninstall 220`，模块就被卸载了，再输入`list`就看不到模块信息了。
* conrtrol + d 可以结束servicemix容器运行。

# Package与服务
该部分可以参考[《OSGi中Bundle间的耦合：Export/Import Package与服务》](https://www.tianmaying.com/tutorial/osgi-bundle-coupling)。
## 注册服务
在BundleActivator的start函数中可以通过BundleContext注册服务。
```
public class MyActivator implements BundleActivator {

	@Override
	public void start(BundleContext context) throws Exception {
		Dictionary<String, String> props = new Hashtable<String, String>();
		props.put("ServiceName", "Calculation");
		context.registerService(ICalculation.class.getName(), new Calculation(), props);
		System.out.println("Service registered!");
	}

	...
}
```
## 引用服务
在BundleActivator的start函数中可以通过BundleContext获得服务。
```
public class MyActivator implements BundleActivator {

	@Override
	public void start(BundleContext context) throws Exception {
		 ServiceReference[] refs = context.getServiceReferences(ICalculation.class.getName(), "(ServiceName=Calculation)");
         System.out.println("demo3:"+ICalculation.class.getName());
		 if(refs!=null && refs.length>0){
             ICalculation service=(ICalculation)context.getService(refs[0]);
             System.out.println("1+1="+service.add(1, 1));
             System.out.println("2-1="+service.sub(2, 1));
             System.out.println("2*3="+service.sub(2, 3));
         }
	}
	...
}
```

## 依赖疑问
在开发中，Demo2中提供服务（包含接口定义），Demo3中使用服务（不包含接口，但依赖接口），但是编译器环境中OSGi模块依赖不能解析，如何表明依赖？翻看作者代码后，发现只要正常添加maven依赖即可，打包的时候不用包含进去。

# 动态的OSGi服务
该部分可以参考[《动态的OSGi服务》](https://www.tianmaying.com/tutorial/osgi-service)。包括：
* OSGI服务的动态性
* Service Listener
* Service Tracker

# 初次接触OSGI Blueprint
该部分可以参考[《初次接触OSGI Blueprint》](https://www.tianmaying.com/tutorial/osgi-blueprint)。包括：
* Blueprint简介
* Blueprint的入门例子：OSGI服务的注册
* Blueprint的入门例子：OSGI服务的引用
* OSGI blueprint的机制原理

# OSGi的配置管理：ConfigAdmin
该部分可以参考[《OSGi的配置管理：ConfigAdmin》](https://www.tianmaying.com/tutorial/osgi-configadmin-service)。包括：
* 动态的OSGI配置
* 利用blueprint来实现ConfigAdmin

# OSGi还有用武之地吗？
* 正面观点，可以参考[《简单了解osgi》](http://blog.csdn.net/u013851082/article/details/70214829)。
* 反面观点，可以参考[《osgi确实面临鸡肋之嫌》](http://blog.csdn.net/itd018/article/details/51035176)。

本人在网上搜索博客的时候，发现OSGi近几年的博客介绍很少，关注度不高，而且学习的OSGi中文网站2015年后也不怎么更新文章了。在分布式微服务流行的现在，OSGi的确有些尴尬，但是个人感觉对于一些复杂的单体应用，OSGi还是有用武之地的，比如Java 9学习了OSGi，增加了模块化功能并重构了Java库。此外，OSGi解决模块依赖的方法和基于接口和模块的设计理念还是很有价值的。

接下来将对反面观点进行一下介绍。

## 明显不足
OSGi的关注不是很高，该文作者也谈了它本身的一些不足：相对来说适合单体应用（但单体应用也可以通过容器的等帮助部署），不适合分布式。具体如下所示。
### 关于隔离与奔溃
OSGi最终基于jvm，无法对bundle对应的服务实现计算资源的隔离，一个服务的故障依然会导致整个jvm crush,这使得在一个运行时的osgi上部署模块级服务只获得了模块部署和启停隔离。
### 关于扩展能力
服务明确依赖的好处，但是没办法实现计算节点的线性扩展，在当前分布式，微服务，网络计算的趋势下，使得osgi只适合构建单一服务节点的内部应用，但是其分离的bundle的部署负担对于微服务架构来说，有点用大炮打蚊子的臭味。

## 推荐的应用架构方式
### 目标
* 采用“进程间构建的分布式应用”和“进程内的单一应用”分开来进行架构设计。
* 对于进程间构建的分布式应用，采取基于soa的理念进行容器模式的服务部署模式，服务交互基于远程服务交互相关协议，采用可忍受网络失败的架构设计原则。
* 对于进程内的应用，如果需要模块级的独立生命周期热部署和模块管理，可以考虑采用OSGi。

### 容器更方便
但是，容器内基于本地进程间通信的模块交付方式不仅能提供同样的独立生命热部署和模块管理，而且具备随时脱离出去部署成单独容器级服务应用的能力，加速进程间的服务交付提供的整体管理和监视环境基础。

### OSGi比较适合嵌入式
 OSGi还有用武之地吗？当然我前述都是以构建分布式企业和面向互联网这类应用为前提来讨论的，对于嵌入式的jvm应用，比如著名的osgi案例宝马的车载系统，osgi依然是最好的原则，不过我怀疑基于andriod系统的机制构建类似应用，osgi的采用依然值得商榷。因此，osgi确实面临鸡肋之嫌。

