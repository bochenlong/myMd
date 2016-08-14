### SPI是什么
**SPI** 即*Service Provider Interface*，是JDK内置的一种服务发现机制。它提供了Java运行时发现添加实现的方式，将实现的装配移到程序外，使得模块之间不对实现进行硬编码，降低耦合实现可插拔。   
### SPI具体约定   
SPI的具体约定：当服务的提供者，提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。 基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里指定。jdk提供服务实现查找的一个工具类：java.util.ServiceLoader。      
### 简单示例   
参考下图我们写一个简单的SPI示例：         
![Java SPI](http://o7ar2k9lr.bkt.clouddn.com/java_spi.png) 
   
**接口IHello**

```java
package demo.spi;

/** 
* @author <a href="mailto:bochenlong@163.com target="_blank">bochenlong</a>
*/
public interface IHello {
	void whoSayHello();
}
```
**接口实现类TextHello/ByteHello**

```java
package demo.spi.impl;
import demo.spi.IHello;

/**
* @author <a href="mailto:bochenlong@163.com" target="_blank">bochenlong</a>
*/
public class ByteHello implements IHello {
    @Override
    public void whoSayHello() {
        System.out.println("ByteHello say hello");
    }	
}
```
<!-- more -->
	

---
```java
package demo.spi.impl;
import demo.spi.IHello;

/**
* @author <a href="mailto:bochenlong@163.com" target="_blank">bochenlong</a>
*/
public class TextHello implements IHello {
	@Override
	public void whoSayHello() {
	    System.out.println("TextHello say hello");
	}
}
```
**测试类**

```java
import demo.spi.IHello;
import java.util.ServiceLoader;

class Test {

	public static void main(String[] args) {
		ServiceLoader<IHello> services = ServiceLoader.load(IHello.class);
		for (IHello service : services) {
		    service.whoSayHello();
		}
	}

}
``` 
**配置文件META/services/demo.spi.IHello**   
> 注意：**文件夹**名需固定为 **META/services/**   **文件**名称固定位 **类** 全称      
> 且文件夹需放置在类路径下，Maven项目结构可放置在resouces里

	demo.spi.impl.TextHello#文本hello实现   
	demo.spi.impl.ByteHello#字节Hello实现
	
**输出结果**   
> TextHello say hello   
> ByteHello say hello   

### 总结   
SPI运行时发现服务的特性，可以很方便的实现服务扩展，比如JDBC、DUBBO都利用SPI机制，实现用户对服务实现的扩展；同时ServiceLoader的reload（）可以对服务进行热更换。