# 一.Gradle

## 1.1 使用

简单的介绍一下gradle:相信大家都使用过Maven这款项目构建工具， gradle也是一款类似的项目构建工具，同时使用与安卓端和后端，适配java和Kotlin两种语言。
### 1.1.1  Gradle Vs Apache Maven
区别：
 - 灵活性
	- 谷歌选择Gradle作为Android的官方构建工具;不是因为构建脚本是代码，而是因为 Gradle 的建模方式能够以最基本的方式进行扩展。Gradle 的模型还允许它用于 C/C++ 的原生开发，并且可以扩展到涵盖任何生态系统。例如，Gradle 在设计时考虑了使用其 Tooling API进行嵌入。
	
	- Gradle 和 Maven 都提供配置上的约定。但是，Maven提供了一个非常严格的模型，这使得自定义变得乏味，有时甚至是不可能的。虽然这可以让你更容易理解任何给定的Maven版本，但只要你没有任何特殊要求，它也使它不适合许多自动化问题。另一方面，Gradle 在构建时考虑了授权和负责任的用户。
```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.18</version>
</dependency>

<build>  
    <plugins>  
        <plugin>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-maven-plugin</artifactId>  
            <configuration>                <excludes>  
                    <exclude>  
                        <groupId>org.projectlombok</groupId>  
                        <artifactId>lombok</artifactId>  
                    </exclude>  
                </excludes>  
            </configuration>  
        </plugin>  
    </plugins>  
</build>
```

```kotlin
dependencies {
// https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter
	implementation group: 'com.alibaba', name: 'druid-spring-boot-starter', version: '1.2.18'
}

plugins {  
    id "java"  
    id "maven"  
    id "idea"  
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"  
}

```
 - 性能
	最大的区别是 Gradle 的工作回避和增量机制。使 Gradle 比 Maven 快得多的前 3 个功能是：
	- 增量 — Gradle 通过跟踪任务的输入和输出并仅运行必要的内容以及仅处理在可能的情况下更改的文件来避免工作。
	
	- 构建缓存 — 重复使用任何其他 Gradle 构建的构建输出，这些输出具有相同的输入，包括在计算机之间。
	
	- Gradle Daemon — 一个长期的进程，可将构建信息保持在内存中的“热”。
 - 用户体验：
	Gradle 提供了一个基于 Web 的交互式 UI，用于调试和优化构建：构建扫描。这些也可以托管在本地，以允许组织收集构建历史记录并进行趋势分析，比较构建以进行调试或优化构建时间。
 - 依赖关系管理
	 - Maven 允许覆盖依赖项，但只能按版本覆盖。Gradle 提供了可自定义的依赖项选择和替换规则，这些规则可以声明一次，并在项目范围内处理不需要的依赖项。这种替换机制使 Gradle 能够一起构建多个源项目以创建复合构建。
	 
	- Maven 的内置依赖项范围很少，这迫使在常见场景中（如使用测试夹具或代码生成）使用笨拙的模块体系结构。例如，单元测试和集成测试之间没有分离。Gradle 允许自定义依赖项范围，从而提供建模效果更好、构建速度更快。
	
	- Maven 依赖项冲突解决方案使用最短路径，该路径受声明排序的影响。Gradle 会执行完整的冲突解决，选择图表中找到的依赖项的最高版本。此外，使用 Gradle，您可以将版本声明为_严格_版本，这允许它们优先于可传递版本，从而允许降级依赖项。
	
	- 作为库生产者，Gradle 允许生产者声明“api”和“实现”依赖项，以防止不需要的库泄漏到消费者的类路径中。Maven 允许发布者通过可选的依赖项提供元数据，但仅作为文档提供。Gradle 完全支持功能变体和可选依赖项。
### 1.1.2 创建项目

不同于正常的Maven项目，需要选择Gradle-groovy仓库管理工具。尽量保持idea，java，jdk，springboot版本较新。其余初始化创建项目方式同springboot+mybatis_plus+Maven的项目格式
### 1.1.3 项目结构
架构：
```Groovy
├── gradle 
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew 
├── gradlew.bat 
├── settings.gradle 
└── lib
    ├── build.gradle 
    └── src
        ├── main
        │   └── java 
        │       └── demo
        │           └── Library.java
        └── test
            └── java 
                └── demo
                    └── LibraryTest.java
```
![[Pasted image 20230904202702.png]]
![[Pasted image 20230904203021.png]]
其中的核心是这个**build.gradle**，这一块是类似maven中**pom.xml**
基础项目架构和Maven框架几乎一样，补充一下
- **gradlew**：gradle命令行工具,gradlew是gradle wrapper的缩写，也就是说它对gradle的命令进行了包装
	- gradlew为Linux下的shell脚本
	-  gradlew.bat是**Windows下的批处理文件**。 
- **settings.gradle**: 多模块项目配置文件
## 1.2 配置
插件注入：在插件框内可以添加相应的插件，较Maven来说，更加简洁
```Groovy
plugins {  
    id 'java'  
    id 'org.springframework.boot' version '3.1.2'  
    id 'io.spring.dependency-management' version '1.1.2'  
}  
```
- `plugins` 块指定了在构建过程中需要使用的插件。其中：
    - `id 'java'` 表示使用Java插件，用于编译和构建Java代码。
    - `id 'org.springframework.boot' version '3.1.2'` 表示使用Spring Boot插件，版本号为3.1.2，用于构建Spring Boot应用。
    - `id 'io.spring.dependency-management' version '1.1.2'` 表示使用Spring依赖管理插件，版本号为1.1.2，用于简化和管理项目的依赖关系。
```Groovy
group = 'com.example'  
version = '0.0.1-SNAPSHOT'  
```
- `group` 和 `version` 分别指定了项目的组名和版本号。
```Groovy
java {  
    sourceCompatibility = '17'  
}  
  ```
  - `java` 块用于配置Java编译相关的属性。`sourceCompatibility` 属性指定了源代码的兼容性版本，这里设置为17，表示使用Java 17的语法。
```Groovy
configurations {  
    compileOnly {  
        extendsFrom annotationProcessor  
    }  
}  
  ```
  - `configurations` 块用于定义配置项，这里定义了一个名为 `compileOnly` 的配置项，并指定它应该继承自 `annotationProcessor`。
    
```Groovy
allprojects {  
    repositories {  
        maven{ url 'https://maven.aliyun.com/nexus/content/groups/public/'}  
    }  
}  
```
- `allprojects` 块用于配置全局的构建设置，这里定义了一个仓库地址 `https://maven.aliyun.com/nexus/content/groups/public/`，用于获取项目的依赖。
```Groovy
dependencies {  
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'  
    implementation 'org.springframework.boot:spring-boot-starter-web'  
    compileOnly 'org.projectlombok:lombok'  
    runtimeOnly 'com.mysql:mysql-connector-j'  
    annotationProcessor 'org.projectlombok:lombok'  
    testImplementation 'org.springframework.boot:spring-boot-starter-test'  
    implementation group: 'com.alibaba', name: 'druid-spring-boot-starter', version: '1.2.18'  
  
}  
  ```
- `dependencies` 块用于定义项目的依赖关系。其中：
    - `implementation` 表示这些依赖将会被编译和打包到最终的构建结果中。
    - `compileOnly` 表示这些依赖仅在编译时被使用，不会被打包到最终的构建结果中。
    - `runtimeOnly` 表示这些依赖仅在运行时被使用，不会被打包到最终的构建结果中。
    - `annotationProcessor` 表示这些依赖是注解处理器，用于在编译时生成代码。
    - `testImplementation` 表示这些依赖仅在测试代码中被使用。
```Groovy
tasks.named('test') {  
    useJUnitPlatform()  
}
```
- `tasks.named('test')` 块用于配置名称为 `test` 的任务的行为，这里使用JUnit Platform来运行测试。
# 二.Jpa

## 2.1 CRUD方法

## 2.2 多表查询

关联关系同Hibernate的查询方式
## 2.3 Querydsl实现动态查询
```java
@Data  
public class ***DTO {  
  
    private Long tagId;  
 
    private String tag;  

    private Integer totalNum;  

}
```
生成我们的 Q实体类在项目中**build**之后会生成一个Q类
```Java

private final JPAQueryFactory jpaQueryFactory;

BooleanBuilder booleanBuilder = new BooleanBuilder();
        // 要查询的条件
        if(!StringUtils.isEmpty(loc.getLoc())){
            // 放入要查询的条件信息
            booleanBuilder.and(qLoc.loc.contains(loc.getLoc()));
        }
        //连接查询条件（Loc.id = User.id ）
        booleanBuilder.and(qLoc.id.eq(qUser.id));


List<***DTO> ***DtoList = jpaQueryFactory.select(Projections.bean(***DTO.class,                 qLabel.id.as("tagId"),  
	    qLabel.name.as("tag"),  
	    q***.totalNum,))  
    .from(q***)  
    .leftJoin(qLabel).on(q***.tagId.eq(qLabel.id)) // join Label  
    .where(booleanBuilder) 
    .fetch();
```


