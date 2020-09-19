---
layout: article
title: SPI 注入失败
tags: []
key: 3da3520e-22e3-44dd-b4a5-f53a372ad1b4
---

使用 `maven-assembly-plugin` 打包可执行 jar 包时, SPI 异常.

<!--more-->

## 问题

使用 `maven-assembly-plugin` 打包可执行 jar 包,启动的时候报如下问题:

```bash
19/12/25 14:53:03 INFO elasticsearch.plugins: [Elizabeth Twoyoungmen] modules [], plugins [], sites []
19/12/25 14:53:04 INFO client.transport: [Elizabeth Twoyoungmen] failed to get local cluster state for {#transport#-1}{172.19.233.149}{172.19.233.149:19590}, disconnecting...
RemoteTransportException[[Failed to deserialize response of type [org.elasticsearch.action.admin.cluster.state.ClusterStateResponse]]]; nested: TransportSerializationException[Failed to deserialize response of type [org.elasticsearch.action.admin.cluster.state.ClusterStateResponse]]; nested: ExceptionInInitializerError; nested: IllegalArgumentException[An SPI class of type org.apache.lucene.codecs.PostingsFormat with name 'Lucene50' does not exist.  You need to add the corresponding JAR file supporting this SPI to your classpath.  The current classpath supports the following names: [es090, completion090, XBloomFilter]];
Caused by: TransportSerializationException[Failed to deserialize response of type [org.elasticsearch.action.admin.cluster.state.ClusterStateResponse]]; nested: ExceptionInInitializerError; nested: IllegalArgumentException[An SPI class of type org.apache.lucene.codecs.PostingsFormat with name 'Lucene50' does not exist.  You need to add the corresponding JAR file supporting this SPI to your classpath.  The current classpath supports the following names: [es090, completion090, XBloomFilter]];
	at org.elasticsearch.transport.netty.MessageChannelHandler.handleResponse(MessageChannelHandler.java:180)
	at org.elasticsearch.transport.netty.MessageChannelHandler.messageReceived(MessageChannelHandler.java:138)
	at org.jboss.netty.channel.SimpleChannelUpstreamHandler.handleUpstream(SimpleChannelUpstreamHandler.java:70)
	at org.jboss.netty.channel.DefaultChannelPipeline.sendUpstream(DefaultChannelPipeline.java:564)
	at org.jboss.netty.channel.DefaultChannelPipeline$DefaultChannelHandlerContext.sendUpstream(DefaultChannelPipeline.java:791)
	at org.jboss.netty.channel.Channels.fireMessageReceived(Channels.java:296)
	at org.jboss.netty.handler.codec.frame.FrameDecoder.unfoldAndFireMessageReceived(FrameDecoder.java:462)
	at org.jboss.netty.handler.codec.frame.FrameDecoder.callDecode(FrameDecoder.java:443)
	at org.jboss.netty.handler.codec.frame.FrameDecoder.messageReceived(FrameDecoder.java:303)
	at org.jboss.netty.channel.SimpleChannelUpstreamHandler.handleUpstream(SimpleChannelUpstreamHandler.java:70)
	at org.jboss.netty.channel.DefaultChannelPipeline.sendUpstream(DefaultChannelPipeline.java:564)
	at org.jboss.netty.channel.DefaultChannelPipeline.sendUpstream(DefaultChannelPipeline.java:559)
	at org.jboss.netty.channel.Channels.fireMessageReceived(Channels.java:268)
	at org.jboss.netty.channel.Channels.fireMessageReceived(Channels.java:255)
	at org.jboss.netty.channel.socket.nio.NioWorker.read(NioWorker.java:88)
	at org.jboss.netty.channel.socket.nio.AbstractNioWorker.process(AbstractNioWorker.java:108)
	at org.jboss.netty.channel.socket.nio.AbstractNioSelector.run(AbstractNioSelector.java:337)
	at org.jboss.netty.channel.socket.nio.AbstractNioWorker.run(AbstractNioWorker.java:89)
	at org.jboss.netty.channel.socket.nio.NioWorker.run(NioWorker.java:178)
	at org.jboss.netty.util.ThreadRenamingRunnable.run(ThreadRenamingRunnable.java:108)
	at org.jboss.netty.util.internal.DeadLockProofWorker$1.run(DeadLockProofWorker.java:42)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.lang.ExceptionInInitializerError
	at org.elasticsearch.Version.fromId(Version.java:564)
	at org.elasticsearch.Version.readVersion(Version.java:308)
	at org.elasticsearch.cluster.node.DiscoveryNode.readFrom(DiscoveryNode.java:339)
	at org.elasticsearch.cluster.node.DiscoveryNode.readNode(DiscoveryNode.java:322)
	at org.elasticsearch.cluster.node.DiscoveryNodes.readFrom(DiscoveryNodes.java:594)
	at org.elasticsearch.cluster.node.DiscoveryNodes$Builder.readFrom(DiscoveryNodes.java:674)
	at org.elasticsearch.cluster.ClusterState.readFrom(ClusterState.java:699)
	at org.elasticsearch.cluster.ClusterState$Builder.readFrom(ClusterState.java:677)
	at org.elasticsearch.action.admin.cluster.state.ClusterStateResponse.readFrom(ClusterStateResponse.java:58)
	at org.elasticsearch.transport.netty.MessageChannelHandler.handleResponse(MessageChannelHandler.java:178)
	... 23 more
Caused by: java.lang.IllegalArgumentException: An SPI class of type org.apache.lucene.codecs.PostingsFormat with name 'Lucene50' does not exist.  You need to add the corresponding JAR file supporting this SPI to your classpath.  The current classpath supports the following names: [es090, completion090, XBloomFilter]
	at org.apache.lucene.util.NamedSPILoader.lookup(NamedSPILoader.java:114)
	at org.apache.lucene.codecs.PostingsFormat.forName(PostingsFormat.java:112)
	at org.elasticsearch.common.lucene.Lucene.<clinit>(Lucene.java:65)
	... 33 more
```

## 排查

重要的是这一句 `An SPI class of type org.apache.lucene.codecs.PostingsFormat with name 'Lucene50' does not exist.`

查看源码可以知道 `org.apache.lucene.codecs.PostingsFormat` 这个服务有多个依赖的 jar 包进行注入:

1. `org.elasticsearch:elasticsearch:2.3.2` 包的`/META-INF/services/org.apache.lucene.codecs.PostingsFormat`,内容为:

    ```bash
    org.elasticsearch.index.codec.postingsformat.Elasticsearch090PostingsFormat
    org.elasticsearch.search.suggest.completion.Completion090PostingsFormat
    org.elasticsearch.index.codec.postingsformat.BloomFilterPostingsFormat
    ```
 2. `org.apache.lucene:lucene-core:5.5.0` 包的 `/META-INF/services/org.apache.lucene.codecs.PostingsFormat`,内容为:

    ```bash
    org.apache.lucene.codecs.lucene50.Lucene50PostingsFormat
    ```
    

使用 `maven-assembly-plugin` 进行打包的时候会将依赖的所有 jar 包解压,然后合并到一个 jar 包,同名文件默认直接覆盖造成了 `org.apache.lucene:lucene-core:5.5.0` 包的 SPI 注入配置被覆盖.

## 解决方法

使用 `maven-shade-plugin` 替代 `maven-assembly-plugin`,并且指定合并 SPI 配置文件: `org.apache.maven.plugins.shade.resource.ServicesResourceTransformer`

pom.xml 文件内容:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.jiguang</groupId>
    <artifactId>iapp</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- JPush Maven Repository  -->
    <repositories>
        <repository>
            <id>nexus</id>
            <name>nexus</name>
            <url>http://maven.jpushoa.com/nexus/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>

    <properties>
        <encoding>UTF-8</encoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <hadoop.version>2.6.0</hadoop.version>
    </properties>

    <dependencies>
        <!--java使用-->
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>2.3.2</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
    </dependencies>


    <build>
        <finalName>es-test</finalName>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                    <encoding>${encoding}</encoding>
                </configuration>
            </plugin>

<!--            <plugin>-->
<!--                <artifactId>maven-assembly-plugin</artifactId>-->
<!--                <configuration>-->
<!--                    <descriptorRefs>-->
<!--                        <descriptorRef>jar-with-dependencies</descriptorRef>-->
<!--                    </descriptorRefs>-->
<!--                    <archive>-->
<!--                        <manifest>-->
<!--                            <mainClass>cn.jiguang.util.ESUtils</mainClass>-->
<!--                        </manifest>-->
<!--                    </archive>-->
<!--                </configuration>-->
<!--                <executions>-->
<!--                    <execution>-->
<!--                        <id>make-assembly</id>-->
<!--                        <phase>package</phase>-->
<!--                        <goals>-->
<!--                            <goal>assembly</goal>-->
<!--                        </goals>-->
<!--                    </execution>-->
<!--                </executions>-->
<!--            </plugin>-->


            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.1</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer"/>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <manifestEntries>
                                        <Main-Class>cn.jiguang.util.ESUtils</Main-Class>
                                    </manifestEntries>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
