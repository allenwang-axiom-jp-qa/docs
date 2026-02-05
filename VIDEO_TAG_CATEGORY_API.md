# 视频标签和分类功能 API 文档

## 目录

- [概述](#概述)
- [数据库设计](#数据库设计)
- [架构设计](#架构设计)
- [RPC API](#rpc-api)
- [HTTP API](#http-api)
- [使用示例](#使用示例)
- [缓存策略](#缓存策略)
- [审计日志](#审计日志)

---

## 概述

视频标签和分类功能提供了完整的视频内容管理能力：

- **标签管理**：为视频打上多个标签，支持标签的增删改查
- **分类管理**：树形分类结构，支持多级分类
- **视频绑定**：一个视频可以关联多个标签和多个分类
- **高级查询**：支持按标签、分类、关键字筛选视频
- **性能优化**：Redis 缓存层提升查询性能
- **审计追踪**：Kafka 记录所有操作日志

---

## 数据库设计

### 表结构

#### 1. video_tag (标签表)

```sql
CREATE TABLE `video_tag` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(64) NOT NULL UNIQUE COMMENT '标签名称',
  `description` VARCHAR(255) DEFAULT '' COMMENT '标签描述',
  `enabled` TINYINT(1) DEFAULT 1 COMMENT '是否启用 1=启用 0=禁用',
  `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_name` (`name`),
  KEY `idx_enabled` (`enabled`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### 2. video_category (分类表)

```sql
CREATE TABLE `video_category` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(64) NOT NULL COMMENT '分类名称',
  `parent_id` BIGINT UNSIGNED DEFAULT 0 COMMENT '父分类ID 0表示顶级分类',
  `sort_order` INT DEFAULT 0 COMMENT '排序权重 数值越大越靠前',
  `enabled` TINYINT(1) DEFAULT 1 COMMENT '是否启用',
  `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_parent` (`parent_id`),
  KEY `idx_sort` (`sort_order`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### 3. video_tag_rel (视频-标签关联表)

```sql
CREATE TABLE `video_tag_rel` (
  `video_id` BIGINT UNSIGNED NOT NULL,
  `tag_id` BIGINT UNSIGNED NOT NULL,
  PRIMARY KEY (`video_id`, `tag_id`),
  KEY `idx_tag` (`tag_id`),
  FOREIGN KEY (`video_id`) REFERENCES `video`(`id`) ON DELETE CASCADE,
  FOREIGN KEY (`tag_id`) REFERENCES `video_tag`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### 4. video_category_rel (视频-分类关联表)

```sql
CREATE TABLE `video_category_rel` (
  `video_id` BIGINT UNSIGNED NOT NULL,
  `category_id` BIGINT UNSIGNED NOT NULL,
  PRIMARY KEY (`video_id`, `category_id`),
  KEY `idx_category` (`category_id`),
  FOREIGN KEY (`video_id`) REFERENCES `video`(`id`) ON DELETE CASCADE,
  FOREIGN KEY (`category_id`) REFERENCES `video_category`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 关系模型

```
video (1) ----< (N) video_tag_rel (N) >---- (1) video_tag
video (1) ----< (N) video_category_rel (N) >---- (1) video_category
video_category (1:parent) ----< (N:children) video_category
```

---

## 架构设计

### 分层架构

```
┌─────────────────────────────────────────┐
│         Client (RPC/HTTP)               │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Handler / Controller Layer             │
│  - handler.go (RPC)                     │
│  - http/controller.go (HTTP)            │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Service Layer (带缓存)                  │
│  - service/service.go                   │
│  - 业务逻辑 + 缓存管理                    │
└─────────────────────────────────────────┘
         ↓                ↓
┌──────────────┐  ┌─────────────────────┐
│ Cache Layer  │  │ Repository Layer    │
│ cache/       │  │ repository/         │
│ - Redis      │  │ - MySQL (sqlx)      │
└──────────────┘  └─────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│  Audit Layer                            │
│  - audit/audit.go                       │
│  - Kafka 异步审计日志                    │
└─────────────────────────────────────────┘
```

### 技术栈

- **RPC**: go-micro v5 (gRPC)
- **HTTP**: Gin
- **数据库**: MySQL 8.0 + sqlx
- **缓存**: Redis 6.0+
- **消息队列**: Kafka
- **服务发现**: etcd
- **追踪**: OpenTelemetry

---

## RPC API

### Proto 定义

位置：`cms-site-svc-api/video/video.proto`

```protobuf
service VideoService {
  // 标签管理
  rpc CreateTag(ReqCreateTag) returns (ResTag) {}
  rpc UpdateTag(ReqUpdateTag) returns (ResTag) {}
  rpc DeleteTag(ReqDeleteTag) returns (google.protobuf.Empty) {}
  rpc ListTags(ReqListTags) returns (ResListTags) {}

  // 分类管理
  rpc CreateCategory(ReqCreateCategory) returns (ResCategory) {}
  rpc UpdateCategory(ReqUpdateCategory) returns (ResCategory) {}
  rpc DeleteCategory(ReqDeleteCategory) returns (google.protobuf.Empty) {}
  rpc ListCategories(ReqListCategories) returns (ResListCategories) {}


}
```

### RPC 调用示例

```go
import (
    pb "ags-git.axiom-gaming.tech/cms-group-service/cms-api/video"
)

// 创建标签
rsp, err := videoClient.CreateTag(ctx, &pb.ReqCreateTag{
    Name:        "动作片",
    Description: "动作类型电影",
    Enabled:     true,
})

// 查询标签列表
tags, err := videoClient.ListTags(ctx, &pb.ReqListTags{
    Keyword:         "动作",
    IncludeDisabled: false,
    Page:            1,
    PageSize:        20,
})
```

---

## HTTP API

### Base URL

```
http://localhost:{HTTPPort}/api/v1
```

### 标签管理 API

#### 1. 创建标签

**请求**
```http
POST /api/v1/tags
Content-Type: application/json

{
  "name": "动作片",
  "description": "动作类型电影",
  "enabled": true
}
```

**响应**
```json
{
  "data": {
    "id": 1,
    "name": "动作片",
    "description": "动作类型电影",
    "enabled": true,
    "created_at": 1702345678,
    "updated_at": 1702345678
  }
}
```

#### 2. 更新标签

**请求**
```http
PUT /api/v1/tags
Content-Type: application/json

{
  "id": 1,
  "name": "动作电影",
  "description": "动作类型电影（更新）",
  "enabled": true
}
```

**响应**
```json
{
  "data": {
    "id": 1,
    "name": "动作电影",
    "description": "动作类型电影（更新）",
    "enabled": true,
    "created_at": 1702345678,
    "updated_at": 1702345700
  }
}
```

#### 3. 删除标签

**请求**
```http
DELETE /api/v1/tags/1
```

**响应**
```json
{
  "message": "ok"
}
```

#### 4. 获取标签详情

**请求**
```http
GET /api/v1/tags/1
```

**响应**
```json
{
  "data": {
    "id": 1,
    "name": "动作片",
    "description": "动作类型电影",
    "enabled": true,
    "created_at": 1702345678,
    "updated_at": 1702345678
  }
}
```

#### 5. 获取标签列表

**请求**
```http
GET /api/v1/tags?keyword=动作&include_disabled=false&page=1&page_size=20
```

**响应**
```json
{
  "data": {
    "tags": [
      {
        "id": 1,
        "name": "动作片",
        "description": "动作类型电影",
        "enabled": true,
        "created_at": 1702345678,
        "updated_at": 1702345678
      },
      {
        "id": 2,
        "name": "动作冒险",
        "description": "动作冒险类型",
        "enabled": true,
        "created_at": 1702345680,
        "updated_at": 1702345680
      }
    ],
    "total": 2
  }
}
```

### 分类管理 API

#### 1. 创建分类

**请求**
```http
POST /api/v1/categories
Content-Type: application/json

{
  "name": "电影",
  "parent_id": 0,
  "sort_order": 100,
  "enabled": true
}
```

**响应**
```json
{
  "data": {
    "id": 1,
    "name": "电影",
    "parent_id": 0,
    "sort_order": 100,
    "enabled": true,
    "created_at": 1702345678,
    "updated_at": 1702345678
  }
}
```

#### 2. 创建子分类

**请求**
```http
POST /api/v1/categories
Content-Type: application/json

{
  "name": "动作电影",
  "parent_id": 1,
  "sort_order": 90,
  "enabled": true
}
```

**响应**
```json
{
  "data": {
    "id": 2,
    "name": "动作电影",
    "parent_id": 1,
    "sort_order": 90,
    "enabled": true,
    "created_at": 1702345700,
    "updated_at": 1702345700
  }
}
```

#### 3. 更新分类

**请求**
```http
PUT /api/v1/categories
Content-Type: application/json

{
  "id": 1,
  "name": "电影类",
  "parent_id": 0,
  "sort_order": 100,
  "enabled": true
}
```

#### 4. 删除分类

**请求**
```http
DELETE /api/v1/categories/1
```

#### 5. 获取分类详情

**请求**
```http
GET /api/v1/categories/1
```

#### 6. 获取分类列表（支持树形结构）

**获取顶级分类**
```http
GET /api/v1/categories?parent_id=0&page=1&page_size=50
```

**获取子分类**
```http
GET /api/v1/categories?parent_id=1&page=1&page_size=50
```

**响应**
```json
{
  "data": {
    "categories": [
      {
        "id": 1,
        "name": "电影",
        "parent_id": 0,
        "sort_order": 100,
        "enabled": true,
        "created_at": 1702345678,
        "updated_at": 1702345678
      },
      {
        "id": 2,
        "name": "电视剧",
        "parent_id": 0,
        "sort_order": 90,
        "enabled": true,
        "created_at": 1702345680,
        "updated_at": 1702345680
      }
    ],
    "total": 2
  }
}
```

### 视频绑定 API

#### 1. 绑定视频标签

**请求**
```http
POST /api/v1/videos/bind-tags
Content-Type: application/json

{
  "video_id": 100,
  "tag_ids": [1, 2, 3]
}
```

**响应**
```json
{
  "message": "ok"
}
```

**说明**：
- 每次调用会清空该视频原有的标签关联，重新绑定新的标签
- 如果 `tag_ids` 为空数组，则清空所有标签

#### 2. 绑定视频分类

**请求**
```http
POST /api/v1/videos/bind-categories
Content-Type: application/json

{
  "video_id": 100,
  "category_ids": [1, 2]
}
```

**响应**
```json
{
  "message": "ok"
}
```

### 视频查询 API

#### 查询视频列表（支持多条件筛选）

**按标签筛选**
```http
GET /api/v1/videos?tag_ids=1,2&page=1&page_size=20
```

**按分类筛选**
```http
GET /api/v1/videos?category_ids=1&page=1&page_size=20
```

**按关键字搜索**
```http
GET /api/v1/videos?keyword=速度与激情&page=1&page_size=20
```

**组合筛选**
```http
GET /api/v1/videos?tag_ids=1,2&category_ids=1&keyword=动作&page=1&page_size=20
```

**响应**
```json
{
  "data": {
    "videos": [
      {
        "id": 100,
        "title": "速度与激情9",
        "cover_url": "https://example.com/cover.jpg",
        "created_at": 1702345678,
        "updated_at": 1702345678
      }
    ],
    "total": 1
  }
}
```

---

## 使用示例

### cURL 示例

#### 1. 创建标签
```bash
curl -X POST http://localhost:8081/api/v1/tags \
  -H "Content-Type: application/json" \
  -d '{
    "name": "动作片",
    "description": "动作类型电影",
    "enabled": true
  }'
```

#### 2. 获取标签列表
```bash
curl -X GET "http://localhost:8081/api/v1/tags?keyword=动作&page=1&page_size=20"
```

#### 3. 创建树形分类
```bash
# 创建顶级分类：电影
curl -X POST http://localhost:8081/api/v1/categories \
  -H "Content-Type: application/json" \
  -d '{
    "name": "电影",
    "parent_id": 0,
    "sort_order": 100,
    "enabled": true
  }'

# 假设返回 id=1，创建子分类：动作电影
curl -X POST http://localhost:8081/api/v1/categories \
  -H "Content-Type: application/json" \
  -d '{
    "name": "动作电影",
    "parent_id": 1,
    "sort_order": 90,
    "enabled": true
  }'
```


#### 5. 查询视频
```bash
curl -X GET "http://localhost:8081/api/v1/videos?tag_ids=1,2&category_ids=2&page=1&page_size=20"
```

### Go 客户端示例

**⚠️ 生产代码注意事项**：
1. ✅ **URL编码**: 使用 `url.QueryEscape` 或 `url.Values` 处理特殊字符
2. ✅ **错误处理**: 不要忽略 `json.Marshal`, `io.ReadAll` 等方法的错误
3. ✅ **结构体**: 使用 `json.Unmarshal` 将响应解析为结构体，方便操作数据
4. ✅ **HTTP状态码**: 检查 `resp.StatusCode` 处理错误响应
5. ✅ **超时设置**: 为HTTP客户端设置合理的超时时间

**❌ 常见错误示例**（仅供参考，不要在生产中使用）：
```go
// ❌ 错误1: 直接拼接URL，没有编码特殊字符
url := fmt.Sprintf("%s/tags?keyword=%s", baseURL, keyword) // keyword="C++"会出错

// ❌ 错误2: 忽略错误
body, _ := io.ReadAll(resp.Body)  // 不应该忽略错误
data, _ := json.Marshal(reqBody)  // 不应该忽略错误

// ❌ 错误3: 只打印字符串，不解析为结构体
fmt.Println(string(body))  // 应该用json.Unmarshal解析
```

**✅ 正确示例**（生产级代码）：

```go
package main

import (
    "bytes"
    "context"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "net/url"
    "strconv"
    "time"
)

const baseURL = "http://localhost:8081/api/v1/x"

// TagResponse 标签响应结构体
type TagResponse struct {
    ID          uint64 `json:"id"`
    Name        string `json:"name"`
    UseCount    int32  `json:"useCount"`
    HotScore    int32  `json:"hotScore"`
    Status      int32  `json:"status"`
    IsRecommend bool   `json:"isRecommend"`
    Description string `json:"description"`
    CreatedTime int64  `json:"createdTime"`
}

// ListTagsResponse 标签列表响应
type ListTagsResponse struct {
    Total int32          `json:"total"`
    Page  int32          `json:"page"`
    List  []*TagResponse `json:"list"`
}

// CreateTagResponse 创建标签响应
type CreateTagResponse struct {
    TagID uint64 `json:"tagId"`
}

// APIError 错误响应
type APIError struct {
    Error string `json:"error"`
}

// createTag 创建标签（正确处理错误）
func createTag(ctx context.Context, name, desc string, status int32, isRecommend bool) (uint64, error) {
    reqBody := map[string]interface{}{
        "name":        name,
        "description": desc,
        "status":      status,
        "isRecommend": isRecommend,
    }

    // ✅ 不忽略Marshal错误
    data, err := json.Marshal(reqBody)
    if err != nil {
        return 0, fmt.Errorf("marshal request: %w", err)
    }

    req, err := http.NewRequestWithContext(ctx, http.MethodPost, baseURL+"/tag", bytes.NewBuffer(data))
    if err != nil {
        return 0, fmt.Errorf("create request: %w", err)
    }
    req.Header.Set("Content-Type", "application/json")

    client := &http.Client{Timeout: 10 * time.Second}
    resp, err := client.Do(req)
    if err != nil {
        return 0, fmt.Errorf("do request: %w", err)
    }
    defer resp.Body.Close()

    // ✅ 不忽略ReadAll错误
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return 0, fmt.Errorf("read response: %w", err)
    }

    // ✅ 检查HTTP状态码
    if resp.StatusCode != http.StatusOK {
        var apiErr APIError
        if err := json.Unmarshal(body, &apiErr); err == nil && apiErr.Error != "" {
            return 0, fmt.Errorf("API error: %s", apiErr.Error)
        }
        return 0, fmt.Errorf("HTTP %d: %s", resp.StatusCode, string(body))
    }

    // ✅ 解析为结构体，不忽略错误
    var result CreateTagResponse
    if err := json.Unmarshal(body, &result); err != nil {
        return 0, fmt.Errorf("unmarshal response: %w", err)
    }

    return result.TagID, nil
}

// listTags 查询标签列表（正确处理URL编码）
func listTags(ctx context.Context, keyword string, status *int32, page, pageSize int32) (*ListTagsResponse, error) {
    // ✅ 使用url.Values进行URL编码
    params := url.Values{}
    params.Set("page", strconv.Itoa(int(page)))
    params.Set("pageSize", strconv.Itoa(int(pageSize)))

    if keyword != "" {
        // 自动处理特殊字符（空格、+、&等）
        params.Set("keyword", keyword)
    }

    if status != nil {
        params.Set("status", strconv.Itoa(int(*status)))
    }

    // 构建完整URL
    fullURL := fmt.Sprintf("%s/tags?%s", baseURL, params.Encode())

    req, err := http.NewRequestWithContext(ctx, http.MethodGet, fullURL, nil)
    if err != nil {
        return nil, fmt.Errorf("create request: %w", err)
    }

    client := &http.Client{Timeout: 10 * time.Second}
    resp, err := client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("do request: %w", err)
    }
    defer resp.Body.Close()

    // ✅ 不忽略ReadAll错误
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("read response: %w", err)
    }

    // ✅ 检查状态码
    if resp.StatusCode != http.StatusOK {
        var apiErr APIError
        if err := json.Unmarshal(body, &apiErr); err == nil && apiErr.Error != "" {
            return nil, fmt.Errorf("API error: %s", apiErr.Error)
        }
        return nil, fmt.Errorf("HTTP %d: %s", resp.StatusCode, string(body))
    }

    // ✅ 解析为结构体
    var result ListTagsResponse
    if err := json.Unmarshal(body, &result); err != nil {
        return nil, fmt.Errorf("unmarshal response: %w", err)
    }

    return &result, nil
}

func main() {
    ctx := context.Background()

    // 示例1: 创建标签
    tagID, err := createTag(ctx, "C++编程", "C++语言相关", 1, true)
    if err != nil {
        fmt.Printf("创建标签失败: %v\n", err)
        return
    }
    fmt.Printf("✓ 创建标签成功, ID: %d\n", tagID)

    // 示例2: 查询标签（关键词包含特殊字符）
    status := int32(1)
    tags, err := listTags(ctx, "C++", &status, 1, 20)
    if err != nil {
        fmt.Printf("查询标签失败: %v\n", err)
        return
    }

    fmt.Printf("✓ 查询到 %d 个标签:\n", tags.Total)
    for _, tag := range tags.List {
        fmt.Printf("  - [%d] %s (状态:%d, 推荐:%v)\n",
            tag.ID, tag.Name, tag.Status, tag.IsRecommend)
    }
}
```

**📝 完整的生产级客户端示例**: 请参考 [examples/client_example.go](../examples/client_example.go)
```

---

## 缓存策略

### Redis 缓存设计

**缓存键设计**：
```
video:tag:{id}              - 单个标签缓存 (TTL: 30分钟)
video:category:{id}         - 单个分类缓存 (TTL: 30分钟)
video:tags:list:{key}       - 标签列表缓存 (TTL: 5分钟)
video:categories:list:{key} - 分类列表缓存 (TTL: 5分钟)
```

**缓存更新策略**：
- **创建/更新**: Write-Through（先写库，后写缓存）
- **删除**: Cache-Aside（先删库，后删缓存）
- **查询**: Cache-Aside（先查缓存，未命中查库后回写缓存）
- **列表查询**: 写操作时清除对应的列表缓存

**缓存失效场景**：
1. 创建标签/分类 → 清除列表缓存
2. 更新标签/分类 → 更新单项缓存 + 清除列表缓存
3. 删除标签/分类 → 删除单项缓存 + 清除列表缓存

### 使用服务层（带缓存）

```go
import (
    "ags-git.axiom-gaming.tech/cms-group-service/cms-video/internal/cache"
    "ags-git.axiom-gaming.tech/cms-group-service/cms-video/internal/service"
    "ags-git.axiom-gaming.tech/cms-group-service/cms-video/internal/repository"
)

// 初始化
repo := repository.NewVideoRepo(db)
videoCache := cache.NewVideoCache(redisClient)
svc := service.NewVideoService(repo, videoCache)

// 使用（自动处理缓存）
tag, err := svc.GetTagByID(ctx, 1)  // 先查缓存，未命中查库
tags, total, err := svc.ListTags(ctx, "动作", false, 0, 20)  // 缓存列表
```

---

## 审计日志

### Kafka 审计事件

**Topic**: `cms-video-oplog`

**事件结构**：
```json
{
  "timestamp": 1702345678,
  "user_id": "admin001",
  "user_ip": "192.168.1.100",
  "operation": "CREATE",
  "resource": "TAG",
  "resource_id": "1",
  "resource_name": "动作片",
  "details": "{\"description\":\"动作类型电影\"}",
  "success": true,
  "error_msg": "",
  "trace_id": "abc123def456"
}
```

**操作类型**：
- `CREATE` - 创建
- `UPDATE` - 更新
- `DELETE` - 删除
- `BIND` - 绑定
- `QUERY` - 查询

**资源类型**：
- `TAG` - 标签
- `CATEGORY` - 分类
- `VIDEO` - 视频

### 审计日志使用

```go
import "ags-git.axiom-gaming.tech/cms-group-service/cms-video/internal/audit"

// 初始化审计日志
auditLogger := audit.NewAuditLogger(
    []string{"localhost:9092"},
    "cms-video-oplog",
)
defer auditLogger.Close()

// 记录标签创建
auditLogger.LogTagCreate(ctx, tagID, tagName, true, "")

// 记录标签更新
auditLogger.LogTagUpdate(ctx, tagID, tagName, details, true, "")

// 记录标签删除
auditLogger.LogTagDelete(ctx, tagID, true, "")

// 自定义审计事件
auditLogger.Log(ctx, &audit.AuditEvent{
    UserID:       "admin001",
    UserIP:       "192.168.1.100",
    Operation:    audit.OpCreate,
    Resource:     audit.ResourceTag,
    ResourceID:   "1",
    ResourceName: "动作片",
    Success:      true,
})
```

---

## 错误处理

### 错误码

| 错误码 | HTTP状态码 | 说明 |
|--------|-----------|------|
| 400 | 400 Bad Request | 请求参数错误 |
| 404 | 404 Not Found | 资源不存在 |
| 500 | 500 Internal Server Error | 服务器内部错误 |

### 错误响应示例

```json
{
  "code": "VIDEO_TAG_NOT_FOUND",
  "message": "标签不存在",
  "status_code": 404
}
```

---

## 性能优化建议

1. **索引优化**
   - `video_tag.name` 唯一索引
   - `video_category.parent_id` 索引
   - `video_tag_rel` 和 `video_category_rel` 的外键索引

2. **缓存预热**
   - 应用启动时加载热门标签和分类到缓存

3. **批量操作**
   - 使用 `BindTags` 和 `BindCategories` 批量绑定

4. **分页查询**
   - 避免一次性查询大量数据
   - 建议 `page_size` 不超过 100

5. **读写分离**
   - 查询操作可走 MySQL 从库
   - 写操作走主库

---

## 部署说明

### 配置文件

`conf/cms-video.json`:
```json
{
  "video-db": {
    "host": "127.0.0.1",
    "port": 3306,
    "user": "root",
    "pwd": "password",
    "name": "cms_video",
    "conn": 10
  },
  "redis": {
    "host": "127.0.0.1",
    "port": 6379,
    "pwd": "",
    "db": 0
  },
  "kafka": {
    "brokers": ["127.0.0.1:9092"],
    "topic_oplog": "cms-video-oplog"
  }
}
```

### 启动服务

```bash
# 编译
go build -o cms-video ./cmd/cms-video

# 运行
./cms-video -conf ./conf/env.json -decryptor ./decryptor/libags.so
```

### 服务端口

- **RPC 端口**: 配置文件中的 `RPCPort`
- **HTTP 端口**: 配置文件中的 `HTTPPort`

---

## 常见问题 FAQ

### Q1: 如何实现树形分类的完整树查询？

A: 当前 API 按层级查询，前端可以递归调用。如需服务端返回完整树，可以扩展 Repository 增加递归查询方法。

### Q2: 视频可以绑定多少个标签和分类？

A: 理论上没有限制，但建议：
- 标签：3-10个
- 分类：1-3个

### Q3: 缓存失效会导致雪崩吗？

A: 采用了不同的 TTL（30分钟和5分钟），缓存失效时间分散，不会集中失效。

### Q4: 审计日志异步写入会丢失吗？

A: Kafka Writer 使用了 `RequiredAcks: RequireOne`，确保至少一个 broker 确认。建议生产环境使用 `RequireAll`。

---

## 开发团队

- **项目**: CMS Video Service
- **版本**: v0.0.1
- **维护**: CMS Service Team

---

## 更新日志

- **2024-12-09**: 初始版本发布
  - 完整的标签和分类功能
  - HTTP RESTful API
  - Redis 缓存层
  - Kafka 审计日志
