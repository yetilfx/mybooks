# 20.开发工具

Spring Boot 中提供了一套开发工具，用于改善研发体验。`spring-boot-devtools` 模块可以给所有项目提供辅助的研发时特性。只需要按照如下方式将该模块包含在Maven或者Gradle工程中即可：

**Maven**

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

**Gradle**

```
dependencies {
    compile("org.springframework.boot:spring-boot-devtools")
}
```

> 开发工具在应用运行时将自动失效掉。当使用`java -jar` 或者其他特定的加载器运行应用时，则被认为是“生产应用实例”。建议。在Maven标记为可选依赖或者在Gradle中使用 `compileOnly` 以免污染使用此工程的其他模块。
>
> 默认在打包时不包含“开发工具”。如果你希望使用开发工具的[远程特性](#20-5)，你需要禁用构建属性 `excludeDevtools` ，将其设置为disable。Maven和Gradle均支持该属性。

## 20.1 默认属性

Spring Boot 支持的一些类库中，通过使用缓存来提升性能。例如，[模板引擎](/27spring-web-mvc-framework.md#27-1-10) 缓存编译后的模板，以避免一再重复的解析模板文件。同样，Spring MVC可以添加HTTP缓存头来缓存静态资源。

有时，在生产环境中缓存起到了很好的作用，但是在开发中却适得其反。因为缓存的原因，导致你对应用的调整无法立即呈现出来。因此，`spring-boot-devtools` 默认禁用缓存选项。

缓存选项通常在 `application.properties` 文件中设置。例如，Thymeleaf 提供的 `spring.thymeleaf.cache` 属性。无需手动设置这些属性，`spring-boot-devtools` 模块将自动采用开发时配置。

> 开发工具自动配置属性的完整清单，参见 [DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.0.3.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)。

## 20.2 自动重启

应用通过 `spring-boot-devtools` 模块，可实现当classpath中任何文件产生变化时自动重启引。这是在使用开发工具工作时，非常用的的一个特性，他能快速的反馈代码变化产生的影响。默认情况下，classpath中所有的文件变化都会被监测到。注意，有些资源的变化，诸如静态文件和视图模板等，是不需要重启应用的，参见 [20.2.2 排除资源](#2022-排除资源)。

```
触发重启

更新classpath中的资源，是唯一触发重启的方式。
如何引起 classpath 中文件更新，取决于你是用的开发工具。
在Eclipse中，保存修改的文件就会引起 classpath 中文件更新并触发重启。
在 IntelliJ IDEA中，构建项目（Build -> Build Project）才会导致上述情况。
```

> 你也可以使用Maven和Gradle的Spring Boot插件来启动用应用，注意插件中“fork”参数需要设置为“启用”，原因是DevTools需要一个独立的应用类加载器（不能多个进程共用）才能运行。默认情况下，当Gradle和Maven的Spring Boot插件检测classpath中包含`spring-boot-devtools` 时，会设置为“启用”。
>
> 关于fork的作用，译者考证如下
>
> 参见：[Devtools reload doesn't work with spring-boot-maven-plugin](https://github.com/spring-projects/spring-boot/issues/3315)
>
> 参见：[Enable forking for \`spring-boot:run\` if devtools is present](https://github.com/spring-projects/spring-boot/issues/5137)
>
> 参见：[Spring Boot Maven Plugin - fork 参数](https://docs.spring.io/spring-boot/docs/current/maven-plugin/run-mojo.html#fork)
>
> 自动重启可以与热部署\(LiveReload\)一起良好的运转。详情参见 “[热部署](#203-热部署（livereload）)”。如果使用JRebel，则禁用自动重启，以支持动态类重载。其他的开发工具特性（例如：热部署和默认属性）仍然有效。
>
> DevTools 依赖于应用上下文（Application Context）中的关闭钩子（shutdown hook）实现重启过程中的关闭应用的动作。如果禁用关闭钩子，自动重启将无法正常工作。（`SpringApplication.setRegisterShutdownHook(false)`）
>
> 当确定设置classpath中某一项，当其改变时将触发重启，DevTools 会自动忽略名称为`spring-boot`,`spring-boot-devtools`,`spring-boot-autoconfigure`,`spring-boot-actuator`, `spring-boot-starter`的项目。
>
> DevTools 需要通过 `ApplicationContext` 来定制 `ResourceLoader` 。如果你的应用中提供了一个，那么它将被封装起来。但直接重载 `ApplicationContext` 中的 `getResource` 方法是不被支持的。

### 20.2.1 记录状态变化

默认情况下，每次应用重启，都会给出**增量**的状态状态变化报告。这个报告反应了，当你添加或移除 bean或修改配置属性等这些变更，导致的应用自动配置的变更。

可通过如下设置，关闭这一报告：

```
spring.devtools.restart.log-condition-evaluation-delta=false
```

### 20.2.2 排除资源

### 20.2.3 监视其他路径

### 20.2.4 禁止重启

### 20.2.5 使用触发文件

### 20.2.6 定制重启类加载器（Classloader）

### 20.2.7 已知局限

## 20.3 热部署（LiveReload）

## 20.5 远程应用



