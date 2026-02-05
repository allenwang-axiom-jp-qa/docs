# 标准部署流程 - Dev环境

## ⚠️ 重要原则
**严格按照标准合并流程操作，不在dev分支上直接开发**

## 📋 标准合并流程（推荐）

### 1. 确保当前在首页配置分支上，提交所有本地更改
```bash
git add .
git commit -m "feat: 推荐算法配置功能完成"
```

### 2. 拉取远程所有分支的最新状态
```bash
git fetch origin
```

### 3. 在当前分支上合并远程最新的 dev，检查冲突
```bash
git merge origin/dev
```

**如果有冲突，解决冲突后：**
```bash
git add .
git commit -m "merge: 解决与 dev 分支的冲突"
```

### 4. 切换到 dev 分支
```bash
git checkout dev
```

### 5. 拉取远程 dev 最新代码
```bash
git pull origin dev
```

### 6. 合并功能分支到 dev
```bash
# 假设你在 feat/recommend-config 分支开发
git merge feat/recommend-config
```

### 7. 推送到远程 dev 分支
```bash
git push origin dev
```

## 🚫 避免的错误做法

❌ **错误**: 直接在 dev 分支上开发
```bash
git checkout dev
# 直接修改代码...
git add .
git commit -m "feat: xxx"
```

✅ **正确**: 创建功能分支开发
```bash
git checkout -b feat/recommend-config dev
# 开发...
git add .
git commit -m "feat: add recommend config"
# 然后按照标准流程合并
```

## 📝 本次推荐配置部署

### 当前状态
- ✅ 数据库已部署（configs表和初始数据）
- ✅ 代码已实现（标准OMS路由模式）
- ✅ 路由路径：`/api/v1/oms/config/recommend`
- ⚠️ 与其他OMS模块保持一致，无特殊处理

### 待完成
1. 按标准流程合并到 dev 分支
2. 触发CI/CD部署到K8s
3. 验证API可用性

### 网关配置说明
- 网关当前期望：`/api/recommend/config` → `cms-config-http`
- OMS实际提供：`/api/v1/oms/config/recommend`
- **需要更新网关配置**将路径路由到 `cms-oms` 服务
