---
title: 在使用 Ecipse Compiler 的 Maven 项目中如何使用 Lombok
tags:
    - Java
    - Lombok
    - Maven
date: 2013-08-08 11:10:00
---

[Lombok](http://projectlombok.org/) 借助 javac 的内部 API 介入 annotation 处理环节，并根据其特有的 annotation 在编译期生成代码，能够大大简化 Java 代码很多冗余的写法。但正是因为其依赖于 javac 内部 API，Lombok 很难在其他 Java 编译器上使用，例如常用的替代编译器 Eclipse Compiler (ecj) 就是如此。

为了在非 javac 编译器上也能享受 Lombok 带来的好处，一个可能的方案是实际编译之前先借助 javac 的内部运行时库对 Java 代码进行 delombok 操作，即将被 Lombok 修改后的 Java 代码导出成实际文件，再使用 Eclipse Compiler 这样的替代编译器对导出代码进行真正的编译操作。方法如下：
- 先让 Maven 项目在编译前预先执行 delombok 操作，修改 `pom.xml` 增加如下内容：
```xml
<build>
    ...
    <!-- 设置 delombok 展开后代码放置位置 -->
    <sourceDirectory>target/generated-sources/delombok</sourceDirectory>
    <testSourceDirectory>target/generated-test-sources/delombok</testSourceDirectory>
    ...
    <plugins>
        ...
        <plugin>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok-maven-plugin</artifactId>
            <version>0.11.8.0</version>
            <executions>
                <execution>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>delombok</goal>
                    </goals>
                    <configuration>
                        <addOutputDirectory>false</addOutputDirectory>
                        <sourceDirectory>src/main/java</sourceDirectory>
                    </configuration>
                </execution>
                <execution>
                    <id>test-delombok</id>
                    <phase>generate-test-sources</phase>
                    <goals>
                        <goal>testDelombok</goal>
                    </goals>
                    <configuration>
                        <addOutputDirectory>false</addOutputDirectory>
                        <sourceDirectory>src/test/java</sourceDirectory>
                    </configuration>
                </execution>
            </executions>
            <dependencies>
                <dependency>
                    <groupId>sun.jdk</groupId>
                    <artifactId>tools</artifactId>
                    <version>1.6</version>
                    <scope>system</scope>
                    <systemPath>${java.home}/../lib/tools.jar</systemPath>
                </dependency>
            </dependencies>
        </plugin>
        ...
    </plugins>
    ...
</build>
```
- 如果 Maven 项目还没有使用 Eclipse Compiler，则还要修改 `pom.xml` 以启用它：
```xml
<build>
    ...
    <plugins>
        ...
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>1.6</source>
                <target>1.6</target>
                <encoding>UTF-8</encoding>
                <compilerId>eclipse</compilerId>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>org.codehaus.plexus</groupId>
                    <artifactId>plexus-compiler-eclipse</artifactId>
                    <version>1.9.1</version>
                </dependency>
            </dependencies>
        </plugin>
        ...
    </plugins>
    ...
</build>
```

经过以上配置之后，Maven 项目的实际和测试 Java 代码中即可正常使用 Lombok annotation，也可以在 `mvn compile` 之后进入 `target/generated-sources/delombok/` 和 `target/generated-test-sources/delombok/` 目录自行查看被 Lombok 修改后的实际和测试代码。
