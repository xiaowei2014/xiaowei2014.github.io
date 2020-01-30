# 深入解析String#intern

https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html

8种基本类型的常量池都是系统协调的，`String`类型的常量池比较特殊。它的主要使用方法有两种：

- 直接使用双引号声明出来的`String`对象会直接存储在常量池中。
- 如果不是用双引号声明的`String`对象，可以使用`String`提供的`intern`方法。intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中

接下来我们主要来谈一下`String#intern`方法。

首先深入看一下它的实现原理。

## 1，JAVA 代码

```
public native String intern();  
```

`String#intern`方法是一个 native 的方法，但注释写的非常明了。“如果常量池中存在当前字符串, 就会直接返回当前字符串. 如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回”。

## 2，native 代码

它的大体实现结构就是: JAVA 使用 jni 调用c++实现的`StringTable`的`intern`方法, `StringTable`的`intern`方法跟Java中的`HashMap`的实现是差不多的, 只是不能自动扩容。默认大小是1009。

要注意的是，String的String Pool是一个固定大小的`Hashtable`，默认值大小长度是1009，如果放进String Pool的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用`String.intern`时性能会大幅下降（因为要一个一个找）。

在 jdk6中`StringTable`是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。在jdk7中，`StringTable`的长度可以通过一个参数指定：

- `-XX:StringTableSize=99991`

相信很多 JAVA 程序员都做做类似 `String s = new String("abc")`这个语句创建了几个对象的题目。 这种题目主要就是为了考察程序员对字符串对象的常量池掌握与否。上述的语句中是创建了2个对象，第一个对象是”abc”字符串存储在常量池中，第二个对象在JAVA Heap中的 String 对象。

来看一段代码：

```
public static void main(String[] args) {
    String s = new String("1");
    s.intern();
    String s2 = "1";
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4);
}
```

打印结果是

- jdk6 下`false false`
- jdk7 下`false true`

具体为什么稍后再解释，然后将`s3.intern();`语句下调一行，放到`String s4 = "11";`后面。将`s.intern();` 放到`String s2 = "1";`后面。是什么结果呢

```
public static void main(String[] args) {
    String s = new String("1");
    String s2 = "1";
    s.intern();
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    String s4 = "11";
    s3.intern();
    System.out.println(s3 == s4);
}
```

打印结果为：

- jdk6 下`false false`
- jdk7 下`false false`

\####小结 从上述的例子代码可以看出 jdk7 版本对 intern 操作和常量池都做了一定的修改。主要包括2点：

- 将String常量池 从 Perm 区移动到了 Java Heap区
- `String#intern` 方法时，如果存在堆中的对象，会直接保存对象的引用，而不会重新创建对象。



## 2，intern 不当使用

在使用 fastjson 进行接口读取的时候，我们发现在读取了近70w条数据后，我们的日志打印变的非常缓慢，每打印一次日志用时30ms左右，如果在一个请求中打印2到3条日志以上会发现请求有一倍以上的耗时。在重新启动 jvm 后问题消失。继续读取接口后，问题又重现。接下来我们看一下出现问题的过程。

\####1，根据 log4j 打印日志查找问题原因

在使用`log4j#info`打印日志的时候时间非常长。所以使用 housemd 软件跟踪 info 方法的耗时堆栈。

- trace SLF4JLogger.
- trace AbstractLoggerWrapper:
- trace AsyncLogger

```
org/apache/logging/log4j/core/async/AsyncLogger.actualAsyncLog(RingBufferLogEvent)                sun.misc.Launcher$AppClassLoader@109aca82            1            1ms    org.apache.logging.log4j.core.async.AsyncLogger@19de86bb  
org/apache/logging/log4j/core/async/AsyncLogger.location(String)                                  sun.misc.Launcher$AppClassLoader@109aca82            1           30ms    org.apache.logging.log4j.core.async.AsyncLogger@19de86bb  
org/apache/logging/log4j/core/async/AsyncLogger.log(Marker, String, Level, Message, Throwable)    sun.misc.Launcher$AppClassLoader@109aca82            1           61ms    org.apache.logging.log4j.core.async.AsyncLogger@19de86bb  
```

代码出在 `AsyncLogger.location` 这个方法上. 里边主要是调用了 `return Log4jLogEvent.calcLocation(fqcnOfLogger);`和`Log4jLogEvent.calcLocation()`

`Log4jLogEvent.calcLocation()`的代码如下:

```
public static StackTraceElement calcLocation(final String fqcnOfLogger) {  
    if (fqcnOfLogger == null) {  
        return null;  
    }  
    final StackTraceElement[] stackTrace = Thread.currentThread().getStackTrace();  
    boolean next = false;  
    for (final StackTraceElement element : stackTrace) {  
        final String className = element.getClassName();  
        if (next) {  
            if (fqcnOfLogger.equals(className)) {  
                continue;  
            }  
            return element;  
        }  
        if (fqcnOfLogger.equals(className)) {  
            next = true;  
        } else if (NOT_AVAIL.equals(className)) {  
            break;  
        }  
    }  
    return null;  
}  
```

经过跟踪发现是 `Thread.currentThread().getStackTrace();` 的问题。

\####2, 跟踪Thread.currentThread().getStackTrace()的 native 代码，验证String#intern

`Thread.currentThread().getStackTrace();`native的方法:

```
public StackTraceElement[] getStackTrace() {  
    if (this != Thread.currentThread()) {  
        // check for getStackTrace permission  
        SecurityManager security = System.getSecurityManager();  
        if (security != null) {  
            security.checkPermission(  
                SecurityConstants.GET_STACK_TRACE_PERMISSION);  
        }  
        // optimization so we do not call into the vm for threads that  
        // have not yet started or have terminated  
        if (!isAlive()) {  
            return EMPTY_STACK_TRACE;  
        }        StackTraceElement[][] stackTraceArray = dumpThreads(new Thread[] {this});  
        StackTraceElement[] stackTrace = stackTraceArray[0];  
        // a thread that was alive during the previous isAlive call may have  
        // since terminated, therefore not having a stacktrace.  
        if (stackTrace == null) {  
            stackTrace = EMPTY_STACK_TRACE;  
        }  
        return stackTrace;  
    } else {  
        // Don't need JVM help for current thread  
        return (new Exception()).getStackTrace();  
    }  
}  

private native static StackTraceElement[][] dumpThreads(Thread[] threads);   
```

下载 openJdk7的源码查询 jdk 的 native 实现代码，列表如下【这里因为篇幅问题，不详细罗列涉及到的代码，有兴趣的可以根据文件名称和行号查找相关代码】：

\openjdk7\jdk\src\share\native\java\lang\Thread.c
\openjdk7\hotspot\src\share\vm\prims\jvm.h line:294: \openjdk7\hotspot\src\share\vm\prims\jvm.cpp line:4382-4414:
\openjdk7\hotspot\src\share\vm\services\threadService.cpp line:235-267: \openjdk7\hotspot\src\share\vm\services\threadService.cpp line:566-577:
\openjdk7\hotspot\src\share\vm\classfile\javaClasses.cpp line:1635-[1651,1654,1658]:

完成跟踪了底层的 jvm 源码后发现，是下边的三条代码引发了整个程序的变慢问题。

```
oop classname = StringTable::intern((char*) str, CHECK_0);  
oop methodname = StringTable::intern(method->name(), CHECK_0);  
oop filename = StringTable::intern(source, CHECK_0);  
```

这三段代码是获取类名、方法名、和文件名。因为类名、方法名、文件名都是存储在字符串常量池中的，所以每次获取它们都是通过`String#intern`方法。但没有考虑到的是默认的 StringPool 的长度是1009且不可变的。因此一旦常量池中的字符串达到的一定的规模后，性能会急剧下降。

\####3,fastjson 不当使用 String#intern

导致这个 intern 变慢的原因是因为 fastjson 对`String#intern`方法的使用不当造成的。跟踪 fastjson 中的实现代码发现，

\####`com.alibaba.fastjson.parser.JSONScanner#scanFieldSymbol()`

```
if (ch == '\"') {
    bp = index;
    this.ch = ch = buf[bp];
    strVal = symbolTable.addSymbol(buf, start, index - start - 1, hash);
    break;
}
```

\####`com.alibaba.fastjson.parser.SymbolTable#addSymbol()`:

```
/**
 * Constructs a new entry from the specified symbol information and next entry reference.
 */
public Entry(char[] ch, int offset, int length, int hash, Entry next){
    characters = new char[length];
    System.arraycopy(ch, offset, characters, 0, length);
    symbol = new String(characters).intern();
    this.next = next;
    this.hashCode = hash;
    this.bytes = null;
}
```

fastjson 中对所有的 json 的 key 使用了 intern 方法，缓存到了字符串常量池中，这样每次读取的时候就会非常快，大大减少时间和空间。而且 json 的 key 通常都是不变的。这个地方没有考虑到大量的 json key 如果是变化的，那就会给字符串常量池带来很大的负担。

这个问题 fastjson 在1.1.24版本中已经将这个漏洞修复了。程序加入了一个最大的缓存大小，超过这个大小后就不会再往字符串常量池中放了。

[1.1.24版本的`com.alibaba.fastjson.parser.SymbolTable#addSymbol()` Line:113]代码

```
public static final int MAX_SIZE           = 1024;

if (size >= MAX_SIZE) {
    return new String(buffer, offset, len);
}
```

这个问题是70w 数据量时候的引发的，如果是几百万的数据量的话可能就不只是30ms 的问题了。因此在使用系统级提供的`String#intern`方式一定要慎重！

本文大体的描述了 `String#intern`和字符串常量池的日常使用，jdk 版本的变化和`String#intern`方法的区别，以及不恰当使用导致的危险等内容，让大家对系统级别的 `String#intern`有一个比较深入的认识。让我们在使用和接触它的时候能避免出现一些 bug，增强系统的健壮性。

以下是几个比较关键的几篇博文。感谢！