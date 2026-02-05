# 钻石配置模块 API 完整文档

## 版本信息

- **文档版本**: v1.3
- **API版本**: v1
- **最后更新**: 2025-12-30
- **服务地址**: http://localhost:10100
- **路由前缀**: /api/v1/oms
- **更新日志**:
  - v1.3 (2025-12-30):
    - 更新接口同步支持标签多选验证
    - UpdateDiamond 接口添加标签字段自定义验证
    - 完善更新接口的错误提示信息
  - v1.2 (2025-12-30):
    - 标签字段支持多选（逗号分隔）
    - 新增"不设置标签"选项
    - status字段改为可选，默认值为"active"
  - v1.1 (2025-12-30): 新增 `total_diamonds` 计算字段
  - v1.0 (2025-12-30): 初始版本

## 📋 目录

- [1. 概览统计](#1-概览统计)
- [2. 套餐列表](#2-套餐列表)
- [3. 创建套餐](#3-创建套餐)
- [4. 获取套餐详情](#4-获取套餐详情)
- [5. 更新套餐](#5-更新套餐)
- [6. 删除套餐](#6-删除套餐)
- [7. 获取汇率配置](#7-获取汇率配置)
- [8. 更新汇率配置](#8-更新汇率配置)
- [9. 同步实时汇率](#9-同步实时汇率)
- [数据模型](#数据模型)
- [业务规则](#业务规则)
- [错误码说明](#错误码说明)

---

## 1. 概览统计

获取钻石套餐的统计概览信息

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds/overview`
- **请求方式**: `GET`
- **是否需要认证**: 否（本地测试）

### 请求参数

无

### 响应数据

#### 成功响应

```json
{
  "code": 0,
  "data": {
    "total_count": 6,
    "active_count": 6,
    "hot_count": 2,
    "average_price": 47.5
  },
  "msg": "成功"
}
```

#### 字段说明

| 字段名 | 类型 | 说明 |
|--------|------|------|
| total_count | int | 套餐总数 |
| active_count | int | 上架套餐数量 |
| hot_count | int | 热门套餐数量（tags包含"热门"） |
| average_price | float | 上架套餐平均价格（元） |

---

## 2. 套餐列表

获取钻石套餐列表（带分页和筛选）

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds`
- **请求方式**: `GET`
- **是否需要认证**: 否（本地测试）

### 请求参数

#### Query参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页数量，默认20，最大100 |
| status | string | 否 | 状态筛选：active/inactive |

#### 请求示例

```bash
GET /api/v1/oms/diamonds?page=1&pageSize=10&status=active
```

### 响应数据

#### 字段说明

| 字段名 | 类型 | 说明 |
|--------|------|------|
| diamonds | int | 基础钻石数量 |
| bonus_diamonds | int | 赠送钻石数量 |
| price_cny_cents | int | 人民币价格（单位：分）10元=1000分 |
| price_usdt_cents | int | USDT价格（单位：分）1USDT=100分 |
| first_charge_bonus | bool | 首充奖励开关 |
| first_charge_bonus_diamonds | int | 首充赠送钻石数量 |
| tags | string | 标签（支持多选，逗号分隔） |
| description | string | 备注说明 |
| sort_order | int | 排序值，数字越小越靠前 |
| status | string | 状态：active（上架）、inactive（下架） |
| total_diamonds | int | **实得钻石总数**（计算字段，后端自动计算） |
| CreatedAt | string | 创建时间 |
| UpdatedAt | string | 更新时间 |

#### 响应示例

```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "ID": 1,
        "CreatedAt": "2025-12-30T15:32:29+08:00",
        "UpdatedAt": "2025-12-30T15:32:29+08:00",
        "diamonds": 100,
        "bonus_diamonds": 0,
        "price_cny_cents": 1000,
        "price_usdt_cents": 150,
        "first_charge_bonus": false,
        "first_charge_bonus_diamonds": 0,
        "tags": "",
        "description": "新手推荐",
        "sort_order": 1,
        "status": "active",
        "total_diamonds": 100
      },
      {
        "ID": 2,
        "CreatedAt": "2025-12-30T15:32:29+08:00",
        "UpdatedAt": "2025-12-30T15:32:29+08:00",
        "diamonds": 500,
        "bonus_diamonds": 50,
        "price_cny_cents": 5000,
        "price_usdt_cents": 750,
        "first_charge_bonus": true,
        "first_charge_bonus_diamonds": 100,
        "tags": "推荐",
        "description": "新手推荐、超值大礼包",
        "sort_order": 2,
        "status": "active",
        "total_diamonds": 650
      }
    ],
    "total": 6,
    "page": 1,
    "size": 10
  },
  "msg": "成功"
}
```

#### 实得钻石计算方式

```
实得钻石 = 基础钻石 + 赠送钻石 + (首充开启 ? 首充钻石 : 0)

示例：
套餐1: total_diamonds = 100 = 100 + 0 + 0
套餐2 (首充): total_diamonds = 650 = 500 + 50 + 100
套餐2 (非首充): total_diamonds = 550 = 500 + 50 + 0

注意：total_diamonds 是后端自动计算的字段，前端无需手动计算
```

---

## 3. 创建套餐

创建新的钻石充值套餐

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds`
- **请求方式**: `POST`
- **是否需要认证**: 否（本地测试）
- **Content-Type**: `application/json`

### 请求参数

| 参数名 | 类型 | 必填 | 校验规则 | 说明 |
|--------|------|------|----------|------|
| diamonds | int | 是 | ≥1 | 基础钻石数量 |
| bonus_diamonds | int | 否 | ≥0 | 赠送钻石数量，默认0 |
| price_cny_cents | int | 是 | ≥1 | 人民币价格（分），如10元=1000 |
| price_usdt_cents | int | 是 | ≥1 | USDT价格（分），如1.5U=150 |
| first_charge_bonus | bool | 否 | - | 首充奖励开关，默认false |
| first_charge_bonus_diamonds | int | 否 | ≥0 | 首充赠送钻石，默认0 |
| tags | string | 否 | 见下方说明 | 标签，支持多选（逗号分隔），可为空 |
| description | string | 否 | - | 备注说明 |
| sort_order | int | 否 | - | 排序值，默认0 |
| status | string | 否 | active/inactive | 状态，默认active |
| meta | string | 否 | - | 扩展字段（JSON字符串） |

**标签 (tags) 字段说明**：
- 可选字段，可以不传或传空字符串
- 有效标签：**"不设置标签"**、**"热门"**、**"推荐"**、**"超值"**
- 支持多选，使用逗号分隔，如：`"热门,推荐"`
- 示例：
  - 不设置：`""` 或不传该字段
  - 单个标签：`"热门"`
  - 多个标签：`"热门,推荐,超值"`
  - 明确标记：`"不设置标签"`

#### 请求示例

```bash
curl -X POST "http://localhost:10100/api/v1/oms/diamonds" \
  -H "Content-Type: application/json" \
  -d '{
    "diamonds": 300,
    "bonus_diamonds": 30,
    "price_cny_cents": 3000,
    "price_usdt_cents": 450,
    "first_charge_bonus": true,
    "first_charge_bonus_diamonds": 50,
    "tags": "推荐",
    "description": "高性价比套餐",
    "sort_order": 7,
    "status": "active"
  }'
```

### 响应数据

#### 成功响应

```json
{
  "code": 0,
  "data": {
    "ID": 7,
    "diamonds": 300,
    "bonus_diamonds": 30,
    "price_cny_cents": 3000,
    "price_usdt_cents": 450,
    "first_charge_bonus": true,
    "first_charge_bonus_diamonds": 50,
    "tags": "推荐",
    "description": "高性价比套餐",
    "sort_order": 7,
    "status": "active",
    "total_diamonds": 380
  },
  "msg": "成功"
}
```

**说明**: `total_diamonds = 300 + 30 + 50 = 380`

#### 失败响应

```json
{
  "code": 7,
  "data": {},
  "msg": "标签只能是：不设置标签、热门、推荐、超值（支持多选，逗号分隔），当前无效标签: 无效标签"
}
```

**其他可能的错误**：
- `"参数验证失败: Key: 'CreateDiamondReq.diamonds' Error:Field validation for 'diamonds' failed on the 'required' tag"` - diamonds字段必填
- `"参数验证失败: Key: 'CreateDiamondReq.status' Error:Field validation for 'status' failed on the 'oneof' tag"` - status只能是active或inactive

---

## 4. 获取套餐详情

根据ID获取单个钻石套餐的详细信息

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds/:id`
- **请求方式**: `GET`
- **是否需要认证**: 否（本地测试）

### 请求参数

#### Path参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | int | 是 | 套餐ID |

#### 请求示例

```bash
GET /api/v1/oms/diamonds/2
```

### 响应数据

#### 成功响应

```json
{
  "code": 0,
  "data": {
    "ID": 2,
    "CreatedAt": "2025-12-30T15:32:29+08:00",
    "UpdatedAt": "2025-12-30T15:32:29+08:00",
    "diamonds": 500,
    "bonus_diamonds": 50,
    "price_cny_cents": 5000,
    "price_usdt_cents": 750,
    "first_charge_bonus": true,
    "first_charge_bonus_diamonds": 100,
    "tags": "推荐",
    "description": "新手推荐、超值大礼包",
    "sort_order": 2,
    "status": "active",
    "total_diamonds": 650
  },
  "msg": "成功"
}
```

**说明**: `total_diamonds = 500 + 50 + 100 = 650`

#### 失败响应（不存在）

```json
{
  "code": 7,
  "data": {},
  "msg": "钻石套餐不存在"
}
```

---

## 5. 更新套餐

更新已有钻石套餐的信息（支持部分更新）

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds/:id`
- **请求方式**: `PUT`
- **是否需要认证**: 否（本地测试）
- **Content-Type**: `application/json`

### 请求参数

#### Path参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | int | 是 | 套餐ID |

#### Body参数（JSON）

**说明**: 所有字段都是可选的，只传需要更新的字段即可

| 参数名 | 类型 | 必填 | 校验规则 | 说明 |
|--------|------|------|----------|------|
| diamonds | int | 否 | ≥1 | 基础钻石数量 |
| bonus_diamonds | int | 否 | ≥0 | 赠送钻石数量 |
| price_cny_cents | int | 否 | ≥1 | 人民币价格（分） |
| price_usdt_cents | int | 否 | ≥1 | USDT价格（分） |
| first_charge_bonus | bool | 否 | - | 首充奖励开关 |
| first_charge_bonus_diamonds | int | 否 | ≥0 | 首充赠送钻石 |
| tags | string | 否 | 见下方说明 | 标签，支持多选（逗号分隔），可为空 |
| description | string | 否 | - | 备注说明 |
| sort_order | int | 否 | - | 排序值 |
| status | string | 否 | active/inactive | 状态 |
| meta | string | 否 | - | 扩展字段 |

**标签 (tags) 字段说明**：
- 可选字段，可以不传或传空字符串
- 有效标签：**"不设置标签"**、**"热门"**、**"推荐"**、**"超值"**
- 支持多选，使用逗号分隔，如：`"热门,推荐"`
- 示例：
  - 不设置：`""` 或不传该字段
  - 单个标签：`"热门"`
  - 多个标签：`"热门,推荐,超值"`
  - 明确标记：`"不设置标签"`

#### 请求示例

```bash
# 示例1: 只更新价格
curl -X PUT "http://localhost:10100/api/v1/oms/diamonds/1" \
  -H "Content-Type: application/json" \
  -d '{
    "price_cny_cents": 990
  }'

# 示例2: 更新多个字段（含多选标签）
curl -X PUT "http://localhost:10100/api/v1/oms/diamonds/2" \
  -H "Content-Type: application/json" \
  -d '{
    "tags": "热门,推荐",
    "sort_order": 1,
    "first_charge_bonus_diamonds": 150
  }'

# 示例3: 下架套餐
curl -X PUT "http://localhost:10100/api/v1/oms/diamonds/3" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "inactive"
  }'
```

### 响应数据

#### 成功响应

```json
{
  "code": 0,
  "data": {},
  "msg": "更新成功"
}
```

#### 失败响应

```json
{
  "code": 7,
  "data": {},
  "msg": "更新钻石套餐失败"
}
```

**标签验证失败**：
```json
{
  "code": 7,
  "data": {},
  "msg": "标签只能是：不设置标签、热门、推荐、超值（支持多选，逗号分隔），当前无效标签: 无效标签"
}
```

**其他可能的错误**：
- `"参数验证失败: Key: 'UpdateDiamondReq.diamonds' Error:Field validation for 'diamonds' failed on the 'min' tag"` - 钻石数量必须≥1
- `"参数验证失败: Key: 'UpdateDiamondReq.status' Error:Field validation for 'status' failed on the 'oneof' tag"` - status只能是active或inactive

---

## 6. 删除套餐

删除指定的钻石套餐（软删除）

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds/:id`
- **请求方式**: `DELETE`
- **是否需要认证**: 否（本地测试）

### 请求参数

#### Path参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | int | 是 | 套餐ID |

#### 请求示例

```bash
DELETE /api/v1/oms/diamonds/3
```

### 响应数据

#### 成功响应

HTTP状态码：`204 No Content`

#### 失败响应

```json
{
  "code": 7,
  "data": {},
  "msg": "删除钻石套餐失败"
}
```

---

## 7. 获取汇率配置

获取USDT对CNY的汇率配置信息

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds/rate`
- **请求方式**: `GET`
- **是否需要认证**: 否（本地测试）

### 请求参数

无

### 响应数据

#### 成功响应

```json
{
  "code": 0,
  "data": {
    "key": "usdt_to_cny_rate",
    "value": {
      "rate": 6.8,
      "auto_sync": false,
      "last_updated": "2024-12-24T10:30:00+08:00"
    }
  },
  "msg": "成功"
}
```

#### 字段说明

| 字段名 | 类型 | 说明 |
|--------|------|------|
| rate | float | USDT对CNY汇率 |
| auto_sync | bool | 自动同步开关 |
| last_updated | string | 最后更新时间 |

---

## 8. 更新汇率配置

手动更新USDT汇率配置

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds/rate`
- **请求方式**: `PUT`
- **是否需要认证**: 否（本地测试）
- **Content-Type**: `application/json`

### 请求参数

| 参数名 | 类型 | 必填 | 校验规则 | 说明 |
|--------|------|------|----------|------|
| rate | float | 是 | ≥0 | USDT对CNY汇率 |
| auto_sync | bool | 否 | - | 自动同步开关，默认false |

#### 请求示例

```bash
curl -X PUT "http://localhost:10100/api/v1/oms/diamonds/rate" \
  -H "Content-Type: application/json" \
  -d '{
    "rate": 6.8,
    "auto_sync": false
  }'
```

### 响应数据

#### 成功响应

```json
{
  "code": 0,
  "data": {
    "key": "usdt_to_cny_rate",
    "value": {
      "rate": 6.8,
      "auto_sync": false,
      "last_updated": "2025-12-30T18:00:00+08:00"
    }
  },
  "msg": "成功"
}
```

#### 失败响应

```json
{
  "code": 7,
  "data": {},
  "msg": "更新汇率配置失败"
}
```

---

## 9. 同步实时汇率

从CoinGecko API同步USDT对CNY的实时汇率

### 基本信息

- **接口地址**: `/api/v1/oms/diamonds/rate/sync`
- **请求方式**: `POST`
- **是否需要认证**: 否（本地测试）

### 请求参数

无

#### 请求示例

```bash
curl -X POST "http://localhost:10100/api/v1/oms/diamonds/rate/sync"
```

### 响应数据

#### 成功响应

```json
{
  "code": 0,
  "data": {
    "key": "usdt_to_cny_rate",
    "value": {
      "rate": 7.23,
      "auto_sync": false,
      "last_updated": "2025-12-30T18:05:00+08:00"
    }
  },
  "msg": "成功"
}
```

#### 失败响应

```json
{
  "code": 7,
  "data": {},
  "msg": "获取实时汇率失败: API请求失败"
}
```

---

## 数据模型

### 钻石套餐模型 (DiamondPackage)

```go
type SysDiamondPackage struct {
    ID                       uint      // 主键ID
    CreatedAt                time.Time // 创建时间
    UpdatedAt                time.Time // 更新时间
    DeletedAt                *time.Time // 删除时间
    Diamonds                 int64     // 钻石数量
    BonusDiamonds            int64     // 赠送钻石
    PriceCnyCents            int64     // 人民币价格（分）
    PriceUsdtCents           int64     // USDT价格（分）
    FirstChargeBonus         bool      // 首充奖励开关
    FirstChargeBonusDiamonds int64     // 首充赠送钻石数量
    Tags                     *string   // 标签（支持多选，逗号分隔）
    Description              *string   // 备注说明
    SortOrder                int       // 排序值
    Status                   string    // 状态（active/inactive）
    Meta                     *string   // 扩展字段（JSON）
    TotalDiamonds            int64     // 实得钻石总数（计算字段，不存储到数据库）
}

// CalculateTotalDiamonds 计算实得钻石总数
func (p *SysDiamondPackage) CalculateTotalDiamonds() {
    p.TotalDiamonds = p.Diamonds + p.BonusDiamonds
    if p.FirstChargeBonus {
        p.TotalDiamonds += p.FirstChargeBonusDiamonds
    }
}
```

**说明**:
- `TotalDiamonds` 字段使用 `gorm:"-"` 标签，表示不存储到数据库
- API返回数据前会自动调用 `CalculateTotalDiamonds()` 方法计算实得钻石总数
- 前端直接使用 `total_diamonds` 字段即可，无需手动计算

### 汇率配置模型 (RateConfig)

```json
{
  "rate": 6.98,           // 汇率值
  "auto_sync": false,     // 自动同步开关
  "last_updated": "2025-12-30T15:56:22+09:00"  // 最后更新时间
}
```

---

## 业务规则

### 1. 标签规则

- **支持多选**：可以同时设置多个标签，使用逗号分隔
- **有效值**：
  - 空字符串 `""` - 不设置任何标签
  - `"不设置标签"` - 明确标记为不设置
  - `"热门"` - 标记为热门套餐
  - `"推荐"` - 标记为推荐套餐
  - `"超值"` - 标记为超值套餐
- **多选示例**：
  - `"热门"` - 单个标签
  - `"热门,推荐"` - 两个标签
  - `"热门,推荐,超值"` - 三个标签
- **前端建议**：使用多选下拉框，允许用户选择多个标签

### 2. 首充奖励规则

- `first_charge_bonus` = false 时，`first_charge_bonus_diamonds` 不生效
- `first_charge_bonus` = true 时，用户首次充值该套餐可额外获得 `first_charge_bonus_diamonds` 钻石

### 3. 排序规则

- `sort_order` 数值越小，套餐排序越靠前
- 建议从1开始递增，预留中间值便于后续调整

### 4. 价格规则

- 价格字段以"分"为单位存储，避免浮点数精度问题
- 人民币：10元 = 1000分
- USDT：1 USDT = 100分

### 5. 软删除规则

- 删除操作不会真正删除数据库记录
- 只是设置 `deleted_at` 字段
- 已删除的套餐不会在列表中显示

---

## 🏷️ 标签字段说明 (tags)

### 字段特性

| 特性 | 说明 |
|------|------|
| 字段名 | `tags` |
| 类型 | string |
| 是否必填 | ❌ 可选 |
| 支持多选 | ✅ 是（逗号分隔） |
| 有效值 | 空字符串、"不设置标签"、"热门"、"推荐"、"超值" |
| 示例 | `"热门,推荐"` |

### 有效标签列表

| 标签值 | 说明 | 使用场景 |
|--------|------|---------|
| `""` (空) | 不设置任何标签 | 普通套餐，无特殊标记 |
| `"不设置标签"` | 明确标记为"不设置" | 与空字符串类似，但更明确 |
| `"热门"` | 热门套餐 | 销量高、用户喜爱的套餐 |
| `"推荐"` | 推荐套餐 | 官方推荐、性价比高 |
| `"超值"` | 超值套餐 | 优惠力度大、赠送多 |

### 多选组合示例

```javascript
// 示例1: 不设置标签
{
  "tags": ""  // 或不传该字段
}

// 示例2: 使用"不设置标签"
{
  "tags": "不设置标签"
}

// 示例3: 单个标签
{
  "tags": "热门"
}

// 示例4: 两个标签
{
  "tags": "热门,推荐"
}

// 示例5: 三个标签
{
  "tags": "热门,推荐,超值"
}
```

### 前端实现建议

#### 方案1：多选下拉框（推荐）

```javascript
const tagOptions = [
  { label: '不设置标签', value: '不设置标签' },
  { label: '热门', value: '热门' },
  { label: '推荐', value: '推荐' },
  { label: '超值', value: '超值' }
];

// 用户选择后，将选中的标签用逗号连接
function handleTagChange(selectedTags) {
  const tagsString = selectedTags.join(',');
  // tagsString: "热门,推荐"
}
```

#### 方案2：标签展示

```javascript
// 接收到数据后，解析标签字符串
function parseTags(tagsString) {
  if (!tagsString) return [];
  return tagsString.split(',').map(tag => tag.trim());
}

// 使用示例
const tags = parseTags("热门,推荐"); // ["热门", "推荐"]

// 渲染标签
tags.map(tag => (
  <span className="tag">{tag}</span>
))
```

### 业务逻辑说明

#### 统计热门套餐数量

在概览统计接口中，只统计 **tags 包含"热门"** 的套餐：

```go
// 后端实现
if item.Tags != nil && strings.Contains(*item.Tags, "热门") {
    hotCount++
}
```

**注意**：
- `"热门"` - 会被统计 ✅
- `"热门,推荐"` - 会被统计 ✅
- `"推荐"` - 不会被统计 ❌
- `""` - 不会被统计 ❌

---

## 🔢 实得钻石字段说明 (total_diamonds)

### 字段特性

| 特性 | 说明 |
|------|------|
| 字段名 | `total_diamonds` |
| 类型 | int64 |
| 数据库存储 | ❌ 不存储（计算字段） |
| API返回 | ✅ 所有套餐相关接口都返回此字段 |
| 计算时机 | API返回数据前自动计算 |
| 计算公式 | `diamonds + bonus_diamonds + (first_charge_bonus ? first_charge_bonus_diamonds : 0)` |

### 返回该字段的接口

| 接口 | 说明 |
|------|------|
| `GET /diamonds` | ✅ 列表中每个套餐都包含 total_diamonds |
| `GET /diamonds/:id` | ✅ 详情中包含 total_diamonds |
| `POST /diamonds` | ✅ 创建成功后返回包含 total_diamonds |
| `PUT /diamonds/:id` | ❌ 更新接口只返回成功消息，不返回完整对象 |

### 计算示例

```javascript
// 示例1: 无首充奖励
{
  "diamonds": 100,
  "bonus_diamonds": 0,
  "first_charge_bonus": false,
  "first_charge_bonus_diamonds": 0,
  "total_diamonds": 100  // = 100 + 0 + 0
}

// 示例2: 有首充奖励
{
  "diamonds": 500,
  "bonus_diamonds": 50,
  "first_charge_bonus": true,
  "first_charge_bonus_diamonds": 100,
  "total_diamonds": 650  // = 500 + 50 + 100
}

// 示例3: 有赠送但无首充
{
  "diamonds": 2000,
  "bonus_diamonds": 400,
  "first_charge_bonus": false,
  "first_charge_bonus_diamonds": 0,
  "total_diamonds": 2400  // = 2000 + 400 + 0
}
```

### 前端使用建议

```javascript
// ✅ 推荐：直接使用 total_diamonds
<td>{item.total_diamonds}</td>

// ❌ 不推荐：前端手动计算
const total = item.diamonds + item.bonus_diamonds +
  (item.first_charge_bonus ? item.first_charge_bonus_diamonds : 0);
<td>{total}</td>
```

---

## 错误码说明

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 7 | 业务错误（参数验证失败、数据不存在等） |

### 常见错误消息

| 错误消息 | 原因 | 解决方案 |
|----------|------|---------|
| 参数验证失败 | 请求参数不符合验证规则 | 检查参数类型和值是否正确 |
| 钻石套餐不存在 | 查询的ID不存在 | 确认ID是否正确 |
| 标签验证失败 | tags字段值不在允许列表 | 使用有效的标签值 |
| 创建/更新钻石套餐失败 | 数据库操作失败 | 检查数据库连接和日志 |
| 获取实时汇率失败 | CoinGecko API调用失败 | 检查网络或使用手动更新 |

---

## 技术支持

如有问题，请查看：
- 快速启动指南：`docs/DIAMOND_CONFIG_QUICK_START.md`
- SQL初始化脚本：`sql/20251230/`

祝使用愉快！🎉
