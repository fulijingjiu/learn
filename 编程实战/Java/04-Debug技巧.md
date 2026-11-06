# Java：Debug 技巧

## 标准复现四步

1. **复现**：固定输入、环境、版本（jar/git commit）
2. **缩小**：单测 / 本地同参 / 关掉无关中间件
3. **观察**：断点 + 日志 + SQL 日志 + 线程栈
4. **验证**：修完写回归用例，防再犯

## 断点（IDEA）

- 行断点、条件断点（`userId==123`）、异常断点（NullPointerException）
- **Evaluate Expression** 看临时表达式；别在生产挂调试端口乱连
- 远程 Debug：只临时开，防火墙收紧

## 日志

- 级别：ERROR 要人处理；WARN 可恢复；INFO 关键节点；DEBUG 细节
- 异常必须打栈：`log.error("xx failed, orderId={}", id, e)`
- 请求入口打 method/path/关键参数（脱敏）；出口打耗时

## 常见异常与排查

### NullPointerException（NPE）

- 看栈顶第一行业务代码；谁为 null
- 可选链：返回值、Map.get、反序列化缺字段、自动拆箱 `Integer`→`int`
- 预防：校验入参、集合 empty、Optional 仅边界

### 并发问题

- 现象：偶发错账、丢更新、死锁
- 查：是否共享可变状态；有无同步；DB 唯一索引；死锁日志（MySQL `show engine innodb status`）
- 工具：线程 dump（`jstack`）、复现加压

### 依赖冲突

- 现象：`NoSuchMethodError` / `ClassNotFoundException` / 同包不同行为
- `mvn dependency:tree` 找重复；`<exclusion>` 或统一 BOM 版本
- Spring Boot 用官方 BOM 管版本，别手写一堆冲突版本

### 其它高频

- `BeanCreationException`：看 caused by；常是循环依赖或缺配置
- `SQLException`：开 `logging.level.jdbc.sqlonly` 或 p6spy；对照面试八股排障篇
- 接口 500：先看全局异常处理器有没有吞掉根因

## 清单（贴显示器）

- [ ] 能稳定复现吗？
- [ ] 完整栈在哪？
- [ ] 最近改动是什么？（git blame / 发布记录）
- [ ] 是数据问题还是代码问题？
- [ ] 修完有没有测试？
