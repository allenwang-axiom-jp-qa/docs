# 内容管理后台 API 文档

--版本历史
> **版本:** V3.1
> **更新时间:** 2026-01-02
> **变更说明:** 添加视频管理接口，统一为"内容管理"模块

### V2.0 (2026-01-02)
- ✅ 添加完整的视频管理API文档
- ✅ 统一为"内容管理"模块，包含视频管理、标签管理、分类管理
- ✅ 更新基础URL为实际部署的域名
- ✅ 添加视频状态管理流程说明
- ✅ 补充收费类型和价格验证说明
- ✅ 完善错误码和注意事项

### V1.0 (2025-12-31)
- 更新实际部署的API路径（`/api/v1/site-svc-video-mgmt`）
- 更新统一响应格式（`code/data/msg`）
- 添加完整的请求示例（基于 dev 环境）
- 补充实际测试数据
- 修正字段类型和默认值说明

### V0.9 (2025-12-23)
- 初始版本
- 标签管理和分类管理API



## 概述

本文档描述内容管理后台的所有API接口，包含三个核心模块：
1. **视频管理** - 视频列表、视频详情、视频状态管理、视频信息编辑
2. **标签管理** - 标签的增删改查、排序、统计
3. **分类管理** - 分类的增删改查、树形结构、拖拽排序

**基础信息:**
- **基础URL (开发环境):** `https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt`
- **基础URL (生产环境):** `https://admin.beauty-666.com/api/v1/site-svc-video-mgmt`
- 所有接口返回JSON格式
- 统一响应格式（见下方）

## 统一响应格式

所有API采用统一的响应格式：

### 成功响应
```json
{
  "code": 0,
  "data": { ... },
  "msg": "获取成功"
}
```

### 错误响应
```json
{
  "code": 错误码,
  "data": null,
  "msg": "错误描述"
}
```

**字段说明:**
- `code`: 状态码（0=成功，其他为错误码）
- `data`: 响应数据（成功时包含实际数据，失败时为null）
- `msg`: 提示信息

---

## 数据模型

### Video（视频）

```json
{
  "id": 30,
  "product_id": "V261292255729101",
  "cover_path": "https://cdn.example.com/covers/xxx.jpg",
  "title": "视频标题示例",
  "category_id": 1,
  "category_name": "电影",
  "tags": [
    { "id": 1, "name": "动作" },
    { "id": 4, "name": "热门推荐" }
  ],
  "status": 3,
  "status_text": "上架",
  "charge_type": "paid",
  "charge_text": "付费",
  "view_count": 12570,
  "like_count": 2300,
  "price": 9.9,
  "created_time": 1704067200
}
```

**字段说明:**
- `id`: 视频ID（数据库自增ID）
- `product_id`: 产品唯一标识符（格式: V + 数字）
- `cover_path`: 封面图片路径/URL
- `title`: 视频标题
- `category_id`: 分类ID
- `category_name`: 分类名称
- `tags`: 标签列表（包含ID和名称）
- `status`: 状态码（-1=上传失败, 0=上传中, 1=转码中, 2=待审核, 3=上架, 4=已拒绝, 5=下架）
- `status_text`: 状态文字描述
- `charge_type`: 收费类型（free=免费, paid=付费, vip=VIP专享）
- `charge_text`: 收费类型文字描述
- `view_count`: 播放次数
- `like_count`: 点赞次数
- `price`: 价格（仅付费视频有值）
- `created_time`: 创建时间戳（Unix timestamp，秒）

### Tag（标签）

```json
{
  "id": 1,
  "tagId": "TAG001",
  "name": "动作",
  "identifier": "action",
  "tagType": 1,
  "useCount": 120,
  "status": 1,
  "sortOrder": 10,
  "description": "动作类型视频",
  "createdTime": 1767117227
}
```

**字段说明:**
- `id`: 数据库自增ID
- `tagId`: 业务唯一标识符（格式: TAG001, TAG002...）
- `name`: 标签名称
- `identifier`: 标签标识符（用于URL和API，如 "action", "comedy"）
- `tagType`: 标签类型（0=普通类型, 1=热门类型）
- `useCount`: 使用次数
- `status`: 状态（0=禁用, 1=启用）
- `sortOrder`: 排序顺序（数字越小越靠前）
- `description`: 描述
- `createdTime`: 创建时间戳（Unix timestamp）

### Category（分类）

```json
{
  "id": 1,
  "categoryId": "CAT001",
  "name": "电影",
  "identifier": "movies",
  "parentId": null,
  "sortOrder": 10,
  "status": 1,
  "description": "电影分类",
  "videoCount": 0,
  "createdTime": 1767115587
}
```

**字段说明:**
- `id`: 数据库自增ID
- `categoryId`: 业务唯一标识符（格式: CAT001, CAT002...）
- `name`: 分类名称
- `identifier`: 分类标识符（用于URL和API）
- `parentId`: 父分类ID（null表示顶级分类）
- `sortOrder`: 排序顺序
- `status`: 状态（0=禁用, 1=启用）
- `description`: 描述
- `videoCount`: 该分类下的视频数量
- `createdTime`: 创建时间戳

---

## 一、视频管理 API

### 1.1 获取视频列表

**GET** `/videos/list`

获取视频列表，支持分页、状态筛选、分类筛选和关键词搜索。

**查询参数:**
- `page` (int): 页码，默认1
- `page_size` (int): 每页数量，默认20
- `status` (int): 状态筛选（可选）
  - `-1`: 上传失败
  - `0`: 上传中
  - `1`: 转码中
  - `2`: 待审核
  - `3`: 上架
  - `4`: 已拒绝
  - `5`: 下架
- `category_id` (uint64): 分类ID筛选（可选）
- `tag_ids` ([]uint64): 标签ID数组筛选（可选，支持多个标签）
- `search` (string): 搜索关键词（可选），支持按标题、产品ID、标签名称搜索

**请求示例:**
```bash
# 获取第一页视频
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/list?page=1&page_size=10'

# 按状态筛选（只看上架的视频）
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/list?status=3&page=1&page_size=10'

# 按分类筛选
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/list?category_id=1&page=1&page_size=10'

# 关键词搜索（需要URL编码）
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/list?search=%E6%B5%8B%E8%AF%95&page=1&page_size=10'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "total": 30,
    "page": 1,
    "page_size": 10,
    "list": [
      {
        "id": 30,
        "product_id": "V261292255729101",
        "cover_path": "https://cdn.example.com/covers/xxx.jpg",
        "title": "视频标题示例1",
        "category_id": 1,
        "category_name": "电影",
        "tags": [
          { "id": 1, "name": "动作" },
          { "id": 4, "name": "热门推荐" }
        ],
        "status": 3,
        "status_text": "上架",
        "charge_type": "paid",
        "charge_text": "付费",
        "view_count": 12570,
        "like_count": 2300,
        "price": 9.9,
        "created_time": 1704067200
      },
      {
        "id": 29,
        "product_id": "V12346",
        "cover_path": "",
        "title": "免费视频示例",
        "category_id": 2,
        "category_name": "电视剧",
        "tags": [
          { "id": 2, "name": "喜剧" },
          { "id": 6, "name": "编辑精选" }
        ],
        "status": 2,
        "status_text": "待审核",
        "charge_type": "free",
        "charge_text": "免费",
        "view_count": 0,
        "like_count": 0,
        "price": 0,
        "created_time": 1704066000
      }
    ]
  },
  "msg": ""
}
```

**字段说明:**
- `total`: 符合条件的视频总数
- `page`: 当前页码
- `page_size`: 每页数量
- `list`: 视频列表数组

---

### 1.2 获取视频状态

**GET** `/videos/status`

获取指定视频的当前状态和转码进度信息。

**查询参数:**
- `video_id` (uint64): 视频ID（必填）

**请求示例:**
```bash
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/status?video_id=30'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "video_id": 30,
    "status": 3,
    "status_text": "上架",
    "transcode_progress": 100,
    "error_message": ""
  },
  "msg": "获取成功"
}
```

**字段说明:**
- `video_id`: 视频ID
- `status`: 当前状态码
- `status_text`: 状态文字描述
- `transcode_progress`: 转码进度（0-100）
- `error_message`: 错误信息（如果有）

---

### 1.3 更新视频状态（支持单个和批量）

**PUT** `/videos/status`

更新视频的审核状态（如通过审核、拒绝、上架、下架）。支持单个和批量操作。

**请求体（JSON或表单）:**
```json
{
  "video_ids": [30, 31, 32],
  "status": 2
}
```

**字段说明:**
- `video_ids` ([]uint64, 必填): 视频ID数组（至少一个）
- `status` (int, 必填): 目标状态
  - `-2`: 下架
  - `-1`: 拒绝
  - `1`: 通过审核
  - `2`: 上架

**状态转换逻辑:**
- 传入 `1`（通过）→ 数据库更新为 `3`（已上架）
- 传入 `2`（上架）→ 数据库更新为 `3`（已上架）
- 传入 `-1`（拒绝）→ 数据库更新为 `4`（已拒绝）
- 传入 `-2`（下架）→ 数据库更新为 `5`（已下架）

**请求示例:**
```bash
# 单个视频上架
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/status' \
  -H "Content-Type: application/json" \
  -d '{"video_ids":[30],"status":2}'

# 批量上架多个视频
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/status' \
  -H "Content-Type: application/json" \
  -d '{"video_ids":[30,31,32],"status":2}'

# 批量审核通过
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/status' \
  -H "Content-Type: application/json" \
  -d '{"video_ids":[30,31,32],"status":1}'

# 批量拒绝审核
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/status' \
  -H "Content-Type: application/json" \
  -d '{"video_ids":[30,31],"status":-1}'

# 批量下架视频
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/status' \
  -H "Content-Type: application/json" \
  -d '{"video_ids":[30,31,32],"status":-2}'
```

**响应示例（批量操作 - 全部成功）:**
```json
{
  "code": 0,
  "data": {
    "results": [
      {
        "video_id": 30,
        "success": true,
        "status": 3,
        "message": "狀態更新成功"
      },
      {
        "video_id": 31,
        "success": true,
        "status": 3,
        "message": "狀態更新成功"
      },
      {
        "video_id": 32,
        "success": true,
        "status": 3,
        "message": "狀態更新成功"
      }
    ],
    "success_count": 3,
    "fail_count": 0,
    "message": "所有影片狀態更新成功（3/3）"
  },
  "msg": ""
}
```

**响应示例（批量操作 - 部分失败）:**
```json
{
  "code": 0,
  "data": {
    "results": [
      {
        "video_id": 30,
        "success": true,
        "status": 3,
        "message": "狀態更新成功"
      },
      {
        "video_id": 999,
        "success": false,
        "error": "影片不存在"
      },
      {
        "video_id": 32,
        "success": true,
        "status": 3,
        "message": "狀態更新成功"
      }
    ],
    "success_count": 2,
    "fail_count": 1,
    "message": "部分影片狀態更新成功（2/3），失敗（1/3）"
  },
  "msg": ""
}
```

**字段说明:**
- `results`: 每个视频的更新结果数组
  - `video_id`: 视频ID
  - `success`: 是否成功
  - `status`: 更新后的状态（成功时返回）
  - `message`: 提示信息（成功时返回）
  - `error`: 错误信息（失败时返回）
- `success_count`: 成功更新的视频数量
- `fail_count`: 失败的视频数量
- `message`: 总体提示信息

---

### 1.4 更新视频信息

**PUT** `/videos`

更新视频的基本信息（标题、描述、分类、标签、收费类型、封面等）。

**请求方式:** `multipart/form-data`（支持文件上传）或 `application/x-www-form-urlencoded`

**请求参数:**
- `video_id` (uint64, 必填): 视频ID
- `title` (string, 可选): 视频标题
- `description` (string, 可选): 视频描述
- `category_id` (uint64, 可选): 分类ID
- `tag_ids` (string, 可选): 标签ID数组，逗号分隔（如 "1,2,3"）
- `charge_type` (string, 可选): 收费类型
  - `free`: 免费
  - `paid`: 付费
  - `vip`: VIP专享
- `price` (float64, 可选): 价格（当 `charge_type` 为 `paid` 时必填，最多两位小数）
- `video_type` (int, 可选): 视频类型（1=短片, 2=长片）
- `cover` (file, 可选): 封面图片文件

**请求示例:**
```bash
# 更新视频基本信息（JSON格式）
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos' \
  -H "Content-Type: application/json" \
  -d '{
    "video_id": 30,
    "title": "更新后的标题",
    "description": "更新后的描述",
    "category_id": 1,
    "tag_ids": "1,4,5",
    "charge_type": "paid",
    "price": 19.9
  }'

# 更新封面图片（multipart/form-data）
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos' \
  -F "video_id=30" \
  -F "title=新标题" \
  -F "cover=@/path/to/cover.jpg"

# 修改为免费视频
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos' \
  -H "Content-Type: application/json" \
  -d '{"video_id":30,"charge_type":"free"}'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "video_id": 30,
    "cover_path": "https://cdn.example.com/covers/new_cover.jpg",
    "message": "视频信息更新成功"
  },
  "msg": "更新成功"
}
```

**注意事项:**
1. 所有字段都是可选的，只传需要更新的字段
2. 当 `charge_type` 为 `paid` 时，`price` 必填且必须大于0
3. `price` 最多只能有两位小数
4. `tag_ids` 是逗号分隔的字符串，不是数组
5. 如果上传了封面，响应中会返回新的 `cover_path`

---

## 二、标签管理 API

### 2.1 获取标签列表

**GET** `/tags`

**查询参数:**
- `page` (int32): 页码，默认1
- `pageSize` (int32): 每页数量，默认20
- `keyword` (string): 搜索关键词（可选），支持按名称、标识符、描述模糊查询
- `tagType` (int32): 标签类型筛选（可选，不传=全部，0=普通类型，1=热门类型）
- `status` (int32): 状态筛选（可选，不传=仅启用，0=禁用，1=启用）

**请求示例:**
```bash
# 获取第一页标签
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags?page=1&pageSize=10'

# 按类型筛选热门标签
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags?tagType=1&page=1&pageSize=10'

# 搜索关键词（需要URL编码）
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags?keyword=%E5%8A%A8%E4%BD%9C&page=1&pageSize=10'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "total": 5,
    "page": 1,
    "list": [
      {
        "id": 4,
        "tagId": "TAG004",
        "name": "热门推荐",
        "identifier": "hot-recommend",
        "tagType": 1,
        "useCount": 0,
        "status": 1,
        "sortOrder": 1,
        "description": "热门推荐视频",
        "createdTime": 1767117227
      },
      {
        "id": 1,
        "tagId": "TAG001",
        "name": "动作",
        "identifier": "action",
        "tagType": 0,
        "useCount": 0,
        "status": 1,
        "sortOrder": 1,
        "description": "动作类型视频",
        "createdTime": 1767117227
      }
    ]
  },
  "msg": "获取成功"
}
```

---

### 2.2 获取单个标签

**GET** `/tag`

**查询参数:**
- `tagId` (uint64): 标签ID（必填）

**请求示例:**
```bash
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag?tagId=1'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "id": 1,
    "tagId": "TAG001",
    "name": "动作",
    "identifier": "action",
    "tagType": 0,
    "useCount": 0,
    "status": 1,
    "sortOrder": 1,
    "description": "动作类型视频",
    "createdTime": 1767117227
  },
  "msg": "获取成功"
}
```

---

### 2.3 创建标签

**POST** `/tag`

**请求体:**
```json
{
  "name": "科幻",
  "identifier": "sci-fi",
  "tagType": 0,
  "status": 1,
  "description": "科幻类型视频"
}
```

**字段说明:**
- `name` (string, 必填): 标签名称
- `identifier` (string, 必填): 标签标识符
- `tagType` (int32): 标签类型，0=普通类型, 1=热门类型，默认0
- `status` (int32): 状态，0=禁用, 1=启用，默认1
- `description` (string): 描述
- **注意**: `sortOrder` 由后端自动生成，前端不需要传递

**请求示例:**
```bash
curl -X POST 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag' \
  -H "Content-Type: application/json" \
  -d '{"name":"科幻","identifier":"sci-fi","tagType":0,"status":1,"description":"科幻类型视频"}'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "tagId": 6
  },
  "msg": "创建成功"
}
```

---

### 2.4 更新标签

**PUT** `/tag`

**查询参数:**
- `tagId` (uint64): 标签ID（必填）

**请求体:**
```json
{
  "name": "动作片",
  "identifier": "action-movies",
  "tagType": 1,
  "status": 1,
  "sortOrder": 20,
  "description": "更新后的描述"
}
```

**字段说明:** 所有字段都是可选的，只传需要更新的字段

**请求示例:**
```bash
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag?tagId=1' \
  -H "Content-Type: application/json" \
  -d '{"name":"动作片","tagType":1}'
```

**响应示例:**
```json
{
  "code": 0,
  "data": null,
  "msg": "更新成功"
}
```

---

### 2.5 删除标签

**DELETE** `/tag`

**查询参数:**
- `tagId` (uint64): 标签ID（必填）

**说明:** 软删除，将status设置为0

**请求示例:**
```bash
curl -X DELETE 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag?tagId=1'
```

**响应示例:**
```json
{
  "code": 0,
  "data": null,
  "msg": "删除成功"
}
```

---

### 2.6 批量重排标签顺序

**PUT** `/tags/reorder`

**请求体:**
```json
{
  "tagIds": [3, 1, 2, 5, 4]
}
```

**字段说明:**
- `tagIds` ([]uint64, 必填): 标签ID数组，按期望的顺序排列

**说明:**
- `tagIds` 是数据库自增ID（不是 TAG001 这种业务ID）
- 系统会按数组顺序重新计算排序值（10, 20, 30...）

**请求示例:**
```bash
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags/reorder' \
  -H "Content-Type: application/json" \
  -d '{"tagIds":[3,1,2,5,4]}'
```

**响应示例:**
```json
{
  "code": 0,
  "data": null,
  "msg": "重排成功"
}
```

---

### 2.7 获取标签统计

**GET** `/tags/stats`

**请求示例:**
```bash
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags/stats'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "total": 5,
    "normalType": 3,
    "hotType": 2,
    "enabled": 5,
    "totalVideos": 0
  },
  "msg": "获取成功"
}
```

**字段说明:**
- `total`: 总标签数
- `normalType`: 普通类型标签数（tagType=0）
- `hotType`: 热门类型标签数（tagType=1）
- `enabled`: 启用的标签数
- `totalVideos`: 标签关联的总视频数

---

## 三、分类管理 API

### 3.1 获取分类列表

**GET** `/categories`

**查询参数:**
- `page` (int32): 页码，默认1
- `pageSize` (int32): 每页数量，默认20
- `parentId` (uint64): 父分类ID（可选，0=顶级分类）
- `status` (int32): 状态筛选（可选，1=仅启用）

**请求示例:**
```bash
# 获取所有分类
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/categories?page=1&pageSize=10'

# 获取顶级分类
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/categories?parentId=0&page=1&pageSize=10'

# 获取某分类的子分类
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/categories?parentId=1&page=1&pageSize=10'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "total": 4,
    "page": 1,
    "list": [
      {
        "id": 3,
        "categoryId": "CAT003",
        "name": "动作电影",
        "identifier": "action",
        "parentId": 1,
        "sortOrder": 10,
        "status": 1,
        "description": "动作类电影",
        "videoCount": 0,
        "createdTime": 1767115587
      },
      {
        "id": 1,
        "categoryId": "CAT001",
        "name": "电影",
        "identifier": "movies",
        "sortOrder": 10,
        "status": 1,
        "description": "电影分类",
        "videoCount": 0,
        "createdTime": 1767115587
      }
    ]
  },
  "msg": "获取成功"
}
```

---

### 3.2 获取单个分类

**GET** `/category`

**查询参数:**
- `categoryId` (uint64): 分类ID（必填）

**请求示例:**
```bash
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category?categoryId=1'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "id": 1,
    "categoryId": "CAT001",
    "name": "电影",
    "identifier": "movies",
    "sortOrder": 10,
    "status": 1,
    "description": "电影分类",
    "videoCount": 0,
    "createdTime": 1767115587
  },
  "msg": "获取成功"
}
```

---

### 3.3 创建分类

**POST** `/category`

**请求体:**
```json
{
  "name": "纪录片",
  "identifier": "documentary",
  "parentId": null,
  "sortOrder": 30,
  "status": 1,
  "description": "纪录片分类"
}
```

**字段说明:**
- `name` (string, 必填): 分类名称
- `identifier` (string, 必填): 分类标识符
- `parentId` (uint64): 父分类ID（null或0表示顶级分类）
- `sortOrder` (uint32): 排序顺序
- `status` (int32): 状态，0=禁用, 1=启用，默认1
- `description` (string): 描述

**请求示例:**
```bash
curl -X POST 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category' \
  -H "Content-Type: application/json" \
  -d '{"name":"纪录片","identifier":"documentary","sortOrder":30,"status":1,"description":"纪录片分类"}'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "categoryId": 5
  },
  "msg": "创建成功"
}
```

---

### 3.4 更新分类

**PUT** `/category`

**查询参数:**
- `categoryId` (uint64): 分类ID（必填）

**请求体:**
```json
{
  "name": "电影院线",
  "identifier": "cinema-movies",
  "parentId": 1,
  "sortOrder": 20,
  "status": 1,
  "description": "更新后的描述"
}
```

**说明:** 所有字段都是可选的，只传需要更新的字段

**请求示例:**
```bash
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category?categoryId=1' \
  -H "Content-Type: application/json" \
  -d '{"name":"电影院线","sortOrder":20}'
```

**响应示例:**
```json
{
  "code": 0,
  "data": null,
  "msg": "更新成功"
}
```

---

### 3.5 删除分类

**DELETE** `/category`

**查询参数:**
- `categoryId` (uint64): 分类ID（必填）

**说明:** 软删除，将status设置为0

**请求示例:**
```bash
curl -X DELETE 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category?categoryId=1'
```

**响应示例:**
```json
{
  "code": 0,
  "data": null,
  "msg": "删除成功"
}
```

---

### 3.6 拖拽调整分类顺序和层级

**PUT** `/category/drag`

**请求体:**
```json
{
  "categoryId": 5,
  "parentId": 2,
  "sortOrder": 30
}
```

**字段说明:**
- `categoryId` (uint64, 必填): 要移动的分类ID
- `parentId` (uint64): 新的父分类ID（null或0表示移到顶级）
- `sortOrder` (uint32): 新的排序值

**说明:**
- 系统会检查循环引用（不能将分类设为自己的子分类或后代）
- 可以同时调整父级和排序

**请求示例:**
```bash
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category/drag' \
  -H "Content-Type: application/json" \
  -d '{"categoryId":5,"parentId":2,"sortOrder":30}'
```

**响应示例:**
```json
{
  "code": 0,
  "data": null,
  "msg": "拖拽成功"
}
```

---

### 3.7 获取分类统计

**GET** `/categories/stats`

**请求示例:**
```bash
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/categories/stats'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "total": 4,
    "level1": 2,
    "level2": 2,
    "enabled": 4
  },
  "msg": "获取成功"
}
```

**字段说明:**
- `total`: 总分类数
- `level1`: 一级分类数（顶级分类）
- `level2`: 二级分类数（有父分类）
- `enabled`: 启用的分类数

---

### 3.8 获取完整分类树

**GET** `/categories/tree`

**查询参数:**
- `status` (int32): 状态筛选（可选，1=仅启用）

**请求示例:**
```bash
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/categories/tree'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "total": 2,
    "tree": [
      {
        "id": 1,
        "categoryId": "CAT001",
        "name": "电影",
        "identifier": "movies",
        "sortOrder": 10,
        "status": 1,
        "description": "电影分类",
        "videoCount": 0,
        "level": 1,
        "children": [
          {
            "id": 3,
            "categoryId": "CAT003",
            "name": "动作电影",
            "identifier": "action",
            "parentId": 1,
            "sortOrder": 10,
            "status": 1,
            "description": "动作类电影",
            "videoCount": 0,
            "level": 2,
            "children": [],
            "createdTime": 1767115587
          },
          {
            "id": 4,
            "categoryId": "CAT004",
            "name": "科幻电影",
            "identifier": "sci-fi",
            "parentId": 1,
            "sortOrder": 20,
            "status": 1,
            "description": "科幻类电影",
            "videoCount": 0,
            "level": 2,
            "children": [],
            "createdTime": 1767115587
          }
        ],
        "createdTime": 1767115587
      },
      {
        "id": 2,
        "categoryId": "CAT002",
        "name": "电视剧",
        "identifier": "tv-series",
        "sortOrder": 20,
        "status": 1,
        "description": "电视剧分类",
        "videoCount": 1,
        "level": 1,
        "children": [],
        "createdTime": 1767115587
      }
    ]
  },
  "msg": "获取成功"
}
```

**字段说明:**
- `level`: 层级（1=一级分类，2=二级分类...）
- `children`: 子分类数组（空数组表示没有子分类）

---

### 3.9 获取分类路径（面包屑）

**GET** `/category/path`

**查询参数:**
- `categoryId` (uint64): 分类ID（必填）

**请求示例:**
```bash
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category/path?categoryId=3'
```

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "path": [
      {
        "id": 1,
        "categoryId": "CAT001",
        "name": "电影",
        "identifier": "movies",
        "sortOrder": 10,
        "status": 1,
        "description": "电影分类",
        "videoCount": 0,
        "createdTime": 1767115587
      },
      {
        "id": 3,
        "categoryId": "CAT003",
        "name": "动作电影",
        "identifier": "action",
        "parentId": 1,
        "sortOrder": 10,
        "status": 1,
        "description": "动作类电影",
        "videoCount": 0,
        "createdTime": 1767115587
      }
    ],
    "level": 2
  },
  "msg": "获取成功"
}
```

**说明:** 返回从根分类到当前分类的完整路径

---

### 3.10 获取子分类

**GET** `/category/children`

**查询参数:**
- `categoryId` (uint64): 分类ID（必填）
- `includeDescendants` (bool): 是否包含所有后代（递归），默认false
- `status` (int32): 状态筛选（可选，1=仅启用）

**请求示例:**
```bash
# 获取直接子分类
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category/children?categoryId=1'

# 获取所有后代（树形）
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category/children?categoryId=1&includeDescendants=true'
```

**响应示例（不递归）:**
```json
{
  "code": 0,
  "data": {
    "total": 2,
    "list": [
      {
        "id": 3,
        "categoryId": "CAT003",
        "name": "动作电影",
        "identifier": "action",
        "parentId": 1,
        "sortOrder": 10,
        "status": 1,
        "description": "动作类电影",
        "videoCount": 0,
        "createdTime": 1767115587
      },
      {
        "id": 4,
        "categoryId": "CAT004",
        "name": "科幻电影",
        "identifier": "sci-fi",
        "parentId": 1,
        "sortOrder": 20,
        "status": 1,
        "description": "科幻类电影",
        "videoCount": 0,
        "createdTime": 1767115587
      }
    ]
  },
  "msg": "获取成功"
}
```

**响应示例（递归 includeDescendants=true）:**
```json
{
  "code": 0,
  "data": {
    "total": 2,
    "tree": [
      {
        "id": 3,
        "categoryId": "CAT003",
        "name": "动作电影",
        "identifier": "action",
        "parentId": 1,
        "sortOrder": 10,
        "status": 1,
        "description": "动作类电影",
        "videoCount": 0,
        "level": 1,
        "children": [],
        "createdTime": 1767115587
      }
    ]
  },
  "msg": "获取成功"
}
```

---

## 使用示例

### 示例1: 视频管理流程

```bash
# 1. 获取待审核的视频列表
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/list?status=2&page=1&page_size=10'

# 2. 审核通过某个视频
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/status' \
  -H "Content-Type: application/json" \
  -d '{"video_id":30,"status":1}'

# 3. 编辑视频信息（添加标题、分类、标签）
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos' \
  -H "Content-Type: application/json" \
  -d '{
    "video_id": 30,
    "title": "精彩动作大片",
    "description": "2024年最新动作电影",
    "category_id": 1,
    "tag_ids": "1,4",
    "charge_type": "paid",
    "price": 9.9
  }'

# 4. 上架视频
curl -X PUT 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/status' \
  -H "Content-Type: application/json" \
  -d '{"video_id":30,"status":2}'

# 5. 查看视频列表（已上架）
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/videos/list?status=3&page=1&page_size=10'
```

---

### 示例2: 创建标签并排序

```bash
# 1. 创建三个标签（tagType: 0=普通类型, 1=热门类型）
curl -X POST https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag \
  -H "Content-Type: application/json" \
  -d '{"name":"动作","identifier":"action","tagType":1,"status":1}'

curl -X POST https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag \
  -H "Content-Type: application/json" \
  -d '{"name":"喜剧","identifier":"comedy","tagType":0,"status":1}'

curl -X POST https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag \
  -H "Content-Type: application/json" \
  -d '{"name":"爱情","identifier":"romance","tagType":1,"status":1}'

# 2. 按类型筛选标签
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags?tagType=1'  # 只获取热门类型标签

# 3. 重新排序（假设创建后的ID分别是 6, 7, 8）
curl -X PUT https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags/reorder \
  -H "Content-Type: application/json" \
  -d '{"tagIds":[8,6,7]}'

# 4. 查看排序结果
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags?page=1&pageSize=10'
```

---

### 示例3: 创建分类树

```bash
# 1. 创建顶级分类
curl -X POST https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category \
  -H "Content-Type: application/json" \
  -d '{"name":"电影","identifier":"movies","sortOrder":10,"status":1}'

# 2. 创建子分类（假设父分类ID是1）
curl -X POST https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category \
  -H "Content-Type: application/json" \
  -d '{"name":"动作片","identifier":"action-movies","parentId":1,"sortOrder":10,"status":1}'

# 3. 获取完整树形结构
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/categories/tree?status=1'

# 4. 拖拽调整分类（假设要移动分类ID=2到新的父分类ID=3）
curl -X PUT https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/category/drag \
  -H "Content-Type: application/json" \
  -d '{"categoryId":2,"parentId":3,"sortOrder":20}'
```

---

### 示例4: 查看统计信息

```bash
# 标签统计
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tags/stats'

# 分类统计
curl 'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/categories/stats'
```

---

## 注意事项

### 1. ID字段区分
- `id`: 数据库自增ID，用于API操作（如 `?tagId=1`，`?video_id=30`）
- `tagId/categoryId/product_id`: 业务唯一标识符（如 "TAG001", "CAT001", "V12345"），由系统自动生成

### 2. 视频状态说明
**数据库存储状态值:**
- `-1`: 上传失败
- `0`: 上传中
- `1`: 转码中
- `2`: 待审核
- `3`: 上架
- `4`: 已拒绝
- `5`: 下架

**API更新状态时的映射:**
- 传入 `1`（通过）→ 数据库更新为 `2`（待审核通过）
- 传入 `2`（上架）→ 数据库更新为 `3`（已上架）
- 传入 `-1`（拒绝）→ 数据库更新为 `4`（已拒绝）
- 传入 `-2`（下架）→ 数据库更新为 `5`（已下架）

### 3. 收费类型
- `free`: 免费视频
- `paid`: 付费视频（必须设置 `price`，最多两位小数）
- `vip`: VIP专享视频

### 4. 软删除
- 删除操作只是将 `status` 设为 0，数据仍保留在数据库
- 列表查询默认只返回 `status=1` 的数据（启用状态）

### 5. 标识符唯一性
- `identifier` 在同类型中必须唯一（标签之间、分类之间）
- 创建和更新时系统会自动校验

### 6. 排序逻辑
- 标签排序：创建时由后端自动生成（按ID递增），可通过 `/tags/reorder` 批量调整
- 分类排序：创建时需要指定 `sortOrder`，可通过 `/category/drag` 调整

### 7. 分类树形结构
- 支持多级分类（通过 `parentId` 实现）
- 系统会检查循环引用，防止数据错误
- `parentId` 为 null 或 0 表示顶级分类

### 8. 分页默认值
- `page`: 默认为 1
- `pageSize`: 默认为 20
- `total`: 返回符合条件的总记录数

### 9. 时间戳
- 所有时间字段返回 Unix 时间戳（秒）
- 前端需要自行转换为可读格式

### 10. URL编码
- 搜索关键词等参数如果包含中文或特殊字符，必须进行URL编码
- 例如："动作" 需要编码为 "%E5%8A%A8%E4%BD%9C"

### 11. 访问地址
- 开发环境: `https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt`
- 生产环境: `https://admin.beauty-666.com/api/v1/site-svc-video-mgmt`

### 12. 文件上传
- 使用 `multipart/form-data` 格式
- 封面图片字段名为 `cover`
- 支持的图片格式：JPG, PNG, WebP

---

## 错误码说明

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 400 | 请求参数错误 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |

常见错误信息:
- `invalid categoryId`: 分类ID格式错误
- `invalid tagId`: 标签ID格式错误
- `invalid video_id`: 视频ID格式错误
- `identifier is required`: 标识符必填
- `identifier already exists`: 标识符已存在
- `category not found`: 分类不存在
- `tag not found`: 标签不存在
- `video not found`: 视频不存在
- `circular reference detected`: 检测到循环引用
- `当收费类型为付款时，金额必填且必须大于 0`: 付费视频价格验证失败
- `金额最多只能有两位小数`: 价格小数位数超过限制
- `無效的狀態值`: 视频状态值不在允许范围内

---
