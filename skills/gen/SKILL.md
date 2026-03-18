---
name: gen
description: 从数据库表结构生成 Dilu 模块代码（Model/Query/DTO/Service/API/Router）。当用户需要为数据库表生成 CRUD 代码时使用。
argument-hint: -t <table> -d <dsn> [--driver <mysql|postgres|sqlite>] [--prefix <api-prefix>]
allowed-tools: Bash, Read, Edit, Glob, Grep
---

# 生成 Dilu 模块代码

使用 `dilu-ctl gen` 从数据库表结构自动生成完整的 CRUD 模块。

## 前置条件

**必须先执行环境检查，再进行后续操作。**

1. 检查 Go 是否已安装：
   ```bash
   go version
   ```
   如果未安装，告知用户需要先安装 [Go](https://go.dev/)（1.21+），然后停止。

2. 检查 `dilu-ctl` 是否已安装：
   ```bash
   which dilu-ctl
   ```
   如果未找到，自动安装：
   ```bash
   go install github.com/baowk/dilu-ctl@latest
   ```
   安装后再次确认：
   ```bash
   dilu-ctl version
   ```
   如果仍然失败，提示用户检查 `$GOPATH/bin` 是否在 `$PATH` 中，然后停止。

3. 当前目录或 `-P` 指定的路径是一个有效的 Dilu 项目（包含 `go.mod`）
4. 数据库可达

## 参数解析

从 `$ARGUMENTS` 中提取：
- `-t <table>` — 表名（必填）
- `-d <dsn>` — 数据库连接字符串（必填）
- `--driver <type>` — 数据库类型：`mysql`（默认）、`postgres`、`sqlite`（可自动推断）
- `-p <package>` — 包名（可选，默认从表名推导）
- `--prefix <path>` — API 路径前缀（默认 `/v1`）
- `-P <path>` — 项目根目录（默认 `.`）
- `-f` — 覆盖已存在文件

## 数据库连接字符串示例

| 数据库 | 格式 |
|--------|------|
| MySQL | `user:password@tcp(host:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local` |
| PostgreSQL | `host=localhost user=postgres password=xxx dbname=mydb port=5432 sslmode=disable` |
| SQLite | `/path/to/database.db` 或 `file:test.db` |

## 执行步骤

### Step 1: 确认项目环境

```bash
# 验证 go.mod 存在
test -f go.mod && head -1 go.mod
```

### Step 2: 生成代码

```bash
dilu-ctl gen $ARGUMENTS
```

### Step 3: 补充依赖

首次生成时需添加 gorm-gen 相关依赖：

```bash
go get github.com/jinzhu/copier gorm.io/gen gorm.io/plugin/dbresolver
```

### Step 4: 验证编译

```bash
go build ./...
```

如果编译失败，检查常见问题：
- 缺少 `common/consts` 包 → 可能需要创建 `common/consts/const.go`
- import 路径不匹配 → 检查 `go.mod` 的 module 名称

### Step 5: 输出结果

告知用户：
1. 生成了哪些文件
2. 路由路径（根据路由规则推导）
3. 下一步：使用 `/dilu-ctl:run` 启动服务测试

## 路由规则

- 包名 == 表名 → `/{prefix}/{package}/page`（如 `/api/articles/page`）
- 包名 != 表名 → `/{prefix}/{package}/{module}/page`（如 `/api/blog/articles/page`）

## 生成的文件列表

```
internal/modules/{pkg}/
├── repository/
│   ├── model/{table}.gen.go        # GORM Model（gorm-gen 生成）
│   └── query/{table}.gen.go        # 类型安全查询（gorm-gen 生成）
│         query/gen.go
├── service/
│   ├── dto/{table}.go              # 请求/响应 DTO
│   └── {table}_service.go          # 业务逻辑层
├── apis/{table}.go                 # HTTP Handler（含 Swagger 注解）
└── router/
    ├── {table}.go                  # 路由注册
    └── router.go                   # 路由初始化

cmd/start/{pkg}.go                  # 模块路由注册入口
```

## 生成的 API 接口

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/{prefix}/{pkg}/page` | 分页查询 |
| GET | `/{prefix}/{pkg}/:id` | 获取单条 |
| POST | `/{prefix}/{pkg}` | 创建 |
| PUT | `/{prefix}/{pkg}/:id` | 更新 |
| DELETE | `/{prefix}/{pkg}/:id` | 删除 |
