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

```
重启与重载
Spring Boot 提供的重启采用双类加载器机制。
未改变的类被加载进基础类加载器（例如，来自与第三方的jar）。
开发中的类被加载进重启类加载器。当应用重启时，重启类加载器将被抛弃并创建一个新的类加载器。
这意味着此时应用重启的速度通常要比“冷重启”快很多，因为基础类加载器已经无需再加载。
```

### 20.2.1 记录状态变化

默认情况下，每次应用重启，都会给出**增量**的状态状态变化报告。这个报告反应了，当你添加或移除 bean或修改配置属性等这些变更，导致的应用自动配置的变更。

可通过如下设置，关闭这一报告：

```
spring.devtools.restart.log-condition-evaluation-delta=false
```

### 20.2.2 排除资源

有些资源改变时，不需要触发应用重启。例如：Thymeleaf模板。默认情况下 `/META-INF/maven` ，`/META-INF/resources` ，`/resources` ，`/static` ， `/public` ，或`/templates` 中资源变化不会触发重启，但是会触发[热部署](#203-热部署（livereload）)。如果你需要自定义排除的资源，你可以通过设置 `spring.devtools.restart.exclude` 实现。例如，仅排除 `/static` 和 `/public` 你需要进行如下设置：

```
spring.devtools.restart.exclude=static/**,public/**
```

> 如果你希望保留默认的配置的基础上再额外添加一些排除的资源，则需要配置 `spring.devtools.restart.additional-exclude` 属性，来取代`spring.devtools.restart.exclude`属性。

### 20.2.3 监视附加路径

当你修改某些不在classpath中的文件时，可能也需要重启应用或重新加载资源。配置 `spring.devtools.restart.additional-paths` 属性可以监控classpath外的其他路径。你也可以使用 `spring.devtools.restart.exclude` 来控制附加路径下的变化，是否触发重启或者热部署。

### 20.2.4 禁止重启

### 20.2.5 使用触发文件

### 20.2.6 定制重启类加载器（Classloader）

### 20.2.7 已知局限

## 20.3 热部署（LiveReload）

`spring-boot-devtools` 模块包含一个内嵌的热部署（LiveReload）服务，它可以在资源变化时自动触发刷新浏览器。在Chrome，Firefox和Safari浏览器上均有免费的LiveReload浏览器扩展。参见，[livereload.com](https://livereload.com/extensions/)。

如果你不希望使用热部署服务来启动并运行你的应用，你可以设置 `spring.devtools.livereload.enabled` 为 `false` 。

> 在同一时间，你仅能运行一个热部署服务。再启动你的应用前，请确认没有其他的热部署服务正在运行。如果从你的IDE启动了多个应用，则仅有第一个支持热部署。

## 20.4 全局配置

## 20.5 远程应用

### 20.5.1 运行远程客户端应用

远程客户端应用程序被设计为从IDE中运行。需要运行 `org.springframework.boot.devtools.RemoteSpringApplication` 并与你连接的远程项目在同一classpath中。这个应用的唯一参数就是他连接的远程URL。

例如，如果你使用Eclipse或者STS开发一个名为 my-app 的项目，并部署在Cloud Foundry上，你需要做下列几件事：

* 从 `Run` 菜单中选择 `Run Configurations` 
* 创建一个新 `Java Application` "运行配置"
* 浏览`my-app` 项目
* 使用 org.springframework.boot.devtools.RemoteSpringApplication做为 main class
* 添加 `https://myapp.cfapps.io (你的远程URL)`到程序参数\(`Program arguments`\)

远程客户端运行时会有类似如下的输出：

      .   ____          _                                              __ _ _
     /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
    ( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
     \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
      '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
     =========|_|==============|___/===================================/_/_/_/
     :: Spring Boot Remote :: 2.0.3.RELEASE

    2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
    2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
    2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
    2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
    2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)

### 20.5.2 远程更新

同本地重启方案一样，远程客户端监测你应用classpath中的变化。任何资源的更新会被推到远程的应用并按需触发重启。

