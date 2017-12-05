+++
date = "2017-11-27T18:53:22+03:00"
draft = false
title = "Spring: destroy prototype beans"
tags = ["spring"]
+++

Prototype-scoped beans destruction is not managed by Spring container (only construction is managed). 
But we can manage it ourselves with Spring `BeanPostProcessor`s. 

<!--more-->

In previos post ([Spring Bean PostProcessors](/post/spring_beanpostprocessors/)) we considered some trivial examples of using `BeanPostProcessor`.
Now let's consider more interesting case - destruction of prototype-scoped beans. 
The reason for this is that **Spring does not manage desctruction phase of prototype-scoped beans**, as it is mentioned in the docs: 
[7.5.2 The prototype scope](https://docs.spring.io/spring/docs/4.3.9.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-prototype):

> In contrast to the other scopes, Spring does not manage the complete lifecycle of a prototype bean: 
> the container instantiates, configures, and otherwise assembles a prototype object, 
> and hands it to the client, with no further record of that prototype instance. 
> Thus, although initialization lifecycle callback methods are called on all objects regardless of scope, 
> in the case of prototypes, configured destruction lifecycle callbacks are not called. 
> The client code must clean up prototype-scoped objects and release expensive resources that the prototype bean(s) are holding. 

And Spring offers to use `BeanPostProcessor` for this: 

>To get the Spring container to release resources held by prototype-scoped beans, 
> try using a custom bean post-processor, which holds a reference to beans that need to be cleaned up.

OK, let's consider how could we do this. 

## DestructionAwareBeanPostProcessor - can we use it?

We have a special `BeanPostProcessor` for applying some custom actions when destroying beans - 
[DestructionAwareBeanPostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/DestructionAwareBeanPostProcessor.html).
It defined 2 new methods: 

* void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException
* boolean requiresDestruction(Object bean)

We should return `true` in `requiresDestruction` when beans comes into it, and then, when destruction phase begin, 
method `postProcessBeforeDestruction` will be called, and this particular bean will be passed to this method. 

Sounds like this is what we need, the only problem is that **this is not applicable to prototype-scoped beans**! 
Yes, for this `DestructionAwareBeanPostProcessor` only singletone-scoped beans are passed inso, so for desctruction of prototype-scoped beans this will not work. 

OK, then it looks that we should create our own implementation of `BeanPostProcessor` that handles destruction of prototype-scoped beans.

## Implement BeanPostProcessor for destruction of prototype-scoped beans

OK, let's implement our prototype beans having one condition in mind: they all should implement `DisposableBean`, and put destruction code
into method `destroy` - to allow unified processing. As we know, Spring container itself will not call this method itself (for prototype beans) - 
so we will take care about it. 

Now let's consider the code of `BeanPostProcessor` that handles destruction of prototype beans: 

```
/**
 * Bean PostProcessor that handles destruction of prototype beans
 */
@Component
public class DestroyPrototypeBeansPostProcessor implements BeanPostProcessor, BeanFactoryAware, DisposableBean {

    private BeanFactory beanFactory;

    private final List<Object> prototypeBeans = new LinkedList<>();

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanFactory.isPrototype(beanName)) {
            synchronized (prototypeBeans) {
                prototypeBeans.add(bean);
            }
        }
        return bean;
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    @Override
    public void destroy() throws Exception {
        synchronized (prototypeBeans) {
            for (Object bean : prototypeBeans) {
                if (bean instanceof DisposableBean) {
                    DisposableBean disposable = (DisposableBean)bean;
                    try {
                        disposable.destroy();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
            prototypeBeans.clear();
        }
    }
}
```

It is simple:
 
 * Save reference to bean if and only if it is of prototype scope
 * This `DestroyPrototypeBeansPostProcessor` is Spring bean itself, so when it's `destroy` method is called - we destroy all prototype beans.

Here is full code of the project: https://github.com/iryndin/misc/tree/master/blogprjs/03-beanpostprocessor-destroyprototypes
 
## Useful

It is useful to take a look at implementation of 
[ScheduledAnnotationBeanPostProcessor](https://github.com/spring-projects/spring-framework/blob/v4.3.13.RELEASE/spring-context/src/main/java/org/springframework/scheduling/annotation/ScheduledAnnotationBeanPostProcessor.java). 
This class handles desctruction actions for sheduled tasks. Exactly the case where additional actions should be taken. 
Also this class implements `DestructionAwareBeanPostProcessor` methods, so it is always useful to see how these methods can be implemented. 
