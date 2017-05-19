---
title: struts2源码阅读
date: 2017-04-25 13:46:42
category: Java
tags: struts2
toc: true
---
<!---->
# Struts2运行原理

Struts2基于Servlet框架的filter机制实现。
```xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<filter-mapping>
     <filter-name>struts2</filter-name>
     <url-pattern>/*</url-pattern>
</filter-mapping>
```
当tomcat启动时，会通过初始化StrutsPrepareAndExecuteFilter加载struts2的相关配置信息。
系统运行时，会拦截request请求，执行struts2的流程。

## request处理流程

1. 客户端发送请求
2. 请求先通过StrutsPrepareAndExecuteFilter
3. StrutsPrepareAndExecuteFilter通过ActionMapper来决定这个Request需要调用哪个Action
4. 如果ActionMapper决定调用某个Action，StrutsPrepareAndExecuteFilter把请求的处理
交给ActionProxy，这儿已经转到它的Delegate--Dispatcher来执行
5. ActionProxy根据ActionMapping和ConfigurationManager找到需要调用的Action类
6. ActionProxy创建一个ActionInvocation的实例
7. ActionInvocation调用真正的Action，当然这涉及到相关拦截器的调用
8. Action执行完毕，ActionInvocation创建Result并返回，当然，如果要在返回之前做些什么，
可以实现PreResultListener。添加PreResultListener可以在Interceptor中实现，不知道其它还有什么方式？

## 相关概念

### ActionMapper
提供Http请求和Action之间的映射，基本上不需要访问struts的配置文件来确定request和action的关系。

默认的使用DefaultActionMapper，同时可以使用自定义Mapper，但是必须要实现`mapper.ActionMapper`接口，并且有一个默认的构造器。

ActionMapper会根据request获取相应的ActionMapping，ActionMapping包含了action的类和方法等详细信息。

### ActionProxy & ActionInvocation
ActionProxy由ActionProxyFactory创建，是Action的代理，用来获取获取Action类，也可以被远程使用。

ActionProxy会使用ActionInvocation封装特定request的action，ActionInvocation决定action的动作，执行、拦截、监听。

简而言之，ActionProxy封装request可以获得的action，ActionInvocation封装action如何执行。

### ConfigurationProvider & Configuration
ConfigurationProvider就是Struts2中配置文件的解析器，Struts2中的配置文件主要是使用StrutsXmlConfigurationProvider来载入xml文件的配置信息。
provider可以通过DispatcherListener进行配置。

# Struts2初始化

Struts2通过初始化`StrutsPrepareAndExecuteFilter`，加载配置信息。StrutsPrepareAndExecuteFilter的初始化过程如下：

```java
 /*
    学习点：1.面向接口编程，耦合度低
           2.适配器模式
 */
 public void init(FilterConfig filterConfig) throws ServletException {
        //封装了初始化动作的类
        InitOperations init = new InitOperations();
        Dispatcher dispatcher = null;
        try {
            //对filter配置进行封装，此处有点类似代理模式类型
            FilterHostConfig config = new FilterHostConfig(filterConfig);
            //初始化log
            init.initLogging(config);
            //创建dispatcher对象
            dispatcher = init.initDispatcher(config);
            //初始化静态资源加载器 
            init.initStaticContentLoader(config, dispatcher);
            
            //初始化prepare（封装请求预处理操作）
            prepare = new PrepareOperations(dispatcher);
            //初始化excute (封装了请求处理操作)
            execute = new ExecuteOperations(dispatcher);
            this.excludedPatterns = init.buildExcludedPatternsList(dispatcher);

            //初始化完成回调方法，默认为空
            postInit(dispatcher, filterConfig);
        } finally {
            //清理
            if (dispatcher != null) {
                dispatcher.cleanUpAfterInit();
            }
            init.cleanup();
        }
    }
```
## 配置信息封装

传入`FilterConfig`接口的实例，然后封装为`HostConfig`的实例`FilterHostConfig`。在创建的过程中，调用方法的入参类型大都为`HostConfig`类型。
```java
/*
    FilterConfig 为接口
*/
public class FilterHostConfig implements HostConfig {

    private FilterConfig config;
    
    public FilterHostConfig(FilterConfig config) {
        this.config = config;
    }
    public String getInitParameter(String key) {
        return config.getInitParameter(key);
    }

    public Iterator<String> getInitParameterNames() {
        return MakeIterator.convert(config.getInitParameterNames());
    }

    public ServletContext getServletContext() {
        return config.getServletContext();
    }
}
```

## logging信息配置

学习:
1. 利用配置信息很灵活的提供数据，创建`LoggerFactory`
2. 命名规范，工厂模式以`Factory`后缀命名

```java
    /**
     * Initializes the internal Struts logging
     */
    public void initLogging( HostConfig filterConfig ) {
        //从配置信息中，读取loggerFactory配置信息
        String factoryName = filterConfig.getInitParameter("loggerFactory");
        if (factoryName != null) {
            try {
                //利用反射创建LoggerFactory
                Class cls = ClassLoaderUtil.loadClass(factoryName, this.getClass());
                LoggerFactory fac = (LoggerFactory) cls.newInstance();
                LoggerFactory.setLoggerFactory(fac);
            } catch ( InstantiationException e ) {
                System.err.println("Unable to instantiate logger factory: " + factoryName + ", using default");
                e.printStackTrace();
            } catch ( IllegalAccessException e ) {
                System.err.println("Unable to access logger factory: " + factoryName + ", using default");
                e.printStackTrace();
            } catch ( ClassNotFoundException e ) {
                System.err.println("Unable to locate logger factory class: " + factoryName + ", using default");
                e.printStackTrace();
            }
        }
    }
```

学习：
1. 在多线程的环境下，创建对象。

```java
  public static void setLoggerFactory(LoggerFactory factory) {
        //创建写锁
        lock.writeLock().lock();
        try {
            LoggerFactory.factory = factory;
        } finally {
            //在finally内释放写锁
            lock.writeLock().unlock();
        }
    }
```

在这个过程中，`ClassLoaderUtil`是一个载入资源和类的工具，反射类的代码如下。

> 学习：
    1. 异常的处理形式 
    2. 获取反射的优先级处理

```java
 /* @param 类名    The name of the class to load
  * @param 调用方法的类 The Class object of the calling object
  * @throws ClassNotFoundException If the class cannot be found anywhere.
  */
public static Class loadClass(String className, Class callingClass) throws ClassNotFoundException {
        try {
            //从当前线程获取ClassLoader，然后反射对象
            return Thread.currentThread().getContextClassLoader().loadClass(className);
        } catch (ClassNotFoundException e) {
            try {
                //直接获取
                return Class.forName(className);
            } catch (ClassNotFoundException ex) {
                try {
                    //从ClassLoaderUtil获取ClassLoader
                    return ClassLoaderUtil.class.getClassLoader().loadClass(className);
                } catch (ClassNotFoundException exc) {
                    //从callingClass获取ClassLoader
                    return callingClass.getClassLoader().loadClass(className);
                }
            }
        }
    }
```

## 创建dispatcher（分配器）

传入配置信息（形参为HostConfig接口，非常灵活！），返回dispatcher。
```java
  public Dispatcher initDispatcher( HostConfig filterConfig ) {
        //利用filterConfig信息，创建Dispatcher对象
        Dispatcher dispatcher = createDispatcher(filterConfig);
        dispatcher.init();
        return dispatcher;
    }
```

`Dispatcher`的创建过程，其实就是数据转换，赋值的过程
```java
 private Dispatcher createDispatcher( HostConfig filterConfig ) {
        //获取配置信息里的信息
        Map<String, String> params = new HashMap<String, String>();
        for ( Iterator e = filterConfig.getInitParameterNames(); e.hasNext(); ) {
            String name = (String) e.next();
            String value = filterConfig.getInitParameter(name);
            params.put(name, value);
        }

        //利用配置信息和ServletContext创建
        return new Dispatcher(filterConfig.getServletContext(), params);
    }
```

`dispatcher`的初始化过程比较复杂，很多初始化过程都在其中包含。
```java
    public void init() {

    	if (configurationManager == null) {
    		configurationManager = createConfigurationManager(DefaultBeanSelectionProvider.DEFAULT_BEAN_NAME);
    	}

        try {
            //初始化文件管理
            init_FileManager();
            //加载org/apache/struts2/default.properties
            init_DefaultProperties(); // [1]
            //加载struts-default.xml,struts-plugin.xml,struts.xml
            init_TraditionalXmlConfigurations(); // [2]      
            init_LegacyStrutsProperties(); // [3]
            //用户自己实现的ConfigurationProviders类 
            init_CustomConfigurationProviders(); // [5]
            //Filter的初始化参数
            init_FilterInitParameters() ; // [6]
            init_AliasStandardObjects() ; // [7]

            //容器
            Container container = init_PreloadConfiguration();
            container.inject(this);
            init_CheckWebLogicWorkaround(container);

            //监听器
            if (!dispatcherListeners.isEmpty()) {
                for (DispatcherListener l : dispatcherListeners) {
                    l.dispatcherInitialized(this);
                }
            }
            errorHandler.init(servletContext);

        } catch (Exception ex) {
            if (LOG.isErrorEnabled())
                LOG.error("Dispatcher initialization failed", ex);
            throw new StrutsException(ex);
        }
    }
```


