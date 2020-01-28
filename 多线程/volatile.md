# volatile

volatile的可见性保证并不是只对于volatile变量本身那么简单。可见性保证遵循以下规则：

　　如果线程A写入一个volatile变量，线程B随后读取了同样的volatile变量，则线程A在写入volatile变量之前的所有可见的变量值，在线程B读取volatile变量后也同样是可见的。

​		如果线程A读取一个volatile变量，那么线程A中所有可见的变量也会同样从主存重新读取。

下面用一段代码来示例说明：

```
public class MyClass {
    private int years;
    private int months
    private volatile int days;

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

　　update()方法写入3个变量，其中只有days变量是volatile。

　　完整的volatile可见性保证意味着，在写入days变量时，线程中所有可见变量也会写入到主存。也就是说，写入days变量时，years和months也会同时被写入到主存。

下面的代码读取了years、months、days变量：

```
public class MyClass {
    private int years;
    private int months
    private volatile int days;

    public int totalDays() {
        int total = this.days;
        total += months * 30;
        total += years * 365;
        return total;
    }

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

　　请注意totalDays()方法开始读取days变量值到total变量。在读取days变量值时，months和years的值也会同时从主存读取。因此，按上面所示的顺序读取时，可以保证读取到days、months、years变量的最新值。

**''**译者注：可以将对volatile变量的读写理解为一个触发刷新的操作，写入volatile变量时，线程中的所有变量也都会触发写入主存。而读取volatile变量时，也同样会触发线程中所有变量从主存中重新读取。因此，应当尽量将volatile的写入操作放在最后，而将volatile的读取放在最前，这样就能连带将其他变量也进行刷新。上面的例子中，update()方法对days的赋值就是放在years、months之后，就是保证years、months也能将最新的值写入到主存，如果是放在两个变量之前，则days会写入主存，而years、months则不会。反过来，totalDays()方法则将days的读取放在最前面，就是为了能同时触发刷新years、months变量值，如果是放后面，则years、months就可能还是从CPU缓存中读取值，而不是从主存中获取最新值。**''**

 

**Java volatile Happens-Before保证**