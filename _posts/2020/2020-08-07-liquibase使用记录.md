---
title: springboot整合liquibase（强烈推荐）
author:
  name: leoz
  link: https://github.com/Arno-Code
date: 2020-08-07 12:10:00 +0800
categories: [springboot, liquibase]
tags: [liquibase, 数据库版本管理]
render_with_liquid: false
---

Liquibase是一个开源数据库模式更改管理解决方案，它使您能够轻松管理数据库更改的修订。Liquibase使参与应用程序发布过程的任何人都可以轻松地：

* 消除发布数据库时的错误和延迟。
* 部署和回滚特定版本的更改，而无需知道已部署的内容。
* 将数据库和应用程序更改一起部署，以便它们始终保持同步。

ps:本人最直观的感觉就是在部署不同环境的时候不用再手动执行初始化sql脚本;

## springboot集成
1. 引入liquibase依赖（以Maven为例）
```xml
        <dependency>
            <groupId>org.liquibase</groupId>
            <artifactId>liquibase-core</artifactId>
            <version>4.3.5</version>
        </dependency>
```
2. 配置liquibase 在application.yml加入以下内容：
```yaml
spring:
  liquibase:
    enabled: true
    change-log: "classpath:/db/master.xml"
    contexts: ${spring.profiles.active}
    url: ${spring.datasource.url}
    user: ${spring.datasource.username}
    password: ${spring.datasource.password}
```
`contexts`：liqubase启动的环境，这里设置为与spring启动环境保持一致
`change-log`：加载主（入口）变更文件位置，这里是`resources/db/master.xml`
3. 配置主变更文件`master.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
	  http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">
  <!-- 引入xml格式的变更文件 -->
    <include file="classpath:/liquibase/xml/01-create-test-schema.xml" relativeToChangelogFile="false"></include>
<!--    <include file="classpath:/liquibase/xml/02-insert-test-data.xml" relativeToChangelogFile="false"></include>-->
  <!-- 引入sql格式的变更文件 -->
    <include file="liquibase/sql/test.sql"/>
<!--    <include file="liquibase/sql/add2.sql"/>-->
  <!-- includeAll：表示sql文件夹下面的所有文件都被liquibase管理-->
<!--    <includeAll path="/sql/" relativeToChangelogFile="true"/>-->

</databaseChangeLog>
```
4. 编辑子变更文件（实际sql变更内容）,子变更文件格式支持xml与sql

编写`01-create-test-schema.xml` （xml格式：优点屏蔽不同数据库sql语言差异；缺点是需要一定的学习成本）
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <changeSet  id="01"  author="classmaster">
        <createTable  tableName="classmaster">
            <column  name="id"  type="int">
                <constraints  primaryKey="true"  nullable="false"/>
            </column>
            <column  name="name"  type="varchar(50)">
                <constraints  nullable="false"/>
            </column>
            <column  name="active"  type="boolean"
                     defaultValueBoolean="true"/>
        </createTable>
    </changeSet>
</databaseChangeLog>
```
xml语法这里不做介绍，详细可以阅读官方文档

编写`test.sql` （sql格式：优点简单处理即可支持liquibase）
```sql
--liquibase formatted sql
--changeset leoz:1
ALTER TABLE DATABASECHANGELOG ADD PRIMARY KEY (ID);

CREATE TABLE IF NOT EXISTS `test`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `name` VARCHAR(100) NOT NULL,
   `author` VARCHAR(40) NOT NULL,
   `date` DATE,
   PRIMARY KEY ( `id` )
);
--changeset leoz:2
INSERT INTO `test` (`id`, `name`, `author`, `date`) VALUES ('2', 'test', 'admin', '2021-10-14');
INSERT INTO `test` (`id`, `name`, `author`, `date`) VALUES ('3', 'test', 'admin', '2021-10-14');
INSERT INTO `test` (`id`, `name`, `author`, `date`) VALUES ('4', 'test', 'admin', '2021-10-14');
```
这里springboot整合liquibase完成，启动项目看看效果把；项目启动后liquibase会在数据库中添加一个日志表和一个锁表用于版本控制；

**注意：**
1. sql文件第一行必须是`--liquibase formatted sql`
2. `--changeset leoz:1`    `--changeset`代表一次变更 leoz:1  指（作者：版本）
3. 一个sql文件可以写多个变更，一个变更内容一旦执行不可修改；（一次 `--changeset` 中的内容代表一个变更）test.sql中就有两次变更；

**常见错误 **
1. 初始化的时候 如果日志报错`DATABASECHANGELOG`没有主键错误则应该把这条`ALTER TABLE DATABASECHANGELOG ADD PRIMARY KEY (ID);`sql语句加上，否则不用加
2. 如果自定义配置了数据源有可能出现其他对象访问数据库在liquibase初始化之前，导致出现xxx表不存在错误；这种情况的解决方案可以在依赖数据库的对象上面添加
   `@DependsOn("liquibase")`注解;例如在对ruoyi框架改造时出现了这种问题，解决代码如下
```java
@DependsOn("liquibase")
@Service
public class SysConfigServiceImpl implements ISysConfigService{}
```
本文简单介绍了liquibase的基本使用方式，如需了解更多知识请移步[liquibase官方文档](https://docs.liquibase.com/concepts/home.html)
