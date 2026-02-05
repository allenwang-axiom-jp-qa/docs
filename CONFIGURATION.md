# 配置管理文档

## 配置加载流程

cms-site-svc-video-mgmt 服务使用多层配置系统，支持从多个来源加载配置。

### 配置优先级

配置加载优先级从高到低：

1. **命令行参数** - 通过 `-conf` 参数指定的配置文件
2. **环境变量** - 通过 `envConfig` 等环境变量覆盖配置
3. **etcd 远程配置** - 从 etcd 配置中心拉取的配置
4. **默认配置文件** - `conf/env.json`

### 启动流程

服务启动时按照以下阶段初始化：

#### 阶段 1: 命令行参数解析
```bash
./cms-site-svc-video-mgmt -conf ./conf/env.json
```

#### 阶段 2: 配置加载
```go
config.SetupConfig(Service, flagEnvName)
```

`SetupConfig` 函数会：
- 读取本地配置文件（JSON 格式）
- 连接 etcd 并拉取远程配置
- 合并本地和远程配置
- 应用环境变量覆盖

#### 阶段 3: 日志系统初始化
根据配置初始化日志系统：
- 日志输出路径
- 是否输出到标准输出
- 日志级别

#### 阶段 4: 链路追踪初始化
如果配置了 APM endpoint，初始化 OpenTelemetry 追踪。

#### 阶段 5-10: 服务组件初始化
依次初始化：
- RPC 服务（gRPC）
- 数据库连接
- 业务逻辑层
- HTTP 服务
- 启动服务

## 配置文件结构

### 本地配置 (conf/env.json)

```json
{
  "local": {
    "http_port": 8080,
    "rpc_port": 9090,
    "etcd_addresses": ["localhost:2379"],
    "etcd_username": "root",
    "etcd_password": "password",
    "log": {
      "out_path": "logs/app.log",
      "stdout": true,
      "level": "info",
      "panic_path": "logs/panic.log"
    }
  },
  "core": {
    "service_ttl_seconds": 30,
    "service_interval_seconds": 15,
    "apm_config": {
      "endpoint": "http://localhost:4318"
    }
  }
}
```

### 环境变量

支持的环境变量：
- `envConfig` - 指定配置环境（dev/staging/prod）

## 最佳实践

### 开发环境

```bash
# 使用本地配置文件
./cms-site-svc-video-mgmt -conf ./conf/dev.json
```

### 生产环境

```bash
# 使用 etcd 远程配置
export envConfig=production
./cms-site-svc-video-mgmt -conf ./conf/prod.json
```

### 配置更新

1. **本地配置更新**：修改配置文件后重启服务
2. **远程配置更新**：更新 etcd 中的配置，服务会自动热加载（如果实现了配置监听）

## 故障排查

### 配置加载失败

检查以下项：
1. 配置文件路径是否正确
2. etcd 连接是否正常
3. 配置文件 JSON 格式是否正确

### 日志输出

服务启动时会输出配置加载信息：
```
开始设置配置...
etcd endpoints=[localhost:2379] user=root
envConfig=production
```

## 配置安全

### 敏感信息处理

- 数据库密码应使用环境变量或加密存储
- etcd 密码不应提交到代码仓库
- 使用 `.gitignore` 排除本地配置文件

### 配置加密

生产环境建议：
1. 使用 Kubernetes Secrets 管理敏感配置
2. 使用 etcd 的加密功能
3. 定期轮换密钥
