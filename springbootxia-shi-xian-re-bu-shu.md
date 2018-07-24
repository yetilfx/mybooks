# QuickStart

###### _for MAC & IntelliJ IDEA & Maven & Chrome_

环境：

macOS High Sierra 10.13.6

IntelliJ IDEA 2018.1.5（Ultimate Edition）

Chrome 版本 67.0.3396.99（正式版本） （64 位）

Maven 3.3.9

1.在pom.xml中添加spring-boot-devtools依赖

```
<dependencies>    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

2.在pom.xml中添加spring-boot-maven-plugin依赖

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

3.开启IDEA的自动构建

在 Intellij IDEA 当中的`Preference`-&gt;`Build, Execution, Deployment`-&gt;`Compiler`中勾选`Build project automatically` \(HotKey : command + ,\)![](/assets/autobuild.png)4.开启IDEA运行时自动构建
