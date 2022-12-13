

主要分为dailect 和 classloader的获取

对于classloader需要加载驱动类
```java
  
// 驱动检查  
// 创建连接前先尝试去加载驱动，把驱动问题暴露出来，有驱动隔离插件的存在，必须这么写，但是又不能写在core里面，因为远程设计的原因也不能写在设计器里面  
String driverClass = null;  
String url = TemplateUtils.renderWithOutException(getURL());  
try {  
    ClassLoader classLoader = DataSourceDriverLoaderUtils.getClassLoader(getDriverSource(), getDatabase(), url);  
    driverClass = TemplateUtils.renderWithOutException(getDriver());  
    if (classLoader != null) {  
        classLoader.loadClass(driverClass);  
    } else {  
        Class.forName(driverClass);  
    }  
} catch (Exception e) {  
    //缺少驱动  
    if (e instanceof ClassNotFoundException && e.getMessage().contains(driverClass)) {  
        throw new DriverClassNotFoundException(driverClass);  
    } else {  
        throw e;  
    }  
}
```






























