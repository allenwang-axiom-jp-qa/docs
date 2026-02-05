# 推荐算法配置 API 文档

## 概览

推荐算法配置模块提供两种独立的权重配置管理，用于优化视频推荐效果。

## 配置类型

### 1. 用户行为权重 (User Behavior Weight)

**用途**：根据用户不同的行为类型，设置对应的得分权重。数值越大，该行为对推荐的影响越大。

**配置键**：`recommend_user_behavior_weights`

**字段说明**：

| 字段 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| view_valid | int | 有效观看得分（观看超过30秒） | 1 |
| view_complete | int | 完播得分 | 3 |
| like | int | 点赞得分 | 5 |
| favorite | int | 收藏得分 | 8 |
| share | int | 分享得分 | 10 |
| pay | int | 付款得分 | 20 |

**权重预览**：
- 有效观看：1.0
- 完播：3.0
- 点赞：5.0
- 收藏：8.0
- 分享：10.0
- 付款：20.0
- **总计**：47.0

**配置建议**：
- 建议按价值递增设置：观看 < 点赞 < 收藏 < 分享 < 付款
- 如：观看 < 点赞 < 收藏 < 分享 < 付款
- 付款行为价值最高，建议设置最高分数

---

### 2. 热度召回权重 (Hot Weight)

**用途**：设置热度计算时各项指标的权重比例。所有权重之和必须等于 1.0。

**配置键**：`recommend_hot_weights`

**字段说明**：

| 字段 | 类型 | 说明 | 默认值 | 占比 |
|------|------|------|--------|------|
| like | float | 点赞权重 | 0.5 | 50% |
| share | float | 分享权重 | 0.3 | 30% |
| favorite | float | 收藏权重 | 0.2 | 20% |

**权重验证**：
- 点赞：0.50 + 分享：0.30 + 收藏：0.20 = **1.00** ✅

**配置建议**：
- 所有权重之和必须等于 1.0
- 建议根据业务目标调整比例
- 点赞通常是最常见的互动，建议权重最高

---

## API 端点

### 获取推荐配置

```http
GET /api/v1/site-svc-video-mgmt/recommend/config
```

**响应示例**：

```json
{
  "code": 0,
  "data": {
    "userBehaviorWeights": {
      "view_valid": 1,
      "view_complete": 3,
      "like": 5,
      "favorite": 8,
      "share": 10,
      "pay": 20
    },
    "hotWeights": {
      "like": 0.5,
      "share": 0.3,
      "favorite": 0.2
    },
    "hotWeightsSumValid": true
  },
  "msg": "获取推荐配置成功"
}
```

**响应字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | int | 业务状态码（0表示成功） |
| data.userBehaviorWeights | object | 用户行为权重配置 |
| data.hotWeights | object | 热度召回权重配置 |
| data.hotWeightsSumValid | boolean | 热度权重总和是否有效（是否等于1.0） |
| msg | string | 响应消息 |

---

### 更新推荐配置

```http
PUT /api/v1/site-svc-video-mgmt/recommend/config
```

**请求体（更新用户行为权重）**：

```json
{
  "configType": "user_behavior_weights",
  "value": {
    "view_valid": 1,
    "view_complete": 3,
    "like": 5,
    "favorite": 8,
    "share": 10,
    "pay": 20
  },
  "updatedBy": "admin"
}
```

**请求体（更新热度召回权重）**：

```json
{
  "configType": "hot_weights",
  "value": {
    "like": 0.5,
    "share": 0.3,
    "favorite": 0.2
  },
  "updatedBy": "admin"
}
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| configType | string | 是 | 配置类型：`user_behavior_weights` 或 `hot_weights` |
| value | object | 是 | 配置值（JSON对象） |
| updatedBy | string | 是 | 更新人 |

**响应示例（成功）**：

```json
{
  "code": 0,
  "data": {
    "configType": "user_behavior_weights",
    "value": {
      "view_valid": 1,
      "view_complete": 3,
      "like": 5,
      "favorite": 8,
      "share": 10,
      "pay": 20
    }
  },
  "msg": "更新推荐配置成功"
}
```

**错误响应**：

```json
{
  "code": 40001,
  "msg": "参数验证失败: Key: 'ReqUpdateRecommendWeights.ConfigType' Error:Field validation for 'ConfigType' failed on the 'required' tag"
}
```

```json
{
  "code": 50001,
  "msg": "hot weights sum must equal 1.0"
}
```

---

## 错误码说明

| 错误码 | 含义 | HTTP状态码 |
|--------|------|-----------|
| 0 | 成功 | 200 |
| 40001 | 参数验证失败 | 400 |
| 50001 | 服务器内部错误 | 500 |

---

## 验证规则

### 用户行为权重验证

✅ **必填字段**：
- view_valid
- view_complete
- like
- favorite
- share
- pay

❌ **错误示例**：
```json
{
  "configType": "user_behavior_weights",
  "value": {
    "like": 5,
    "share": 10
  }
}
```
错误响应：
```json
{
  "code": 50001,
  "msg": "invalid user behavior weights: missing required field: view_valid"
}
```

---

### 热度召回权重验证

✅ **权重总和必须等于 1.0**（允许 ±0.0001 误差）

❌ **错误示例**：
```json
{
  "configType": "hot_weights",
  "value": {
    "like": 0.5,
    "share": 0.3,
    "favorite": 0.3
  }
}
```
错误响应：
```json
{
  "code": 50001,
  "msg": "hot weights sum must equal 1.0"
}
```

---

## 前端集成示例

### JavaScript/TypeScript

```javascript
// 获取推荐配置
async function getRecommendConfig() {
  const response = await fetch('/api/v1/site-svc-video-mgmt/recommend/config');
  const result = await response.json();

  if (result.code === 0) {
    console.log('用户行为权重:', result.data.userBehaviorWeights);
    console.log('热度召回权重:', result.data.hotWeights);
    console.log('权重总和是否有效:', result.data.hotWeightsSumValid);
    return result.data;
  } else {
    throw new Error(result.msg);
  }
}

// 更新用户行为权重
async function updateUserBehaviorWeights() {
  const response = await fetch('/api/v1/site-svc-video-mgmt/recommend/config', {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      configType: 'user_behavior_weights',
      value: {
        view_valid: 1,
        view_complete: 3,
        like: 5,
        favorite: 8,
        share: 10,
        pay: 20
      },
      updatedBy: 'admin'
    })
  });

  const result = await response.json();
  if (result.code === 0) {
    return result.data;
  } else {
    throw new Error(result.msg);
  }
}

// 更新热度召回权重（带验证）
async function updateHotWeights(like, share, favorite) {
  // 前端验证
  const sum = like + share + favorite;
  if (Math.abs(sum - 1.0) > 0.0001) {
    throw new Error(`权重总和必须为1.0，当前为: ${sum}`);
  }

  const response = await fetch('/api/v1/site-svc-video-mgmt/recommend/config', {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      configType: 'hot_weights',
      value: { like, share, favorite },
      updatedBy: 'admin'
    })
  });

  const result = await response.json();
  if (result.code === 0) {
    return result.data;
  } else {
    throw new Error(result.msg);
  }
}
```

---

## 数据库表结构

### t_recommend_config

```sql
CREATE TABLE IF NOT EXISTS `t_recommend_config` (
    `id` BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    `config_key` VARCHAR(128) NOT NULL COMMENT '配置键',
    `config_value` TEXT COMMENT '配置值（JSON格式）',
    `description` VARCHAR(255) DEFAULT NULL COMMENT '配置描述',
    `created_by` VARCHAR(64) NOT NULL COMMENT '创建人',
    `created_time` TIMESTAMP(3) DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
    `last_updated_by` VARCHAR(64) NOT NULL COMMENT '最后更新人',
    `last_updated_time` TIMESTAMP(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '最后更新时间',
    UNIQUE KEY `uk_config_key` (`config_key`),
    INDEX `idx_created_time` (`created_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='推荐算法配置表';
```

---

## 注意事项

1. **权重调整建议**：
   - 定期根据用户行为数据调整权重
   - 监控推荐效果，迭代优化配置

2. **热度权重验证**：
   - 后端强制验证总和为1.0
   - 前端建议添加实时验证和提示

3. **配置生效**：
   - 配置更新后立即生效
   - 建议在低峰期调整配置

4. **数据一致性**：
   - 使用事务确保配置更新的原子性
   - 保留配置更新历史便于回滚

---

## 版本历史

### v1.1 (2026-01-07)
- ✅ 统一API响应格式为 `{code, data, msg}`
- ✅ JSON字段命名改为 camelCase (`configType`, `updatedBy`)
- ✅ 添加错误码常量 (0, 40001, 50001)
- ✅ 优化错误响应格式
- ✅ 更新前端集成示例

### v1.0 (2026-01-07)
- 初始版本

---

**最后更新**: 2026-01-07
**文档版本**: v1.1
**API版本**: v1
