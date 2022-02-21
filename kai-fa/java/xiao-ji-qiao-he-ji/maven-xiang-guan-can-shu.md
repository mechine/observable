# maven相关参数

### 编码格式与java版本



```xml
<properties>    
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>    
<maven.compiler.source>1.8</maven.compiler.source>    
<maven.compiler.target>1.8</maven.compiler.target>    
<java.version>1.8</java.version>
</properties>
```



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



### 冲突解决

> mvn dependency:tree

剔除

```
<exclusions>
    <exclusion>
    <groupId>org.eclipse.jdt</groupId>
    <artifactId>core</artifactId>
    </exclusion>
</exclusions>
```

shadow

```
<relocations>
    <relocation>
    <pattern>cn.tass</pattern>
    <shadedPattern>com.sf.bdp</shadedPattern>
    </relocation>
</relocations>
```

### all in one

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>${shade.version}</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```



```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>${assembly.version}</version>
    <inherited>false</inherited>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <descriptors>
            <descriptor>src/main/assembly/bin.xml</descriptor>
        </descriptors>
    </configuration>
</plugin>
```



### 组件

tomcat

```
```

jetty

```
```



