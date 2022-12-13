


```java
/**  
 * 标记类/方法/字段需谨慎修改，要考虑前后端、移动端兼容性或性能. * * @author richie  
 * created on 2019-05-14 * @IncludeIntoJavadoc  
 */  
@Retention(RetentionPolicy.CLASS)  
@Target({  
        ElementType.ANNOTATION_TYPE,  
        ElementType.CONSTRUCTOR,  
        ElementType.FIELD,  
        ElementType.METHOD,  
        ElementType.TYPE  
})  
@Documented  
public @interface Careful {  
}



/**  
 * 标记类/方法/字段修改时必须考虑兼容性. * * @author richie  
 * created on 2019-05-14 * @IncludeIntoJavadoc  
 */  
@Retention(RetentionPolicy.CLASS)  
@Target({  
        ElementType.FIELD,  
        ElementType.METHOD,  
        ElementType.TYPE  
})  
@Documented  
public @interface Compatible {  
  
    /**  
     * 表示什么类型的兼容 默认空     *     * @return 兼容类型  
     */    CompatibleType type() default CompatibleType.NONE;  
  
}

/**  
 * 标记一个类/方法仅供测试使用，在{@code review}时可以忽略.  
 * * @author richie  
 * created on 2019-05-14 * @IncludeIntoJavadoc  
 */  
@Inside(String.class)  
public @interface Inside {  
    /**  
     * 被Test的类数组，可以只写一个.     *     * @return 被测试类数组  
     */    @Inside(value = {Inside.class, String.class}, function = "value")  
    Class[] value() default {Inside.class};  
  
    /**  
     * 测试的方法名称.     *     * @return 方法名称  
     */    @Inside(value = Inside.class, function = "function")  
    String[] function() default "";  
}


/**  
 * 标记类为开放类，可在插件使用. * * @author richie  
 * created on 2019-08-15 * @IncludeIntoJavadoc  
 */  
@Retention(RetentionPolicy.CLASS)  
@Target(ElementType.TYPE)  
public @interface Open {  
}


```