---
title: "Akka集群分布式主题消息及订阅"
subtitle: "「AKKA」- 2.6.8"
layout: post
author: "LiuL"
header-style: text
tags:
  - Akka
  - Akka cluster topic
---

# 模块依赖

分布式发布订阅主题API可用且可与核心akka-actor-typed模块一起使用，但是只有在集群应用程序中使用时才分发:

```xml
<properties>
  <akka.version>2.6.8</akka.version>
  <scala.binary.version>2.13</scala.binary.version>
</properties>
<dependency>
  <groupId>com.typesafe.akka</groupId>
  <artifactId>akka-cluster-typed_${scala.binary.version}</artifactId>
  <version>${akka.version}</version>
</dependency>
```

# Topic Actor

分布式发布订阅是通过用actor代表每个pub子主题来实现的akka.actor.typed.pubsub.Topic。

Topic Actor需要在订阅者将要居住的每个节点上运行，或在要向该主题发布消息的节点上运行。

Topic的标识是可以发布的消息类型的元组和字符串主题名称，但是建议不要定义具有不同类型和相同主题名称的多个主题。

```java
import akka.actor.typed.pubsub.Topic;

Behaviors.setup(
    context -> {
      ActorRef<Topic.Command<Message>> topic =
          context.spawn(Topic.create(Message.class, "my-topic"), "MyTopic");
          ...
```

然后，本地Actor可以订阅主题（并取消订阅）：

```java
topic.tell(Topic.subscribe(subscriberActor));

topic.tell(Topic.unsubscribe(subscriberActor));
```

并将消息发布到该主题：

```java
topic.tell(Topic.publish(new Message("Hello Subscribers!")));
```

# Pub Sub可伸缩性

每个主题都由一个“接待员Receptionist”服务键表示，这意味着主题的数量将扩展到数千或数万，但对于更多的主题，则需要自定义解决方案。这也意味着独特主题的很高周转率将无法很好地解决，并且针对此类用例，建议使用自定义解决方案。

主题参与者充当代理，并委派给处理重复数据删除的本地订户，因此，发布的消息仅发送到节点一次，无论该节点上有多少订户。

当主题参与者没有主题订阅者时，它将从接待员中注销自己，这意味着该主题的已发布消息不会发送给该主题参与者。

# 交货保证

正如在消息传递的可靠性Akka，在分布式订阅模式的消息传递保证是在-最多一次交付。换句话说，消息可能会丢失。除了具有订阅者的节点的注册表最终是一致的之外，这意味着在一个节点上订阅一个actor会有很短的延迟，之后它才能在其他节点上已知并发布。

如果您希望至少一次交货，我们建议您选择Alpakka Kafka（企业版，收费的）。







