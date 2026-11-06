# MyBatis：一级/二级缓存与 #{} / ${}

> 范围：MyBatis 3、MyBatis-Spring。先分清 SqlSession、Spring 事务、Mapper namespace；以下片段是机制示意，不是完整可启动项目。

## 1. 概念：两级缓存不是“请求缓存 + 全局 Redis”

| 维度 | 一级缓存（Local Cache） | 二级缓存（Second Level Cache） |
|---|---|---|
| 范围 | 同一个 SqlSession | 同一 namespace 或明确共享的 Cache |
| 开启 | 默认 SESSION 范围 | 全局允许缓存之外，还需 Mapper 显式配置，例如 `<cache/>` |
| 内容 | 查询结果及对象引用 | 映射语句查询结果；默认实现不等于分布式缓存 |
| 生命周期 | 清理、提交/回滚、关闭会话等会影响缓存 | 受事务提交、失效、淘汰和 Mapper 配置影响 |
| 核心风险 | 同会话结果对象被修改、长会话读旧值 | 跨 namespace / 外部写入无法自动完整失效 |

为什么有缓存？减少重复查询、支持嵌套结果映射等。但**一致性边界**比命中率更重要，订单/库存场景不应因为“框架有缓存”就默认启用所有缓存。

## 2. 一级缓存：什么时候重复查询可能不发 SQL？

同一 SqlSession 中，映射语句、最终 SQL、参数、分页范围等缓存键一致，且未被清理时，第二次查询可能命中本地缓存。

```java
try (SqlSession session = factory.openSession()) {
    UserMapper mapper = session.getMapper(UserMapper.class);
    User a = mapper.findById(1L);
    User b = mapper.findById(1L);  // 可能直接来自本会话缓存
    session.clearCache();
    User c = mapper.findById(1L);  // 清理后需重新获取
}
```

同一个会话可能拿到相同对象引用：给 a 改字段，b 观察到的对象也可能变化；这不代表数据库已 UPDATE。不要把 Mapper 返回实体当无限期共享的可变全局对象。

DML、清理、提交/回滚、关闭等会使本地缓存失效或结束。`localCacheScope=STATEMENT` 限制跨语句复用，但不是禁用所有内部缓存机制。

### Spring 下“同一次 HTTP 请求”不等于“同一个 SqlSession”

- `SqlSessionTemplate` 可作为线程安全的共享组件；底层实际会话由 Spring 事务上下文等管理。
- 同一正确配置的 Spring 事务内，通常绑定并复用相应会话；没有事务时，连续两次 Mapper 调用可能使用不同会话。
- 原生 `DefaultSqlSession` **不线程安全**，不能放单例字段供线程共享。
- 同一个事务仍不代表无限期实时读取：本地缓存与数据库隔离级别都影响可见性，`clearCache()` 也不能突破 RR 的快照语义。

## 3. 二级缓存：开关、提交与失效

```xml
<!-- Mapper namespace 内显式选择开启；生产前先证明失效策略正确 -->
<cache eviction="LRU" size="512" readOnly="false"/>

<select id="findById" resultType="com.example.User" useCache="true">
  SELECT id, name FROM users WHERE id = #{id}
</select>
```

- `cacheEnabled=true` 只是允许配置的二级缓存生效，不是每个 namespace 自动拥有缓存。
- 二级缓存有事务性管理：结果通常在会话提交/正常完成相应事务边界后才对别的会话可见；回滚不应把未提交结果发布出去。不要在另一会话尚未完成事务时断言“缓存失效了”。
- 默认 SELECT 一般 `useCache=true`、`flushCache=false`，DML 默认 `flushCache=true`；修改配置前理解影响。
- `useCache=false` 针对二级缓存，不等于关闭一级缓存。确需新查询时还需考虑本地缓存策略与事务隔离。
- 默认 `readOnly=false` 的内置缓存通过序列化等方式返回拷贝，结果对象图需满足相应序列化要求；`readOnly=true` 可共享实例，只适合真正不可变使用。
- 容量、淘汰、刷新时间只能限制陈旧范围，不能代替业务正确的失效通知。

### 最易漏的跨 Mapper 问题

UserMapper 缓存了一条 JOIN users + departments 的查询；DepartmentMapper 更新部门名。缓存按 namespace 管理，不会自动建立“SQL 依赖哪些表”的全图，所以 UserMapper 旧结果可能还在。

可选择不开二级缓存、把相关查询/更新纳入同一明确失效边界、谨慎使用 `cache-ref` 共享缓存，或在服务层设计可验证的缓存策略。定时任务、其他服务、DBA 直改等外部写入同样要覆盖，不能只测单 Mapper 的增删改查。

## 4. #{} 与 ${}：绑定值和拼 SQL 文本

| 语法 | 机制 | 常见用途 | 安全边界 |
|---|---|---|---|
| `#{name}` | 生成 `?`，通过 PreparedStatement 绑定值 | id、姓名、时间、过滤值 | 值不会作为 SQL 结构解析 |
| `${column}` | 将字符串原样替换进 SQL | 少数必须动态的标识符/语法片段 | 不转义、不绑定，必须由程序白名单控制 |

```xml
<!-- 输入 O'Reilly 等普通字符，仍按一个值绑定 -->
SELECT id FROM users WHERE name = #{name}
```

不能写 `ORDER BY #{column}` 来动态选择列名：占位符代表值，不是 SQL 标识符。表名、列名、ASC/DESC 不能直接靠值绑定变成语法。

安全的排序方式之一是固定分支：

```xml
ORDER BY
<choose>
  <when test="sort == 'createdAt'">created_at</when>
  <when test="sort == 'name'">name</when>
  <otherwise>id</otherwise>
</choose>
DESC
```

另一种是把外部 sort key 映射为程序内固定 SQL 片段，再使用替换；必须拒绝不在白名单中的值。不能把用户的 `orderBy` 原样传入 `${}`，也不能认为加个正则“去单引号”就可靠。

模糊查询可在应用把关键字组成参数 `%keyword%` 后 `LIKE #{pattern}`；不要改成 `LIKE '%${keyword}%'`。用户输入中的 `%`、`_` 是否应该当通配符，是**业务匹配语义**问题，需要独立转义策略；参数绑定不会自动替你改成字面匹配。

## 5. 常见坑与验证清单

| 现象 | 优先检查 |
|---|---|
| 连续调用两次仍发两次 SQL | 是否同会话？是否有 Spring 事务？缓存键/动态 SQL 是否相同？ |
| 二级缓存一直不命中 | namespace 是否配置缓存？前会话是否完成事务？结果是否可序列化？ |
| 更新后仍读旧值 | 跨 namespace、外部写入、本地缓存、隔离级别分别检查 |
| 动态排序失效 / 注入风险 | 是否误用值占位符作列名？是否原样拼外部输入？ |
| 压测多实例后不一致 | 默认本地二级缓存是否跨进程共享？失效广播有没有设计？ |

测试至少包括：同会话、跨会话、提交后、回滚后、同 Mapper 更新、跨 Mapper 更新、多实例/外部更新。记录 SQL 次数与返回值，不只看“执行很快”。日志不要打印密码、令牌和完整个人信息。

## 6. 面试口述版

> MyBatis 一级缓存跟 SqlSession 走，不是跟 HTTP 请求走；Spring 中由事务边界影响会话复用。二级缓存是 namespace 范围的显式选择，事务完成后再发布或失效，跨 Mapper 和外部写入容易留下旧值。#{} 是预编译参数绑定，用来传值；${} 是原样拼接，动态列名等只能走受控白名单。我会先证明一致性和注入边界，再谈缓存命中率。

## 延伸阅读

- 本地补缺依据：[编程对照补缺](../开源精选/对照补缺.md)；相邻开源主题：[Spring 事务摘录](../开源精选/java/toBeBetterJavaer-springboot-transaction.md)。本地摘录未覆盖 MyBatis 全部细节，以下官方文档作机制核验。
- 官方：[Mapper XML、缓存和参数](https://mybatis.org/mybatis-3/sqlmap-xml.html)、[MyBatis-Spring 会话](https://mybatis.org/spring/sqlsession.html)。
