# API 文档快速开始指南

## 已完成 ✓

从 OpenAPI JSON 文件成功生成了 **48 个** Mintlify 格式的 API 文档！

---

## 文档位置

```
/Users/allen/Desktop/docs/api-reference/
```

包含 11 个分类目录，共 48 个 `.mdx` 文件。

---

## 快速集成（3 步）

### 第 1 步：复制文档

将整个 `api-reference` 目录复制到你的 Mintlify 项目：

```bash
# 方式一：复制到已有的 Mintlify 项目
cp -r /Users/allen/Desktop/docs/api-reference /path/to/your/mintlify-project/

# 方式二：如果还没有 Mintlify 项目，先创建
npx create-mintlify-app
# 然后复制文档
cp -r /Users/allen/Desktop/docs/api-reference /path/to/your/mintlify-project/
```

### 第 2 步：更新导航配置

打开你的 Mintlify 项目中的 `mint.json`，在 `navigation` 部分添加：

```json
{
  "navigation": [
    {
      "group": "API 参考",
      "pages": [
        {
          "group": "系统接口",
          "pages": [
            "api-reference/system/get-health",
            "api-reference/system/get-version"
          ]
        },
        {
          "group": "视频管理",
          "pages": [
            "api-reference/video/get-videos-list",
            "api-reference/video/get-videos-detail",
            "api-reference/video/get-videos-status",
            "api-reference/video/update-videos"
          ]
        },
        {
          "group": "会员配置",
          "pages": [
            "api-reference/vip/get-subscription-plans",
            "api-reference/vip/create-subscription-plans",
            "api-reference/vip/get-subscription-plans-by-id",
            "api-reference/vip/update-subscription-plans-by-id",
            "api-reference/vip/delete-subscription-plans-by-id"
          ]
        },
        {
          "group": "分类管理",
          "pages": [
            "api-reference/categories/get-categories",
            "api-reference/categories/get-categories-tree",
            "api-reference/categories/get-categories-stats",
            "api-reference/categories/get-category",
            "api-reference/categories/get-category-children",
            "api-reference/categories/create-category",
            "api-reference/categories/update-category",
            "api-reference/categories/update-category-drag",
            "api-reference/categories/delete-category",
            "api-reference/categories/get-tags"
          ]
        },
        {
          "group": "标签管理",
          "pages": [
            "api-reference/tags/get-tag",
            "api-reference/tags/get-tags-stats",
            "api-reference/tags/create-tag",
            "api-reference/tags/update-tag",
            "api-reference/tags/update-tags-reorder",
            "api-reference/tags/delete-tag"
          ]
        },
        {
          "group": "钻石配置",
          "pages": [
            "api-reference/diamond/get-oms-diamonds",
            "api-reference/diamond/get-oms-diamonds-overview",
            "api-reference/diamond/get-oms-diamonds-by-id",
            "api-reference/diamond/create-oms-diamonds",
            "api-reference/diamond/update-diamonds-by-id",
            "api-reference/diamond/delete-oms-diamonds-by-id"
          ]
        },
        {
          "group": "汇率配置",
          "pages": [
            "api-reference/exchange-rate/get-oms-diamonds-rate",
            "api-reference/exchange-rate/update-diamonds-rate",
            "api-reference/exchange-rate/create-oms-diamonds-rate-sync"
          ]
        },
        {
          "group": "首页模块管理",
          "pages": [
            "api-reference/homepage-modules/get-admgmt-homepage-overview",
            "api-reference/homepage-modules/get-admgmt-homepage-modules",
            "api-reference/homepage-modules/get-admgmt-homepage-modules-by-moduleId",
            "api-reference/homepage-modules/update-admgmt-homepage-modules-by-moduleId"
          ]
        },
        {
          "group": "模块子项管理",
          "pages": [
            "api-reference/homepage-items/get-admgmt-homepage-modules-by-moduleId-items",
            "api-reference/homepage-items/get-admgmt-homepage-items-by-itemId",
            "api-reference/homepage-items/create-admgmt-homepage-modules-by-moduleId-items",
            "api-reference/homepage-items/update-admgmt-homepage-items-by-itemId",
            "api-reference/homepage-items/delete-admgmt-homepage-items-by-itemId"
          ]
        },
        {
          "group": "预览接口",
          "pages": [
            "api-reference/preview/get-admgmt-homepage-preview-h5"
          ]
        },
        {
          "group": "推荐算法",
          "pages": [
            "api-reference/algorithm/get-api-recommend-config",
            "api-reference/algorithm/update-api-recommend-config"
          ]
        }
      ]
    }
  ]
}
```

或者使用我们提供的配置文件：

```bash
# 参考完整配置
cat /Users/allen/Desktop/docs/api-reference/navigation_config.json
```

### 第 3 步：启动预览

```bash
cd /path/to/your/mintlify-project
mintlify dev
```

访问 `http://localhost:3000` 查看文档效果！

---

## 文档特性

每个 API 接口文档包含：

### 基本信息
- 接口标题
- 完整 API URL (带 Base URL)
- 接口功能描述

### 参数文档
- 请求头 (Authorization)
- 路径参数 (如 `{id}`)
- 查询参数 (如 `page`, `pageSize`)
- 请求体参数 (POST/PUT)

### 代码示例
- **cURL**: 命令行请求
- **JavaScript**: fetch API 示例
- **Python**: requests 库示例

### 响应文档
- 响应字段说明
- 嵌套字段展开
- 成功响应示例 (200/201)
- 错误响应示例 (400/401/404)

---

## 文档结构

```
api-reference/
├── system/             # 系统接口 (2)
├── video/              # 视频管理 (4)
├── vip/                # 会员配置 (5)
├── categories/         # 分类管理 (10)
├── tags/               # 标签管理 (6)
├── diamond/            # 钻石配置 (6)
├── exchange-rate/      # 汇率配置 (3)
├── homepage-modules/   # 首页模块管理 (4)
├── homepage-items/     # 模块子项管理 (5)
├── preview/            # 预览接口 (1)
└── algorithm/          # 推荐算法 (2)
```

---

## 重新生成文档

如果 OpenAPI 规范更新，可以重新生成：

```bash
cd /Users/allen/Desktop
python3 generate_mintlify_docs.py
```

所有文档将自动更新。

---

## 辅助文件

### 1. 文件清单
`api-reference/generated_files.json` - 包含所有文件的元数据

### 2. 导航配置
`api-reference/navigation_config.json` - Mintlify 导航配置模板

### 3. 集成指南
`MINTLIFY_INTEGRATION_GUIDE.md` - 详细的集成说明

### 4. 总结报告
`GENERATION_SUMMARY.md` - 完整的生成报告

### 5. 验证脚本
`/Users/allen/Desktop/verify_docs.py` - 文档完整性验证

---

## 自定义修改

### 修改 Base URL

如果需要更改 API 服务器地址，编辑生成脚本：

```python
# 在 generate_mintlify_docs.py 中修改
BASE_URL = "https://your-new-domain.com"
```

然后重新运行生成脚本。

### 手动编辑文档

所有 `.mdx` 文件都是标准 Markdown 格式，可以直接编辑：

```bash
# 编辑某个文档
vim api-reference/video/get-videos-list.mdx
```

### 添加自定义内容

在 front matter 后面可以添加任意 Markdown 内容：

```mdx
---
title: '获取视频列表'
api: 'GET https://...'
description: '...'
---

这是接口的详细说明。

## 使用场景

1. 场景一：...
2. 场景二：...

## 注意事项

- 注意事项 1
- 注意事项 2

## 查询参数

...
```

---

## 验证文档

运行验证脚本检查文档完整性：

```bash
cd /Users/allen/Desktop
python3 verify_docs.py
```

---

## 部署到生产

### Mintlify Cloud

```bash
# 推送到 GitHub
git add .
git commit -m "Add API documentation"
git push

# 在 Mintlify Dashboard 中连接仓库
# https://mintlify.com/dashboard
```

### 自托管

```bash
# 构建静态文件
mintlify build

# 部署到任何静态托管服务
# (Vercel, Netlify, GitHub Pages 等)
```

---

## 技术信息

- **文档格式**: Mintlify MDX
- **源文件**: OpenAPI 3.0.1 JSON
- **总文档数**: 48 个
- **分类数**: 11 个
- **Base URL**: `https://dev-admin.beauty-666.com`
- **生成日期**: 2026-01-07

---

## 常见问题

**Q: 文档中有些描述显示"字段说明"？**
A: 这是因为原 OpenAPI JSON 中缺少详细描述。可以：
1. 在 OpenAPI JSON 中添加 description 后重新生成
2. 直接编辑 `.mdx` 文件手动添加

**Q: 如何更改导航顺序？**
A: 修改 `mint.json` 中 `navigation` 数组的顺序即可。

**Q: 可以添加认证说明吗？**
A: 可以！创建一个 `api-reference/authentication.mdx` 文件，然后在导航中引用。

**Q: 需要手动更新每个文档吗？**
A: 不需要。更新 OpenAPI JSON 后重新运行生成脚本即可。

---

## 下一步

1. [x] 生成文档
2. [ ] 集成到 Mintlify 项目
3. [ ] 自定义样式和主题
4. [ ] 添加认证说明
5. [ ] 添加使用指南
6. [ ] 部署到生产环境

---

## 需要帮助？

- Mintlify 文档: https://mintlify.com/docs
- OpenAPI 规范: https://swagger.io/specification/

---

生成时间: 2026-01-07
版本: 1.0.0
