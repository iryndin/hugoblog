+++
date = "2017-11-27T15:45:43+03:00"
draft = false
title = "Spring Bean PostProcessors"
tags = ["spring"]
+++

Let's speak about Bean PostProcessors in Spring Framework. 

The [BeanPostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html) 
interface defines callback methods that you can implement to provide your own (or override the container’s default) instantiation logic, 
dependency-resolution logic, and so forth. In Spring Framework Reference you can read about Bean PostProcessors in more details here: 
[7.8.1 Customizing beans using a BeanPostProcessor](https://docs.spring.io/spring/docs/4.3.9.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension-bpp).

Shortly, you can use `BeanPostProcessor` for various initialization actions, as well as destroying actions, do some smart init/destroy actions *en masse* etc.
Here let me give some examples on using BeanPostProcessors.

## Example 1. Inject some random variable on integer fields in beans

Here let's create `BeanPostProcessor` that injects random integer variable for bean fields marked with special annotation: `InjectRandomInt`.

Here is this annotation code: 

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface InjectRandomInt {
    int min();
    int max();
}
```

And here is `BeanPostProcessor` code, which is really self-descriptive: 

```
/**
 * Bean PostProcessor that injects random int value for fields marked with annotation <link>@InjectRandomInt</link>
 */
public class InjectRandomIntBeanPostProcessor implements BeanPostProcessor {

    static final Class[] allowedTypes = new Class[]{Integer.class, Long.class, int.class, long.class};

    private Random random = new Random();

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        Field[] fields = bean.getClass().getDeclaredFields();
        for (Field field : fields) {
            InjectRandomInt annotation = field.getAnnotation(InjectRandomInt.class);
            if (annotation != null) {
                if (!isAllowedType(field)) {
                    throw new RuntimeException("Don't put @InjectRandomInt above " + field.getType());
                }
                if (Modifier.isFinal(field.getModifiers())) {
                    throw new RuntimeException("Can't inject to final fields");
                }
                int randomInt = generateRadomIntValue(annotation);
                try {
                    field.setAccessible(true);
                    field.set(bean, randomInt);
                    System.out.println(format("Init bean '%s.%s' with value %d", beanName, field.getName(), randomInt));
                } catch (IllegalAccessException e) {
                    throw new RuntimeException(e);
                }
            }
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    private int generateRadomIntValue(InjectRandomInt annotation) {
        int min = annotation.min();
        int max = annotation.max();
        int randomInt = min + random.nextInt(max - min);
        return randomInt;
    }

    private boolean isAllowedType(Field field) {
        for (Class c : allowedTypes) {
            if (field.getType().equals(c)) {
                return true;
            }
        }
        return false;
    }
}
```

Please see full code of the project here: https://github.com/iryndin/misc/tree/master/blogprjs/01-beanpostprocessor-injectrandomvar

This example is inspired by this article: [Small Spring secrets](https://www.dataart.ru/news/malen-kie-sekrety-spring/).

## Example 2. Inject logger with annotation

This is another example of using `BeanPostProcessor` - here we inject logger for each bean that is marked with proper annotation. 
This example is inspired by this article: [Spring Inject Logger by Annotation Example](https://memorynotfound.com/spring-inject-logger-annotation-example/).

Here is `InjectLog` annotation:

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
    public @interface InjectLog {
}
```

and its processor:

```
/**
 * Bean PostProcessor that injects properly created logger into Spring bean
 */
@Component
public class InjectLogPostProcessor implements BeanPostProcessor {

    private Logger log = LoggerFactory.getLogger(InjectLogPostProcessor.class);

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        ReflectionUtils.doWithFields(bean.getClass(), field -> {
            if (field.getAnnotation(InjectLog.class) != null) {
                ReflectionUtils.makeAccessible(field);
                Logger logger = LoggerFactory.getLogger(bean.getClass());
                field.set(bean, logger);
                log.info("Injected logger into bean '{}'", beanName);
            }
        });
        return bean;
    }
}
```

Here is example of bean with this annotation:

```
@Component
public class SimpleBean {

    @InjectLog
    private Logger log;

    public void doSmth() {
        log.info("doSmth is called");
    }
}
```

You can see that this is pretty much similar to the first example. 

You can see the code of the project here: https://github.com/iryndin/misc/tree/master/blogprjs/02-beanpostprocessor-injectlogger

## Links 

Here are some good examples of usage of `BeanPostProcessor`:

* [Small Spring secrets](https://www.dataart.ru/news/malen-kie-sekrety-spring/) (in Russian)
* [Spring Inject Logger by Annotation Example](https://memorynotfound.com/spring-inject-logger-annotation-example/)
* [A Spring Custom Annotation for a Better DAO](http://www.baeldung.com/spring-annotation-bean-pre-processor)
