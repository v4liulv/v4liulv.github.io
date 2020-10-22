---
title: "SpringBoot容器启动完成后执行"
subtitle: "延迟启动"
layout: post
author: "LiuL"
header-style: text
tags:
  - SpringBoot
---


# 场景需求

在开发中可能会有这样的情景。需要在容器启动的时候执行一些内容。比如读取配置文件，数据库连接，执行初始化等之类的。SpringBoot给我们提供了两个接口来帮助我们实现这种需求。这两个接口分别为CommandLineRunner和ApplicationRunner。他们的执行时机为容器启动完成的时候。

# 实现

这两个接口中有一个run方法，我们只需要实现这个方法即可。这两个接口的不同之处在于：ApplicationRunner中run方法的参数为ApplicationArguments，而CommandLineRunner接口中run方法的参数为String数组。目前我在项目中用的是ApplicationRunner。是这么实现的：

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
 
@Component
public class JDDRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(args);
        System.out.println("这个是测试ApplicationRunner接口");
    }
}
```

执行结果如下：

![20201009103734](https://liulv.work/images/img/20201009103734.png)
