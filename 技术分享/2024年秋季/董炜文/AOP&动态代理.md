
AOP（Aspect-Oriented Programming），即面向切面编程。

在Java中，动态代理是一种在运行时动态创建代理对象的技术。Spring AOP使用动态代理来实现方法的增强（即添加额外的行为）。当您定义一个切面（Aspect）并声明通知（Advice）时，Spring AOP框架会在运行时创建一个代理对象，该对象会在调用目标方法前后、抛出异常时或最终执行时动态地插入额外的逻辑。

## 1. 动态代理

通过特定的设置，在程序运行期间指示JVM动态地生成类的字节码。这种动态生成的类往往被用作代理类，即动态代理类。也就是运行时做了编译的事情并且把生成的字节码加载成这个类的Class对象。

静态代理：就是手动为每一个目标类的每一个方法都增加交叉业务，也就是手动为每一个目标类增加代理类。

缺点：如果目标类数量非常多或者目标类中的功能非常多，直接使用静态代理的方式来为目标类增加交叉业务会非常的繁琐。

Spring AOP使用了两种动态代理：

- 基于接口的动态代理—JDK动态代理
- 基于类的动态代理—cglib动态代理（默认常用）

### 附：静态代理

#### 1.1 业务接口

**IUserService.java**
```java
public interface IUserService {
    void add(String name);
}
```

#### 1.2 被代理类实现业务接口

**UserServiceImpl.java**
```java
public class UserServiceImpl implements IUserService {
    @Override
    public void add(String name) {
        System.out.println("向数据库中插入名为：" + name + "的用户");
    }
}
```

#### 1.3 定义代理类并实现业务接口

因为代理对象和被代理对象需要实现相同的接口。所以代理类源文件UserServiceProxy.java这么写：

其实就是类的聚合（）

**UserServiceProxy.java**
```java
public class UserServiceProxy implements IUserService {
    // 被代理对象
    private IUserService target;
    // 通过构造方法传入被代理对象
    public UserServiceProxy(IUserService target) {
        this.target = target;
    }
    @Override
    public void add(String name) {
        System.out.println("准备向数据库中插入数据");
        target.add(name);
        System.out.println("插入数据库成功");
    }
}
```
由于代理类(UserServiceProxy )和被代理类(UserServiceImpl )都实现了IUserService接口，所以都有add方法，在代理类的add方法中调用了被代理类的add方法，并在其前后各打印一条日志。
#### 1.4 客户端调用


**StaticProxyTest.java**
```java
public class StaticProxyTest {
    public static void main(String[] args) {
        IUserService target = new UserServiceImpl();
        UserServiceProxy proxy = new UserServiceProxy(target);
        proxy.add("新用户");
    }
}
```

### 1.1 JDK动态代理

核心：InvocationHandler接口和Proxy类

前提：
1. 目标类必须实现某些接口
2. 被代理的方法必须是public

**业务接口**
```java
public interface IUserService {
    void add(String name);
}
```

**实现类**
```java
public class UserServiceImpl implements IUserService {
    @Override
    public void add(String name) {
        System.out.println("向数据库中插入名为：" + name + "的用户");
    }
}
```

**代理类实现InvocationHandler并定义invoke函数**
```java
public class MyInvocationHandler implements InvocationHandler {
    // 被代理对象，Object类型
    private Object target;
    public MyInvocationHandler(Object target) {
        this.target = target;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("准备向数据库中插入数据");
        Object returnValue = method.invoke(target, args);
        System.out.println("插入数据库成功");
        return returnValue;
    }
}
```

**测试类**
```java
public class ProxyTest {
    public static void main(String[] args) {
        IUserService target = new UserServiceImpl();
        MyInvocationHandler handler = new MyInvocationHandler(target);
        IUserService proxyObject = (IUserService)
                Proxy.newProxyInstance(IUserService.class.getClassLoader(),
                        new Class[] {IUserService.class}, handler);
        proxyObject.add("新用户");
    }
}
```

### newProxyInstance源码
```java
@CallerSensitive // 当一个方法标记为 @CallerSensitive 时，它可以使用 java.security.Principal 获取调用者的信息。这通常涉及到使用 java.lang.reflect.ReflectAccess 获取调用者的类，这在默认情况下是受限的。
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h) {
    // 检查h不为null
    Objects.requireNonNull(h);
    // 抑制编译器警告，可能使用了过时的api
    @SuppressWarnings("removal")
    // 使用Reflection.getCallerClass()方法获取调用newProxyInstance方法的类的Class对象
    // 如果系统中没有设置SecurityManager，则caller被设置为null。
    final Class<?> caller = System.getSecurityManager() == null
            ? null
            : Reflection.getCallerClass();
    /*
     * Look up or generate the designated proxy class and its constructor.
     * 如果代理类已经被创建过，则直接返回；否则，创建一个新的代理类，并缓存它以便将来使用。
     */
    Constructor<?> cons = getProxyConstructor(caller, loader, interfaces);
    return cons.newInstance(new Object[]{h});
}
```

### SecurityManager 相关代码
`SecurityManager`：Java提供的安全管理器框架，它允许应用程序在执行某些敏感操作之前进行安全检查。

##### SecurityManagerDemo

创建一个`SecurityManager`的子类，并重写其方法以实现你的安全策略。例如，你可以重写`checkPermission`方法来检查特定的权限：
```java
public class MySecurityManager extends SecurityManager {
    @Override
    public void checkPermission(Permission perm) {
        // 自定义权限检查逻辑
        if (perm instanceof RuntimePermission) {
            String name = perm.getName();
            if (name.equals("createClassLoader")) {
                throw new SecurityException("ClassLoader创建被禁止");
            }
        }
    }
    // 可以根据需要重写其他方法，如checkRead, checkWrite等
}
```

在你的应用程序启动时，设置自定义的 SecurityManager。通常，这可以在 main 方法中完成：
```java
public class Application {
    public static void main(String[] args) {
        System.setSecurityManager(new MySecurityManager());
        // 应用程序的其他代码
    }
}
```

在你的代码中，你可以使用安全管理器来检查权限。例如，你可以在执行某些敏感操作之前调用 checkPermission 方法：
```java
public void someSensitiveOperation() {
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        sm.checkPermission(new RuntimePermission("somePermission"));
    }
    // 执行敏感操作
}
```

当 SecurityManager 检查到权限不足时，会抛出 SecurityException。因此，你需要在你的代码中处理这个异常：
```java
try {
    someSensitiveOperation();
} catch (SecurityException e) {
    System.err.println("安全检查失败：" + e.getMessage());
}
```

### getProxyConstructor源码
```java
private static Constructor<?> getProxyConstructor(Class<?> caller,
                                                  ClassLoader loader,
                                                  Class<?>... interfaces) {
    // optimization for single interface
    // if内是针对单接口的优化
    if (interfaces.length == 1) {
        Class<?> intf = interfaces[0];
        if (caller != null) {
            checkProxyAccess(caller, loader, intf); // 检查调用者的权限
        }
        // 使用proxyCache（一个缓存结构）来获取或生成代理类的构造器。proxyCache.sub(intf)可以为每个不同的接口组合创建一个子缓存，然后使用computeIfAbsent方法查找是否存在已经存在的构造器，如果没有则new一个新的
        proxyCache.sub(intf)可以为每个不同的接口组合创建一个子缓存，然后使用computeIfAbsent方法查找是否存在已经存在的构造器，如果没有则new一个新的
        return proxyCache.sub(intf).computeIfAbsent(
            loader,
            (ld, clv) -> new ProxyBuilder(ld, clv.key()).build()
        );
    } else {
        // 克隆接口
        final Class<?>[] intfsArray = interfaces.clone();
        if (caller != null) {
            checkProxyAccess(caller, loader, intfsArray);
        }
        // 创建接口列表
        final List<Class<?>> intfs = Arrays.asList(intfsArray);
        return proxyCache.sub(intfs).computeIfAbsent(
            loader,
            (ld, clv) -> new ProxyBuilder(ld, clv.key()).build()
        );
    }
}
```

### newProxyInstance源码
```java
private static Object newProxyInstance(Class<?> caller, // null if no SecurityManager
                                      Constructor<?> cons,
                                      InvocationHandler h) {
    /*
     * Invoke its constructor with the designated invocation handler.
     */
    try {
        if (caller != null) {
            checkNewProxyPermission(caller, cons.getDeclaringClass());
        }
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException | InstantiationException e) {
        throw new InternalError(e.toString(), e);
    } catch (InvocationTargetException e) {
        Throwable t = e.getCause();
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else {
            throw new InternalError(t.toString(), t);
        }
    }
}
...

cons.newInstance(Object ... initargs) -> newInstanceWithCaller(initargs, !override, caller)

newInstanceWithCaller(Object[] args, boolean checkAccess, Class<?> caller) ->ca.newInstance(args)...
代码很长，总之当 `Proxy.newProxyInstance` 方法被调用时，JDK 会动态生成一个代理类的字节码。

在main中添加代码System._setProperty_("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");保存并查看
```

### 代理类生成示例
```java
public final class $Proxy0 extends Proxy implements IUserService {
    private static final Method m0;
    private static final Method m1;
    private static final Method m2;
    private static final Method m3;
    public $Proxy0(InvocationHandler var1) {
        super(var1);
    }
    //
 以下省略hashcode等方法
    ...
    public final void add(String var1) {
        try {
            super.h.invoke(this, m3, new Object[]{var1}); // 传递代理类本身，Method实例，入参
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    // 静态初始化代码，创建方法引用，方便后续调用接口方法
    static {
        ClassLoader var0 = $Proxy0.class.getClassLoader();
        try {
            m0 = Class.forName("java.lang.Object", false, var0).getMethod("hashCode");
            m1 = Class.forName("java.lang.Object", false, var0).getMethod("equals", Class.forName("java.lang.Object", false, var0));
            m2 = Class.forName("java.lang.Object", false, var0).getMethod("toString");
            m3 = Class.forName("org.dww.aopdemo.service.IUserService", false, var0).getMethod("add", Class.forName("java.lang.String", false, var0));
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
    private static MethodHandles.Lookup proxyClassLookup(MethodHandles.Lookup var0) throws IllegalAccessException {
        if (var0.lookupClass() == Proxy.class && var0.hasFullPrivilegeAccess()) {
            return MethodHandles.lookup();
        } else {
            throw new IllegalAccessException(var0.toString());
        }
    }
}
代理类继承了Proxy类，其主要目的是为了传递InvocationHandler。

代理类实现了被代理的接口JDKService，这也是为什么代理类可以直接强转成接口。

有一个公开的构造函数，参数为指定的InvocationHandler，并将参数传递到父类Proxy中。

每一个实现的方法，都会调用InvocationHandler中的invoke方法，并将代理类本身、Method实例、入参三个参数进行传递。这也是为什么调用代理类中的方法时，总会分派到InvocationHandler中的invoke方法的原因。```

### 1.2 cglib动态代理

依赖：包含在org.springframework.boot中

使用：业务类
```java
public class CglibService {
    public void test1() { // 可以被cglib代理对象代理执行
        System.out.println("CglibService正在执行test1方法");
    }
    private static void test2() { // private或static的方法无法被cglib代理对象代理执行
        System.out.println("CglibService正在执行test2静态方法");
    }
    public final void test4() { // final方法无法被cglib代理对象代理执行
        System.out.println("CglibService正在执行test4公有final方法");
    }
}
```

### 自定义回调方法实现MethodInterceptor
```java
public class TestCglibProxy implements MethodInterceptor {
    /**
     *
     * @param proxy cglib代理对象
     * @param method 目标方法Method对象
     * @param objects 方法入参
     * @param methodProxy cglib方法代理
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object proxy, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("cglib回调函数开始工作...");
        methodProxy.invokeSuper(proxy, objects); // 调用代理类的父类——目标类的方法
        return null;
    }
}
```

### 测试类
```java
public class CglibProxyTest {
    public static void main(String[] args) {
        // ASM框架生成的字节码对象是存在内存中，我们可以设置此配置值将该字节码文件写进项目路径下的文件中
        System.setProperty("cglib.debugLocation", "D:\\javafile\\AopDemo\\CGLIB");
        Callback callback = new TestCglibProxy();
        // 通过Enhancer去创建cglib代理对象
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(CglibService.class); // 设置要继承的超类，即目标类Class对象
        enhancer.setCallback(callback); // 设置回调函数，没有它，目标方法无法被拦截调用
        CglibService proxy = (CglibService) enhancer.create();
        proxy.test1();
        proxy.test4(); // test4作为final方法，无法被代理类覆盖，因此不会执行额外行为，使用的是原方法
    }
}
```

### 查看生成的字节码（代理类）

初始化方法，通过反射获取方法引用，并创建对应的MethodProxy对象

> CGLIB 动态代理使用 `MethodProxy` 来封装对目标方法的调用：
> 
> - `MethodProxy` 是 CGLIB 中的一个类，它封装了对特定方法的调用细节，包括方法的签名、名称和如何调用目标方法。
>     
> - `MethodProxy` 提供了一个快速的方法调用路径，允许在 `MethodInterceptor` 的 `intercept` 方法中直接调用原始方法，而不需要每次都进行反射查找。
>     
> 
> `MethodProxy` 的作用类似于一个桥梁，连接了代理对象和目标对象之间的方法调用，同时允许在调用前后插入额外的行为。

```java
static void CGLIB$STATICHOOK1() {
    CGLIB$THREAD_CALLBACKS = new ThreadLocal<>();
    CGLIB$emptyArgs = new Object[0];
    Class var0 = Class.forName("org.dww.aopdemo.service.CglibService$$EnhancerByCGLIB$$89c37a51");
    Class var1;
    Method[] var10000 = ReflectUtils.findMethods(new String[]{"equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;"}, (var1 = Class.forName("java.lang.Object")).getDeclaredMethods());
    CGLIB$equals$1$Method = var10000[0];
    CGLIB$equals$1$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$1");
    CGLIB$toString$2$Method = var10000[1];
    CGLIB$toString$2$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/String;", "toString", "CGLIB$toString$2");
    CGLIB$hashCode$3$Method = var10000[2];
    CGLIB$hashCode$3$Proxy = MethodProxy.create(var1, var0, "()I", "hashCode", "CGLIB$hashCode$3");
    CGLIB$clone$4$Method = var10000[3];
    CGLIB$clone$4$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/Object;", "clone", "CGLIB$clone$4");
    CGLIB$test1$0$Method = ReflectUtils.findMethods(new String[]{"test1", "()V"}, (var1 = Class.forName("org.dww.aopdemo.service.CglibService")).getDeclaredMethods())[0];
    CGLIB$test1$0$Proxy = MethodProxy.create(var1, var0, "()V", "test1", "CGLIB$test1$0");
}
```

### test1方法
```java
public final void test1() {
    MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
    if (var10000 == null) {
        CGLIB$BIND_CALLBACKS(this);
        var10000 = this.CGLIB$CALLBACK_0;
    }
    if (var10000 != null) {
        var10000.intercept(this, CGLIB$test1$0$Method, CGLIB$emptyArgs, CGLIB$test1$0$Proxy);
    } else {
        super.test1();
    }
}
```
->执行代理类test1方法

->var10000=CGLIB$CALLBACK_0（在创建代理对象时传入的参数，这里为TestCglibProxy实例），通过var10000调用其中的intercept方法

->intercept方法执行额外逻辑，并通过MethodProxy.invokeSuper调用fci.f2.invoke

> `FastClassInfo fci`：这是一个包含快速类信息的内部类实例。`FastClassInfo` 包含两个 `FastClass` 实例（`f1` 和 `f2`）和两个索引（`i1` 和 `i2`）。在这里，`f2` 和 `i2` 用于访问父类的方法。

->FastClass中的invoke方法调用被代理类的方法

> `FastClass` 是 CGLIB 用来优化方法调用的工具。它通过生成一个包含特定类方法快速访问路径的代理类来实现。这个代理类包含了直接调用原始类方法的能力，绕过了一些 Java 反射的开销。

```java
public Object invokeSuper(Object obj, Object[] args) throws Throwable {
    try {
        // 初始化 MethodProxy 实例，确保 fastClassInfo 被正确设置。
        this.init();
        // 从 MethodProxy 实例中获取 FastClassInfo 对象，该对象包含了与代理类和父类相关的 FastClass 对象和方法索引。
        FastClassInfo fci = this.fastClassInfo;
        // 使用 FastClass 对象 fci.f2（它封装了对代理类的父类的方法访问）和方法索引 fci.i2 来调用父类中的方法。
        // 这个方法调用将传递 obj（代理对象的实例）和 args（方法参数）。
        return fci.f2.invoke(fci.i2, obj, args);
    } catch (InvocationTargetException var4) {
        // 如果被调用的方法抛出了异常，InvocationTargetException 会被抛出，我们捕获它并抛出其目标异常（cause），
        // 这样原始异常的类型和堆栈跟踪得以保留。
        throw var4.getTargetException();
    }
}
```
```
