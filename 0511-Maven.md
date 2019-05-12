[TOC]



# 0511-Maven

## 坐标和依赖

Maven坐标包括groupId, artifactId, version, packaging, classifier。对于依赖配置，只需要配置基本坐标：groupId, artifactId 和version

### 依赖范围

- compile：编译依赖范围。默认。使用此依赖范围的Maven dependency在编译、测试、运行三种classpath都有效
- test：测试依赖范围，只在测试时有效
- provided：以提供依赖范围。dependency对于编译和测试classpath有效，典型例子就是lambok
- runtime：运行时依赖范围，dependency对于测试和运行classpath有效。典型的就是jdbc的驱动实现
- system：系统依赖范围，依赖范围与provided一致，但是使用system范围的依赖时必须通过systemPath元素显示指定依赖文件的路径
- import：导入依赖范围，对三种classpath都不产生影响。This scope is only supported on a dependency of type `pom` in the `<dependencyManagement>` section. It indicates the dependency to be replaced with the effective list of dependencies in the specified POM's `<dependencyManagement>` section. Since they are replaced, dependencies with a scope of `import` do not actually participate in limiting the transitivity of a dependency.

还有一个不属于依赖范围内的测试类代码打包，测试类中可能有些常用的测试工具类，或者测试基类想重用，这时候可以通过plugin将test目录下的代码打包：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.2</version>
    <executions>
        <execution>
            <goals>
                <goal>test-jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 传递性依赖

|          | compile    | provided | runtime  | test |
| -------- | ---------- | -------- | -------- | ---- |
| compile  | compile(*) | -        | runtime  | -    |
| provided | provided   | -        | provided | -    |
| runtime  | runtime    | -        | runtime  | -    |
| test     | test       | -        | test     | -    |

第一纵列表示第一直接依赖（if a dependency is set to the scope in the left column）

第一行列表示第二传递（transitive dependencies of that dependency with the scope across the top row will result in a dependency in the main project with the scope listed at the intersection.）

### 依赖调解(Depenency Mediation)

例如有这样的依赖关系：A -> B -> C -> X(1.0); A -> D -> X(2.0)

maven依赖调解的第一原则是路径近者优先，所以上例会采用第二个中的X

再假设：A->B->Y(1.0); C->D->Y(2.0)

第二原则：第一声明者优先，所以在依赖长度相等的前提下，在POM中声明顺序靠前的那个依赖。此例中采用1.0版本的Y

### 可选依赖

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook</groupId>
    <artifactId>project-b</artifactId>
    <version>1.0.0</version>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.10</version>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>8.4-701.jdbc3</version>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

可能项目实现了两个特性，其中特性一依赖于X，特性二依赖于Y，而且这两个特性是互斥的，用户不可能同时使用两个特性，比如支持多种持久化的方式。

可选依赖只会对当前项目产生影响，当其他项目依赖于此项目的时候，这两个可选依赖都不会传递，因此当依赖的项目需要用到具体的持久化jar包时，需要显式声明

### 排除依赖

比较简单，使用$<exclution>$，在这个元素里面只需要提供groupId和artifactId，不需要version信息

### 归类依赖

就是使用定义一些properties常量，然后使用这些属性

### 优化依赖

- 查看当前项目已解析依赖：mvn dependency:list
- 查看依赖树：mvn dependency:tree
- 分析依赖：mvn dependency:analyze

## 仓库

### 仓库的布局

本地仓库包位置：groupId/artifactId/version/artifactId-version.packaging

### 远程仓库的配置

```xml
<project>
……
    <repositories>
        <repository>
            <id>jboss</id>
            <name>JBoss Repository</name>
            <url>http://repository.jboss.com/maven2/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
                <updatePolicy>daily</updatePolicy>
                <checksumPolicy>ignore</checksumPolicy>
            </snapshots>
            <layout>default</layout>
        </repository>
    </repositories>
……
</project>
```

在这里配置了一个id叫jboss的仓库，任何一个仓库声明的id必须唯一。Maven自带的中央仓库使用的id为central，如果其他仓库声明也使用了该id，就会覆盖中央仓库的配置

对于release和snapshot的enable表示是否支持release和snapshot的下载，updatePolicy用来配置Maven从远程仓库检查更新的频率，默认为daily，可用的值还有never, always(每次构建), interval:X(X分钟)，checksumPolicy用来配置Maven检查检验和文件的策略。如果校验和验证失败，会根据checksumPolicy的配置输出，默认为warn，其他可用值包括fail, ignore

当依赖版本不明确时，如RELEASE，LATEST和SNAPSHOT的时候，maven就需要拿到最新的元数据计算出对应的版本值，这里需要release或snapshots中的enabled为true

另外用户可以在命令行值加入参数 -U强制检查更新忽略updatePolicy的配置

maven-metadata.xml示例：

```xml
<?xml version="1.0"encoding="UTF-8"?>
<metadata>
    <groupId>org.sonatype.nexus</groupId>
    <artifactId>nexus</artifactId>
    <versioning>
        <latest>1.4.2-SNAPSHOT</latest>
        <release>1.4.0</release>
        <versions>
            <version>1.3.5</version>
            <version>1.3.6</version>
            <version>1.4.0-SNAPSHOT</version>
            <version>1.4.0</version>
            <version>1.4.0.1-SNAPSHOT</version>
            <version>1.4.1-SNAPSHOT</version>
            <version>1.4.2-SNAPSHOT</version>
        </versions>
        <lastUpdated>20091214221557</lastUpdated>
    </versioning>
</metadata>
```



### 从仓库解析依赖的机制

1. 当依赖的范围时system的时候，maven直接从本地文件系统解析构件
2. 根据依赖坐标计算仓库路径后，尝试直接从本地仓库寻找构件，如果发现，解析成功
3. 本地仓库不存在相应构件的情况下，如果依赖的是显式的发布版本，如1.2，则遍历所有远程仓库，发现后下载并解析使用
4. 如果依赖的版本是RELEASE或者LATEST，则基于更新策略读取所有远程仓库的元数据(/groupId/argifactId/maven-metadata.xml)将其与本地仓库的对应元数据合并后，计算出RELEASE或LATEST真实的值，然后基于这个真实的值检查本地和远程仓库
5. 如果依赖的版本是SNAPSHOT，则基于更新策略读取所有远程仓库的元数据(groupId/artifactId/version/maven-metadata.xml),将其与本地仓库的对应元数据合并后，得到最新快照版本的值，基于该值从本地或远程仓库中下载
6. 如果最后解析得到的构件版本是时间戳格式的快照如1.4.1-20091104.121450-121，则复制其时间戳格式的文件至非时间戳格式，如SNAPSHOT，并使用该非时间戳格式的构件。

## 生命周期



### Clean

包含的阶段有：pre-clean, clean, post-clean

当调用clean的时候，pre-clean和clean阶段会顺序执行；当调用post-clean的时候，pre-clean, clean 和 post-clean会顺序执行。

### Default

- validate
- initialize
- generate-sources
- process-resources: 处理项目主资源文件。一般来说，是对src/main/resources目录的内容进行变量替换等工作后，复制到项目输出的主classpath目录中。
- generate-resources
- process-resources
- compile: 编译项目的主源码。一般来说，是编译src/main/java目录下的Java文件至项目输出的主classpath目录中。
- process-classes
- generate-test-sources
- process-test-sources: 处理项目测试资源文件。一般来说，是对src/test/resources目录的内容进行变量替换等工作后，复制到项目输出的测试classpath目录中。
- generate-test-resources
- process-test-resources
- test-compile: 编译项目的测试代码。一般来说，是编译src/test/java目录下的Java文件至项目输出的测试classpath目录中。
- process-test-classes
- test: 使用单元测试框架运行测试，测试代码不会被打包或部署。
- prepare-package
- package: 接受编译好的代码，打包成可发布的格式，如jar
- pre-integration-test
- post-integration-test
- verify
- install: 将抱安装到Maven本地仓库，供本地其他Maven项目使用
- deploy: 将最终的包复制到远程仓库，供其他开发人和Maven项目使用

### Site

建立和发布项目站点

pre-site, site, post-site, site-deploy

Maven拥有三套相互独立的生命周期，可以仅调用clean生命周期的某个阶段，不会对其他生命周期产生任何影响。

## Goal

mvn clean install: 该命令调用clean生命周期的pre-clean, clean 阶段以及default生命周期validate至install的所有阶段。

Maven的生命周期与插件的目标相互绑定，已完成具体的构建任务，内置的绑定有：

| Phase                    | plugin:goal               |
| :----------------------- | :------------------------ |
| `process-resources`      | `resources:resources`     |
| `compile`                | `compiler:compile`        |
| `process-test-resources` | `resources:testResources` |
| `test-compile`           | `compiler:testCompile`    |
| `test`                   | `surefire:test`           |
| `package`                | `jar:jar`                 |
| `install`                | `install:install`         |
| `deploy`                 | `deploy:deploy`           |
| clean                    | clean:clean               |

### 自定义绑定

这里利用source plugin在verify的时候执行jar-no-fork的goal.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.1.1</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



## 插件配置

1. 命令行插件配置：mvn install -Dmaven.test.skip=true, -D是java自带的，可以设置一个Java系统属性

2. POM中插件全局配置：例如给compiler配置target为1.8版本

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-compiler-plugin</artifactId>
               <version>2.1</version>
               <configuration>
                   <source>1.8</source>
                   <target>1.8</target>
               </configuration>
           </plugin>
       </plugins>
   </build>
   ```

3. POM中插件任务配置

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>1.3</version>
            <executions>
                <execution>
                    <id>ant-validate</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    <configuration>
                        <tasks>
                            <echo>I'm bound to validate phase.</echo>
                        </tasks>
                    </configuration>
                </execution>
                <execution>
                    <id>ant-verify</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>run</goal>
                    </goals>
                    <configuration>
                        <tasks>
                            <echo>I'm bound to verify phase.</echo>
                        </tasks>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



## 命令相关

1. 使用maven-help-plugin描述插件：mvn help:describe -Dplugin=org.apache.maven.plugins:maven-compiler-plugin:2.1，或者简化为：mvn help:describe -Dplugin=compiler
2. 命令行调用插件：usage: mvn [options] [<goal (s)>] [<phase(s)>], 例如：mvn dependency:tree   <!--dependency是maven-dependency-plugin的目标前缀，有了目标前缀，maven就能找到对应的artifactId-->

## 插件解析机制

### 插件仓库

插件构件同样基于坐标存储在Maven仓库中。Maven会从本地仓库寻找，如果不存在，从远程插件仓库查找，找到就会下载到本地仓库。插件的远程仓库配置使用的是pluginRepository

### 解析插件版本

Maven在超级POM中为所有核心插件设定了版本，超级POM是所有Maven项目的父POM，即使用户没有显式指定父POM。如果用户使用某个插件时没有设定版本，Maven会检查仓库中可用的版本，找到release版本

### 解析插件前缀

前缀信息配置在仓库元数据

```xml
<metadata>
    <plugins>
        <plugin>
            <name>Maven Clean Plugin</name>
            <prefix>clean</prefix>
            <artifactId>maven-clean-plugin</artifactId>
        </plugin>
        <plugin>
            <name>Maven Compiler Plugin</name>
            <prefix>compiler</prefix>
            <artifactId>maven-compiler-plugin</artifactId>
        </plugin>
        <plugin>
            <name>Maven Dependency Plugin</name>
            <prefix>dependency</prefix>
            <artifactId>maven-dependency-plugin</artifactId>
        </plugin>
    </plugins>
</metadata>
```

插件元数据的地址为groupId/maven-metadata.xml<!--jar包本地仓库元数据的位置为groupId/artifactId/maven-metadata.xml-->Maven默认使用org.apache.maven.plugins 和 org.codehaus.mojo两个groupId. 可以通过配置setting.xml让Maven检查其他groupId上的插件仓库元数据。所以当Maven解析dependency:tree这样的命令是，首先找到org/apache/maven/plugins/maven-metadata.xml;其次找到归并后的元数据，找到对应的artifactId为maven-dependency-plugin;然后结合当前元数据的groupId，在通过找version的方式找到version，就得到了完整的插件坐标。如果org/apache/maven/plugins/maven-metadata.xml没有记录该插件前缀，则接着检查其他groupId下的元数据，如果所有元数据都不含该前缀则报错。

## 聚合与继承

## 聚合

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-aggregator</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Aggregator</name>
    <modules>
        <module>../account-email</module>
        <module>../account-persist</module>
    </modules>
</project>
```

聚合就是放在一起方便编译，packaging的方式为pom，module的值为当前pom的相对目录

## 继承

Parent

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Parent</name>
</project>
```

Child

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.juvenxu.mvnbook.account</groupId>
        <artifactId>account-parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../account-parent/pom.xml</relativePath>
    </parent>
    <artifactId>account-email</artifactId>
    <name>Account Email</name>
    <dependencies>
……
    </dependencies>
    <build>
        <plugins>
……
        </plugins>
    </build>
</project>
```

child项目构建时，maven会首先根据relativePath检查父POM，如果找不到，再从本地仓库查找，relativePom默认值为../pom.xml

child项目会继承父pom中一些属性包括：groupId, version, description, organization, inceptionYear, url, developers, contributors, distributionManagement, issueManagement, ciManagement, scm, mailingLists, properties, dependencies, dependencyManagement, repositories, build, reporting

### 依赖管理

dependencyManagement既能让子模块继承到父模块的依赖配置，又能保证子模块依赖使用的灵活性。在dependencyManagement元素下的依赖声明不会引入实际的依赖，不过它能够约束dependencies下的依赖使用，例如

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Parent</name>
    <properties>
        <springframework.version>2.5.6</springframework.version>
        <junit.version>4.7</junit.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-support</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

在这里父pom定义了一些framework版本的约束，子pom会直接继承这些约束，当需要配置约束的依赖时，不需要指定版本：

```xml
<properties>
    <javax.mail.version>1.4.1</javax.mail.version>
    <greenmail.version>1.3.1b</greenmail.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
    <dependency>
        <groupId>javax.mail</groupId>
        <artifactId>mail</artifactId>
        <version>${javax.mail.version}</version>
    </dependency>
    <dependency>
        <groupId>com.icegreen</groupId>
        <artifactId>greenmail</artifactId>
        <version>${greenmail.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

当dependencyManagement中定义依赖范围为import的时候，是指将目标POM中的dependencyManagement配置导入并合并到当前POM的dependencyManagement元素中，例如想要一份parent一样的dependencyManagement，除了复制配置或者继承还可以使用import范围依赖这一配置导入如：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.juvenxu.mvnbook.account</groupId>
            <artifactId>account-parent</artifactId>
            <version>1.0-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

type值为pom

### 插件管理

parent:

```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.1.1</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

child:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

子模块声明使用了source plugin并继承parent，那么就会自动合并parent中其他的元素，当子模块不需要使用父模块中的pluginManagement配置的插件，自行配置以覆盖父模块中的配置

### 超级POM

因为超级POM会被所有的POM默认继承，这个POM中定义了所有的约定，位于$MAVEN_HOME/lib/maven-model-build-x.x.x.jar中的org/apache/maven/model/pom-4.0.0.xml路径下。

## 反应堆

在一个多模块的maven项目中，反应堆(Reactor)是指所有模块组成的一个构建结构，对于单模块的项目，反应堆就是该模块本身，对于多模块项目来说，反应堆就包含了各模块之间的继承和依赖关系，从而能够自动计算出合理的模块构建顺序

举例有聚合模块account-aggregator中的模块配置为：

``` xml
<modules>
    <module>account-email</module>
    <module>account-persist</module>
    <module>account-parent</module>
</modules>
```

构建这个聚合假设得到这么一份log

```log
[INFO] －－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
[INFO] Reactor Build Order:
[INFO]
[INFO] Account Aggregator
[INFO] Account Parent
[INFO] Account Email
[INFO] Account Persist
[INFO]
[INFO] －－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
```

上述输出告诉我们反应堆的构建顺序，以此为account-aggregator, account-parent, account-email, account-persist.可以看出构建的顺序与aggregator配置的模块顺序无强相关性

Maven还需要考虑模块之间的继承和依赖关系，模块的依赖关系会将反应堆构成一个有向非循环图(DAG)。这个图不允许出现循环。

### 裁剪反应堆

参数：

- -am also make，同时构建所列模块的依赖模块
- -amd also makde dependents 同时构建依赖于所列模块的模块
- -pl projects <arg> 构建指定的模块, 模块间用逗号分隔
- -rf resume from <arg> 在完整的反应堆构建顺序基础上指定从哪个模块开始构建

举例：

- mvn clean install -pl account-email,account-persist，得到的反应堆为Account Email, Account Persist
- mvn clean install -pl account-email -am，得到反应堆：Account Parent, Account Email
- mvn clean install -pl account-parent -amd，得到反应堆：Account Parent, Account Email, Account Persist
- mvn clean install -rf account-email，得到反应堆：Account Email, Account Persist
- mvn clean install -pl account-parent -amd -rf account-email，该命令中-pl -amd会裁剪出一个account-parent, account-email, account-persist的反应堆，再次基础上，-rf 指定从account-email 开始，所以会得到反应堆：Account Email, Account Persist

[1]: ExcerptFrom:许晓斌.“Maven实战.”AppleBooks.

