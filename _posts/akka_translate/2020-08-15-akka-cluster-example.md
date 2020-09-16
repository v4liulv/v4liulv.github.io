---
title: "Akka 集群示例"
subtitle: "「AKKA官方文档」- 2.6.8"
layout: post
author: "LiuL"
header-style: text
tags:
  - Akka
  - Akka cluster
  - Akka cluster example
---

# 1. 前台Maven依赖

开发Akka集群可使用Maven添加最低依赖配置如下

```xml
<properties>
        <akka.version>2.6.8</akka.version>
        <scala.binary.version>2.13</scala.binary.version>
        <lombok.version>1.18.12</lombok.version>
        <logback.version>1.2.3</logback.version>
        <junit.version>4.12</junit.version>
</properties>

<dependencies>
  <!-- lombok -->
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>${lombok.version}</version>
      <scope>provided</scope>
  </dependency>
  <!-- logback -->
  <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>${logback.version}</version>
  </dependency>
  <!-- akka actor typed -->
  <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-actor-typed_2.13</artifactId>
      <version>${akka.version}</version>
  </dependency>
  <!-- akka actor test -->
  <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-actor-testkit-typed_${scala.binary.version}</artifactId>
      <version>${akka.version}</version>
      <!--<scope>test</scope>-->
  </dependency>
  <!-- akka remote -->
  <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-remote_${scala.binary.version}</artifactId>
      <version>${akka.version}</version>
      <scope>compile</scope>
  </dependency>
  <!-- akka cluster -->
  <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-cluster-typed_${scala.binary.version}</artifactId>
      <version>${akka.version}</version>
  </dependency>
  <!-- akka serialization jackson-->
  <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-serialization-jackson_${scala.binary.version}</artifactId>
      <version>${akka.version}</version>
  </dependency>
</>
```

# 常用配置config方式

默认情况下创建ActorSystem时如不指定具体的config，那么会默认读取application.conf配置文件，但是通常建议都自己固定配置的配置文件，并且程序动态指定一些定制需要的配置。

常用方式：

```java
从写覆盖定制配置，比如端口和角色
Map<String, Object> overrides = new HashMap<>();
overrides.put("akka.remote.artery.canonical.port", port);
overrides.put("akka.cluster.roles", Collections.singletonList(role));

然后添加stats.conf配置，文件名可直接根据项目调整
 Config config = ConfigFactory.parseMap(overrides)
.withFallback(ConfigFactory.load("stats"));

//指定config创建ActorSystem
 ActorSystem.create(RootBehavior.create(), "ClusterSystem", config);

//RootBehavior中创建集群
private static class RootBehavior {
    static Behavior<Void> create() {
        return Behaviors.setup(context -> {
            //创建集群
            Cluster cluster = Cluster.get(context.getSystem());
            ...
            ...
        }
    }
}
```


# 2. Hello AkkaCluster示例

## 2.1. 最小集群配置如下

最小的Akka集群示例，配置如下

```conf
akka { 
  actor.provider = cluster 
  remote.artery { 
    canonical { 
      hostname = "127.0.0.1" 
      port = 2551 
    } 
  } 
} 
```

可直接解析生成Akka Config，也可配置到application.conf，默认创建ActorSystem如果未指定Config会加载application.conf配置。也可配置其他conf文件配置，通过ConfigFactory.load加载配置文件创建Config。


示例中使用直接String方法创建Config配置并覆盖`akka.remote.classic.netty.tcp.port`和`akka.remote.artery.canonical.port`配置，如下：

```java
 private Config clusterConfig =
            ConfigFactory.parseString(
                    "akka { \n"
                            + "  actor.provider = cluster \n"
                            + "  remote.artery { \n"
                            + "    canonical { \n"
                            + "      hostname = \"127.0.0.1\" \n"
                            + "      port = 2551 \n"
                            + "    } \n"
                            + "  } \n"
                            + "}  \n");
    private Config noPort =
            ConfigFactory.parseString(
                    "      akka.remote.classic.netty.tcp.port = 0 \n"
                            + "      akka.remote.artery.canonical.port = 0 \n"
                            + "      akka.cluster.jmx.multi-mbeans-in-same-jvm = on \n"
                            );
```

> **注意**
> > 为了测试需要，我们设置支持同一个JVM进程允许启动多个集群，需要配置`akka.cluster.jmx.multi-mbeans-in-same-jvm = on`

## 2.2. 根据配置创建ActorSystem

```java
ActorSystem<Object> system =
                ActorSystem.create(Behaviors.empty(), "ClusterSystem", noPort.withFallback(clusterConfig));

ActorSystem<Object> system2 =
        ActorSystem.create(Behaviors.empty(), "ClusterSystem2", noPort.withFallback(clusterConfig));
```

## 2.3. 创建、加入、离开集群

```java
 try {
    // #cluster-create
    Cluster cluster = Cluster.get(system);
    Cluster cluster2 = Cluster.get(system2);

    // #cluster-join
    cluster.manager().tell(Join.create(cluster.selfMember().address()));
    cluster2.manager().tell(Join.create(cluster2.selfMember().address()));

    // TODO wait for/verify cluster to form

    // #cluster-leave
    cluster.manager().tell(Leave.create(cluster.selfMember().address()));
    cluster2.manager().tell(Leave.create(cluster2.selfMember().address()));

    // TODO wait for/verify node 2 leaving

} finally {
    // #actorSystem-terminate终止
    system.terminate();
    system2.terminate();
}
```

## 2.4. 测试集群创建、加入、离开

直接@test测试，测试运行结果如下：

```log
SLF4J: A number (1) of logging calls during the initialization phase have been intercepted and are
SLF4J: now being replayed. These are subject to the filtering rules of the underlying logging system.
SLF4J: See also http://www.slf4j.org/codes.html#replay
[2020-09-15 16:51:27,369] [INFO] [akka.event.slf4j.Slf4jLogger] [ClusterSystem-akka.actor.default-dispatcher-3] [] - Slf4jLogger started
[2020-09-15 16:51:28,049] [INFO] [akka.remote.artery.tcp.ArteryTcpTransport] [ClusterSystem-akka.actor.default-dispatcher-3] [ArteryTcpTransport(akka://ClusterSystem)] - Remoting started with transport [Artery tcp]; listening on address [akka://ClusterSystem@127.0.0.1:64600] with UID [4688141471941267513]
[2020-09-15 16:51:28,089] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-3] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Starting up, Akka version [2.6.8] ...
[2020-09-15 16:51:28,288] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-3] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Registered cluster JMX MBean [akka:type=Cluster,port=64600]
[2020-09-15 16:51:28,288] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-3] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Started up successfully
[2020-09-15 16:51:28,326] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - No downing-provider-class configured, manual cluster downing required, see https://doc.akka.io/docs/akka/current/typed/cluster.html#downing
[2020-09-15 16:51:28,327] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - No seed-nodes configured, manual cluster join required, see https://doc.akka.io/docs/akka/current/typed/cluster.html#joining
[2020-09-15 16:51:29,704] [INFO] [akka.event.slf4j.Slf4jLogger] [ClusterSystem2-akka.actor.default-dispatcher-3] [] - Slf4jLogger started
[2020-09-15 16:51:29,745] [INFO] [akka.remote.artery.tcp.ArteryTcpTransport] [ClusterSystem2-akka.actor.default-dispatcher-3] [ArteryTcpTransport(akka://ClusterSystem2)] - Remoting started with transport [Artery tcp]; listening on address [akka://ClusterSystem2@127.0.0.1:64609] with UID [420093861921554813]
[2020-09-15 16:51:29,747] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-3] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Starting up, Akka version [2.6.8] ...
[2020-09-15 16:51:29,758] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-3] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Registered cluster JMX MBean [akka:type=Cluster,port=64609]
[2020-09-15 16:51:29,758] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-3] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Started up successfully
[2020-09-15 16:51:29,767] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - No downing-provider-class configured, manual cluster downing required, see https://doc.akka.io/docs/akka/current/typed/cluster.html#downing
[2020-09-15 16:51:29,768] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - No seed-nodes configured, manual cluster join required, see https://doc.akka.io/docs/akka/current/typed/cluster.html#joining

[2020-09-15 16:51:29,892] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Node [akka://ClusterSystem@127.0.0.1:64600] is JOINING itself (with roles [dc-default]) and forming new cluster
[2020-09-15 16:51:29,892] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Node [akka://ClusterSystem2@127.0.0.1:64609] is JOINING itself (with roles [dc-default]) and forming new cluster
[2020-09-15 16:51:29,898] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - is the new leader among reachable nodes (more leaders may exist)
[2020-09-15 16:51:29,898] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - is the new leader among reachable nodes (more leaders may exist)
[2020-09-15 16:51:29,917] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Leader is moving node [akka://ClusterSystem@127.0.0.1:64600] to [Up]
[2020-09-15 16:51:29,917] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Leader is moving node [akka://ClusterSystem2@127.0.0.1:64609] to [Up]
[2020-09-15 16:51:29,931] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-3] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Marked address [akka://ClusterSystem@127.0.0.1:64600] as [Leaving]
[2020-09-15 16:51:29,933] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-15] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Marked address [akka://ClusterSystem2@127.0.0.1:64609] as [Leaving]
[2020-09-15 16:51:30,365] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Leader is moving node [akka://ClusterSystem@127.0.0.1:64600] to [Exiting]
[2020-09-15 16:51:30,369] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Exiting completed
[2020-09-15 16:51:30,371] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Shutting down...
[2020-09-15 16:51:30,373] [INFO] [akka.cluster.Cluster] [ClusterSystem-akka.actor.default-dispatcher-6] [Cluster(akka://ClusterSystem)] - Cluster Node [akka://ClusterSystem@127.0.0.1:64600] - Successfully shut down
[2020-09-15 16:51:30,391] [INFO] [akka.remote.RemoteActorRefProvider$RemotingTerminator] [ClusterSystem-akka.actor.default-dispatcher-3] [akka://ClusterSystem@127.0.0.1:64600/system/remoting-terminator] - Shutting down remote daemon.
[2020-09-15 16:51:30,394] [INFO] [akka.remote.RemoteActorRefProvider$RemotingTerminator] [ClusterSystem-akka.actor.default-dispatcher-3] [akka://ClusterSystem@127.0.0.1:64600/system/remoting-terminator] - Remote daemon shut down; proceeding with flushing remote transports.
[2020-09-15 16:51:30,419] [INFO] [akka.remote.RemoteActorRefProvider$RemotingTerminator] [ClusterSystem-akka.actor.default-dispatcher-3] [akka://ClusterSystem@127.0.0.1:64600/system/remoting-terminator] - Remoting shut down.
[2020-09-15 16:51:30,778] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Leader is moving node [akka://ClusterSystem2@127.0.0.1:64609] to [Exiting]
[2020-09-15 16:51:30,779] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Exiting completed
[2020-09-15 16:51:30,779] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Shutting down...
[2020-09-15 16:51:30,780] [INFO] [akka.cluster.Cluster] [ClusterSystem2-akka.actor.default-dispatcher-16] [Cluster(akka://ClusterSystem2)] - Cluster Node [akka://ClusterSystem2@127.0.0.1:64609] - Successfully shut down
[2020-09-15 16:51:30,789] [INFO] [akka.remote.RemoteActorRefProvider$RemotingTerminator] [ClusterSystem2-akka.actor.default-dispatcher-15] [akka://ClusterSystem2@127.0.0.1:64609/system/remoting-terminator] - Shutting down remote daemon.
[2020-09-15 16:51:30,789] [INFO] [akka.remote.RemoteActorRefProvider$RemotingTerminator] [ClusterSystem2-akka.actor.default-dispatcher-15] [akka://ClusterSystem2@127.0.0.1:64609/system/remoting-terminator] - Remote daemon shut down; proceeding with flushing remote transports.
[2020-09-15 16:51:30,791] [INFO] [akka.actor.LocalActorRef] [ClusterSystem2-akka.actor.default-dispatcher-16] [akka://ClusterSystem2/system/cluster/core/publisher] - Message [akka.cluster.InternalClusterAction$PublishChanges] from Actor[akka://ClusterSystem2/system/cluster/core/daemon#-964571468] to Actor[akka://ClusterSystem2/system/cluster/core/publisher#-1466189001] was not delivered. [1] dead letters encountered. If this is not an expected behavior then Actor[akka://ClusterSystem2/system/cluster/core/publisher#-1466189001] may have terminated unexpectedly. This logging can be turned off or adjusted with configuration settings 'akka.log-dead-letters' and 'akka.log-dead-letters-during-shutdown'.

Process finished with exit code 0
```

## 2.5. 集群订阅

集群订阅：可用于在集群状态更改时接收消息,此示例订阅了一个ActorRef<MemberEvent>订阅服务器，控制集群成员添加和成员离开时，订阅用户subscriber将收到事件MemberUp，MemberLeft，MemberExited和MemberRemoved等信息。


```java
 /**
     * 集群订阅：可用于在集群状态更改时接收消息
     *
     * @throws Exception
     */
    @Test
    public void clusterSubscribe() throws Exception {
        ActorSystem<Object> system =
                ActorSystem.create(Behaviors.empty(), "ClusterSystem", noPort.withFallback(clusterConfig));

        try {
            Cluster cluster = Cluster.get(system);

            TestProbe<ClusterEvent.MemberEvent> testProbe = TestProbe.create(system);
            ActorRef<ClusterEvent.MemberEvent> subscriber = testProbe.getRef();
            // #cluster-subscribe
            
            cluster.subscriptions().tell(Subscribe.create(subscriber, ClusterEvent.MemberEvent.class));
            // #cluster-subscribe

            Address anotherMemberAddress = cluster.selfMember().address();
            // #cluster-leave-example
            cluster.manager().tell(Join.create(anotherMemberAddress));
            cluster.manager().tell(Leave.create(anotherMemberAddress));
            // subscriber will receive events MemberLeft, MemberExited and MemberRemoved
            // #cluster-leave-example
            testProbe.expectMessageClass(ClusterEvent.MemberUp.class);
            testProbe.expectMessageClass(ClusterEvent.MemberLeft.class);
            testProbe.expectMessageClass(ClusterEvent.MemberExited.class);
            testProbe.expectMessageClass(ClusterEvent.MemberRemoved.class);

        } finally {
            system.terminate();
        }
    }
```


## 2.6. 设置种子节点

**种子节点简介**

种子节点是加入集群的初始接触点，可以通过不同的方式实现
* 配置文件指定
* JVM启动参数指定
* 程序中添加

在加入过程之后，种子节点并不特殊，它们以与其他节点完全相同的方式参与到集群中。

通过集群引导自动连接到种子节点，使用开源的Akka管理项目的模块Cluster Bootstrap，可以自动发现连接过程的节点。请参阅其文档以了解更多细节。

当一个新节点启动时，它向所有配置的种子节点发送一条消息，然后向第一个响应的节点发送一个join命令。如果没有一个种子节点响应(可能还没有启动)，它将重试此过程，直到成功或关闭。

种子节点可以按照任何顺序启动。没有必要让所有种子节点都运行，但是在初始启动集群时必须启动配置为种子节点列表中的第一个元素的节点。否则，其他种子节点将不会被初始化，并且没有其他节点可以加入集群。

**种子节点加入方式**

具体加入方式
1. conf配置文件指定
akka{
     cluster{
        seed-nodes = [“akka://ClusterSystem@host1:2552”,
        “akka://ClusterSystem@host2:2552”]
     }
 }
1. 可在JVM启动是指定
-Dakka.cluster.seed-nodes.0=akka://ClusterSystem@host1:2552
-Dakka.cluster.seed-nodes.1=akka://ClusterSystem@host2:2552

3. 通过本示例程序中加入种子节点
 Cluster.get(system).manager().tell(new JoinSeedNodes(seedNodes));

 **示例程序添加种子节点**

```java
 public void illustrateJoinSeedNodes() {
    ActorSystem<Object> system =
            ActorSystem.create(Behaviors.empty(), "ClusterSystem", noPort.withFallback(clusterConfig));

    // #join-seed-nodes
    List<Address> seedNodes = new ArrayList<>();
    seedNodes.add(AddressFromURIString.parse("akka://ClusterSystem@127.0.0.1:2551"));
    seedNodes.add(AddressFromURIString.parse("akka://ClusterSystem@127.0.0.1:2552"));

    Cluster.get(system).manager().tell(new JoinSeedNodes(seedNodes));
    // #join-seed-nodes
}
```

## 2.7. 集群会员角色

集群的角色通过指定配置akka.cluster.roles，即overrides.put("akka.cluster.roles", Collections.singletonList(role))即可指定集群会员角色，获取角色可通过下面示例方式获取集群角色。

```java
 /**
     * 集群角色示例
     *
     * 通过指定配置akka.cluster.roles
     * overrides.put("akka.cluster.roles", Collections.singletonList(role));
     */
    void illustrateRoles() {
        ActorContext<Void> context = null;

        // #hasRole
        Member selfMember = Cluster.get(context.getSystem()).selfMember();
        if (selfMember.hasRole("backend")) {
            context.spawn(Backend.create(), "back");
        } else if (selfMember.hasRole("front")) {
            context.spawn(Frontend.create(), "front");
        }
        // #hasRole
    }
```



 






