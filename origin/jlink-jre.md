# 使用 Jlink构建最小化依赖的 JRE

# 一、简介

JDK11之前的版本安装JDK的时候，JDK安装完成后会弹出一个框，让我们安装公共JRE，当公共JRE安装完成后，我们就拥有了一个JDK中封装的专用JRE和一个独立的公共JRE。从JDK11开始及之后的版本，Oracle把JRE集成到了JDK中，默认是不会自动安装JRE的。

​	**从JDK 9开始商业化运作，引入JPMS ( Java Platform Module System )模块化系统。之前的JDK类库目前太臃肿了，在一些微型设备上可能用不到全部的功能，却不得不引用全部的类库。模块功能后，JDK、JRE、甚至是JAR都可以把用不到的类库排除掉，大大降低了依赖库的规模 。同时还自带了一个叫 Jlink的工具可以让开发者手动编译自定义modules的JRE 。**

​	**而开发者如何知道自己的项目都需要哪些 java modules来最小化运行。这就需要Java8中提供的一个工具jdeps(java dependencies)。它可以显示Java类文件的包级或类级依赖关系。输入类可以是.class文件、目录、jar文件的路径名，或者可以是完全限定的类名称。具体命令参数可以参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jdeps.html**

当开发者知道了自己项目所依赖的 Java Modules，就可以构建出来最小化的 JRE。

最小化的 JRE 有哪些好处：

- 加快项目启动速度

- 减小构建项目 docker镜像的大小，进而也可以快速分发镜像

- 增加安全性

上述不管 Java底层代码 module化，还是开发 jlink 构建 jre，都是在适应开发者项目的容器化。

# 二、示例操作

## 1、查看当前 Java 版本所支持的 Modules

> java --list-modules

```bash
jdk.internal.jvmstat@17.0.7                 jdk.naming.dns@17.0.7               java.rmi@17.0.7
jdk.internal.le@17.0.7                      jdk.naming.rmi@17.0.7               java.scripting@17.0.7
jdk.internal.opt@17.0.7                     jdk.net@17.0.7                      java.se@17.0.7
jdk.internal.vm.ci@17.0.7                   jdk.nio.mapmode@17.0.7              java.security.jgss@17.0.7
jdk.internal.vm.compiler@17.0.7             jdk.random@17.0.7                   java.security.sasl@17.0.7
jdk.internal.vm.compiler.management@17.0.7  jdk.sctp@17.0.7                     java.smartcardio@17.0.7
jdk.jartool@17.0.7                          jdk.security.auth@17.0.7            java.sql@17.0.7
jdk.javadoc@17.0.7                          jdk.security.jgss@17.0.7            java.sql.rowset@17.0.7
jdk.jcmd@17.0.7                             jdk.unsupported@17.0.7              java.transaction.xa@17.0.7
jdk.jconsole@17.0.7                         jdk.unsupported.desktop@17.0.7      java.xml@17.0.7
jdk.jdeps@17.0.7                            jdk.xml.dom@17.0.7                  java.xml.crypto@17.0.7
jdk.jdi@17.0.7                              jdk.zipfs@17.0.7                    jdk.accessibility@17.0.7
jdk.jdwp.agent@17.0.7                       java.base@17.0.7                    jdk.attach@17.0.7
jdk.jfr@17.0.7                              java.compiler@17.0.7                jdk.charsets@17.0.7
jdk.jlink@17.0.7                            java.datatransfer@17.0.7            jdk.compiler@17.0.7
jdk.jpackage@17.0.7                         java.desktop@17.0.7                 jdk.crypto.cryptoki@17.0.7
jdk.jshell@17.0.7                           java.instrument@17.0.7              jdk.crypto.ec@17.0.7
jdk.jsobject@17.0.7                         java.logging@17.0.7                 jdk.dynalink@17.0.7
jdk.jstatd@17.0.7                           java.management@17.0.7              jdk.editpad@17.0.7
jdk.localedata@17.0.7                       java.management.rmi@17.0.7          jdk.hotspot.agent@17.0.7
jdk.management@17.0.7                       java.naming@17.0.7                  jdk.httpserver@17.0.7
jdk.management.agent@17.0.7                 java.net.http@17.0.7                jdk.incubator.foreign@17.0.7
jdk.management.jfr@17.0.7                   java.prefs@17.0.7                   jdk.incubator.vector@17.0.7
jdk.internal.ed@17.0.7
```

## 2、查看项目所依赖的 Java Modules

由于项目使用了 SpringBoot 。所以先查看SpringBoot 所依赖的 Java Modules

```bash
# 查看Springboot 3所依赖的 Java Modules (springboot小版本对依赖的 java modules基本一致)
jdeps -R -s --multi-release 13 -cp 'path-to-dependencies/*'  ~/.m2/repository/org/springframework/boot/spring-boot/3.0.7/spring-boot-3.0.7.jar 

# spring-boot-3.0.7.jar -> java.base
# spring-boot-3.0.7.jar -> java.compiler
# spring-boot-3.0.7.jar -> java.desktop
# spring-boot-3.0.7.jar -> java.logging
# spring-boot-3.0.7.jar -> java.management
# spring-boot-3.0.7.jar -> java.naming
# spring-boot-3.0.7.jar -> java.sql
# spring-boot-3.0.7.jar -> java.xml
# spring-boot-3.0.7.jar -> 找不到
```

> 上面依赖的 Java modules针对Springboot 3依旧缺少相关的 module,运行 jar包时报：`org.ietf.jgss.GSSException`，而这个 package 是在 `java.security.jgss`这个 module 中。详情查看：https://docs.oracle.com/en/java/javase/11/docs/api/java.security.jgss/org/ietf/jgss/package-summary.html

```bash
# 查看Springboot 2所依赖的 Java Modules (springboot小版本对依赖的 java modules基本一致)
jdeps -R -s --multi-release 13 -cp 'path-to-dependencies/*'  ~/.m2/repository/org/springframework/boot/spring-boot/2.7.5/spring-boot-2.7.5.jar
# spring-boot-2.7.5.jar -> java.base
# spring-boot-2.7.5.jar -> java.desktop
# spring-boot-2.7.5.jar -> java.logging
# spring-boot-2.7.5.jar -> java.management
# spring-boot-2.7.5.jar -> java.naming
# spring-boot-2.7.5.jar -> java.sql
# spring-boot-2.7.5.jar -> java.xml
# spring-boot-2.7.5.jar -> 找不到
```

再查看当前项目所需的  Java Modules

```bash
jdeps -s target/demo-1.0-SNAPSHOT.jar      
demo-1.0-SNAPSHOT.jar -> java.base
demo-1.0-SNAPSHOT.jar -> java.logging
demo-1.0-SNAPSHOT.jar -> 找不到
```

> 基本上项目的代码所需的大部分java modules都包含在SpringBoot 所依赖的 Java Modules中。

## 3、构建能运行 Springboot 所运行的 JRE

```bash
jlink \
    --no-header-files --no-man-pages --compress=2 --strip-debug \
    --add-modules java.base,java.logging,java.naming,java.desktop,java.management,java.security.jgss,java.instrument,java.sql \
    --output ./custom-jre-runtime
```

> 根据Spring Boot 3所需Modules构建的 JRE 大小有 40 多 MB

## 4、测试 jre中的 java运行 jar包

```bash
./custom-jre-runtime/bin/java -jar target/demo-1.0-SNAPSHOT.jar
```

# 参考

- https://access.redhat.com/documentation/en-us/openjdk/17/html-single/using_jlink_to_customize_java_runtime_environment/index
- https://gist.github.com/mballoni/4324d7e89e43103778b2cc0e24d26e42