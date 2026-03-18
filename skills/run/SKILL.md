---
name: run
description: 编译并启动 Dilu 应用服务。当用户需要运行 Dilu 项目进行本地开发或测试时使用。
argument-hint: [-c <config-file>] [--build-only]
allowed-tools: Bash, Read, Glob, Grep
---

# 运行 Dilu 应用

编译并启动 Dilu 项目的 HTTP/gRPC 服务。

## 参数解析

从 `$ARGUMENTS` 中提取：
- `-c <config>` — 指定配置文件（如 `resources/config.sqlite.yaml`）
- `--build-only` — 只编译不运行

如果未指定配置文件，自动查找可用配置。

## 前置条件

**必须先执行环境检查，再进行后续操作。**

1. 检查 Go 是否已安装：
   ```bash
   go version
   ```
   如果未安装，告知用户需要先安装 [Go](https://go.dev/)（1.21+），然后停止。

2. 当前目录是一个有效的 Dilu 项目（包含 `go.mod`）

## 执行步骤

### Step 1: 确认项目

```bash
# 获取项目名
head -1 go.mod | awk '{print $2}'
```

### Step 2: 编译

```bash
go build -o ./<项目名> .
```

如果编译失败，分析错误并修复。

### Step 3: 查找配置文件

如果用户未指定 `-c`：

```bash
ls resources/config.*.yaml
```

优先级：`config.sqlite.yaml` > `config.dev.yaml` > 其他

对于 SQLite 配置，确保数据库目录存在：
```bash
mkdir -p temp
```

### Step 4: 停止旧实例

```bash
# 查找并停止同名进程
pkill -f "<项目名> start" 2>/dev/null
sleep 1
```

### Step 5: 启动服务

```bash
(./<项目名> start -c <config-file> > /tmp/<项目名>.log 2>&1 &)
sleep 3
```

### Step 6: 验证启动

```bash
# 检查进程
pgrep -f "<项目名> start"

# 检查日志中的错误
grep -c "ERROR\|panic" /tmp/<项目名>.log

# 提取端口和路由
grep "Server started" /tmp/<项目名>.log
grep "\[GIN-debug\]" /tmp/<项目名>.log | grep -v WARNING
```

### Step 7: 输出结果

告知用户：
- 服务地址和端口
- 已注册的路由列表
- 日志文件位置
- 如何停止：`pkill -f "<项目名> start"`
- 下一步：使用 `/dilu-ctl:test` 测试接口

## 配置文件说明

Dilu 配置文件 (`resources/config.*.yaml`) 关键字段：

```yaml
server:
  port: 7888        # 服务端口
  mode: dev         # dev/test/prod
dbcfg:
  driver: sqlite    # mysql/postgres/sqlite
  dns: temp/dilu.db # 连接字符串
cache:
  type: memory      # memory/redis
```

## 停止服务

```bash
pkill -f "<项目名> start"
```
