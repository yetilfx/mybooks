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



