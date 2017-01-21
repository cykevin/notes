# JDK、JRE与JVM的区别
JDK是Java开发包，为java开发者提供，包含了编译java源代码、发布Java程序的工具、Java类库和运行java程序的必须的环境。

JRE是运行java程序必须的环境，包含了程序会调用到的类库以及JVM

JVM是Java跨平台的核心技术，JVM将java字节码转换成对应操作系统的CPU指令。

来自官网的介绍图

![java 结构示例图](java_images\java.png)

# 安装JDK
从官网下载java se development kit安装包

* 双击安装，一路下一步直到完成。
*  配置3个环境变量
    *  JAVA_HOME：**jdk安装路径**
    *  CLASSPATH：**.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;**
    *  追加Path环境变量：**;%JAVA_HOME%\bin**
JDK下载地址：[http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
能看出来，这三者是层层包含的关系。

验证安装：在命令行下输入：java -version，能查看安装的java版本说明安装成功。

eclipse下载地址：[https://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/mars1](https://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/mars1)

