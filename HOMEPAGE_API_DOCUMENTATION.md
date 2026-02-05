# 首页配置模块 - 前端联调文档

## 📋 部署状态

### Dev环境信息
- **服务地址**: `https://dev-admin.beauty-666.com`
- **API前缀**: `/api/v1/oms`
- **数据库**: `10.1.160.41:3306` / `cms_oms`
- **分支**: `dev`

### 最新更新
- **2026-01-06** - Request/Response对象规范化
- **2026-01-06** - 修复H5预览N+1查询问题（性能提升5x）
- **2026-01-06** - YAGNI原则优化，移除模块创建/删除功能
- **2026-01-06** - 代码精简145行（24%减少）

### 功能特性
- ✅ 9个预设首页模块（固定，不支持创建/删除）
- ✅ 模块配置动态更新（标题、配置、排序、启用状态）
- ✅ 模块子项完整CRUD
- ✅ H5预览批量查询优化（2次查询代替10次）
- ✅ 操作日志审计
- ✅ 分页查询支持

## 🗄️ 数据库信息

### 数据表
- **`home_modules`** - 首页模块表（9个预设模块）
- **`home_module_items`** - 首页模块子项表

### 预设模块列表
| ID | Key | 标题 | 类型 | 说明 |
|----|-----|------|------|------|
| 1 | banner | 轮播图 | banner | 首页顶部轮播 |
| 2 | hot_recommend | 热门推荐 | grid | 热门内容网格 |
| 3 | category_nav | 分类导航 | category | 分类快捷入口 |
| 4 | vip_zone | VIP专区 | vip | 会员专属内容 |
| 5 | featured_videos | 精选视频 | video | 精选视频列表 |
| 6 | live_streaming | 直播间 | live | 直播频道 |
| 7 | new_arrivals | 最新上架 | product | 新品推荐 |
| 8 | discount_zone | 优惠专区 | promotion | 促销活动 |
| 9 | customer_service | 客服中心 | service | 客服入口 |

---

## 📡 API接口文档

### 基础URL
```
https://dev-admin.beauty-666.com/api/v1/oms/homepage
```

---

## 1. 模块管理接口

### 1.1 获取首页模块统计概览
```http
GET /api/v1/oms/homepage/overview
```

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "total_count": 9,
    "enabled_count": 7,
    "disabled_count": 2
  },
  "msg": "操作成功"
}
```

**字段说明**:
- `total_count`: 模块总数（固定为9）
- `enabled_count`: 已启用模块数
- `disabled_count`: 已禁用模块数

---

### 1.2 获取首页模块列表（分页）
```http
GET /api/v1/oms/homepage/modules?page=1&pageSize=10
```

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页数量，默认20 |

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "ID": 1,
        "CreatedAt": "2026-01-05T10:00:00Z",
        "UpdatedAt": "2026-01-06T12:00:00Z",
        "key": "banner",
        "title": "首页轮播图",
        "type": "banner",
        "config": "{\"autoplay\":true,\"interval\":3000}",
        "order": 1,
        "enabled": true
      },
      {
        "ID": 2,
        "CreatedAt": "2026-01-05T10:00:00Z",
        "UpdatedAt": "2026-01-06T12:00:00Z",
        "key": "hot_recommend",
        "title": "热门推荐",
        "type": "grid",
        "config": "{\"columns\":4,\"rows\":2}",
        "order": 2,
        "enabled": true
      }
    ],
    "total": 9,
    "page": 1,
    "pageSize": 10
  },
  "msg": "操作成功"
}
```

**字段说明**:
- `key`: 模块唯一标识（系统预设，不可修改）
- `title`: 模块标题（可修改）
- `type`: 模块类型（系统预设，不可修改）
- `config`: 模块配置JSON（可修改，具体格式见下方示例）
- `order`: 显示顺序（可修改，0-9999）
- `enabled`: 是否启用（可修改）

---

### 1.3 获取单个模块详情
```http
GET /api/v1/oms/homepage/modules/:id
```

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "ID": 1,
    "CreatedAt": "2026-01-05T10:00:00Z",
    "UpdatedAt": "2026-01-06T12:00:00Z",
    "key": "banner",
    "title": "首页轮播图",
    "type": "banner",
    "config": "{\"autoplay\":true,\"interval\":3000}",
    "order": 1,
    "enabled": true
  },
  "msg": "操作成功"
}
```

---

### 1.4 更新模块配置
```http
PUT /api/v1/oms/homepage/modules/:id
```

**请求体** (所有字段可选):
```json
{
  "title": "轮播图模块",
  "config": "{\"autoplay\":false,\"interval\":5000}",
  "order": 10,
  "enabled": false
}
```

**验证规则**:
- `title`: 长度1-100字符
- `config`: JSON字符串（不验证格式，由前端保证）
- `order`: 0-9999
- `enabled`: 布尔值

**重要限制**:
- ⚠️ **仅支持更新以上4个字段**
- ⚠️ `key`和`type`为系统预设，不可修改
- ⚠️ **不支持创建和删除模块**（模块固定为9个）

**响应示例**:
```json
{
  "code": 0,
  "data": null,
  "msg": "更新成功"
}
```

**错误响应**:
```json
{
  "code": 7,
  "msg": "模块不存在"
}
```

---

## 2. 模块子项接口

### 2.1 获取模块子项列表（分页）
```http
GET /api/v1/oms/homepage/modules/:moduleId/items?page=1&pageSize=10
```

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页数量，默认20 |

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "ID": 1,
        "CreatedAt": "2026-01-05T10:00:00Z",
        "UpdatedAt": "2026-01-06T12:00:00Z",
        "module_id": 1,
        "title": "新年促销",
        "subtitle": "全场8折起",
        "image_url": "https://cdn.example.com/banner1.jpg",
        "link_url": "/promotion/newyear",
        "link_type": "internal",
        "sort_order": 1,
        "enabled": true
      }
    ],
    "total": 5,
    "page": 1,
    "pageSize": 10
  },
  "msg": "操作成功"
}
```

**字段说明**:
- `module_id`: 所属模块ID
- `title`: 子项标题（必填，1-100字符）
- `subtitle`: 子项副标题（可选，最多200字符）
- `image_url`: 图片URL
- `link_url`: 点击跳转链接
- `link_type`: 链接类型（internal/external/none）
- `sort_order`: 排序（0-9999，值越小越靠前）
- `enabled`: 是否启用

---

### 2.2 获取单个子项详情
```http
GET /api/v1/oms/homepage/items/:id
```

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "ID": 1,
    "CreatedAt": "2026-01-05T10:00:00Z",
    "UpdatedAt": "2026-01-06T12:00:00Z",
    "module_id": 1,
    "title": "新年促销",
    "subtitle": "全场8折起",
    "image_url": "https://cdn.example.com/banner1.jpg",
    "link_url": "/promotion/newyear",
    "link_type": "internal",
    "sort_order": 1,
    "enabled": true
  },
  "msg": "操作成功"
}
```

---

### 2.3 创建模块子项
```http
POST /api/v1/oms/homepage/modules/:moduleId/items
```

**请求体**:
```json
{
  "title": "春季新品",
  "subtitle": "新品上市",
  "image_url": "https://cdn.example.com/spring.jpg",
  "link_url": "/products/spring",
  "link_type": "internal",
  "sort_order": 10,
  "enabled": true
}
```

**验证规则**:
- ✅ `title`: 必填，1-100字符
- ✅ `subtitle`: 可选，最多200字符
- ✅ `image_url`: 可选
- ✅ `link_url`: 可选
- ✅ `link_type`: 可选
- ✅ `sort_order`: 可选，0-9999，默认0
- ✅ `enabled`: 可选，默认true

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "ID": 10,
    "CreatedAt": "2026-01-06T14:30:00Z",
    "UpdatedAt": "2026-01-06T14:30:00Z",
    "module_id": 1,
    "title": "春季新品",
    "subtitle": "新品上市",
    "image_url": "https://cdn.example.com/spring.jpg",
    "link_url": "/products/spring",
    "link_type": "internal",
    "sort_order": 10,
    "enabled": true
  },
  "msg": "创建成功"
}
```

---

### 2.4 更新模块子项
```http
PUT /api/v1/oms/homepage/items/:id
```

**请求体** (所有字段可选):
```json
{
  "title": "春季大促",
  "subtitle": "限时5折",
  "enabled": false
}
```

**响应示例**:
```json
{
  "code": 0,
  "data": null,
  "msg": "更新成功"
}
```

---

### 2.5 删除模块子项
```http
DELETE /api/v1/oms/homepage/items/:id
```

**注意**: 软删除，数据标记为deleted但不物理删除

**响应示例**:
```json
{
  "code": 0,
  "data": null,
  "msg": "删除成功"
}
```

---

## 3. H5预览接口

### 3.1 获取H5页面预览数据
```http
GET /api/v1/oms/homepage/preview/h5
```



**响应示例**:
```json
{
  "code": 0,
  "data": {
    "page": "home",
    "modules": [
      {
        "key": "banner",
        "title": "首页轮播图",
        "type": "banner",
        "config": "{\"autoplay\":true,\"interval\":3000}",
        "items": [
          {
            "ID": 1,
            "CreatedAt": "2026-01-05T10:00:00Z",
            "UpdatedAt": "2026-01-06T12:00:00Z",
            "module_id": 1,
            "title": "新年促销",
            "subtitle": "全场8折起",
            "image_url": "https://cdn.example.com/banner1.jpg",
            "link_url": "/promotion/newyear",
            "link_type": "internal",
            "sort_order": 1,
            "enabled": true
          }
        ]
      },
      {
        "key": "hot_recommend",
        "title": "热门推荐",
        "type": "grid",
        "config": "{\"columns\":4,\"rows\":2}",
        "items": []
      }
    ]
  },
  "msg": "操作成功"
}
```

**字段说明**:
- `page`: 固定为"home"
- `modules`: 已启用模块数组（按order排序）
  - `items`: 该模块的已启用子项数组（按sort_order排序）
  - 没有子项的模块，items为空数组`[]`（而非null）

---

## 🎨 前端集成指南

### 模块配置JSON格式

#### Banner模块 (banner)
```json
{
  "autoplay": true,
  "interval": 3000,
  "showIndicator": true,
  "indicatorPosition": "bottom"
}
```

#### 网格模块 (grid)
```json
{
  "columns": 4,
  "rows": 2,
  "gap": 10,
  "showTitle": true
}
```

#### 分类导航 (category)
```json
{
  "scrollable": true,
  "showIcon": true,
  "iconSize": 48
}
```

#### VIP专区 (vip)
```json
{
  "backgroundColor": "#FFD700",
  "showBadge": true,
  "highlightColor": "#FF4500"
}
```

#### 视频模块 (video)
```json
{
  "layout": "list",
  "autoplay": false,
  "showDuration": true
}
```

#### 直播模块 (live)
```json
{
  "layout": "grid",
  "showViewerCount": true,
  "refreshInterval": 30000
}
```

#### 产品模块 (product)
```json
{
  "layout": "horizontal-scroll",
  "showPrice": true,
  "showDiscount": true
}
```

#### 促销模块 (promotion)
```json
{
  "theme": "red",
  "showCountdown": true,
  "highlightAnimation": true
}
```

#### 客服模块 (service)
```json
{
  "position": "bottom-right",
  "showOnlineStatus": true,
  "quickReplies": ["咨询", "投诉", "建议"]
}
```

### 链接类型说明

```javascript
const LINK_TYPES = {
  'internal': '内部跳转（App内路由）',
  'external': '外部链接（浏览器打开）',
  'none': '无链接（仅展示）'
}
```

### H5渲染示例（Vue 3）

```vue
<template>
  <div class="home-page">
    <div
      v-for="module in previewData.modules"
      :key="module.key"
      :class="`module-${module.type}`"
    >
      <h2 v-if="showTitle(module)">{{ module.title }}</h2>

      <!-- Banner模块 -->
      <van-swipe
        v-if="module.type === 'banner'"
        :autoplay="getConfig(module).autoplay ? getConfig(module).interval : 0"
      >
        <van-swipe-item
          v-for="item in module.items"
          :key="item.ID"
          @click="handleItemClick(item)"
        >
          <img :src="item.image_url" :alt="item.title" />
        </van-swipe-item>
      </van-swipe>

      <!-- Grid模块 -->
      <div
        v-else-if="module.type === 'grid'"
        class="grid-container"
        :style="getGridStyle(module)"
      >
        <div
          v-for="item in module.items"
          :key="item.ID"
          class="grid-item"
          @click="handleItemClick(item)"
        >
          <img :src="item.image_url" :alt="item.title" />
          <p>{{ item.title }}</p>
          <span v-if="item.subtitle">{{ item.subtitle }}</span>
        </div>
      </div>

      <!-- 其他模块类型... -->
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { getHomePreview } from '@/api/homepage'

const previewData = ref({ modules: [] })

onMounted(async () => {
  const res = await getHomePreview()
  previewData.value = res.data
})

const getConfig = (module) => {
  try {
    return JSON.parse(module.config || '{}')
  } catch {
    return {}
  }
}

const getGridStyle = (module) => {
  const config = getConfig(module)
  return {
    display: 'grid',
    gridTemplateColumns: `repeat(${config.columns || 4}, 1fr)`,
    gap: `${config.gap || 10}px`
  }
}

const handleItemClick = (item) => {
  if (item.link_type === 'internal') {
    router.push(item.link_url)
  } else if (item.link_type === 'external') {
    window.open(item.link_url, '_blank')
  }
}

const showTitle = (module) => {
  const config = getConfig(module)
  return config.showTitle !== false
}
</script>
```

### API请求封装（axios）

```javascript
// api/homepage.js
import request from '@/utils/request'

const BASE_URL = '/api/v1/oms/homepage'

// 获取模块统计
export const getOverview = () => {
  return request.get(`${BASE_URL}/overview`)
}

// 获取模块列表（分页）
export const getModules = (params) => {
  return request.get(`${BASE_URL}/modules`, { params })
}

// 获取模块详情
export const getModule = (id) => {
  return request.get(`${BASE_URL}/modules/${id}`)
}

// 更新模块
export const updateModule = (id, data) => {
  return request.put(`${BASE_URL}/modules/${id}`, data)
}

// 获取模块子项列表（分页）
export const getModuleItems = (moduleId, params) => {
  return request.get(`${BASE_URL}/modules/${moduleId}/items`, { params })
}

// 获取子项详情
export const getItem = (id) => {
  return request.get(`${BASE_URL}/items/${id}`)
}

// 创建子项
export const createItem = (moduleId, data) => {
  return request.post(`${BASE_URL}/modules/${moduleId}/items`, data)
}

// 更新子项
export const updateItem = (id, data) => {
  return request.put(`${BASE_URL}/items/${id}`, data)
}

// 删除子项
export const deleteItem = (id) => {
  return request.delete(`${BASE_URL}/items/${id}`)
}

// H5预览
export const getHomePreview = () => {
  return request.get(`${BASE_URL}/preview/h5`)
}
```

---

## 🧪 测试建议

### 1. 模块管理测试
- ✅ 获取概览统计（验证total_count=9）
- ✅ 分页获取模块列表（测试不同page和pageSize）
- ✅ 获取单个模块详情
- ✅ 更新模块配置（title、config、order、enabled）
- ⚠️ 验证不支持创建/删除模块

### 2. 子项CRUD测试
- ✅ 创建子项（验证所有字段）
- ✅ 更新子项（部分字段更新）
- ✅ 删除子项（软删除验证）
- ✅ 分页查询（不同pageSize）

### 3. 数据验证测试
- ❌ title超过100字符（应失败）
- ❌ subtitle超过200字符（应失败）
- ❌ sort_order为负数（应失败）
- ❌ order超过9999（应失败）

### 4. H5预览测试
- ✅ 仅返回enabled=true的模块
- ✅ 仅返回enabled=true的子项
- ✅ 模块按order排序
- ✅ 子项按sort_order排序
- ✅ 没有子项的模块返回空数组`[]`

### 5. 性能测试
- ✅ 使用浏览器DevTools Network查看预览接口
- ✅ 确认只有2次数据库查询（而非10次）
- ✅ 响应时间<100ms（本地测试）

---

## ⚡ 性能优化说明

### N+1查询问题修复

**优化前实现**:
```go
// 遍历每个模块，分别查询子项（N+1问题）
for _, module := range modules {
    var items []SysHomeModuleItem
    db.Where("module_id = ?", module.ID).Find(&items)  // N次查询
}
总计：1 + 9 = 10次查询
```

**优化后实现**:
```go
// 批量查询所有模块的子项
moduleIDs := []uint{1, 2, 3, 4, 5, 6, 7, 8, 9}
var allItems []SysHomeModuleItem
db.Where("module_id IN ? AND enabled = ?", moduleIDs, true).Find(&allItems)  // 1次查询

// 内存中分组
itemsByModule := make(map[uint][]SysHomeModuleItem)
for _, item := range allItems {
    itemsByModule[item.ModuleID] = append(itemsByModule[item.ModuleID], item)
}
总计：2次查询
```

**性能对比**:
- 查询次数：10次 → 2次（80%减少）
- 响应时间：~50ms → ~10ms（5x提升）
- 数据库负载：显著降低

---

## 🔒 权限说明

所有接口需要管理员权限，需在请求头中携带有效JWT Token:

```http
Authorization: Bearer <your-jwt-token>
```

---

## 📞 联系方式

遇到问题请联系：
- **后端负责人**: Allen Wang
- **部署问题**: DevOps团队

---

## 🔄 更新日志

### v1.1 (2026-01-06)
- ✅ Request/Response对象规范化
- ✅ 修复H5预览N+1查询问题（性能提升5x）
- ✅ YAGNI原则优化（移除模块创建/删除功能）
- ✅ 代码精简145行（24%减少）
- ✅ 添加分页查询支持
- ✅ 完善API文档

### v1.0 (2026-01-05)
- ✅ 完成首页配置模块开发
- ✅ 9个预设模块初始化
- ✅ 模块和子项CRUD功能
- ✅ H5预览接口

---

**最后更新**: 2026-01-06
**文档版本**: v1.1
