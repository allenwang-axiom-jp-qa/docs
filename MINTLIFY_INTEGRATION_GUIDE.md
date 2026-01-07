# Mintlify API 文档集成指南

## 生成概览

已成功从 OpenAPI JSON 文件生成 **48 个** API 接口文档！

### 文档统计

- **系统接口**: 2 个
- **视频管理**: 4 个
- **会员配置**: 5 个
- **分类管理**: 10 个
- **标签管理**: 6 个
- **钻石配置**: 6 个
- **汇率配置**: 3 个
- **首页模块管理**: 4 个
- **模块子项管理**: 5 个
- **预览接口**: 1 个
- **推荐算法**: 2 个

---

## 目录结构

```
/Users/allen/Desktop/docs/api-reference/
├── algorithm/              # 推荐算法（2个文件）
├── categories/             # 分类管理（10个文件）
├── diamond/                # 钻石配置（6个文件）
├── exchange-rate/          # 汇率配置（3个文件）
├── homepage-items/         # 模块子项管理（5个文件）
├── homepage-modules/       # 首页模块管理（4个文件）
├── preview/                # 预览接口（1个文件）
├── system/                 # 系统接口（2个文件）
├── tags/                   # 标签管理（6个文件）
├── video/                  # 视频管理（4个文件）
├── vip/                    # 会员配置（5个文件）
├── generated_files.json    # 文件清单
└── navigation_config.json  # 导航配置
```

---

## 集成到 Mintlify 项目

### 步骤 1: 复制文档文件

将 `api-reference` 目录复制到你的 Mintlify 项目根目录：

```bash
cp -r /Users/allen/Desktop/docs/api-reference /path/to/your/mintlify-project/
```

### 步骤 2: 更新 mint.json 配置

打开你的 Mintlify 项目中的 `mint.json` 文件，在 `navigation` 部分添加以下内容：

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

或者，如果你希望平铺展示（不使用嵌套分组），可以参考 `navigation_config.json` 文件中的配置。

### 步骤 3: 验证文档

在 Mintlify 项目目录中运行：

```bash
mintlify dev
```

访问 `http://localhost:3000` 查看生成的 API 文档。

---

## 文档特性

### 1. 完整的参数说明
每个接口文档包含：
- **请求头**: Authorization token 说明
- **路径参数**: 动态路径参数（如 `{id}`）
- **查询参数**: URL 查询字符串参数
- **请求体**: POST/PUT 请求的 body 参数

### 2. 多语言请求示例
每个接口提供三种语言的请求示例：
- **cURL**: 命令行工具
- **JavaScript**: 使用 fetch API
- **Python**: 使用 requests 库

### 3. 响应示例
包含：
- **200/201 成功响应**: 标准成功响应格式
- **401 未授权**: 认证失败响应
- **400/404 错误响应**: 根据接口定义的错误响应

### 4. 响应字段说明
详细的响应字段文档，包括：
- 字段名称
- 字段类型
- 字段说明
- 嵌套字段展开（使用 Expandable 组件）

---

## 自定义配置

### Base URL
当前使用的 Base URL: `https://dev-admin.beauty-666.com`

如需修改，可以编辑生成脚本：
```python
BASE_URL = "https://your-new-domain.com"
```

然后重新运行：
```bash
python3 /Users/allen/Desktop/generate_mintlify_docs.py
```

### 重新生成文档

如果 OpenAPI 规范更新，只需重新运行生成脚本：

```bash
cd /Users/allen/Desktop
python3 generate_mintlify_docs.py
```

所有文档将被重新生成并覆盖。

---

## 文件清单

完整的文件列表保存在：
```
/Users/allen/Desktop/docs/api-reference/generated_files.json
```

该文件包含每个生成文件的：
- 文件路径
- API 分类
- 接口标题
- API 方法和路径

---

## 常见问题

### Q: 如何更新某个接口的文档？
A: 直接编辑对应的 `.mdx` 文件即可。所有文档都是标准的 Markdown 格式，支持手动修改。

### Q: 如何添加自定义说明？
A: 在每个 `.mdx` 文件的 front matter 之后，可以添加任意 Markdown 内容。

### Q: 如何调整导航顺序？
A: 修改 `mint.json` 中的 `navigation` 数组顺序即可。

### Q: 某些接口缺少描述怎么办？
A: 在 OpenAPI JSON 文件中添加 `description` 字段，然后重新生成文档。或者手动编辑 `.mdx` 文件。

---

## 技术支持

如有问题，请检查：
1. Mintlify 官方文档: https://mintlify.com/docs
2. OpenAPI 规范: https://swagger.io/specification/
3. 生成脚本: `/Users/allen/Desktop/generate_mintlify_docs.py`

---

生成时间: 2026-01-07
文档版本: 1.0.0
