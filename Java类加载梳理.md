### 什么是类加载器
> 虚拟机设计团队把类加载阶段中的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。实现这个动作的代码模块成为“类加载器”。

简单的讲，类加载器就是Java程序在运行时动态加载class文件到内存中的模块。类加载器是Java语言的一项创新，在类层次划分、OSGi、热部署、代码加密等领域都有着极其重要的应用。

### Java默认的类加载器
* 启动类加载器（Bootstrap ClassLoader):这个类加载器负责将存放在**<JAVA_HOME>\lib目录中，或者被-Xbootclasspath参数所指定的路径**中的，并且是虚拟机识别的（仅按照文件名称识别，如rt.jar,名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用，这个类加载器使用C++实现，是虚拟机自身的一部分。比如打包dmeo.jar，含类到
* 扩展类加载器（Extension ClassLoader）：这个加载器由sun.misc.Luncher$ExtClassLoader实现，它负责加载**<JAVA_HOME>\lib\ext目录中的，或者被java.ext.dirs系统变量所指定的路径**中的所有类库，开发者可以直接使用扩展类加载器。
* 应用程序类加载器（Applicaition ClassLoader）：这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值，也成为系统类加载器。它负责加载用户**类路径（ClassPath）**上所指定的类库，开发者可以直接使用这个类加载器，如果应用中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

<!-- more -->
### 类加载器加载类的原理
#### 类与类加载器
对于任意一个类，都需要由它加载它的类加载器和这个类本身一同确立其在Java虚拟中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。如果两个类源于同一个class文件，被同一个虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等（包括equals/isAssignableFrom/isInstance）。测试代码：

```java
package demo.classloader;

/**
 * @author <a href="mailto:bochenlong@163.com" target="_blank">bochenlong</a>.
 * 定义测试Object
 */
public class TestObject {
}

```
---

```java
package demo.classloader;

import com.sun.tools.javac.util.Assert;

import java.io.InputStream;

/**
 * @author <a href="mailto:bochenlong@163.com" target="_blank">bochenlong</a>.
 * 自定义ClassLoader
 */
public class MyClassLoader extends ClassLoader {
    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        Assert.checkNonNull(name);
        try {
            String className = name.substring(name.lastIndexOf(".") + 1) + ".class";
            // 获取当前路径下className文件的字节流
            InputStream in = getClass().getResourceAsStream(className);
            // 如果为空的话，由父类加载处理
            if(in == null) {
                return super.loadClass(name);
            }
            byte[] bytes = new byte[in.available()];
            // 读取字节
            in.read(bytes);
            // 实例化class
            return super.defineClass(name,bytes,0,bytes.length);
        } catch (Exception e) {
            throw new ClassNotFoundException(name);
        }
    }
}
```
---
```
package demo.classloader;

/**
 * @author <a href="mailto:bochenlong@163.com" target="_blank">bochenlong</a>.
 */
public class Test {
    public static void main(String[] args) throws Exception {
   		// 实例化自定义加载
    	ClassLoader classLoader = new MyClassLoader();
    	// 通过自定义类加载器生成TestObject
    	Object o = classLoader.loadClass("demo.classloader.TestObject").newInstance();
    	System.out.println(o instanceof  TestObject);
      	// 输出false，即o不是TestObject类型，因为这里的TestObject是应用类加载器加载的类型
    }
}

```

#### 双亲委派模型
我们的应用程序都是都上面三种加载器互相配合进行加载的，如果有必要，还可以加入自己定义的类加载器。这些加载器之间的关系如图所示：![类双亲委派模型](http://i.imgur.com/8QUH9PZ.png)   
这种层次关系称之为**双亲委派模型（Parents Delegation Model)**，类加载器使用这种模型呢来加载类，即：*如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个了你，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。*   
注意，双亲委派模型除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。这里类加载器之间的关系不是以继承的关系来实现，而是都使用组合的关系来复用父加载器的代码。核心实现如下：
  
```java
// java.lang.ClassLoader
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);// 查找已加载的类
        if (c == null) {// 如果为空，加载
            long t0 = System.nanoTime();
            try {
                if (parent != null) {// 如果父加载器不为空，则使用父加载器进行记载
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);// 父加载器加载不了，尝试使用启动类加载器加载
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {// 如果都加载不了，不作处理尝试自己加载
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}

```
#### 为什么要使用双亲委派模型
使用双亲委派模型组织类加载器之间的关系，使Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类java.lang.Object，它存放在rt.jar之中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型顶端的启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是一个类。相反，如果没有使用双亲委派模型，由个各类加载器自行去加载的话，系统中将出现多个不同的Object类，Java将无法识别哪一个才是我们要用到的。
#### 双亲委派模型验证
* 应用程序类加载器加载TestObject：

	```java
	package demo.classloader;
	
	/**
	 * Created by bcl on 2016/5/17.
	 */
	public class Test {
	    public static void main(String[] args) throws Exception {
	        System.out.println(Test.class.getClassLoader().loadClass("demo.classloader.TestObject").getClassLoader());
	    }
	}
	```
	执行结果：
	
	```
	sun.misc.Launcher$AppClassLoader@232204a1
	```
* 扩展类加载器加载TestObject:   
	将TestObject.class打包到demo.jar中（jar -cfv demo.jar demo/classloader/TestObject.class），将demo.jar放到${JAVA_HOME}/jre/lib/ext路径下。执行上面Test类：    
	执行结果：
	
	```java
	sun.misc.Launcher$ExtClassLoader@7ea987ac
	```
* 启动类加载器加载TestObject,两种验证方式:   
	1 在jvm中添加-Xbootclasspath参数，指定启动类加载类的加载路径，并追加我们自已的jar（demo.jar）   
	> -Xbootclasspath:路径，指定启动类加载器的加载路径，一般结合参数使用   
	> 1 -Xbootclasspath/a:路径，/a指后缀在核心class搜索路径后面
	> 2 -Xbootclasspath/p:路径，/p前缀在class搜索路径前面,一般不使用，避免和核心类冲突   
	
	执行上面test类，执行运行参数-Xbootclasspath/a:D:\demo.jar，执行结果：
	
	```
	null
	```
	从类的层次关系，可以知道，ExtClassLoader的parent加载器为null，实际上指的就是启动类加载器。
	
	2 将class文件放到${JAVA_HOME}/jre/classes/目录下   
	验证略

#### 扩展类加载器
了解了各个类加载器的执行原理和自带类加载器的加载用途，我们就可以利用双亲委派模式自定义类加载器来扩展Java的类加载器了。扩展很简单，自定义扩展类，重写findClass()就可以，这样类在请求加载的时候会依次请求父类加载，显然父类加载不了的时候，自定义类加载器会尝试自己加载而调用重写`findClass()`方法。  
> 为什么偏偏只重写findClass方法？   
> 在loadClass方法中JDK帮我们实现了双亲委派模式加载类的逻辑，如果父类加载器都加载不了类的时候，loadClass方法就会调用findClass方法来搜索类。所以我们只需重写该方法即可。如没有特殊的要求，一般不建议重写loadClass加载类的算法。   
 
下面我们使用类加载器加载非类路径下的d:\TestObject.class:

```java
package demo.classloader;

import jdk.internal.util.xml.impl.Input;

import java.io.*;

/**
 * Created by bcl on 2016/5/19.
 * 自定义类加载器
 */
public class MyClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        try {
            File file = new File("D:\\TestObject.class");
            if (file == null) {
                super.findClass(name);
            }
            String path = name.substring(name.lastIndexOf(".") + 1).concat(".class");
            InputStream in = new FileInputStream(file);
            if(in == null) {
                super.findClass(name);
            }
            byte[] bytes = new byte[in.available()];
            in.read(bytes);
            return super.defineClass(name,bytes,0,bytes.length);
        } catch (Exception e) {
            throw new ClassNotFoundException(name);
        }
    }
}

```
---
```java
package demo.classloader;

/**
 * Created by bcl on 2016/5/17.
 * 注意确保应用、扩展、启动类加载器加载路径下都没有TestObject.class，否则由于优先级则会被其他类加载器加载。
 */
public class Test {
    public static void main(String[] args) throws Exception {
        MyClassLoader myClassLoader = new MyClassLoader();
        System.out.println(myClassLoader.loadClass("demo.classloader.TestObject").getClassLoader());
    }
}
```
输出结果:

```
demo.classloader.MyClassLoader@7f31245a
```

   
### 经典应用场景   
* Tomcat，类加载器架构，自己定义了多个类加载器，   
1, 保证了同一个服务器的两个Web应用程序的Java类库隔离；    
2, 保证了同一个服务器的两个Web应用程序的Java类库又可以相互共享；比如多个Spring组织的应用程序不能共享，会造成资源浪费；    
3, 保证了服务器尽可能保证自身的安全不受不受部署Web应用程序影响；
4, 支持JSP应用的服务器，大多需要支持热替换(HotSwap)功能。   
* OSGi(Open Service GateWay Initiative)，是基于Java语言的动态模块化规范。已成为Java世界的“事实上”的模块化标准，最为熟悉的案例的Eclipse IDE。

具体的相关应用知识可以自定Google，下面简单的实现一个热加载。
#### 实现简单的热加载

首先，先介绍一下java.lang.ClassLoader中和热替换有关的一些重要方法：   

* `findLoadedClass`：每个类加载器都维护有自己的一份已加载类名字空间，其中不能出现两个同名的类。凡是通过该类加载器加载的类，无论是直接的还是间接的，都保存在自己的名字空间中，该方法就是在该名字空间中寻找指定的类是否已存在，如果存在就返回给类的引用，否则就返回 null。这里的直接是指，存在于该类加载器的加载路径上并由该加载器完成加载，间接是指，由该类加载器把类的加载工作委托给其他类加载器完成类的实际加载。

* `getSystemClassLoader`：Java2 中新增的方法。该方法返回系统使用的 ClassLoader。可以在自己定制的类加载器中通过该方法把一部分工作转交给系统类加载器去处理。

* `defineClass`：该方法是 ClassLoader 中非常重要的一个方法，它接收以字节数组表示的类字节码，并把它转换成 Class 实例，该方法转换一个类的同时，会先要求装载该类的父类以及实现的接口类。

* `loadClass`：加载类的入口方法，调用该方法完成类的显式加载。通过对该方法的重新实现，我们可以完全控制和管理类的加载过程。
resolveClass：链接一个指定的类。这是一个在某些情况下确保类可用的必要方法，详见 Java 语言规范中“执行”一章对该方法的描述。

了解了上面的这些方法，下面我们来实现一个定制的类加载器，这个类加载器只加载指定的类集合，否则就把类加载的工作委托给系统类加载器完成。   
在实现代码之前，有两点需要注意：1、要想实现同一个类的不同版本的共存，那么这些不同版本必须由不同的类加载器进行加载，因此就不能把这些类的加载工作委托给系统加载器来完成，因为它们只有一份。2、为了做到这一点，就不能采用系统默认的类加载器委托规则，也就是说我们定制的类加载器要重写loadclass()方法。   
下面给出代码：

```java
package demo.classloader;

import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.util.HashMap;
import java.util.Map;

/**
 * @author <a href="mailto:bochenlong@163.com" target="_blank">bochenlong</a>.
 */
public class HotSwapClassLoader extends ClassLoader {
    private String baseDir;// 热替换路径
    private Map<String, String> clazz2Path;// 类名称和文件名称对应关系

    public HotSwapClassLoader(String baseDir, String[] clazzs) {
        this.baseDir = baseDir;
        clazz2Path = new HashMap();
        initClazz2Path(baseDir, clazzs);// 初始化类名称和文件名称map
    }

    private void initClazz2Path(String baseDir, String[] clazzs) {
        for (String className : clazzs) {
            clazz2Path.put(className, className2FileName(className));
        }
    }

    private String className2FileName(String className) {
        return baseDir + className.replace(".", "/").concat(".class");
    }


    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        Class<?> c = findLoadedClass(name);
        if (!clazz2Path.containsKey(name) && c == null) {
            // 如果不在热加载范围内且没有加载，则使用Java默认加载规则
            c = super.loadClass(name);        }
        if (c == null) {
        	  // 否则没有加载的话，使用热加载器加载
        	  // 获取文件
            File file = new File(clazz2Path.get(name));
            if (file != null) {
                try {
                    // 获取字节流
                    InputStream in = new FileInputStream(file);
                    byte[] bytes = new byte[in.available()];
                    in.read(bytes);
                    // 生成class类
                    return defineClass(name, bytes, 0, bytes.length);
                } catch (Exception e) {
                    super.findClass(name);
                }
            }
        }
        return c;
    }


}
```
---

```
package demo.classloader;

/**
 * @author <a href="mailto:bochenlong@163.com" target="_blank">bochenlong</a>.
 */
public class SayHello {
    public void sayHello() {
        System.out.println("Hello world!");
    }
}

```
```
import demo.classloader.HotSwapClassLoader;

/**
 * Created by bochenlong on 16/5/18.
 */
public class Test {
    public static void main(String[] args) throws Exception {
        while (true) {
            Class c = new HotSwapClassLoader("/Users/bochenlong/Architect/IdeaProjects/demo/java-demo/target/classes/",
                    new String[]{"demo.classloader.SayHello"})
                    .loadClass("demo.classloader.SayHello");
            Object o = c.newInstance();
            c.getMethod("sayHello").invoke(o);
            Thread.sleep(2000L);
        }

    }
}

```
	
修改SayHello输出为Hello world2，替换原SayHello.class，输出hello world2

```
Hello world!
Hello world!
Hello world!
Hello world2!
Hello world2!
Hello world2!
```
