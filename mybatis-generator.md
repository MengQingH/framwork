mybatis-generator(MBG)，mybatis中的一个插件，使用该插件可以很方便的生成实体类、Mapper接口以及对应的xml文件、以及example文件。

插件代码在tk.mybatis.mapper.generator包下，一共有如下两个类：
* MapperCommentGenerator：该类用于生成数据库备注字段的注释，以及实体类字段的注解
* MapperPlugin：插件的实现类，该类默认使用上面这两个注释生成器，插件屏蔽了一般的CRUD方法（保留了Example），插件可以生成实体的@Table注释。

# 使用方法：
1. 在pom.xml文件中添加generator插件：
```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.2</version>
    <!-- 声明配置文件的位置 -->
    <configuration>
        <configurationFile>src/main/resources/mybatis-generator/generatorConfig.xml</configurationFile>
        <verbose>true</verbose>
        <overwrite>true</overwrite>
    </configuration>
    <executions>
        <execution>
            <id>Generate MyBatis Artifacts</id>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.2</version>
        </dependency>
    </dependencies>
</plugin>
```
2. 声明generator的配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--数据库驱动,在generator工作的时候，需要额外加载的依赖包 location属性指明加载jar包的全路径,具体版本根据自己仓库选择-->
    <classPathEntry
            location="C:\Users\\Administrator\.m2\repository\mysql\mysql-connector-java\5.1.47\mysql-connector-java-5.1.47.jar"/>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库链接地址账号密码-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://172.20.176.213:3306/autotax"
                        userId="test"
                        password="kdzwytest">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!--生成Model类存放位置，如果targetPackage不存在，会自动建立-->
        <javaModelGenerator targetPackage="com.zwy.tdc.entity" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!--生成XML映射文件存放位置-->
        <sqlMapGenerator targetPackage="mappers" targetProject="src/main/resources">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <!--生成Dao类存放位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.zwy.tdc.dao" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>

        
        <!--对应的数据库表及model中生成的类名,默认启用Example-->
        <!-- 只有配置了table标签才能生成entity、xml、接口、example文件，可以用通配符%代替所有表 -->
        <table tableName="t_tdc_task"
               domainObjectName="Task"
               enableSelectByPrimaryKey="false"
               enableUpdateByPrimaryKey="false"
               enableDeleteByPrimaryKey="false">
               <!-- 使用数据库表字段名作为实体字段名(不使用下划线转驼峰)-->
            <property name="useActualColumnNames" value="true"/>
            <!-- 配置字段类型的映射 -->
            <columnOverride column="extraData" javaType="java.lang.String" jdbcType="VARCHAR"/>
        </table>
    </context>
</generatorConfiguration>
```
3. 点击generator插件生成代码。

