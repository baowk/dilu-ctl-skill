---
name: create
description: 创建新的 Dilu 项目脚手架。当用户需要初始化一个基于 Gin+GORM 的 Go Web 项目时使用。
argument-hint: <project-name> [--https] [--all] [-o <output-dir>]
allowed-tools: Bash, Read, Edit, Glob, Grep
---

# 创建 Dilu 项目

使用 `dilu-ctl create` 创建一个新的 Dilu 项目。

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

3. 检查 Git 是否已安装（create 需要 git clone）：
   ```bash
   git --version
   ```
   如果未安装，告知用户需要先安装 Git，然后停止。

## 参数解析

从 `$ARGUMENTS` 中提取：
- 第一个参数为项目名称（必填）
- `--https` — 使用 HTTPS 协议克隆（默认 SSH）
- `--all` 或 `-a` — 使用 dilu-all 仓库（含管理后台）
- `-o <path>` — 输出路径（默认当前目录）

## 执行步骤

### Step 1: 创建项目

```bash
dilu-ctl create -n $0 [其他参数]
```

### Step 2: 初始化依赖

进入项目目录后：

1. 如果同级目录存在 `dilu-core/`，启用 replace 指令：
   ```bash
   # 将 go.mod 中的 //replace github.com/baowk/dilu-core 取消注释
   sed -i 's|^//replace github.com/baowk/dilu-core|replace github.com/baowk/dilu-core|' go.mod
   ```
2. 执行依赖整理：
   ```bash
   go mod tidy
   ```

### Step 3: 验证编译

```bash
go build ./...
```

### Step 4: 输出结果

告知用户：
- 项目路径
- 可用的配置文件（`resources/config.*.yaml`）
- 下一步建议：使用 `/dilu-ctl:gen` 生成业务模块

## 项目结构说明

创建后的项目结构：
```
<project>/
├── cmd/start/          # 启动命令、路由注册
├── internal/
│   ├── bootstrap/      # 应用初始化
│   ├── common/         # 公共组件（middleware, config, utils）
│   └── modules/        # 业务模块（gen 生成到这里）
├── resources/          # 配置文件
├── docs/               # Swagger 文档
└── go.mod
```
