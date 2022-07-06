# 什么是Flyway？

我们先看一下项目的演进。

![项目演进过程](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061541016.png)

项目的不断迭代中，可能会有不同的版本，我们可以根据不同环境部署不同版本的应用，这没什么问题，但是数据库呢？在不同的项目版本中数据库的表结构等信息可能有变化的。对于本次数据库修改，可能上一版本的项目就无法使用。而之后的项目中也可能删除了某个表。总之都会出现问题。



那么能够根据不同的项目版本，维护对应的数据库版本呢？Flayway就是做这个的，他通过对数据库进行版本管理来实现。

# Flyway如何工作

最简单的就是Flyway指向了一个空的数据库。

![img](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061548585.png)

Flyway会创建一个叫做*flyway_schema_history*的表，该表用于跟踪数据库的状态。

![img](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061548885.png)



然后Flyway会扫描数据库文件路径，根据版本号进行排序。

![数据库迭代图](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061519624.png)

然后按照顺序执行脚本

![img](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061551312.png)

# SQL脚本文件规则

官网有详细解释：https://flywaydb.org/documentation/concepts/migrations#naming

下图说明了版本化迁移，撤销迁移，重复迁移的格式。

![SQL脚本文件规则](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061724988.png)

- **前缀**：`V`表示版本化（可配置），`U`表示撤销（可配置），`R`表示可重复迁移（可配置）
- **版本**：带有点或下划线的版本，随需要分开任意数量的部分（不适用于可重复迁移）
- **分隔符**：`__`（两个下划线）（可配置）
- **描述**：下划线或空格将单词分开
- **后缀**：`.sql`（可配置）

# 实战

Spring Boot： 2.7.1

JDK：17

![image-20220706155626689](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061556746.png)

pom配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.7.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.itlab1024</groupId>
	<artifactId>flyway-tutorial</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>flyway-tutorial</name>
	<description>flyway-tutorial</description>
	<properties>
		<java.version>17</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.flywaydb</groupId>
			<artifactId>flyway-core</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

application.yaml配置

```yaml
spring:
  flyway:
    # 启用或禁用 flyway
    enabled: true
    # flyway 的 clean 命令会删除指定 schema 下的所有 table, 生产务必禁掉。这个默认值是 false 理论上作为默认配置是不科学的。
    clean-disabled: true
    # SQL 脚本的目录,多个路径使用逗号分隔 默认值 classpath:db/migration
    locations: classpath:db/migration
    #  metadata 版本控制信息表 默认 flyway_schema_history
    table: flyway_schema_history
    # 如果没有 flyway_schema_history 这个 metadata 表， 在执行 flyway migrate 命令之前, 必须先执行 flyway baseline 命令
    # 设置为 true 后 flyway 将在需要 baseline 的时候, 自动执行一次 baseline。
    baseline-on-migrate: true
    # 指定 baseline 的版本号,默认值为 1, 低于该版本号的 SQL 文件, migrate 时会被忽略
    baseline-version: 1
    # 字符编码 默认 UTF-8
    encoding: UTF-8
    # 是否允许不按顺序迁移 开发建议 true  生产建议 false
    out-of-order: false
    # 需要 flyway 管控的 schema list,这里我们配置为flyway  缺省的话, 使用spring.datasource.url 配置的那个 schema,
    # 可以指定多个schema, 但仅会在第一个schema下建立 metadata 表, 也仅在第一个schema应用migration sql 脚本.
    # 但flyway Clean 命令会依次在这些schema下都执行一遍. 所以 确保生产 spring.flyway.clean-disabled 为 true
    schemas: spring-flyway
    # 执行迁移时是否自动调用验证   当你的 版本不符合逻辑 比如 你先执行了 DML 而没有 对应的DDL 会抛出异常
    validate-on-migrate: true
  datasource:
    url: jdbc:mysql://localhost:3306
    password: qwe!@#123
    username: root
```

1. 创建数据库（数据库需要预先创建），db/migration此时并未有任何sql文件。

   控制台打印结果如下

   ![image-20220706164558926](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061645107.png)

   可以看到创建了`flyway_schema_history`表，表数据是空的。

2. 创建一个sql文件，并重启。

   文件名是：V1__init.sql

   内容是

   ```sql
   create table itlab
   (
       id int auto_increment,
       name varchar(30),
       constraint itlab_pk
           primary key (id)
   );
   ```

   启动后，脚本会被执行，itlab表会被创建，看下flyway_schema_history表记录信息如下：

   | installed\_rank | version | description | type | script         | checksum    | installed\_by | installed\_on       | execution\_time | success |
   | :-------------- | :------ | :---------- | :--- | :------------- | :---------- | :------------ | :------------------ | :-------------- | :------ |
   | 1               | 1       | init        | SQL  | V1\_\_init.sql | -2138043076 | root          | 2022-07-06 17:19:05 | 81              | 1       |

   可以看到version=1,description=init，请对照文件名体会。

3. 撤销

   社区版不支持此功能。

4. 重复迁移

   增加文件R__ADD.sql。

   ```sql
   insert into itlab(name) values ("张三")
   ```

   历史表增加记录。

   | installed\_rank | version | description | type | script       | checksum   | installed\_by | installed\_on       | execution\_time | success |
   | :-------------- | :------ | :---------- | :--- | :----------- | :--------- | :------------ | :------------------ | :-------------- | :------ |
   | 2               | NULL    | ADD         | SQL  | R\_\_ADD.sql | 2087458821 | root          | 2022-07-06 17:52:20 | 76              | 1       |

   itlab表增加了一条记录。

   多次运行会一直增加记录。

---

还有很多细节...慢慢完善！！！



> 个人网站：https://itlab1024.com
>
> 知乎：https://www.zhihu.com/people/xpp1109
>
> Github：https://github.com/itlab1024/flyway-tutorial
>
> 微信公众号：![微信公众号](https://raw.githubusercontent.com/itlab1024/picgo-images/main/202207061803337.png)
