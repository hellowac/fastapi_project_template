# FastAPI Example Project

有些人在阅读有关 [FastAPI 最佳实践](https://github.com/zhanymkanov/fastapi-best-practices) 的文章后，在我的 GitHub 个人资料中搜索项目示例。 不幸的是，我没有有用的公共存储库，只有我的旧概念验证项目。

因此，在获得一些实际经验之后，我决定解决这个问题并展示我现在如何开始项目。

这个 repo 是我在启动新的 FastAPI 项目时使用的一种模板：

- 生产就绪
  - 具有动态工作人员配置的 gunicorn（从 [@tiangolo](https://github.com/tiangolo) 窃取）
  - Dockerfile 针对非 root 用户的小尺寸和快速构建进行了优化
  - JSON 日志
  - 已部署环境支持sentry
- 易于本地开发
  - 配置了 postgres 和 redis 的环境
  - 使用 `black`、`autoflake`、`isort` 对代码进行 lint 的脚本（也从 [@tiangolo](https://github.com/tiangolo) 窃取）
  - 使用pytest配置了 `async-asgi-testclient`、`pytest-env`、`pytest-asyncio`
  - 全部已做类型注解以符合`mypy`
- SQLAlchemy 稍微配置了 `alembic`
  - 使用 `asyncpg` 进行异步数据库调用
  - 设置`sqlalchemy2-stubs`
  - 以易于排序的格式设置的迁移 (`YYYY-MM-DD_slug`)
- 预装 JWT 授权
  - 短期访问令牌
  - 存储在 http-only cookie 中的长期刷新令牌
  - 使用 `bcrypt` 的加盐密码存储
- 全局 pydantic 模型支持
  - `orjson`
  - JSON 导出期间的显式时区设置
- 和其他一些附加功能，如全局异常、sqlalchemy 键命名约定、alembic 的快捷脚本等。

## 本地开发

### 第一次构建需要

1. `cp .env.example .env`
2. `docker network create app_main`
3. `docker-compose up -d --build`

### Linters

格式化代码

```shell
docker compose exec app format
```

### 迁移

- 从 `src/database.py` 中的更改创建自动迁移

```shell
docker compose exec app makemigrations *migration_name*
```

- 运行迁移

```shell
docker compose exec app migrate
```

- 迁移降级

```shell
docker compose exec app downgrade -1  # or -2 or base or hash of the migration
```

### 测试

所有测试都是集成的，需要数据库连接。

我做出的选择之一是使用默认数据库 (`postgres`)，与应用程序的 `app` 数据库分开。

- 使用默认数据库可以更轻松地在 CI/CD 环境中运行测试，因为无需设置额外的数据库
- 测试使用 `force_rollback=True` 运行，即每笔交易都会被还原

运行测试

```shell
docker compose exec app pytest
```
