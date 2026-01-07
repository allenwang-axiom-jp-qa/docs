# Mintlify 文档平台设置指南

本指南将帮助你使用 Mintlify 管理和展示 CMS OMS 的 API 文档。

## 什么是 Mintlify？

Mintlify 是一个现代化的文档平台，可以将 Markdown 文件转换为美观、交互式的文档网站。它特别适合 API 文档，提供了：

- 🎨 美观的界面和主题
- 📱 响应式设计
- 🔍 强大的搜索功能
- 📊 代码示例和交互式组件
- 🚀 快速的页面加载
- 🔗 自动生成的导航和目录

## 已完成的设置

我已经为你完成了以下配置：

### 1. Mintlify 配置文件

- **文件**: `mint.json`
- **内容**: 包含项目名称、导航结构、主题颜色等配置

### 2. 文档结构

创建了以下文档文件：

```
cms-site-svc-oms/
├── mint.json                          # Mintlify 配置
├── introduction.mdx                   # 首页介绍
├── quickstart.mdx                     # 快速开始
├── authentication.mdx                 # 认证说明
├── api-reference/
│   ├── overview.mdx                   # API 总览
│   ├── homepage/
│   │   ├── overview.mdx               # Homepage API 概览
│   │   ├── modules.mdx                # 模块管理接口
│   │   ├── module-items.mdx           # 模块子项接口
│   │   └── preview.mdx                # H5 预览接口
│   └── diamond/
│       ├── overview.mdx               # Diamond API 概览
│       ├── packages.mdx               # 钻石套餐接口
│       └── exchange-rate.mdx          # 汇率管理接口
└── deployment/
    ├── setup.mdx                      # 部署设置
    └── ci-cd.mdx                      # CI/CD 说明
```

## 本地预览

### 安装 Mintlify CLI

```bash
npm install -g mintlify
```

### 启动本地预览服务器

在项目根目录运行：

```bash
cd /Users/allen/Desktop/cms-site-svc-oms
mintlify dev
```

这将启动一个本地开发服务器，默认地址为 `http://localhost:3000`

### 预览效果

打开浏览器访问 `http://localhost:3000`，你将看到：

- 📖 左侧导航栏：包含所有文档页面
- 📄 主内容区：当前页面内容
- 📑 右侧目录：当前页面的标题导航
- 🔍 搜索栏：快速查找文档

## 修改文档

### 编辑现有页面

1. 找到对应的 `.mdx` 文件
2. 使用任何文本编辑器编辑
3. 保存后，本地预览会自动刷新

### 添加新页面

1. 创建新的 `.mdx` 文件
2. 在 `mint.json` 的 `navigation` 部分添加页面路径
3. 示例：

```json
{
  "group": "你的分组",
  "pages": [
    "path/to/new-page"
  ]
}
```

### Markdown 增强语法

Mintlify 支持特殊的 MDX 组件：

#### Card 组件

```mdx
<Card title="标题" icon="icon-name" href="/link">
  描述文字
</Card>
```

#### CardGroup 组件

```mdx
<CardGroup cols={2}>
  <Card title="卡片1" icon="icon1">内容1</Card>
  <Card title="卡片2" icon="icon2">内容2</Card>
</CardGroup>
```

#### 代码示例

```mdx
<CodeGroup>

```bash cURL
curl https://api.example.com
```

```javascript JavaScript
fetch('https://api.example.com')
```

</CodeGroup>
```

#### 提示框

```mdx
<Info>这是一个信息提示</Info>
<Warning>这是一个警告</Warning>
<Check>这是一个成功提示</Check>
```

#### API 请求/响应示例

```mdx
<RequestExample>
```bash cURL
curl https://api.example.com
```
</RequestExample>

<ResponseExample>
```json Response
{
  "success": true
}
```
</ResponseExample>
```

## 部署到线上

### 方式 1：Mintlify 托管（推荐）

1. 访问 [mintlify.com](https://mintlify.com)
2. 使用 GitHub 账号登录
3. 连接你的 GitHub 仓库
4. Mintlify 会自动检测 `mint.json`
5. 每次推送到 GitHub，文档会自动更新

**优点**:
- 免费托管
- 自动部署
- CDN 加速
- 自定义域名支持

### 方式 2：自己构建部署

```bash
# 构建静态文件
mintlify build

# 输出在 _site 目录
# 可以部署到任何静态网站托管服务
```

### 方式 3：集成到现有网站

如果你想将文档集成到现有的网站中，可以：

1. 使用 iframe 嵌入
2. 使用 Mintlify 的 API 获取内容
3. 将生成的静态文件托管在子路径

## 自定义配置

### 修改主题颜色

编辑 `mint.json`：

```json
{
  "colors": {
    "primary": "#你的主色",
    "light": "#浅色主题的主色",
    "dark": "#深色主题的主色"
  }
}
```

### 添加 Logo

1. 将 logo 图片放在项目根目录的 `logo/` 文件夹
2. 在 `mint.json` 中配置：

```json
{
  "logo": {
    "light": "/logo/light.svg",
    "dark": "/logo/dark.svg"
  }
}
```

### 添加自定义链接

在 `mint.json` 中：

```json
{
  "topbarLinks": [
    {
      "name": "GitHub",
      "url": "https://github.com/your-repo"
    }
  ]
}
```

## 常见问题

### 1. 本地预览报错？

确保已安装 Node.js (v14+) 和 Mintlify CLI：

```bash
node --version
npm install -g mintlify
```

### 2. 页面不显示？

检查 `mint.json` 中的 navigation 配置，确保路径正确。

### 3. 图片不显示？

图片路径应该相对于项目根目录，或使用绝对 URL。

### 4. 如何添加搜索功能？

Mintlify 自动提供搜索功能，无需配置。

## 下一步

### 推荐操作

1. **本地预览**: 运行 `mintlify dev` 查看效果
2. **自定义品牌**: 添加你的 logo 和颜色
3. **完善内容**: 根据实际 API 调整文档内容
4. **部署上线**: 连接 Mintlify 或部署到你的服务器

### 维护文档

- 每次 API 更新时，更新对应的 `.mdx` 文件
- 定期检查链接是否有效
- 添加更多代码示例和使用场景
- 收集用户反馈改进文档

## 参考资源

- [Mintlify 官方文档](https://mintlify.com/docs)
- [MDX 语法指南](https://mdxjs.com/)
- [Mintlify 组件库](https://mintlify.com/docs/components)
- [图标库](https://fontawesome.com/icons)

## 获取帮助

如果遇到问题：

1. 查看 [Mintlify 官方文档](https://mintlify.com/docs)
2. 访问 [Mintlify Discord 社区](https://discord.gg/mintlify)
3. 查看 [GitHub Issues](https://github.com/mintlify/mint/issues)

---

**祝你文档编写愉快！** 🎉
