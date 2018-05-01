---
title: Java中的SPI机制
date: 2018-03-03 16:38:16
tags: [Java,SPI] 
categories: Java
---

本文对 Java 中的 SPI 机制进行学习。是对以下文章的摘录：
* [《Java spi机制浅谈》](http://singleant.iteye.com/blog/1497259)
* [《Dubbo原理解析-Dubbo内核实现之SPI简单介绍》](http://blog.csdn.net/quhongwei_zhanqiu/article/details/41577159)
* [《Dubbo源码分析 ---- 基于SPI的扩展实现机制》](http://blog.csdn.net/guhong5153/article/details/61210511)

<!-- more -->

# 具体约定
当服务的提供者，提供了服务接口的一种实现之后，在 jar 包的 `META-INF/services/` 目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该 jar 包 `META-INF/services/` 里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。

基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里制定。JDK 提供服务实现查找的一个工具类：`java.util.ServiceLoader`

# 例子
__1.common-logging__
apache最早提供的日志的门面接口。只有接口，没有实现。具体方案由各提供商实现，发现日志提供商是通过扫描 `META-INF/services/org.apache.commons.logging.LogFactory` 配置文件，通过读取该文件的内容找到日志提工商实现类。只要我们的日志实现里包含了这个文件，并在文件里制定 `LogFactory` 工厂接口的实现类即可。

__2.jdbc__
jdbc4.0 以前，开发人员还需要基于 `Class.forName("xxx")` 的方式来装载驱动。jdbc4 开始也基于 spi 的机制来发现驱动提供商了，可以通过 `META-INF/services/java.sql.Driver` 文件里指定实现类的方式来暴露驱动提供者。

__3.自己编写简单例子__
假设有一个内容搜索系统，分为展示和搜索两个模块。展示和搜索基于接口编程。搜索的实现可能是基于文件系统的搜索，也可能是基于数据库的搜索。实例代码如下

以下代码托管在 [__java-spi-demo__](https://github.com/mindawei/java-spi-demo)。

`Search.java` : 搜索接口
```
package search;
import java.util.List;
/**
 * @author <url>http://singleant.iteye.com/blog/1497259</url>
 */
public interface Search {
    List<Object> search(String keyword);
}
```

`FileSearch.java` : 文件系统的搜索实现
```
package search;
import java.util.List;
/**
 * @author <url>http://singleant.iteye.com/blog/1497259</url>
 */
public class FileSearch implements Search {
    @Override
    public List<Object> search(String keyword) {
        System.out.println("now use file system search. keyword:" + keyword);
        return null;
    }
}
```

`DatabaseSearch.java` : 数据库的搜索实现
```
package search;
import java.util.List;
/**
 * @author <url>http://singleant.iteye.com/blog/1497259</url>
 */
public class DatabaseSearch implements Search {
    @Override
    public List<Object> search(String keyword) {
        System.out.println("now use database search. keyword:" + keyword);
        return null;
    }
}
```

`SearchTest.java` : 测试类
```
package search;
import java.util.Iterator;
import java.util.ServiceLoader;
/**
 * @author <url>http://singleant.iteye.com/blog/1497259</url>
 */
public class SearchTest {
    public static void main(String[] args) {
        ServiceLoader<Search> s = ServiceLoader.load(Search.class);
        Iterator<Search> searchIterator = s.iterator();
        if (searchIterator.hasNext()) {
            Search curSearch = searchIterator.next();
            curSearch.search("test");
        }else{
            System.out.println("请检 META-INF 查文件路径是否正确");
        }
    }
}
```

当 `META-INF/services/search.Search` 中是 `search.DatabaseSearch` 输出结果是：
`now use database search. keyword:test` 。
![](/images/00023/01.png)

当 `META-INF/services/search.Search` 中是 `search.FileSearch` 输出结果是：
`now use file system search. keyword:test` 。
![](/images/00023/02.png)

__可以看出SearchTest里没有任何和具体实现有关的代码，而是基于spi的机制去查找服务的实现__。

__4.另一个例子__
可以查看[《Dubbo原理解析-Dubbo内核实现之SPI简单介绍》](http://blog.csdn.net/quhongwei_zhanqiu/article/details/41577159)。

# Dubbo 扩展
Dubbo 的 SPI 机制是在标准的 jdk 的 SPI 的机制上扩展加强而来的，SPI 的实现在 dubbo 中由以下几个 annotation 来实现。 
1. `SPI` 注解，，使用 SPI 注解来标识一个扩展点，该注解一般是打在接口上的，dubbo 的扩展点都是基于接口的。 
2. `Adaptive` 注解 该注解主要作用在方法上，使用该注解可以根据方法的参数值来调用的具体的实现类的对应方法。
3. `Activate` 注解，该注解一般作用在实现类上，使用该注解一般是对于 `Filter` 类型的类，来决定是该类是否加入到 `Filter` 的执行器责任链中。

`ExtensionLoader` 是 SPI 机制实现的核心类，该类提供了以下静态方法 `getExtensionLoader（Class type）`，该方法返回了加载此 `class` 的 `ExtensionLoader`， 通过该 `ExtensionLoader` 来创建对应的 type 的类的实例，该类还提供了以下方法来获取对应的加载类的实例 
1. `getAdaptiveExtension（String name）` 
2. `public T getExtension(String name)` 

具体可参考 [《Dubbo源码分析 ---- 基于SPI的扩展实现机制》](http://blog.csdn.net/guhong5153/article/details/61210511)。

