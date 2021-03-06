---
title: Java 9，OSGi和模块化的未来（1）
date: 2018-02-05 13:53:50
tags: [翻译,OSGi,模块化,Java] 
categories: 模块化
---

Java 9 中一个重要的新特性就是模块化。它的实现机制是什么那？它和已有的模块框架OSGi有什么差异那？为了回答这些问题，本人在网上找到了一篇比较好的介绍文章，为了加深理解，对文章进行了翻译。由于原文分为2个部分，所以翻译对应也分为2篇：

1）[《Java 9，OSGi和模块化的未来（1）》](https://mindawei.github.io/2018/02/05/Java-9%EF%BC%8COSGi%E5%92%8C%E6%A8%A1%E5%9D%97%E5%8C%96%E7%9A%84%E6%9C%AA%E6%9D%A5%EF%BC%881%EF%BC%89/)是对[《Java 9, OSGi and the Future of Modularity (Part 1)》](https://www.infoq.com/articles/java9-osgi-future-modularity)的翻译，文章日期为2016年9月22日。介绍的内容包括：背景、高层次比较、复杂性、依赖粒度对比、模块导出对比、模块导入对比、反射和服务。

2）[《Java 9，OSGi和模块化的未来（2）》](https://mindawei.github.io/2018/02/06/Java-9%EF%BC%8COSGi%E5%92%8C%E6%A8%A1%E5%9D%97%E5%8C%96%E7%9A%84%E6%9C%AA%E6%9D%A5%EF%BC%882%EF%BC%89/)是对[《Java 9, OSGi and the Future of Modularity (Part 2)》](https://www.infoq.com/articles/java9-osgi-future-modularity-part-2)的翻译，文章日期为2016年10月4日。介绍的内容包括：动态性、二者协同工作、未来发展、结论。

__本文是对原文第一部分的翻译。__

<!-- more -->

# 引言
关键点
* Java 9 在2017年发布，其中一个重要的特性就是新的模块化系统，被称作Java平台模块系统（Java Platform Module System，JPMS）。本文将介绍它与Java已有的模块标准化机制OSGi的关系，以及对OSGi的影响。
* 自1.0版本以来Java平台已经增长了20倍，整个平台存在着模块化的需求。为了解决这个问题，进行了很多不成功的尝试。与之相对的是，OSGi已经提供了16年的应用模块化服务。
* OSGi和JPMS在实现细节上差别很大。如果将JPMS作为模块化的一般解决方案，会存在一些重大的缺陷，并且缺少OSGi的一些特性。
* JMPS的目标是比OSGi更简单和更容易，但是对一个非模块化的产品进行模块化设计本身就是一件很复杂的事情，JMPS看起来好像还没有实现这个目标。
* JMPS在对Java平台本身的模块化方面做得非常好，这意味着我们可以在运行时只加载和任务相关的Java平台组件。对于应用模块化，OSGi有很多的优点。我们已经证明这两者可以很好地结合在一起。


这篇文章是“Java 9, OSGi and the Future of Modularity”文章系列的第一部分。第二部分请查看[《Java 9, OSGi and the Future of Modularity (Part 2)》](https://www.infoq.com/articles/java9-osgi-future-modularity-part-2)。

当Java 9发布的时候，其中一个重要的特性就是模块化：Java平台模块系统（Java Platform Module System，JPMS）。虽然JPMS的具体细节还不是很清楚，但我们已经对其基本内容有了一定的了解。

从2000年开始，Java中的模块系统就以各种形式存在了。其中一个比较著名的模块化系统叫做OSGi，这是一个独立于供应商的行业标准，由OSGi联盟制定。OSGi联盟由领先的软件供应商、电信公司和其他组织组成，包括：Adobe、Bosch、Huawei、IBM、Liferay、NTT、Oracle、Paremus、Software AG。它几乎能够支持所有的Java EE应用服务器、最流行的IDE、web应用（eBay、Salesforce.com、Liferay），并且它也被政府和军方使用（the U.S. Air Force、Federal Aviation Administration）。

OSGi适用于IoT（物联网）。一开始，OSGi是为嵌入式系统设计的。在几年前，这些系统的内存和CPU资源都是很有限的。现在，这些设备具备更多的资源，这为构建复杂的应用系统提供了机会，并且催生一大批企业和个人对软件和硬件进行着贡献，形成了一个生态。这样的生态系统存在于多个市场，包括：智能家居联网、车联网、智慧城市、工业4.0（IIoT）。此外，网关常用于传感器与设备、设备与后端系统的互联。应用和服务可以在本地网关运行，也可以在云端运行。

OSGi提供了很多规范，从而使得构建开放物联网生态的基本特性能够实现。这些特性包括：设备管理、软件配置、对设备的底层通信协议进行抽象。一些公司，包括：AT&T、Bosch、NTT、Deutsche Telekom、General Electric、Hitachi、Miele、Schneider Electric等，在构建IoT解决方案的时候，可以通过使用OSGi技术获得更好的收益，而这也有段时间了。通过OSGi和IoT，现在已经是万物互联的社会了。

当然，OSGi使用者比较关注的是：Java 9中的模块化系统会对OSGi造成什么影响，包括短期和长期。

由于技术、政治、经济等原因，Java生态中存在了两个模块系统。在这篇文章中，我们避开了政治的原因，从技术的角度进行了比较。我们描绘了一个JPMS和OSGi一起工作的场景，在这个场景中，它们有各自擅长的领域和机会。

需要注意的是，本文使用了截至2016年8月能够获得的公开信息，有些细节可能在规范最终出来之前还会更改。

# 背景
Java平台从1990年代开始迅速发展。从下载大小来看，JDK 1.1的大小在 10Mb 之内，而 Mac OS X 平台下载的JDK 8u77 达到了227M。安装空间和内存需求也相应需要增加。Java平台扩大是由增加新特性驱动的，这些特性的大部分都是受欢迎和有用的。但是，平台中的每个新功能都会给不需要它的用户造成膨胀，并且没有人会使用到全部的特性。此外，为了使Java具备向后的兼容性，其中一些特征在废弃之后仍会被保留。

很多年来，Java平台变大并不是一个大问题。Java是最受欢迎的企业级平台，并且它的竞争对手（微软的 .NET）也和Java有着相似的发展轨迹。但是，现在Java面临了不同的挑战。IoT的发展使得空间占用被重新关注了起来，并且一些新的灵活的平台和语言（如Node.js、Go）也是非常厉害的竞争对手。

此外，安全同样是一个大问题：为了避免来自Java的攻击，有安全意识的组织会从用户桌面完全删除它。如果JVM内部和用户空间的应用代码有更好的隔离性，那么很多攻击可能就不会发生。

很长一段时间来，大家都很明确的是：需要对Java平台进行一些模块化工作。在2000年代中期，有过一些努力，但是都没被采用，包括JSR 294和它的“superpackages”、JSR 277的“Java Module System”。最终，一个称为 Jigsaw 的原型项目出现了。这本来打算在2011年的Java 7中发布的，但是后来推迟到了Java 8，接着又被推迟到了Java 9。作为一个原型工程，Jigsaw 为 JPMS 规范提供了实践参考。

与此同时，OSGi已经发展和改进了16年。OSGi是应用模块化的标准，由于它不是Java平台本身的一部分，所以它不能影响平台本身的模块化。但是另一方面，许多应用程序都得益于它在JVM级别之上提供的模块化模型。

# 高层次比较
JPMS和OSGi之间在细节上有很多不同点，但是它们之间最大的区别在于：实现隔离的方法。

对于任何一个模块化系统，隔离性都是最基本的特性。必须有一些方法来保护每个模块不受同一应用程序中其它运行模块的干扰。隔离并不是一个二元概念（隔离或不隔离），OSGi和JPMS都不能阻止一个模块干这些“坏事”：耗尽JVM中的可用内存、启动数千个线程、通过忙循环挂起CPU。将模块运行在OS独立的处理器中可以提供一些隔离保护，但是即使这样也不是万无一失的，一些人仍旧可以使OS奔溃或者擦除磁盘中的数据。

OSGi和JPMS提供的都是代码级别的隔离，这意味着一个模块不能随意访问另一个模块内部的类型，除非有另一个模块的显示允许。

OSGi通过类加载器来实现它的隔离性。每个模块（OSGi中的术语是“bundle”）有它自己的类加载器，这些类加载器知道如何加载bundle中的类型。此外，也可将类加载的请求委托给依赖的bundle。OSGi的系统是高度优化的，例如：OSGi直到bundle真正需要被使用的时候才会创建类加载器；并且事实上每个类加载器只需要加载部分类型，这也意味着加载速度可以有所提高。

这个系统有一个很大的优点是，多个bundle可以包含重复的包和类型，并且这些元素不会相互干扰。实际的效果是：可以同时在一个虚拟机上运行包或库的多个版本。当使用构建工具（例如 Maven）时，这个模块化机制有助于处理复杂的传递依赖。在很多企业级Java应用中，几乎不可能实现应用的每个依赖里面使用的都是同一个版本的类库。

例如，我们看一下JitWatch库。JitWatch 依赖 slf4j-api 1.7.7、logback-classic 1.1.2，但是 logback-classic 1.1.2 依赖 slf4j-api 1.7.6，与 JitWatch 直接依赖的版本发生了冲突。JitWatch 同样传递依赖 jansi 的1.6和1.9版本，并且如果我们包括测试域的依赖，我们还会依赖另一个1.6版本的slf4j-api。这种混乱状况非常常见，传统的Java并没有真正有效的解决办法，除了在依赖树上逐渐添加“excludes”，直到我们可以奇迹般地让一堆依赖工作。但不幸的是，JPMS对此还没有确切的解决方案，等Java 9发布后我们就可以看到是否如此。

正是这个缺点阻止了OSGi被用来模块化JDK本身。JDK的很多部分都有一个隐含的假设就是：任何一个JDK类型都可以从JDK的其它部分中加载进来。因此，如果使用类似OSGi的模型的话，很多事情都无法完成。为了解决这个问题，同时为了可以轻松地从使用 `Class.forName` 的代码迁移过来，JPMS并没有全部选用类加载机制来实现模块的隔离。当你启动了一个应用，并且使用了“modulepath”下的一些模块，那么这些模块会被同一个类加载器加载。JPMS引入了一个新的访问机制来实现这个目标。

这个隔离栅栏在OSGi中被称为可见性。在OSGi中，我们不能加载模块内部的类，因为这在外部是不可见的。这也意味着，我模块的类加载器只能看到我自己模块内的类型、以及从其它模块明确引入的类型。如果我尝试从你的模块中加载一个类，我的模块是无法看见那些类型的，看起来好像那些类并不存在一样。如果我无论如何一定要加载那些类的话，会得到 `NoClassDefFoundError` 或者 `ClassNotFoundException` 。

在JPMS中，每个类型可以看到其它任何类型，因为它们都在一个类加载器中。但JPMS增加了额外的检查，从而确保尝试加载的类有正确的访问权限。其它模块中的内部类型实际上是私有的，即使被声明为 public。我们如果强制要加载的话，会得到 `IllegalAccessError` 和 `IllegalAccessException`。加载其它模块中的私有类型和默认类型也会得到同样的错误，并且调用 `setAccessible` 这样的函数是没有作用的。这改变了Java中public的语义，原先意味着任何地方都可以访问，现在意味着只有在模块内部或者声明了require的地方才可见。

JPMS这种做法的缺陷是：不能实现多个模块都加载重叠的内容。这意味着，如果两个模块都包含一个私有（没有导出）的包 `org.example.util`，那么这些模块不能在模块路径下被同时加载，这会导致 `LayerInstantiationException`。虽然我们可以在自己的应用中创建类加载器来绕过这个限制，但是OSGi已经为我们实现了。

这完全是通过设计，来使得JPMS可以对JDK内部进行模块化。但是这样造成的影响是，你不能让内部实现细节冲突的模块一起工作。

# 复杂性
关于OSGi，大家经常抱怨的一点是：它增加了开发复杂度。这有一些道理，但是提出这些抱怨的人可能没有完全弄清状况。

模块化并不是一个神奇到只要在发布前一刻添加上去就能使用的技术。它是一个必须贯穿于设计和开发的所有阶段的理念和原则。如果开发者较早采用OSGi，或者在开发前就已经考虑了模块化设计，那这会有很多益处。并且他们会发现，OSGi实际上非常简单，尤其是还可以使用先进的OSGi工具，这些工具可以帮助开发者自动生成元数据，并可以在运行前检查出一致性方面的错误。

另一方面，那些想要在现有大型项目中使用OSGi的人之所以遭受痛苦，是因为那些代码很少有良好的模块化设计，这会导致迁移工作很困难。如果没有强制性的模块化开发原则，那么很容易导致这样的情况：为了开发方便而破除封装性。一个BEA WebLogic的开发人员告诉我，在BEA被Oracle收购之前：“我们一直以为我们的产品是模块化设计的，但是当我们开始使用OSGi时，我们改变了之前的看法。”

除了非模块化的应用之外，非模块化的库也阻碍了OSGi的使用。一些非常流行的Java库都基于类加载机制和全局可见性的假设，而这些假设在模块化结构中不能成立。OSGi为了能够使用这样的类库做了很多工作，而这也是OSGi显得复杂的原因之一。事实上，我们需要一些复杂的技术来处理混乱的局面，因为真正复杂的是现实世界。

在JPMS下也会出现同样的问题，可能更多，我们很快就会看到。如果你所在的组织之前曾尝试采用OSGi，但是因为迁移的工作量而放弃，那么你们如果打算迁移到JPMS的话，工作量至少会同样多。你只需看看Oracle对JDK进行模块化的经验：工作量是如此之大，以至于Jigsaw从Java 7被推迟到Java 8，之后又被推迟到了Java 9，即使Java 9已经延迟一年发布了。

Jigsaw是以简洁性为目标开始的，但是JPMS规范已经变得越来越复杂：模块与类加载器的相互作用、分层与配置、再次导出依赖、弱模块、静态依赖、规范的导出、动态导出、跨层的读可继承性、多个Jar包的多个模块、自动化模块、未命名模块，等等。这些所有的特性都被添加到规范中去了，因为它们需要被清晰地说明。OSGi也发生了相似的过程，只是领先了16年而已。

# 依赖：包 vs 模块
隔离只是完成了模块化难题的一半，另一半是：模块需要一起协同工作。在构建了模块之间的隔离之后，我们还需要引进一个可以控制的交流机制。这个可以在类型级别静态实现，也可以通过对象动态实现。

静态依赖是指那些在构建时期就能被知道和控制的部分。如果需要穿过模块的边界访问一个类型，那么模块需要提供一种提供类型可见性和访问控制的方法。这包括两个方面：模块需要选择性地暴露一些它们封装的类型，并且它们需要明确指出需要使用其它模块中的哪些类型。

# 导出：Exports
在OSGi和JPMS中。暴露类型的粒度都是基于Java包的。在OSGi中，我们使用 Export-Package 声明，它可以确切表明哪些包是可以被其它模块可见的。示例如下：
```
Export-Package: org.example.foo; version=1.0.1,
	  org.example.bar; version=2.1.0
```
这个声明在 META-INF/MANIFEST.MF 文件中。在早期，一些OSGi开发者需要手动地配置这些声明，但是逐渐地，我们更喜欢使用构建工具来生成。目前最流行的模式是在Java源代码中使用注解。在Java 5中 package-info.java 被引入，并且允许包级别的注解和文档，所以在OSGi中我们可以有以下写法：
```
@org.osgi.annotation.versioning.Version("1.0.1")
	package org.example.foo;
```
这是一个有用的模式，因为导出包的意图可以直接在该包中表示。版本号也可以在这里声明，而且当包中内容发生变动时，也很容易在附件进行修改。

在JPMS中，包导出声明是在 module-info.java 文件中，举例如下：
```
	module A {
		exports org.example.foo;
		exports org.example.bar;
	}
```
需要注意的是：在JPMS中，模块和包都无法标注版本号。这部分内容我们之后会再次讨论。

# 导入：Imports/Requires
OSGi和JPMS在模块导出的部分很相似，但是它们在导入或者依赖其它模块的部分差异很大。

在OSGi中，导入包和导出包是互补的。我们使用 Import-Package 声明来导入包，例如：
```
Import-Package: org.example.foo; version='[1,2)',
	  org.example.bar; version='[2.0,2.1)'
```
导入的规则是：OSGi的bundle需要导入每个依赖的包，除了以 `java.*` 开始的包（例如，`java.util`）。具体来说，如果bundle中的代码依赖了类型 `org.slf4j.Logger` (并且bundle中没有包括 `org.slf4j` 包)，那么这个包需要在导入列表中标明。同样，如果你依赖 `org.w3c.dom.Element`，那么你必须导入 `org.w3c.dom`。但是如果你需要依赖 `java.math.BigInteger` 的话，你__不需要__导入 `java.math`，因为 `java.*` 的包是由 JVM 的 `bootstrap` 类加载器加载的。

OSGi存在并行机制来导入所有的模块，被称作 Require-Bundle，但是这个在OSGi规范中已经废弃了，存在也只是为了支持一些非常稀少的边缘案例。Import-Package 机制的最大优点是：可以在不影响下游模块的情况下，实现模块的重构和重命名。图1和图2对此进行了说明。

在图1中，模块A被重构为两个模块，A和A'，但是模块B没有受到这个操作的影响，因为它依赖的是包。在图2中，我们对A进行了同样的操作，但是现在B奔溃了，因为它可能使用的包在A中不再存在了（这里说“可能”是因为我们无法知道B使用了A中哪些部分，因为我们只说明了依赖的模块，这就是问题所在）。
![图1](/images/00007/图1.jpg "图1 Import机制中重构模块")

![图2](/images/00007/图2.jpg "图2 Require机制中重构模块")

Import-Package 声明手动书写很麻烦，所以我们并没有这么做。OSGi工具可以通过检查bundle中编译类型的依赖来生成它。这是非常可靠的，比开发人员自己声明运行时依赖要可靠得多。当然，开发者还是需要管理它们的构建依赖，这通常可以通过Maven进行管理（或者你可以自己选择构建工具）。在构建期间，如果你在类路径上放置了很多依赖，这并不会造成特别大的问题：最糟糕的情况也只是编译失败，而这仅会对开发者造成影响，并且很容易被修复。另一方面，在运行时如果有太多依赖的话会降低模块的可移植性，因为所有的这些依赖都需要被移植，这可能会与移植环境中其它模块的依赖形成冲突。

这引出了OSGi和JPMS之间的另一个重要哲学差异。在OSGi中，我们总是意识到构建时期的依赖和运行时的依赖常常会不同。例如，通常我们需要构建API和运行API的具体实现。进一步，开发者通常会构建老版本的API来保持兼容，而我们在运行时会选择最新的实现版本。即使是非OSGi的开发者也会对此非常熟悉：通常构建的时候会支持低版本的JDK，但还是会鼓励使用者运行高版本的JDK，因为高版本中会有一些安全补丁，并对运行能力有所增强。

另一方面，JPMS则采取了不同的做法。JPMS的目标是实现“所有阶段的保真度”, 所以“模块系统应该在编译时、运行时以及在开发或部署的每个阶段中，都以完全相同的方式工作”（摘自 [JPMS Requirements](http://openjdk.java.net/projects/jigsaw/spec/reqs/#fidelity-across-all-phases)）。因此运行时依赖是以整个模块为粒度定义的，因为这样能够和它们编译时期的定义保持一致。示例如下：
```
	module B {
		require A;
	}
```

这个 require 声明和OSGi中被废弃的 Require-Bundle 有着相同的效果：模块A中所有被导出的包都可以在模块B中使用。这也导致了它和 Require-Bundle 有着相同的问题，从模块依赖声明当中没有办法得知：对A中内容进行重构是否安全。所以，通常来说这样做并不安全。

我们发现在依赖关系的树形结构中，使用 require 的声明会比使用 imports 的声明具有更大的出度（节点出去的边数）：每个模块需要关注比实际需要的更多的依赖。这些问题是真实并且重要的。Eclipse的插件开发者尤其受到了它们的影响，因为历史原因，Eclipse bundles 往往使用 require 而不是 import。我们觉得JPMS采用这种方案是很不幸的一件事情。有趣的是，尽管编译/运行时保真是JPMS的基本目标，但是JPMS最近的一些改变显著地降低了保真度。当前早期的一个版本允许require声明使用静态修饰符，这意味着依赖在编译时强制性的，但是在运行时是可选的。相反的是，export 声明可以被动态修饰符修饰，这意味着导出的包在编译时期是无法访问的，但是在运行时期是可以访问的（通过反射）。这些特性会导致这样的情况：在编译时期可以成功地创建编译和链接模块，但是在运行时期会抛出 `IllegalAccessError` / `Exception` 。

# 反射和服务
Java的生态系统非常巨大，包含一系列用于各种目的的框架，如依赖注入、模拟对象、远程调用、对象映射等等。这些框架中的大部分都使用了反射来实例化和管理用户代码中的对象。例如，Java 持久化框架（JPA），它是Java EE规范中的一部分：作为一个 O/R 映射器，它需要从用户的代码中加载和实例化领域类，从而使得这些实例能够匹配从数据库中加载到的记录。另一个例子是，Spring 框架中会加载和实例化一些接口的实现类 “bean”。

这会对模块化系统带来问题，包括OSGi和JPMS。理想情况下，一个领域或对象类应该被隐藏在模块中：如果它被暴露了，那么它成了一个公共的API，如果有消费者依赖它的话，可能会对它造成影响，而我们希望能够按照意愿灵活地改变我们的内部类。另一方面，可以通过反射来访问一些没有被模块导出的类型，这对于一些框架是非常有用的。

由于OSGi是基于类加载机制设计的，模块可以获得那些没有导出包和类型的模块的可见性，只要它知道类型的完全限定名和需要访问的模块（注意，多个模块可以包含任意的类型名）。Java开发中，反射思想的长期运用会破坏隔离性，因为即使是私有的域也可以通过 `setAccessible` 方法来修改可见性。

通过这种功能，OSGi通常可以提供模块的实现，但是不需要导出。此外，它们可能包含一些内部类型的声明，而这些类型可能是由框架加载。例如，一个使用JPA来说进行持久化的模块，可以通过 persistence.xml 文件来引用领域类型，JPA实现模块会在需要使用的时候加载引用的类型。

最大的用例在于实现服务组件。OSGi规范中有一章内容是声明性服务（Declarative Services，DS）,它定义了一个模块如何声明组件（那些生命周期由框架管理的类）。组件可以在OSGi注册中心中绑定服务，并且可以为它们自己提供服务。例如：
```
@Component
    public class CartMgrComponent implements CartManager {

    	@Reference
    	UserAdmin users;

        @Override
        public Cart getOrCreateCart(String user) {
            // ...
        }
    }
```

在这个例子中，`CartMgrComponent` 是一个提供 `CartManager` 服务的组件。它指向一个服务 `UserAdmin`，这个类的声明周期由DS框架管理。当 `UserAdmin` 服务可用的时候，`CartMgrComponent` 将会被创建，并且会发布一个 `CartManager` 服务，这个服务可能也会被其它模块中的组件通过相似的方法引用。

这个框架可以运行是因为它能够加载 `CartMgrComponent` 类，这个类通过使用 `@Component` 注解来表明它是一个组件。定义组件和服务是OSGi应用设计和编程的主要方式。

在JPMS中，只有在导出包内的类型可以被访问，即使使用反射也还是如此。即使模块中没有导出的类型是可见的（你可以通过调用 `Class.forName` 来获得一个类对象），但是它们还是还是无法在模块外部被访问。当框架调用 `newInstance` 来实例化对象的时候，将会抛出一个 `IllegalAccessException`。这视乎削减了框架的很多可能性，但是还是有一些可以采用的方法。

第一种方法是提供个人类型来作为服务，这可以通过 `java.util.ServiceLoader` 来加载。ServiceLoader 从Java 6开始就已经成为平台标准的一部分了，并且在Java 9中会被更新来支持进行模块间的工作。ServiceLoader 可以访问没有导出的包，只要模块提供者包含一个提供声明。不幸的是，ServiceLoader 是原生的，而且无法为当前的一些框架（比如，DS 或 Spring）提供灵活性。

第二种方法是通过使用“合格”的包导出。这是只对单个命名模块访问的导出，而不是所有模块都能访问。例如，你可以导出类所在的包给Spring 框架。但是，这样的方式会在JPA中的某些方面遇到失败，因为JPA是一个规范，而不是一个单一的模块，它可以由多个不同的模块实现，比如：Hibernate、EclipseLink、等等。

第三种方法是“动态”导出，这样包可以被任何人访问，但是只能通过反射的方式，在编译时期是无法访问的。这是JPMS很新的一个特性，而且它还有所争议。这是最接近OSGi允许策略的方法，但是对于那些可能会被反射加载的类型，还需要为它们所在的每个包都添加 `dynamic`  导出来获得访问权限。对于OSGi用户而言，这就像是一种不必要的并发症。

# 下一篇
本文是文章的第一部分。下一篇将介绍OSGi和JPMS中的这些主题：版本、动态加载、未来二者协同工作的可能性。


 



