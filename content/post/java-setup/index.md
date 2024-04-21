---
title: Java 循序渐进 1.1：配置环境
slug: java-setup
date: 2024-04-20 02:00:00+0000
categories:
  - development
tags:
  - java
  - Java 循序渐进
links:
  - title: 上一节
    description: 1：基础知识
    website: ../java-1
  - title: 下一节
    description: 1.2 JShell
    website: ../java-jshell
---

在开始前，我们先理清以下概念：

- Java：指 Java 编程语言。
    - Java 源文件：指使用 Java 编写的文件。文件后缀名是`.java`。
- JVM：指 Java 虚拟机，可运行字节码文件。
    - 字节码：指 [JVM Specification](https://docs.oracle.com/javase/specs/jvms/se22/html/index.html)
      规定的一系列指令，用单个或多个字节（Byte）表示。
- JDK：指 Java 开发包，包含 Java 的编译器和运行时。
- JRE：指 Java 运行时环境，现已不可单独下载，可用 jlink 自行制作。

Java 与 JVM 无直接关系。JVM 能运行任何规范的字节码文件。

## 下载 JDK

JDK 有多个供应商的版本。Oracle 自己的版本称为 Oracle JDK，其他则是基于 OpenJDK 开发。

OpenJDK 的供应商可在[这里](https://sdkman.io/jdks)找到。

{{< admonition note >}}
不推荐用 Oracle JDK，因为商用收费。
{{< /admonition >}}

本书使用 GraalVM 22。

## 环境变量

下载完 JDK 后，将压缩包内容解压到你能记住的位置，例如`D:/java`。

根据自己的操作系统设置环境变量。

{{< admonition example "以 Windows 10 为例：" >}}
1. 右击开始按钮，点击设置→系统→关于→高级系统设置→环境变量。
2. 这时，请无视**用户变量**中的任何内容。
3. 点击**系统变量**下的**新建**按钮。
4. 在**变量名**内填入`JAVA_HOME`，点击浏览**目录**，选择你刚刚下载的 JDK 的目录（**不包括**`bin`！）。
5. 找到变量 **`Path`**，点击**编辑**按钮。
6. 在弹出的窗口中，点击**新建**按钮，输入`%JAVA_HOME%\bin\`。
7. 确定。
{{< /admonition >}}

## 测试

找到你的操作系统的终端，输入`javac --version`，运行。
输出`javac 22`或类似的内容，则说明成功了。
如果提示找不到文件，请检查环境变量是否配置成功。

{{< admonition question "如果你是 Windows 用户，且不知道什么是终端：" >}}
打开开始菜单，搜索 cmd。打开命令提示符。
{{< /admonition >}}
