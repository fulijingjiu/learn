# Python：Debug 技巧

## Traceback 怎么读

1. 从下往上：最后一行是异常类型与信息
2. 找**你自己项目路径**的那一帧（跳过 site-packages 噪音）
3. 看该行变量：谁是 `None`、类型是否不对、文件是否不存在

```text
Traceback (most recent call last):
  File "app/api/order.py", line 42, in create
    total = item.price * qty
TypeError: ...
```

先看 `order.py:42`，不是先怪框架。

## 复现步骤

1. 固定：Python 版本、venv 依赖哈希、输入 JSON、环境变量
2. `pytest` 最小复现；或 `curl` 同路径同 body
3. 打开更详细日志：`LOG_LEVEL=DEBUG`

## 断点

- IDE 断点；或 `breakpoint()`（pdb）
- 远程/容器里慎用交互 pdb；优先日志与测试
- 条件：只在 `user_id==...` 停

## venv 与包冲突

- 现象：本机能跑 CI 挂；或 `ModuleNotFoundError`；或奇怪 API 行为
- 检查：
  - `which python` / `sys.executable` 是否在 venv 内
  - `pip freeze` 与锁文件一致吗？
  - 是否装到了用户目录/系统目录混用
- 依赖冲突：版本范围互斥；用锁文件；必要时新建干净 venv
- `pip check`；看谁拉了旧传递依赖

## 常见异常

| 异常 | 常见原因 | 动作 |
|---|---|---|
| `TypeError`/`AttributeError` | None、类型不符 | 断言入参、类型提示、Pydantic |
| `KeyError`/`IndexError` | 缺字段、空列表 | `.get`、先判空 |
| `ImportError`/`ModuleNotFoundError` | 路径/venv/包名 | 工作目录、`PYTHONPATH`、是否安装 |
| `ConnectionError`/超时 | 下游挂、DNS、防火墙 | 重试策略、超时、熔断 |
| 异步 `RuntimeError: Event loop` | 嵌套 loop/错上下文 | 统一入口 uvicorn；别乱 `asyncio.run` |

## 数据库相关

- SQLAlchemy：看 `echo=True` 打出的 SQL；N+1 用 eager load
- 连接泄漏：确保 session 关闭（依赖 `yield` + finally）
- 细节排障步骤对齐：`/workspace/面试八股/06-实战/`

## 清单

- [ ] Traceback 定位到业务行了吗？
- [ ] 虚拟环境对吗？
- [ ] 能写成一个失败单测吗？
- [ ] 修的是根因还是只 catch 掉？
