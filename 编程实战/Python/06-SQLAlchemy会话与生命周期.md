# SQLAlchemy 2.x：Session、事务与 FastAPI 生命周期

> 范围：SQLAlchemy 2.x，示例采用 FastAPI + 异步驱动。Session 不是数据库连接池，也不是任意查询的结果缓存；理解资源所有权，比记住依赖注入写法更重要。

## 1. 概念：谁长寿，谁短寿？

| 对象 | 职责 | 常见作用域 |
|---|---|---|
| Engine / AsyncEngine | 连接方言、连接池与执行入口 | 应用进程级复用，退出时释放 |
| sessionmaker / async_sessionmaker | Session 工厂，保存配置 | 应用进程级复用 |
| Session / AsyncSession | 工作单元、identity map、事务状态 | 每次请求/业务单元独立创建 |
| 数据库事务 | 保证一组操作的原子性与隔离性 | 明确的业务边界，不等于进程生命周期 |
| ORM 对象 | 映射记录及其加载/过期状态 | 不应脱离会话后随意触发隐式查库 |

Session 往往按需取得连接，开始数据库事务后占用相应资源；所以“创建了 Session”不等于“立刻开一条专用永久连接”。同样，`close()` 不是提交。

## 2. 为什么需要 Session？identity map + unit of work

- identity map 在同一会话里为同一映射主键维护对象身份，避免一条记录出现多个互不关联的实例。
- 工作单元跟踪新增、修改、删除，在 flush 时发出对应 SQL。
- `session.get(User, id)` 可能直接命中会话里已有且未过期的对象；普通 `select(User)...` 通常仍发送查询。Session 不是 Redis 式“相同 SQL 自动免查库”的通用缓存。
- 身份一致不保证永远最新；刷新后是否看到别人的新提交仍受数据库隔离级别影响。

## 3. flush、commit、rollback、close 对比

| 操作 | 做什么 | 不代表什么 |
|---|---|---|
| `flush()` | 将待处理变更发为 SQL，仍在当前事务中 | 其他事务立刻可见、不可回滚 |
| `commit()` | 提交事务，通常先自动 flush | 只在内存改个标记 |
| `rollback()` | 回滚当前事务，恢复可用事务状态 | 任意外部 HTTP 副作用也回滚 |
| `close()` | 结束使用，释放连接/事务等资源，默认行为可重置会话 | 自动把未提交修改保存下来 |
| `refresh(obj)` | 重新从数据库加载对象属性 | 绕过隔离级别、打穿所有数据库缓存 |

默认 autobegin：查询也可能让 Session 进入事务。因此不要先查几条，再随手 `async with session.begin()`，否则可能提示事务已开始；应先定义整体边界。

autoflush 常发生在 ORM 查询前，commit 也会 flush。`autoflush=False` 不会阻止 commit 发出变更。flush 发生约束错误后，即使底层已回滚，也需显式 `rollback()` 或让事务上下文正确退出后再使用会话；不能捕获异常后继续在失败事务里查库。

## 4. FastAPI 异步最小生命周期骨架

先安装与连接 URL 匹配的异步驱动，例如 PostgreSQL 的 asyncpg、MySQL 的 asyncmy 等。数据库地址由部署环境注入；不把真实凭据写进笔记或仓库。下面只演示连接生命周期，`User` 等映射类需在项目中定义。

```python
import os
from collections.abc import AsyncIterator
from contextlib import asynccontextmanager
from typing import Annotated
from fastapi import Depends, FastAPI
from sqlalchemy.ext.asyncio import (
    AsyncSession, async_sessionmaker, create_async_engine,
)

engine = create_async_engine(os.environ["DATABASE_URL"], pool_pre_ping=True)
SessionFactory = async_sessionmaker(engine, expire_on_commit=False)

@asynccontextmanager
async def lifespan(app: FastAPI):
    yield
    await engine.dispose()

app = FastAPI(lifespan=lifespan)

async def get_session() -> AsyncIterator[AsyncSession]:
    async with SessionFactory() as session:
        yield session

DbSession = Annotated[AsyncSession, Depends(get_session)]
```

这里依赖只负责创建和关闭，业务层显式决定提交；未提交事务在关闭时结束/回滚，**不能指望 yield 后关闭自动 commit**。不要一部分地方在依赖中提交、一部分地方在 service 提交，形成两个边界拥有者。

写操作形状：

```python
async def create_user(session: AsyncSession, user: "User") -> "User":
    async with session.begin():
        session.add(user)
        await session.flush()   # 例如需要数据库分配的主键
    return user                # 正常退出 begin 后已提交；异常则回滚
```

前提：调用者还没有启动另一个事务；整个业务单元只由一层管理 begin/commit。多个 repository 参与同一业务时，repository 可以执行/flush，但别各自 commit 切断原子性。复杂嵌套场景要明确 SAVEPOINT 等语义，不是随便再包一层 begin。

## 5. 异步并发：Session 不能共享给多个任务

`Session` 通常每线程一份，`AsyncSession` 每并发任务一份。它们是带状态的事务对象，不是线程安全/协程安全的全局客户端。

错误心智：把一个 AsyncSession 放到 `asyncio.gather` 的多个并发查询里，认为“都是读就安全”。正确选择：同一事务顺序执行，或每个独立任务自己建会话/事务；后者不再是同一个原子事务。

- 同步数据库驱动不会因为外层写了 `async def` 就变异步；阻塞调用仍可能卡事件循环。
- 背景任务不要继续借请求已关闭的会话，要创建自己的会话并定义重试/幂等。
- 流式响应、长时间网络调用不要无意中长持数据库事务。先完成必要 DB 工作，再处理耗时外部 IO，或者明确拆为可恢复流程。

## 6. 对象生命周期、过期与 N+1

默认 `expire_on_commit=True` 时提交会使属性过期，后续访问可能重新查库。在异步场景，属性访问无法随意隐式 await；常见 `MissingGreenlet` 来自过期属性或关系懒加载触发隐式 IO，而不是“网络一定断了”。

- 异步应用常用 `expire_on_commit=False` 方便提交后读取已加载字段，但它不保证对象永远最新。
- 响应需要的关系提前加载，例如 `select(User).options(selectinload(User.orders))`；这里 User 与关系是示意映射。
- `selectinload` 常用额外批量 SELECT 降低 N+1；`joinedload` 通过 JOIN，可能放大行数，集合场景还需留意去重等 API 要求。按数据规模验证。
- 在会话有效时构造 DTO/响应模型；不要把带未加载属性的 ORM 对象交给序列化器后才发现它要查库。
- 会话关闭后对象常处于 detached 状态：已加载普通字段可能还可读，未加载/过期属性不能靠原会话补查。

## 7. 常见坑与排障检查清单

| 报错/现象 | 优先检查 |
|---|---|
| PendingRollbackError | 是否此前 flush/commit 失败却没 rollback？ |
| A transaction is already begun | 是否查询已 autobegin，又嵌套 begin？ |
| MissingGreenlet | 是否隐式关系加载/过期属性 IO？是否用对异步驱动？ |
| DetachedInstanceError | 是否会话已关才读未加载字段？ |
| 连接池耗尽 | 是否会话未关闭、事务太长、并发过大？池总量是否乘了 worker 数？ |
| API 成功但数据没落库 | 是否只有 flush，或 close 前未 commit？ |
| 返回了不该存在的“成功” | 提交错误是否被吞掉？响应是否早于事务成功？ |

`pool_pre_ping` 可发现部分失效连接，不会自动安全重放失败的业务事务。连接池大小应综合数据库连接上限、应用副本数与 worker 数，而非每个进程都设很大。

## 8. 面试口述版

> Engine 和 Session 工厂可以按进程复用，但 Session 是短生命周期、带事务状态的工作单元，每个并发任务独立使用。flush 只是把变更发到当前事务，commit 才提交，close 不能代替提交。FastAPI 中我让依赖负责资源释放、业务层统一拥有事务边界，异常走回滚。异步还要防止懒加载和属性过期触发隐式 IO，提前加载响应需要的数据。

## 延伸阅读

- 补缺依据：[编程对照补缺](../开源精选/对照补缺.md)；框架背景：[FastAPI 开源 README](../开源精选/python/fastapi-README.md)。README 是入口，不是 Session 机制完整依据。
- 官方：[Session basics](https://docs.sqlalchemy.org/en/20/orm/session_basics.html)、[AsyncIO extension](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)。
