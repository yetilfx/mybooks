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

开发工具自动配置属性的完整清单，参见 DevToolsPropertyDefaultsPostProcessor。

## 20.5 远程应用



