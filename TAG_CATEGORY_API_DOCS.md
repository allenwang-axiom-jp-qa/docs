# Tag Category Management API Documentation

标签分类管理 API 文档

**Base URL**: `/api/v1/site-svc-video-mgmt`

---

## 目录

1. [概述](#概述)
2. [数据模型](#数据模型)
3. [获取分类列表](#1-获取分类列表)
4. [获取单个分类](#2-获取单个分类)
5. [创建分类](#3-创建分类)
6. [更新分类](#4-更新分类)
7. [删除分类](#5-删除分类)
8. [获取分类统计](#6-获取分类统计)

---

## 概述

标签分类管理API允许您对标签进行分类管理，建立层次化的标签组织结构。

### 功能特性

- **分类创建与管理**: 创建、更新、删除标签分类
- **标签关联**: 管理分类下的标签
- **排序管理**: 支持调整分类顺序
- **状态控制**: 启用/禁用分类
- **统计信息**: 获取分类使用统计

### 接口列表

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tag-categories` | 获取分类列表 |
| GET | `/tag-category` | 获取单个分类详情 |
| POST | `/tag-category` | 创建新分类 |
| PUT | `/tag-category` | 更新分类信息 |
| DELETE | `/tag-category` | 删除分类 |
| GET | `/tag-categories/stats` | 获取分类统计 |

### 标签与分类的关系

- 每个标签必须属于一个分类
- 一个分类可以包含多个标签
- 分类被禁用时，其下的标签仍然可以正常使用

---

## 数据模型

### Tag Category Object

| Field | Type | Description |
|-------|------|-------------|
| id | int | 数据库主键ID |
| categoryId | string | 分类唯一标识符 (CAT001) |
| name | string | 分类名称 |
| icon | string | 分类图标URL |
| description | string | 分类描述 |
| tagCount | int | 该分类下的标签数量 |
| sortOrder | int | 排序顺序 |
| status | int | 状态: 0=禁用, 1=启用 |
| createdAt | int | 创建时间 (Unix时间戳) |
| updatedAt | int | 更新时间 (Unix时间戳) |
| createdBy | string | 创建人 |
| updatedBy | string | 更新人 |

---

## 1. 获取分类列表

### `GET /api/v1/site-svc-video-mgmt/tag-categories`

获取标签分类列表，支持分页、关键词搜索和状态筛选。

### 请求参数 (Query Parameters)

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| page | int | 否 | 1 | 页码，从1开始 |
| pageSize | int | 否 | 20 | 每页数量 |
| keyword | string | 否 | - | 搜索关键词，模糊匹配分类名称 |
| status | int | 否 | - | 状态筛选：0=禁用，1=启用 |

### 响应参数

| 参数 | 类型 | 说明 |
|------|------|------|
| code | int | 业务状态码（0表示成功） |
| msg | string | 响应消息 |
| data.total | int | 总记录数 |
| data.page | int | 当前页码 |
| data.list | array | 分类列表 |

### 响应示例

```json
{
  "code": 0,
  "data": {
    "total": 5,
    "page": 1,
    "list": [
      {
        "id": 1,
        "categoryId": "CAT001",
        "name": "内容类型",
        "icon": "https://cdn.example.com/icons/content.png",
        "description": "视频内容分类标签",
        "tagCount": 15,
        "sortOrder": 1,
        "status": 1,
        "createdAt": 1704067200,
        "updatedAt": 1704067200,
        "createdBy": "admin",
        "updatedBy": "admin"
      },
      {
        "id": 2,
        "categoryId": "CAT002",
        "name": "风格类型",
        "icon": "https://cdn.example.com/icons/style.png",
        "description": "视频风格分类标签",
        "tagCount": 8,
        "sortOrder": 2,
        "status": 1,
        "createdAt": 1704067200,
        "updatedAt": 1704067200,
        "createdBy": "admin",
        "updatedBy": "admin"
      }
    ]
  },
  "msg": "获取成功"
}
```

### 代码示例

**JavaScript**
```javascript
async function getTagCategories(params = {}) {
  const query = new URLSearchParams({
    page: params.page || 1,
    pageSize: params.pageSize || 20,
    ...(params.keyword && { keyword: params.keyword }),
    ...(params.status !== undefined && { status: params.status })
  });

  const response = await fetch(
    `https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-categories?${query}`
  );
  const result = await response.json();

  if (result.code === 0) {
    return result.data;
  } else {
    throw new Error(result.msg);
  }
}

// 使用示例
getTagCategories({ keyword: '内容', status: 1 });
```

**Python**
```python
import requests

def get_tag_categories(page=1, page_size=20, keyword=None, status=None):
    params = {
        'page': page,
        'pageSize': page_size
    }
    if keyword:
        params['keyword'] = keyword
    if status is not None:
        params['status'] = status

    response = requests.get(
        'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-categories',
        params=params
    )
    result = response.json()

    if result['code'] == 0:
        return result['data']
    else:
        raise Exception(result['msg'])
```

**cURL**
```bash
curl "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-categories?page=1&pageSize=20&status=1"
```

### 错误码

| 错误码 | 含义 | HTTP状态码 |
|--------|------|-----------|
| 0 | 成功 | 200 |
| 400 | 参数错误 | 400 |
| 500 | 服务器内部错误 | 500 |

---

## 2. 获取单个分类

### `GET /api/v1/site-svc-video-mgmt/tag-category`

根据分类ID获取单个标签分类的完整信息，包括该分类下的标签数量。

### 请求参数 (Query Parameters)

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | int | 是 | 分类数据库主键ID |

### 响应示例

```json
{
  "code": 0,
  "data": {
    "id": 1,
    "categoryId": "CAT001",
    "name": "内容类型",
    "icon": "https://cdn.example.com/icons/content.png",
    "description": "视频内容分类标签",
    "tagCount": 15,
    "sortOrder": 1,
    "status": 1,
    "createdAt": 1704067200,
    "updatedAt": 1704067200,
    "createdBy": "admin",
    "updatedBy": "admin"
  },
  "msg": "获取成功"
}
```

### 代码示例

**JavaScript**
```javascript
async function getTagCategory(id) {
  const response = await fetch(
    `https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category?id=${id}`
  );
  const result = await response.json();

  if (result.code === 0) {
    return result.data;
  } else {
    throw new Error(result.msg);
  }
}
```

**Python**
```python
import requests

def get_tag_category(category_id):
    response = requests.get(
        'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category',
        params={'id': category_id}
    )
    result = response.json()

    if result['code'] == 0:
        return result['data']
    else:
        raise Exception(result['msg'])
```

**cURL**
```bash
curl "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category?id=1"
```

### 错误码

| 错误码 | 含义 | HTTP状态码 |
|--------|------|-----------|
| 0 | 成功 | 200 |
| 400 | 参数错误（缺少id或格式错误） | 400 |
| 404 | 分类不存在 | 404 |
| 500 | 服务器内部错误 | 500 |

---

## 3. 创建分类

### `POST /api/v1/site-svc-video-mgmt/tag-category`

创建一个新的标签分类。分类名称在系统中必须唯一，系统会自动生成分类标识符（categoryId）。

### 请求参数 (Request Body)

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| name | string | 是 | - | 分类名称，必须唯一，最大长度100字符 |
| icon | string | 否 | - | 分类图标URL |
| description | string | 否 | - | 分类描述 |
| sortOrder | int | 否 | 0 | 排序顺序，数字越小越靠前 |
| status | int | 否 | 1 | 状态：0=禁用，1=启用 |

### 响应示例

```json
{
  "code": 0,
  "data": {
    "id": 6,
    "categoryId": "CAT006"
  },
  "msg": "创建成功"
}
```

### 代码示例

**JavaScript**
```javascript
async function createTagCategory(categoryData) {
  const response = await fetch(
    'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: categoryData.name,
        icon: categoryData.icon || '',
        description: categoryData.description || '',
        sortOrder: categoryData.sortOrder || 0,
        status: categoryData.status ?? 1
      })
    }
  );
  const result = await response.json();

  if (result.code === 0) {
    return result.data;
  } else {
    throw new Error(result.msg);
  }
}

// 使用示例
createTagCategory({
  name: '主题类型',
  icon: 'https://cdn.example.com/icons/theme.png',
  description: '视频主题分类',
  sortOrder: 3,
  status: 1
});
```

**Python**
```python
import requests

def create_tag_category(name, icon='', description='', sort_order=0, status=1):
    response = requests.post(
        'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category',
        json={
            'name': name,
            'icon': icon,
            'description': description,
            'sortOrder': sort_order,
            'status': status
        }
    )
    result = response.json()

    if result['code'] == 0:
        return result['data']
    else:
        raise Exception(result['msg'])
```

**cURL**
```bash
curl -X POST "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "主题类型",
    "icon": "https://cdn.example.com/icons/theme.png",
    "description": "视频主题分类",
    "sortOrder": 3,
    "status": 1
  }'
```

### 错误码

| 错误码 | 含义 | HTTP状态码 |
|--------|------|-----------|
| 0 | 成功 | 200 |
| 400 | 参数错误（缺少必填字段） | 400 |
| 1002 | 分类名称已存在 | 400 |
| 500 | 服务器内部错误 | 500 |

---

## 4. 更新分类

### `PUT /api/v1/site-svc-video-mgmt/tag-category`

更新指定标签分类的信息。只需传入需要更新的字段，未传入的字段将保持原值。

### 请求参数

**Query Parameters**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | int | 是 | 要更新的分类数据库主键ID |

**Request Body**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 否 | 分类名称，如果修改需确保唯一 |
| icon | string | 否 | 分类图标URL |
| description | string | 否 | 分类描述 |
| sortOrder | int | 否 | 排序顺序 |
| status | int | 否 | 状态：0=禁用，1=启用 |

### 响应示例

```json
{
  "code": 0,
  "msg": "更新成功"
}
```

### 代码示例

**JavaScript**
```javascript
async function updateTagCategory(id, updates) {
  const response = await fetch(
    `https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category?id=${id}`,
    {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(updates)
    }
  );
  const result = await response.json();

  if (result.code === 0) {
    return true;
  } else {
    throw new Error(result.msg);
  }
}

// 使用示例：更新分类名称和状态
updateTagCategory(1, {
  name: '更新后的分类名',
  status: 0
});
```

**Python**
```python
import requests

def update_tag_category(category_id, **updates):
    response = requests.put(
        'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category',
        params={'id': category_id},
        json=updates
    )
    result = response.json()

    if result['code'] == 0:
        return True
    else:
        raise Exception(result['msg'])

# 使用示例
update_tag_category(1, name='更新后的分类名', status=0)
```

**cURL**
```bash
curl -X PUT "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category?id=1" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "更新后的分类名",
    "status": 0
  }'
```

### 错误码

| 错误码 | 含义 | HTTP状态码 |
|--------|------|-----------|
| 0 | 成功 | 200 |
| 400 | 参数错误（缺少id或格式错误） | 400 |
| 404 | 分类不存在 | 404 |
| 1002 | 分类名称已被其他分类使用 | 400 |
| 500 | 服务器内部错误 | 500 |

---

## 5. 删除分类

### `DELETE /api/v1/site-svc-video-mgmt/tag-category`

删除指定的标签分类。

> ⚠️ **警告**: 删除分类前，请确保该分类下没有关联的标签。如果分类下还有标签，建议先将标签迁移到其他分类，或者将分类状态设为禁用。

### 请求参数 (Query Parameters)

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | int | 是 | 要删除的分类数据库主键ID |

### 响应示例

```json
{
  "code": 0,
  "msg": "删除成功"
}
```

### 代码示例

**JavaScript**
```javascript
async function deleteTagCategory(id) {
  const response = await fetch(
    `https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category?id=${id}`,
    {
      method: 'DELETE'
    }
  );
  const result = await response.json();

  if (result.code === 0) {
    return true;
  } else {
    throw new Error(result.msg);
  }
}
```

**Python**
```python
import requests

def delete_tag_category(category_id):
    response = requests.delete(
        'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category',
        params={'id': category_id}
    )
    result = response.json()

    if result['code'] == 0:
        return True
    else:
        raise Exception(result['msg'])
```

**cURL**
```bash
curl -X DELETE "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category?id=1"
```

### 错误码

| 错误码 | 含义 | HTTP状态码 |
|--------|------|-----------|
| 0 | 成功 | 200 |
| 400 | 参数错误（缺少id或格式错误） | 400 |
| 404 | 分类不存在 | 404 |
| 409 | 分类下还有关联标签，无法删除 | 409 |
| 500 | 服务器内部错误 | 500 |

---

## 6. 获取分类统计

### `GET /api/v1/site-svc-video-mgmt/tag-categories/stats`

获取标签分类的整体统计信息，包括总分类数、启用分类数、关联标签总数和平均标签数。

### 响应参数

| 参数 | 类型 | 说明 |
|------|------|------|
| code | int | 业务状态码（0表示成功） |
| msg | string | 响应消息 |
| data.totalCount | int | 分类总数 |
| data.activeCount | int | 启用状态的分类数量 |
| data.totalTags | int | 所有分类下的标签总数 |
| data.avgTags | float | 平均每个分类的标签数量 |

### 响应示例

```json
{
  "code": 0,
  "data": {
    "totalCount": 5,
    "activeCount": 4,
    "totalTags": 50,
    "avgTags": 10.0
  },
  "msg": "获取成功"
}
```

### 代码示例

**JavaScript**
```javascript
async function getTagCategoryStats() {
  const response = await fetch(
    'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-categories/stats'
  );
  const result = await response.json();

  if (result.code === 0) {
    console.log('分类总数:', result.data.totalCount);
    console.log('启用分类:', result.data.activeCount);
    console.log('标签总数:', result.data.totalTags);
    console.log('平均标签数:', result.data.avgTags);
    return result.data;
  } else {
    throw new Error(result.msg);
  }
}
```

**Python**
```python
import requests

def get_tag_category_stats():
    response = requests.get(
        'https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-categories/stats'
    )
    result = response.json()

    if result['code'] == 0:
        print(f"分类总数: {result['data']['totalCount']}")
        print(f"启用分类: {result['data']['activeCount']}")
        print(f"标签总数: {result['data']['totalTags']}")
        print(f"平均标签数: {result['data']['avgTags']}")
        return result['data']
    else:
        raise Exception(result['msg'])
```

**cURL**
```bash
curl "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-categories/stats"
```

### 错误码

| 错误码 | 含义 | HTTP状态码 |
|--------|------|-----------|
| 0 | 成功 | 200 |
| 500 | 服务器内部错误 | 500 |

---

## 快速开始

### 1. 获取所有启用的分类

```bash
curl "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-categories?status=1"
```

### 2. 创建新分类

```bash
curl -X POST "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-category" \
  -H "Content-Type: application/json" \
  -d '{"name": "内容类型", "description": "按内容分类的标签"}'
```

### 3. 获取分类统计

```bash
curl "https://dev-admin.beauty-666.com/api/v1/site-svc-video-mgmt/tag-categories/stats"
```

---

## 注意事项

1. **分类名称唯一性**: 分类名称在系统中必须唯一。创建分类时，系统会自动生成唯一的 categoryId。

2. **删除分类**: 删除分类前，请确保该分类下没有关联的标签。建议先将标签迁移到其他分类。

3. **时间戳格式**: 所有时间字段（createdAt、updatedAt）均为 Unix 时间戳（秒级）。
