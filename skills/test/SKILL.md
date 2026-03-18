---
name: test
description: 对运行中的 Dilu 服务执行 E2E 接口测试（CRUD 全流程）。当用户需要验证生成的 API 是否正常工作时使用。
argument-hint: "[<base-url>] [--resource <path>] [--port <port>]"
metadata: {"openclaw":{"requires":{"bins":["curl"]},"emoji":"🧪","os":["darwin","linux"]}}
---

# E2E 接口测试

对运行中的 Dilu 服务执行完整的 CRUD 测试。

## 参数解析

从 `$ARGUMENTS` 中提取：
- `<base-url>` — 服务地址（默认 `http://localhost:7888`）
- `--resource <path>` — 资源路径（如 `/api/articles`，可自动从日志发现）
- `--port <port>` — 端口号（默认从配置推断）

## 前置条件

**必须先执行环境检查，再进行后续操作。**

1. 检查 `curl` 是否可用：
   ```bash
   which curl
   ```
   如果未安装，告知用户需要先安装 curl，然后停止。

2. 当前目录是一个有效的 Dilu 项目（包含 `go.mod`），用于从日志和 Model 文件推断信息。

## 执行步骤

### Step 1: 确认服务运行中

从配置文件获取端口，或使用默认端口：

```bash
curl -sf http://localhost:<port>/api/health && echo "服务正常" || echo "服务未启动"
```

如果服务未启动，提示用户先使用 `/dilu-ctl:run` 启动。

### Step 2: 发现路由

从日志或运行中的服务获取路由信息：

```bash
# 从日志获取
grep "\[GIN-debug\]" /tmp/<项目名>.log | grep -v WARNING
```

从输出中提取所有资源路径（去重）。

### Step 3: 执行 CRUD 测试

对每个发现的资源路径，按顺序执行：

#### 3.1 Create（创建）

```bash
curl -s <base>/<resource> \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '<根据资源推断的测试数据>'
```

从响应中提取创建的记录 ID。

#### 3.2 Page（分页查询）

```bash
curl -s <base>/<resource>/page \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"pageIndex":1,"pageSize":10}'
```

验证：`code == 200`，`total >= 1`。

#### 3.3 Get（查询单条）

```bash
curl -s <base>/<resource>/<id>
```

验证：`code == 200`，返回的数据与创建一致。

#### 3.4 Update（更新）

```bash
curl -s <base>/<resource>/<id> \
  -X PUT \
  -H 'Content-Type: application/json' \
  -d '<部分更新数据>'
```

验证：`code == 200`。

#### 3.5 Delete（删除）

```bash
curl -s <base>/<resource>/<id> -X DELETE
```

验证：`code == 200`。

#### 3.6 验证删除

再次查询被删除的记录，确认已不存在或总数减少：

```bash
curl -s <base>/<resource>/page \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"pageIndex":1,"pageSize":10}'
```

### Step 4: 输出测试报告

以表格形式汇总：

```
| 接口     | 方法   | 路径                    | 状态 |
|----------|--------|-------------------------|------|
| Create   | POST   | /api/articles           | ✅   |
| Page     | POST   | /api/articles/page      | ✅   |
| Get      | GET    | /api/articles/1         | ✅   |
| Update   | PUT    | /api/articles/1         | ✅   |
| Delete   | DELETE | /api/articles/1         | ✅   |
| Verify   | POST   | /api/articles/page      | ✅   |
```

### Step 5: 错误日志检查

```bash
grep -c "ERROR\|panic" /tmp/<项目名>.log
```

如果有错误，输出最近的错误详情辅助排查。

## 测试数据推断

如果用户未提供测试数据，根据 Model 文件推断：

1. 读取 `internal/modules/<pkg>/repository/model/<table>.gen.go`
2. 提取结构体字段和类型
3. 生成合理的测试数据（string → "test"，int → 1，bool → true）

## 注意事项

- 测试会实际写入数据库，测试结束后不自动清理
- SQLite 数据库可通过重新复制原始文件来重置
- 如果服务使用了认证中间件，需在请求头中添加 Token
