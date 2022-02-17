# maven相关参数

### 跳过测试

* \-DskipTests，不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下
* \-Dmaven.test.skip=true，不执行测试用例，也不编译测试用例类

可以在pom.xml中添加如下配置来跳过测试：

```java
<build>
  <plugins>
<!-- maven 打包时跳过测试 -->
	<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <skip>true</skip>
                </configuration>
        </plugin>
       </plugins>
   <build>         
```

### 执行测试



### 打印依赖



### all in one



### 编码格式与java版本



```xml
<properties>    
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>    
<maven.compiler.source>1.8</maven.compiler.source>    
<maven.compiler.target>1.8</maven.compiler.target>    
<java.version>1.8</java.version>
</properties>
```

###



### 组件







