+++
date = "2017-09-20T15:08:51+03:00"
draft = false
title = "Spring Boot and AWT headless"

+++


## The problem with AWT headless app in Spring Boot

Recently I needed to create console Java Spring Boot app that does some stuff with AWT (not so important what exactly). 
I found that simple run with a normat Sprint Boot stub simply does not work. Running this:

```
@SpringBootApplication
public class MyAwtApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyAwtApplication.class, args);
    }
}
```

generates following exception when AWT-related processing happens: 

```
java.awt.HeadlessException: null at 
sun.java2d.HeadlessGraphicsEnvironment.getScreenDevices(HeadlessGraphicsEnvironment.java:72)
...
```

All right, how to fix this?

We can fix this in two ways. 

## Set java.awt.headless to false in command line

One way, is to pass `-Djava.awt.headless=false` command line argument to this app 

## Use SpringApplicationBuilder to construct your Application properly

Another way is to tweak a little the way how `SpringApplication` is built using class `SpringApplicationBuilder`:

```
@SpringBootApplication
public class MyAwtApplication {

    public static void main(String[] args) {
        SpringApplicationBuilder builder = new SpringApplicationBuilder(MyAwtApplication.class);
        builder.headless(false).run(args);
    }
}
```

## Useful links

[Run Spring Boot Headless](https://stackoverflow.com/questions/23553755/run-spring-boot-headless)
