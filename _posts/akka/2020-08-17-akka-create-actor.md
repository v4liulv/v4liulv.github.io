---
title: "Akka创建Actor"
subtitle: "「AKKA」- 2.6.8"
layout: post
author: "LiuL"
header-style: text
tags:
  - Akka
  - Akka create actor
---

# 1. 简单介绍

在新版的Akka中，创建Actor是我们最常用的，而创建Actor使用新版的AbstractBehavior<POJO>有两种常用的方式创建，太常用了所以单独列出两种创建Actor的示例。

# 2. 创建Actor的方式

创建Actor新版本支持以下两种方式创建：
* 继承AbstractBehavior<POJO>方式
* 方法直接函数式构造创建

两种方式最终效果完全一致，可根据自己喜欢选择使用，函数式方式创建代码比较简洁推荐使用。

# 3. 继承AbstractBehavior<POJO>方式

```java
public class ActorTest extends AbstractBehavior<ActorTest.Command> {

    //先定义Actor请求消息类型接口
    interface Command {}

    //请求消息交互的POJO
    ...

    //响应消息接口
     interface Ref {}

    //响应消息交互的POJO
    ...

    //静态方法创建actor入口方法
    public static Behavior<Command> create() {
        return Behaviors.setup(context -> new ActorTest(context));
    }
 
    //构造方法创建Actor
    public ActorTest(Context context){
        super(context);
        ...
    }

    //消息接收处理
    @Override
    public Receive<Command> createReceive() {
        return newReceiveBuilder()
                .onMessage(Request.class, this::onRequest)
                .build();
    }
    
    //实际消息处理
    private Behavior<Command> onUpdate(Request request) {
        request.ref.tell(response);

        return this;
        //或return Behaviors.same();
    }
    
    //...
}
```

# 4. 方法直接函数式构造创建


```java
public class ActorTest {
    //这种方式无法getContext()获取context，通过静态全局属性设置
    private static ActorContext<Request> getContext;
    

    //先定义Actor请求消息类型接口
    interface Command {}

    //请求消息交互的POJO
    ...

    //响应消息接口
     interface Ref {}

    //响应消息交互的POJO
    ...

    public static Behavior<ActorTest.Command> create() {
        return Behaviors.setup(context -> {
                getContext = context;
                return Behaviors.receive(Request.class)
                .onMessage(Request.class, this::onRequest)
                .build();
        });
   }

   private static Behavior<Request> onRequest(Request request) {
        getContext.getLog().info("收到请求了， 我在回复...");
        request.replyTo.tell(new Response("Here are the cookies for " + request.query));

        return Behaviors.same();
    }
}
```

# 5. 创建ActorSystem、RootBehavior组装Actor

```java
public class App {

    public static void main(String[] args) {
        ActorSystem actorSystem = ActorSystem.create(RootBehavior.create(), "RequestResponseAsk");
    }
   
    /**
     * RootBehavior
     */
    private static class RootBehavior {
        static Behavior<Void> create() {
            return Behaviors.setup(context -> {
                
                //创建ActorTest Actor
                ActorRef<ActorTest.Command> actorTest =
                        context.spawn(ActorTest.create(), "ActorTest");

                //...

                return Behaviors.empty();
            });
        }
    }
}
```