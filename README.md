# CMS 服务系统 API 文档

自动从 OpenAPI 规范生成的 Mintlify 格式 API 文档

---

## 概览

本项目包含从 `CMS服务系统API.openapi.json` 自动生成的完整 API 文档。

**统计数据：**
- **总接口数**: 48 个
- **分类数**: 11 个
- **文档格式**: Mintlify MDX
- **支持语言**: 简体中文
- **Base URL**: `https://dev-admin.beauty-666.com`

---

## 目录结构

```
docs/
├── api-reference/                    # API 文档目录
│   ├── system/                      # 系统接口 (2)
│   ├── video/                       # 视频管理 (4)
│   ├── vip/                         # 会员配置 (5)
│   ├── categories/                  # 分类管理 (10)
│   ├── tags/                        # 标签管理 (6)
│   ├── diamond/                     # 钻石配置 (6)
│   ├── exchange-rate/               # 汇率配置 (3)
│   ├── homepage-modules/            # 首页模块管理 (4)
│   ├── homepage-items/              # 模块子项管理 (5)
│   ├── preview/                     # 预览接口 (1)
│   ├── algorithm/                   # 推荐算法 (2)
│   ├── generated_files.json         # 文件清单
│   └── navigation_config.json       # 导航配置模板
├── QUICK_START.md                   # 快速开始指南 ⭐
├── MINTLIFY_INTEGRATION_GUIDE.md    # 详细集成指南
├── GENERATION_SUMMARY.md            # 生成总结报告
└── README.md                        # 本文件
```

---

## 快速开始

### 1. 查看快速开始指南

```bash
cat /Users/allen/Desktop/docs/QUICK_START.md
```

这是最简单快速的使用指南！

### 2. 复制文档

```bash
cp -r /Users/allen/Desktop/docs/api-reference /path/to/mintlify-project/
```

### 3. 更新 mint.json

参考 `navigation_config.json` 更新你的 Mintlify 配置。

### 4. 启动预览

```bash
cd /path/to/mintlify-project
mintlify dev
```

---

## 文档分类

### 系统接口 (2)
基础的系统健康检查和版本信息接口。

### 视频管理 (4)
视频列表查询、详情获取、状态管理等核心功能。

### 会员配置 (5)
订阅套餐的 CRUD 操作，包括价格、时长等配置。

### 分类管理 (10)
视频分类的完整管理功能，支持树形结构、拖拽排序等。

### 标签管理 (6)
标签的创建、更新、删除、排序和统计功能。

### 钻石配置 (6)
钻石套餐管理，包括价格、赠送、首充优惠等配置。

### 汇率配置 (3)
USDT 对 CNY 汇率的配置和实时同步功能。

### 首页模块管理 (4)
首页模块的列表、详情、配置更新和统计功能。

### 模块子项管理 (5)
首页模块子项的 CRUD 操作。

### 预览接口 (1)
H5 页面预览数据接口。

### 推荐算法 (2)
推荐算法配置的查询和更新。

---

## 文档特性

每个 API 文档包含：

### 完整的参数说明
- **请求头**: Authorization token
- **路径参数**: 动态路径变量
- **查询参数**: URL 查询字符串
- **请求体**: POST/PUT 请求体

### 多语言示例代码
- **cURL**: 命令行工具
- **JavaScript**: fetch API
- **Python**: requests 库

### 响应文档
- **字段说明**: 详细的响应字段描述
- **嵌套字段**: 支持展开查看
- **示例数据**: 成功和错误响应

### 格式特点
- **简体中文**: 所有内容使用简体中文
- **格式统一**: 使用 Mintlify 标准组件
- **干净整洁**: 页面布局清晰简洁

---

## 相关文件

### 文档文件

| 文件 | 说明 |
|------|------|
| `QUICK_START.md` | 快速开始指南，推荐首先阅读 ⭐ |
| `MINTLIFY_INTEGRATION_GUIDE.md` | 详细的 Mintlify 集成步骤 |
| `GENERATION_SUMMARY.md` | 完整的生成过程和统计报告 |
| `README.md` | 项目概述（本文件）|

### 配置文件

| 文件 | 说明 |
|------|------|
| `generated_files.json` | 所有生成文件的元数据清单 |
| `navigation_config.json` | Mintlify 导航配置模板 |

### 工具脚本

| 文件 | 说明 |
|------|------|
| `/Users/allen/Desktop/generate_mintlify_docs.py` | 文档生成脚本 |
| `/Users/allen/Desktop/verify_docs.py` | 文档验证脚本 |

---

## 使用场景

### 场景 1: 新建 Mintlify 项目

```bash
# 1. 创建 Mintlify 项目
npx create-mintlify-app my-api-docs
cd my-api-docs

# 2. 复制文档
cp -r /Users/allen/Desktop/docs/api-reference .

# 3. 更新 mint.json
# (参考 navigation_config.json)

# 4. 启动
mintlify dev
```

### 场景 2: 集成到现有项目

```bash
# 1. 复制文档到现有项目
cp -r /Users/allen/Desktop/docs/api-reference /path/to/existing-project/

# 2. 更新现有的 mint.json
# (添加 api-reference 相关的导航配置)

# 3. 启动预览
cd /path/to/existing-project
mintlify dev
```

### 场景 3: 更新文档

```bash
# 1. 更新 OpenAPI JSON 文件
# /Users/allen/Desktop/CMS服务系统API.openapi.json

# 2. 重新生成文档
cd /Users/allen/Desktop
python3 generate_mintlify_docs.py

# 3. 验证文档
python3 verify_docs.py

# 4. 复制到项目
cp -r /Users/allen/Desktop/docs/api-reference /path/to/project/
```

---

## 自定义配置

### 修改 Base URL

编辑 `generate_mintlify_docs.py`:

```python
BASE_URL = "https://your-domain.com"
```

重新运行生成脚本。

### 修改目录映射

编辑 `generate_mintlify_docs.py`:

```python
TAG_TO_DIR = {
    "系统": "system",
    "视频管理": "video",
    # 添加或修改映射...
}
```

### 自定义文档内容

直接编辑生成的 `.mdx` 文件：

```bash
vim api-reference/video/get-videos-list.mdx
```

---

## 验证和测试

### 验证文档完整性

```bash
cd /Users/allen/Desktop
python3 verify_docs.py
```

输出示例：
```
============================================================
文档验证报告
============================================================

总计应有 48 个文档文件
✓ 存在的文件: 48
✗ 缺失的文件: 0

...

🎉 所有文档验证通过！
============================================================
```

### 本地预览

```bash
cd /path/to/mintlify-project
mintlify dev
```

访问 `http://localhost:3000`

---

## API 端点示例

### 获取视频列表
```
GET https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/list
```

### 创建钻石套餐
```
POST https://dev-admin.beauty-666.com/api/v1/oms/diamonds
```

### 获取首页模块
```
GET https://dev-admin.beauty-666.com/api/v1/admgmt/homepage/modules
```

完整的 48 个接口文档请查看 `api-reference/` 目录。

---

## 部署

### Mintlify Cloud (推荐)

1. 推送到 GitHub
   ```bash
   git init
   git add .
   git commit -m "Add API documentation"
   git remote add origin <your-repo-url>
   git push -u origin main
   ```

2. 连接到 Mintlify
   - 访问 https://mintlify.com/dashboard
   - 连接 GitHub 仓库
   - 自动部署

### 自托管

```bash
# 构建
mintlify build

# 部署到 Vercel
vercel deploy

# 或部署到 Netlify
netlify deploy
```

---

## 技术栈

- **文档格式**: Mintlify MDX
- **源规范**: OpenAPI 3.0.1
- **生成语言**: Python 3
- **组件库**: Mintlify Components
- **内容语言**: 简体中文

---

## 更新日志

### 2026-01-07 - v1.0.0
- ✓ 初始版本
- ✓ 生成 48 个 API 接口文档
- ✓ 支持 11 个分类
- ✓ 完整的请求和响应示例
- ✓ 多语言代码示例（cURL、JavaScript、Python）
- ✓ 验证脚本和集成指南

---

## 常见问题

### Q1: 如何更新某个接口的文档？
**A**: 有两种方式：
1. 手动编辑对应的 `.mdx` 文件
2. 更新 OpenAPI JSON 后重新生成

### Q2: 文档中部分字段显示"字段说明"？
**A**: 这是因为 OpenAPI JSON 中缺少详细描述。建议：
- 在 OpenAPI JSON 中添加 `description` 字段
- 或手动编辑 `.mdx` 文件补充

### Q3: 如何自定义文档样式？
**A**: 在 Mintlify 项目的 `mint.json` 中配置主题：
```json
{
  "primaryColor": "#FF6B6B",
  "theme": "dark",
  ...
}
```

### Q4: 支持其他语言吗？
**A**: 当前生成的是简体中文文档。如需其他语言：
1. 修改生成脚本中的文本
2. 或手动翻译生成的 `.mdx` 文件

### Q5: 如何添加认证说明？
**A**: 创建一个 `authentication.mdx` 文件：
```mdx
---
title: '认证说明'
description: 'API 认证方式'
---

## JWT Token

所有接口需要在请求头中携带 JWT token：
\`\`\`
Authorization: Bearer YOUR_TOKEN
\`\`\`

...
```

然后在 `mint.json` 中引用。

---

## 支持与反馈

如遇到问题或有改进建议，请：

1. 检查 `QUICK_START.md` 和 `MINTLIFY_INTEGRATION_GUIDE.md`
2. 运行 `verify_docs.py` 验证文档完整性
3. 参考 Mintlify 官方文档: https://mintlify.com/docs

---

## 许可证

本文档生成工具和生成的文档可自由使用和修改。

---

**生成时间**: 2026-01-07
**文档版本**: 1.0.0
**OpenAPI 版本**: 3.0.1
**项目**: CMS 服务系统 API 文档

---

## 推荐阅读顺序

1. **本文件** (README.md) - 了解整体概况 ✓
2. **QUICK_START.md** - 快速集成和使用 ⭐
3. **MINTLIFY_INTEGRATION_GUIDE.md** - 详细集成步骤
4. **GENERATION_SUMMARY.md** - 生成过程和统计
5. **api-reference/** - 查看具体的 API 文档

开始使用吧！祝你使用愉快！
