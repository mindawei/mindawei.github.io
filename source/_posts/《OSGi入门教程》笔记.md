---
title: 《OSGi入门教程》笔记
date: 2018-01-31 17:52:51
tags: [笔记,OSGi,模块化,Java] 
categories: 模块化
---

本文是对[《OSGi入门教程》](https://course.tianmaying.com/osgi-toturial+osgi-concept#0)课程的笔记。

# OSGi基础概念

## 基本概念
### OSGi的一些定义
* Open Services Gateway initative 开发服务网关协议
* 是指Java的动态模块化系统的一系列规范。（OSGi联盟，osgi.org）
* OSGi Alliance 组织 以及该组织指制定的一个基于Java语言的服务规范。(Wiki)
* Java 平台的模块层。(《OSGi in Action》) 

<!-- more -->

### OSGi生态
![OSGi生态.png](/images/00006/OSGi生态.png)

### 模块化
将大型系统分解为多个模块，通过设置模块的边界来改善系统的可维护性和封装性。

## Java模块化的局限性
### 可见性控制不够
* 通过Package来组织和划分代码，可见范围可以分为Private、Package、Protected、Public这几类。
* 存在的问题：如果是Public的话，任何人都可以访问，希望可以进一步控制（一种方案可以是通过一些模式进行模拟实现，另一种可以是OSGi）。

### Jar包灾难
* ClassPath中的Jar包在运行时没有明确地模块边界。
* 多个版本的Jar同时存在时，加载具有不缺定性。Jar Hell（Jar包灾难）。

### 部署和管理缺少支持
* 模块的动态更、系统动态演化。
* 插件化开发。
 
## OSGi三层架构
### 模块层：定义了OSGi中的模块Bundle。
* Bundle是一个具有额外元数据的Jar包。
* Bundle定义了包含的包的可见性（对外暴露哪些模块）。
* Bundle定义了所依赖的包。

![Bundle所包含内容.png](/images/00006/Bundle所包含内容.png)

### 生命周期层：提供运行时管理和框架访问接口。
* 提供对模块生命周期的操作（install,update,start,stop...），使得程序外部可以动态管理和演化系统。
* 定义了运行时上下文的接口，bundle本身可以和OSGi框架进行交互，从而实现内部自我管理。

### 服务层：关注模块间的交互。
* 服务层是JVM中的SOA（面向服务的架构）。
* 服务是动态的，促使你使用基于接口编程。
* OSGi服务是Java Interface，服务调用是Java对象方法调用（Lightweight Tech,轻量级技术）。

## OSGi介绍补充
### OSGi严格的模块化特性的最大优势
* 基于接口编程，完全隐藏实现（促进你从架构上思考）。
* 动态新（对扩展开发，即使是运行时）。

### OSGi传播的阻力
* 最初面向的嵌入式领域，容易被误解为只是嵌入式技术。
* 有些人觉得OSGi复杂，人为属于重量级。
* Lib支持不够：很多Lib不能再OSGi环境下运行。

### 企业中的OSGi
* 企业应用特点：持久化数据、数据量大、访问并发等。
* 需要模块化：前端/事务/持久化分离、运行多个服务器上、需要协作化开发。
* 存在问题：企业开发中其它框架会使用TCCL（SPI模式）、反射，这些技术在OSGi类加载机制下会有问题。

## 小节
1. OSGi提供了更粗粒度的模块化特征，可以解决Java模块化的局限性。
2. OSGi中声明式和基于元数据的方法是非侵入式的。
3. 生命周期层定义了模块动态且可控的生命周期模型，简化了系统管理。
4. 服务层鼓励采用基于接口编程的方法，从而将接口与实现进行分类。

# OSGi模块层
## 模块化
### 与面向对象的关系
* 都是“关注点分离”（分治）思想的体现，但关注的粒度不一样。
* 在实现特定功能时，需要设计类以及类之间的关系，此时需要面向对象的原则和模式。
* 当把相关的类在逻辑上组织在一起的时候，需要关注系统模块和模块间的关系。
![模块化设计.png](/images/00006/模块化设计.png)

### 模块化的意义
* 解决Java模块化的局限：可见性控制不够、Jar包灾难、部署和管理缺少支持。
* 通过显示定义能力（Export-Package）和依赖（Import-Package）,可以优化设计，让系统更加“高内聚低耦合”（软件开发的终极目标）。
* 促进协同开发，提高开发效率。

## Bundle
### 基本概念
* Bundle是一个包含代码、资源、元数据，以Jar的形式存在的模块化单元。
* Jar文件的边界也是模块的边界。Jar包是bundle中代码的物理容器。
* Mainifest.mf文件保存了模块的元数据。

### 元数据定义
* 元数据信息定义在/META-INF/MANIFEST.MF中，OSGi R5规范定义了28个标记。
* 元数据有三类标记：可读信息、Bundle标识（Identification）、代码可见性。

## 依赖解析
### OSGi类查找顺序
* 如果类所在包以“java.”开头，委托给父类加载器。
* 如果类所在的包在导入包中，委托给导出该包的Bundle。
* 在Bundle自身的类路径上查找。

### 多个提供者的选取规则
* 已解析的（resolved）的bundle优先级高，未解析的（installed）bundle优先级低。
* 相同优先级多个匹配时，版本高者优先，版本相同则先安装的优先。
![解析问题.png](/images/00006/解析问题.png)

### uses子句使用
1. 使用uses子句来解决类空间不一致的问题。
2. uses约束是可传递的。
3. 谨慎使用uses,会大大限制解析的灵活性。
4. 使用场景如下。
* 导出包中的类，其方法签名中包含了其Import-Package中的类。
* 导出包中的类，继承了其Import-Packeage中的类。

![use子句使用](/images/00006/use子句使用.png)

# OSGi生命周期层
## 基本概念
### 生命周期管理
* 通过外部或内部对应用进行操作，完成对应用的“生命周期管理”过程。
* 对于非模块应用，这些操作是以整个应用为对象。
* 对于模块化应用，可以有更细粒度（针对某个模块）的生命周期管理。
![生命周期.png](/images/00006/生命周期.png)

### 生命周期层的作用
* 在应用外部，该层精确定义了对bundle生命周期的相关操作。
* 在应用内部，该层定义了bundle访问其执行上下文的方式，为bundle提供了一种与OSGi框架交互的途径。
* 对生命周期的操作允许你动态地改变运行于框架汇中的bundle组成，并以此来管理和演化应用程序。

## 生命周期层状态转移

![生命周期层状态转移](/images/00006/生命周期层状态转移.png)

## 使用生命周期层
* BundleActivator是生命周期层的基础设施,如下所示。
```
public interface BundleActivator{
	public void start(BundleContext context) throws Exception;
	public void stop(BundleContext context) throws Exception;
}
```
* bundle属于active时，BundleContext才有意义，即start方法被调用和stop方法被调用的两个时间点之间。
* BundleContext包含部署和生命周期管理相关接口、与bundle间服务交互相关的接口。
* Bundle定义了一系列API,用于管理已安装的bundle的生命周期。

![BundleContext接口](/images/00006/BundleContext接口.png)

![Bundle接口](/images/00006/Bundle接口.png)

## Bundle的更新
### 两阶段更新
* 两阶段更新：先update,再显示refresh。
* 为什么两阶段？对外输出后，其它模块会使用旧版本，需要刷新，可参考[《bundle动态更新》](http://blog.csdn.net/saloon_yuan/article/details/7902569)。
![Bundle更新](/images/00006/Bundle更新.png)


### 刷新流程
1. 从bundle开始计算受影响的bundle有向图。
2. 处于ACTIVE状态的bundle被停止并被切换至RESOLVED状态。
3. 处于RESOLVED状态的bundle，切换至INSTALLED状态，这些bundle的依赖关系不再被解析。
4. 处于UNINSTALLED状态的bundle会从图中移除，同时也会被彻底地从框架中移除（GC）。
5. 其它bundle如果在框架重启前处于ACTIVE状态，重启框架会对这些bundle及其依赖的bundle进行解析。
6. 所有工作完成后，框架会触发一个FrameworkEvent.PACKAGES_REFRESHED事件。

## 小节
1. BundleActivator是bundle的入口，与Java应用中的main函数类似。
2. BundleContext为应用提供执行时操作OSGi框架的方法。
3. Bundle代表了一个已安装到框架中的bundle,允许对其执行状态进行操作。

# OSGi服务层
## 基本概念
### 什么是服务
* 为别人完成的工作（经典定义）。
* 指提供者及其使用者之间的一个契约。服务可以被替代，能够发布和查找。
* 使用者不关心具体实现，只关心约定的契约。

### 面向服务的设计
* 降低服务提供者和使用者之间的耦合，更容易重用组件。
* 更强调抽象接口而不是具体实现。
* 清晰描述依赖关系（可以通过附加元数据描述）。
* 支持服务的多个实现方案以及动态替换。

## OSGi服务 
### OSGi服务模型
* 拥有一个集中的服务注册中心，遵循发布-查询-绑定模型。
* 提供者bundle可以将POJOs（Plain Ordinary Java Object）发布为服务。
* 使用者bundle可以找到并绑定服务。
![OSGi服务](/images/00006/OSGi服务.png)

### OSGi服务注册、更新与注销
* 服务注册对象是私有的，不能被别的bundle共享，它们与发布服务的bundle的生命周期是绑定的。
* 不推荐使用具体类名进行服务注册。
* 当一个bundle停止时，任何没有被移除的服务都会被框架自动移除。
* 服务排序：先按service.ranking由大到小排序，然后再按service.id由小到大排序。
* 按属性查询：使用LDAP过滤字符串（LDAP，Lightweight Directory Access Protocol，轻量目录访问协议）。

### 服务使用
```
// 注册中心会增加一个使用计数。
A serviceA = (A)bundleContext.getService(reference); 
// 完成服务时应该通知注册中心。
bundleContext.ungetService(reference);  
```

### 服务监听
服务可以监听的事件包括：REGISTERED（注册）、MODIFIED（更改）、UNREGISTERING（注销）。
![服务监听事件](/images/00006/服务监听事件.png)

# OSGi开发环境
略，<a href="https://course.tianmaying.com/osgi-toturial+osgi-development-environment#0">点击可查看原文</a>。













