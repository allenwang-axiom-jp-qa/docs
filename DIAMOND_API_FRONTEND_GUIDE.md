# 钻石配置模块 - 前端联调文档

## 📋 部署状态

### Dev环境信息
- **服务地址**: `https://dev-admin.beauty-666.com`
- **API前缀**: `/api/v1/oms`
- **K8s Pod**: `cmsoms-76cd68fdfd-lf29q` (namespace: `x-dev`)
- **最新镜像**: `ghcr.io/axiom888/cms-site-svc-oms:dev-latest`
- **最新提交**: `81879c2` - 钻石配置路由注册修复部署

### 部署时间线
1. **2026-01-05 20:59** - 合并钻石配置到dev分支
2. **2026-01-05 21:27** - 统一表名使用oms_前缀
3. **2026-01-06 09:40** - 修复路由注册问题
4. **2026-01-06 09:47** - 触发CI/CD构建新镜像
5. **2026-01-06 09:49** - ✅ 新镜像已部署，API正常运行

## 🗄️ 数据库信息

### Dev环境数据库
- **主机**: `10.1.160.41:3306`
- **数据库**: `cms_oms`
- **表名**:
  - `oms_diamond_packages` - 钻石套餐表
  - `oms_diamond_configs` - 钻石配置表

### 已有测试数据
- ✅ 6个钻石套餐（¥10 ~ ¥999）
- ✅ USDT汇率配置

## 📡 API接口文档

### 基础URL
```
https://dev-admin.beauty-666.com/api/v1/oms/diamonds
```

---

## 1. 查询类接口

### 1.1 获取钻石套餐概览统计
```http
GET /api/v1/oms/diamonds/overview
```

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "total_count": 6,
    "active_count": 6,
    "hot_count": 2,
    "average_price": 296.5
  },
  "msg": "success"
}
```

---

### 1.2 获取钻石套餐列表（分页）
```http
GET /api/v1/oms/diamonds?page=1&pageSize=10&status=1
```

**查询参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页数量，默认20 |
| status | int | 否 | 状态筛选：0=下架 1=上架 |

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1,
        "diamonds": 100,
        "bonus_diamonds": 0,
        "price_cny_cents": 1000,
        "price_usdt_cents": 150,
        "first_charge_bonus": false,
        "first_charge_bonus_diamonds": 0,
        "tags": 0,
        "description": "新手推荐",
        "sort_order": 1,
        "status": 1,
        "total_diamonds": 100,
        "created_at": "2026-01-06T10:00:00Z",
        "updated_at": "2026-01-06T10:00:00Z"
      }
    ],
    "total": 6,
    "page": 1,
    "pageSize": 10
  },
  "msg": "success"
}
```

**字段说明**:
- `diamonds`: 基础钻石数量
- `bonus_diamonds`: 额外赠送钻石
- `price_cny_cents`: 人民币价格（分），需除以100显示为元
- `price_usdt_cents`: USDT价格（分），需除以100
- `first_charge_bonus`: 是否开启首充奖励
- `first_charge_bonus_diamonds`: 首充额外赠送钻石
- `tags`: 标签（0=无 1=热门 2=推荐 3=超值）
- `status`: 状态（0=下架 1=上架）
- `total_diamonds`: 实得钻石总数（计算字段）

---

### 1.3 获取单个套餐详情
```http
GET /api/v1/oms/diamonds/:id
```

**响应示例**:
```json
{
  "code": 0,
  "data": {
    "id": 1,
    "diamonds": 100,
    "total_diamonds": 100,
    ...
  },
  "msg": "success"
}
```

---

## 2. 管理类接口

### 2.1 创建钻石套餐
```http
POST /api/v1/oms/diamonds
```

**请求体**:
```json
{
  "diamonds": 2000,
  "bonus_diamonds": 200,
  "price_cny_cents": 20000,
  "price_usdt_cents": 3000,
  "first_charge_bonus": false,
  "first_charge_bonus_diamonds": 0,
  "tags": 2,
  "description": "测试套餐",
  "sort_order": 100,
  "status": 1
}
```

**验证规则**:
- ✅ `diamonds`: 必填，1-10,000,000
- ✅ `bonus_diamonds`: 0-20,000,000，不超过基础钻石2倍
- ✅ `price_cny_cents`: 必填，1-10,000,000（1分-10万元）
- ✅ `price_usdt_cents`: 必填，1-1,000,000（0.01-1万USDT）
- ✅ `tags`: 必填，0-3（单选）
- ✅ `status`: 必填，0或1
- ✅ 首充关闭时，`first_charge_bonus_diamonds`必须为0
- ✅ 首充赠送不超过基础钻石

---

### 2.2 更新钻石套餐
```http
PUT /api/v1/oms/diamonds/:id
```

**请求体** (所有字段可选):
```json
{
  "price_cny_cents": 1200,
  "status": 0
}
```

---

### 2.3 删除钻石套餐
```http
DELETE /api/v1/oms/diamonds/:id
```

**注意**: 软删除，不会真正删除数据

---

## 3. 汇率配置接口

### 3.1 获取USDT汇率配置
```http
GET /api/v1/oms/diamonds/rate
```

**响应示例**:
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
  "msg": "success"
}
```

---

### 3.2 更新USDT汇率
```http
PUT /api/v1/oms/diamonds/rate
```

**请求体**:
```json
{
  "rate": 7.2,
  "auto_sync": false
}
```

---

### 3.3 同步USDT汇率（从CoinGecko）
```http
POST /api/v1/oms/diamonds/rate/sync
```

**功能**: 自动从CoinGecko API获取最新USDT对CNY汇率

---

## 🎨 前端展示建议

### 标签映射
```javascript
const TAG_LABELS = {
  0: { text: '无标签', color: 'default' },
  1: { text: '热门', color: 'red' },
  2: { text: '推荐', color: 'blue' },
  3: { text: '超值', color: 'orange' }
}
```

### 状态映射
```javascript
const STATUS_LABELS = {
  0: { text: '已下架', color: 'gray' },
  1: { text: '已上架', color: 'green' }
}
```

### 价格格式化
```javascript
// 分转元
const formatPrice = (cents) => (cents / 100).toFixed(2)

// 示例
formatPrice(1000) // "10.00"
```

### 实得钻石计算
```javascript
const calculateTotalDiamonds = (pkg) => {
  let total = pkg.diamonds + pkg.bonus_diamonds
  if (pkg.first_charge_bonus) {
    total += pkg.first_charge_bonus_diamonds
  }
  return total
}
```

---

## 🧪 测试建议

### 1. 测试数据验证
- 尝试创建赠送钻石超过2倍的套餐（应失败）
- 尝试首充关闭但赠送不为0（应失败）
- 尝试首充赠送超过基础钻石（应失败）

### 2. 测试状态切换
- 上架 → 下架
- 下架 → 上架

### 3. 测试分页
- 不同pageSize
- 不同page
- status筛选

---

## 📞 联系方式

遇到问题请联系：
- 后端负责人：Allen Wang
- 部署问题：DevOps团队

---

## 🔄 部署检查清单

✅ **所有部署验证已完成**

1. ✅ 访问概览API：`GET /api/v1/oms/diamonds/overview` - 返回6个套餐统计
2. ✅ 访问列表API：`GET /api/v1/oms/diamonds` - 成功返回6个测试套餐
3. ✅ 访问汇率API：`GET /api/v1/oms/diamonds/rate` - USDT汇率配置正常
4. ✅ 数据库已部署：oms_diamond_packages, oms_diamond_configs
5. ✅ 路由已注册：所有10个API端点正常工作

**API可用性**: 🟢 所有接口正常，可以开始前端联调

---

**最后更新**: 2026-01-06 09:50
**文档版本**: v1.1
