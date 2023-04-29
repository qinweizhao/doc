# Maven 常用插件

> Maven 本质上是一个插件框架，它的核心并不执行任何具体的构建任务，所有这些任务都交给插件来完成。

## 一、分类

### 1、spring-boot-maven-plugin

 负责将源码和依赖、以及 springboot loader 相关内容打成一个包。保证打出的包能独立运行。

常用配置：

```xml
<plugin>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-maven-plugin</artifactId>
     <version>x.x.x</version>
     <configuration>
      <mainClass>com.xx.xx</mainClass>
     </configuration>
     <executions>
      <execution>
      <goals>
        <goal>repackage</goal>
      </goals>
      </execution>
     </executions>
</plugin>
```

### 2、maven-jar-plugin

仅负责将源码打成 jar 包，不能独立运行。另外，可以根据你的设置，将依赖 jar 包路径和程序的主入口定义在所打 jar 包中的 MANIFEST.MF 文件里。这里只会对当前的主项目打出包，其主项目依赖的 jar 包则不管。

常用配置：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <!-- 打包时包含的文件配置，在暗黑我的这个工程中，只打包 com 文件夹 -->
        <includes>
            <include>
                **/com/**
            </include>
        </includes>
 
        <archive>
            <manifest>
                <!-- 配置加入依赖包 -->
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
                <useUniqueVersions>false</useUniqueVersions>
                <!-- Spring Boot 启动类(自行修改) -->
                <mainClass>com.ccccit.springdockerserver.SpringDockerServerApplication</mainClass>
            </manifest>
            <manifestEntries>
                <!-- 外部资源路径加入 manifest.mf 的 Class-Path -->
                <Class-Path>resources/</Class-Path>
            </manifestEntries>
        </archive>
        <!-- jar 输出目录 -->
        <outputDirectory>${project.build.directory}/pack/</outputDirectory>
    </configuration>
</plugin>
```

### 3、maven-dependency-plugin

负责将各种依赖打包。也可以根据设置，将所打的依赖 jar 包输出到指定位置。maven-dependency-plugin最大的用途是帮助分析项目依赖，dependency:list 能够列出项目最终解析到的依赖列表，dependency:tree 能进一步的描绘项目依赖树。maven-dependency-plugin 还有很多目标帮助你操作依赖文件，例如 dependency:copy-dependencies 能将项目依赖从本地 Maven 仓库复制到某个特定的文件夹下面。

**注意**：通常 maven-dependency-plugin 与 maven-jar-plugin 一块使用，可以打出当前项目包及其依赖包，但是要配置 maven-dependency-plugin 的输出路径，否则 target 无法看到依赖包。

常见配置：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.2.0</version>
    <!-- 复制依赖 -->
    <executions>
        <execution>
            <!-- 这里的id可以随意写，与下面的goal无名称上的必然联系 -->
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <!-- 依赖包 输出目录 -->
                <outputDirectory>${project.build.directory}/pack/lib</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 4、maven-resource-plugin

负责将正式与测试用到的资源文件导出到指定位置。还负责用资源文件中的参数替换 pom 文件中对应的占位符。

常用配置：

```xml
<!-- 在打包时，动态将maven的参数传给resource文件夹下的参数 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <encoding>UTF-8</encoding>
        <!-- 当这里为true，那么resource文件夹下的配置文件，比如application.yml这些文件里面的${}包起来的内容就可以被pom文件中profiles标签下的对应名称部分行替换了 -->
        <useDefaultDelimiters>true</useDefaultDelimiters>
    </configuration>
    <!-- 复制资源和bin文件夹  -->
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>package</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <resources>
                    <resource>
                        <!-- 文件来源 -->
                        <directory>src/main/resources</directory>
                    </resource>
                </resources>
                <!-- 资源文件的输出目录 -->
                <outputDirectory>${project.build.directory}/pack/resources</outputDirectory>
            </configuration>
        </execution>
        <execution>
            <id>copy-bin</id>
            <phase>package</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <resources>
                    <resource>
                        <!-- 文件来源 -->
                        <directory>src/main/bin</directory>
                    </resource>
                </resources>
                <!-- 资源文件的输出目录 -->
                <outputDirectory>${project.build.directory}/pack/bin</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 5、maven-assembly-plugin

它是 maven 中针对打包任务而提供的标准插件。允许用户将项目输出与它的依赖项、模块、站点文档、和其他文件一起组装成一个可分发的归档文件。即结构定制化的打包。

常用配置：

**assembly.xml**

```xml
<assembly>
    <id>assembly</id>
    <formats>
        <!--zip/tar/tar.gz或tgz/tar.bz2或tbz2/tar.snappy/tar.xz或txz/jar/dir/war-->
        <format>tar.gz</format>
    </formats>
    <includeBaseDirectory>true</includeBaseDirectory>
 
    <fileSets>
        <fileSet>
            <lineEnding>unix</lineEnding>
            <directory>${project.basedir}/src/main/resources</directory>
            <outputDirectory>./config</outputDirectory>
            <includes>
                <include>application*.properties</include>
                <include>log*.xml</include>
            </includes>
        </fileSet>
        <fileSet>
            <lineEnding>unix</lineEnding>
            <directory>${project.basedir}/bin</directory>
            <outputDirectory>./bin</outputDirectory>
        </fileSet>
        <!-- 把项目自己编译出来的jar文件，打包进zip文件的根目录 -->
        <fileSet>
            <directory>${project.build.directory}</directory>
            <outputDirectory>./lib</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>
    </fileSets>
 
</assembly>
```

 **pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <dependencies>
        <dependencie>
            ...
        </dependencie>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                ...
            </plugin>
 
            <!--   这个插件是关键   -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <!--   这个是assembly 所在位置；${basedir}是指项目的的根路径  -->
                    <descriptors>
                        descriptor>${basedir}/src/main/assembly/assembly.xml</descriptor>
                    </descriptors>
                    <!--打包解压后的目录名；${project.artifactId}是指：项目的artifactId-->
                    <finalName>${project.artifactId}</finalName>
                    <!-- 打包压缩包位置-->
                    <outputDirectory>${project.build.directory}/release</outputDirectory>
                    <!-- 打包编码 -->
                    <encoding>UTF-8</encoding>
                </configuration>
                <executions>
                    <execution><!-- 配置执行器 -->
                        <id>make-assembly</id>
                        <phase>package</phase><!-- 绑定到package生命周期阶段上 -->
                        <goals>
                            <goal>single</goal><!-- 只运行一次 --> 
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## 二、补充

Maven 官方文档：

[Maven – Available Plugins (apache.org)](https://maven.apache.org/plugins/)
