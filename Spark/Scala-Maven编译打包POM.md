```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>spark2-hbase2-demo</artifactId>
    <version>1.0</version>

    <properties>
        <spark.version>2.4.5</spark.version>
        <scala.prefix>2.11</scala.prefix>
        <scala.version>2.11.12</scala.version>
        <hbase.version>2.0.2</hbase.version>
    </properties>

    <profiles>
    <!-- dev -->
    <profile>
        <id>dev</id>
        <properties>
            <jar.scope>compile</jar.scope>
        </properties>
        <activation>
            <!-- 设置默认激活这个配置 -->
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <!-- prod -->
    <profile>
        <id>prod</id>
        <properties>
            <jar.scope>provided</jar.scope>
        </properties>
    </profile>
    </profiles>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_${scala.prefix}</artifactId>
            <version>${spark.version}</version>
            <scope>${jar.scope}</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>${hbase.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>*</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-mapreduce</artifactId>
            <version>${hbase.version}</version>
        </dependency>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>

    <build>

        <finalName>spark2-hbase2-bulkload</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!--支持scala打包-->
            <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
                <version>2.15.1</version>
                <executions>
                    <execution>
                        <id>scala-compile-first</id>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <configuration>
                            <includes>
                                <include>**/*.scala</include>
                            </includes>
                        </configuration>
                    </execution>
                    <execution>
                        <id>scala-test-compile</id>
                        <goals>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <scalaVersion>${scala.version}</scalaVersion>
                    <args>
                        <arg>-target:jvm-1.8</arg>
                    </args>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

