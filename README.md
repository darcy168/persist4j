# Use by Jitpack

[![JitPack](https://jitpack.io/v/devezhao/persist4j.svg)](https://jitpack.io/#devezhao/persist4j)

## Maven

```
<repositories>
  <repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>com.github.devezhao</groupId>
    <artifactId>persist4j</artifactId>
    <version>LAST_VERSION</version>
  </dependency>
</dependencies>
```

# 如何使用

## 基础对象

使用 [persist4j](https://github.com/devezhao/persist4j) 首先需要了解两个基础对象 `Record` 和 `Query`。`Record` 是一个 Map 实现，他承担了 DAO 职责，而 `Query` 是查询入口，是日常编码中使用最为频繁的类。

`Record` 包含了诸多 setter 和 getter 方法，在实现动态化 DAO 职责的同时，还具备对不同数据（字段）类型进行校验的能力。以下是一个基本的使用示例。

```
// 获取数据管理对象
PersistManagerFactory PMF = ...
PersistManager PM = PMF.createPersistManager();

Record record = new StandardRecord()
record.setString("FieldName", "FieldValue");

PM.create(record);
```


`Query` 的使用主要是对 persist4j 独有的 AJQL （Auto Join Query Language）语句进行理解。AJQL 是 Like-SQL 语法，因此学习/理解起来并不困难。

```
// 获取数据管理对象
PersistManagerFactory PMF = ...
PersistManager PM = PMF.createPersistManager();

String ajql = "select Field1, ReferenceField1.Field2 from Table1 where Field2 = ?";
Record found = PM.createQuery(ajql).setParameter(1, "SomeValue").unique();
```

请特别注意上例中 select 子句中的 `ReferenceField1.Field2` 字段，`ReferenceField1` 是一个引用型字段，因此可以使用 `.` 进行（自动）关联查询（其中 `Field2` 是关联表中的字段）。如果 `Field2` 也是一个引用字段，那么可以继续进行关联查询。 

可以看出，通过 AJQL 可以大大简化我们日常查询 SQL 中的表关联操作，这也正是 AJQL 名称的由来。


## 元数据

动态元数据是支撑 [persist4j](https://github.com/devezhao/persist4j) 的核心，使用 XML 配置，也可以将其存储在数据库中动态加载。无论何种方式没有本质区别，仅在与来源不同。

```
<entity name="ProjectConfig" type-code="050" description="项目配置" queryable="false">
  <field name="configId" type="primary" />
  <field name="projectName" type="string" max-length="100" nullable="false" description="项目名称" />
  <field name="projectCode" type="string" max-length="10" nullable="false" updatable="false" description="项目代号" />
  <field name="comments" type="string" max-length="300" description="备注" />
  <field name="principal" type="reference" ref-entity="User" description="负责人" />
  <field name="members" type="string" max-length="420" default-value="ALL" description="项目成员(可选值: ALL/$MemberID)" />
  <field name="showConfig" type="text" max-length="3000" description="扩展配置(JSON Map)" />
</entity>
```

以上示例片段来自 [rebuild](https://github.com/getrebuild/rebuild/) 项目，他定义了一个 `ProjectConfig` 实体及其所拥有的字段。关于各配置项的说明，请参考 [metadata.dtd](https://github.com/devezhao/persist4j/blob/master/src/main/resources/metadata.dtd) 。

元数据的配置最终会被映射为 `Entity` 和 `Field` 对象，可以对元数据进行操作。


## 在真实环境中使用

我们建议你将 [persist4j](https://github.com/devezhao/persist4j) 与 Spring 配合使用，可以大大简化编码。当然，这不是必须的。通过参考 [rebuild](https://github.com/getrebuild/rebuild/blob/master/src/main/resources/application-ctx.xml#L47) 项目你可以进行更多的了解。