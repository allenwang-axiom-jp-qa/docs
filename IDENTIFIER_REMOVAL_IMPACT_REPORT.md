# Identifier 字段移除影响分析报告

**变更日期:** 2026-01-08
**变更范围:** 标签管理系统（Tag & TagCategory）
**提交记录:** d3ab43a - refactor: 移除标签管理的 identifier 字段

---

## 一、变更概述

从标签（Tag）和标签分类（TagCategory）中移除 `identifier` 字段，简化系统设计，降低维护复杂度。

### 变更原因
1. identifier 字段与 name 字段功能重复
2. 实际使用中主要依赖 name 字段进行标识
3. 减少唯一性约束可以简化代码逻辑
4. 降低前端使用复杂度

---

## 二、代码变更清单

### 已修改文件（已提交）

1. **数据模型层**
   - `internal/model/tag.go` - 移除 Identifier 字段
   - `internal/model/tag_category.go` - 移除 Identifier 字段

2. **数据访问层**
   - `internal/data/tag_repo.go`
     - 移除 `CheckTagIdentifierExists()` 方法
     - 移除 `GetDeletedTagByIdentifier()` 方法
     - 修改 `CreateTag()` - 移除 identifier 验证逻辑
     - 修改 `UpdateTag()` - 移除 identifier 重复检查
     - 修改 `ListTags()` - 关键词搜索不再包含 identifier

   - `internal/data/tag_category_repo.go`
     - 移除 `GetTagCategoryByIdentifier()` 方法
     - 修改 `CreateTagCategory()` - 移除 identifier 验证
     - 修改 `UpdateTagCategory()` - 移除 identifier 检查

3. **服务层**
   - `internal/service/tag.go` - 移除创建和更新时的 identifier 处理

4. **API层**
   - `internal/api/tag.go`
     - `ResTag` - 移除 Identifier 字段
     - `ReqCreateTag` - 移除 Identifier 字段
     - `ReqUpdateTag` - 移除 Identifier 字段

   - `internal/api/tag_category.go`
     - `ResTagCategory` - 移除 Identifier 字段
     - `ReqCreateTagCategory` - 移除 Identifier 字段
     - `ReqUpdateTagCategory` - 移除 Identifier 字段

5. **转换器**
   - `internal/converter/converter.go` - 移除 Identifier 字段映射

6. **数据库迁移**
   - `migrate/003_init_cms_tag.sql` - 移除 identifier 列定义和索引
   - `migrate/008_create_tag_category.sql` - 移除 identifier 列定义和索引
   - `migrate/009_drop_identifier_fields.sql` - **新增**删除现有表的 identifier 列

---

## 三、功能影响分析

### ✅ 不受影响的功能

#### 标签管理
- ✅ 标签列表查询（使用 name 或 description 搜索）
- ✅ 标签详情查询（使用 ID）
- ✅ 标签创建（仅需 name + categoryId）
- ✅ 标签更新（基于 ID）
- ✅ 标签删除（基于 ID）
- ✅ 标签排序（使用 sort_order）
- ✅ 标签统计

#### 标签分类管理
- ✅ 分类列表查询
- ✅ 分类详情查询
- ✅ 分类创建（仅需 name）
- ✅ 分类更新
- ✅ 分类删除
- ✅ 分类统计

#### 视频-标签关联
- ✅ 视频标签关联（使用 tag.id）
- ✅ 根据标签筛选视频
- ✅ 标签使用次数统计

### 📝 变更的行为

#### 1. 唯一性约束
**变更前:**
- Tag: name（唯一） + identifier（唯一）
- TagCategory: name + identifier（唯一）

**变更后:**
- Tag: name（唯一）
- TagCategory: name

**影响:**
- 简化了重复性检查逻辑
- 创建标签时不再需要考虑 identifier 冲突

#### 2. API 请求/响应格式

**创建标签 - 请求变化**
```json
// 变更前
{
  "name": "动作片",
  "identifier": "action",  // ❌ 不再需要
  "categoryId": 1
}

// 变更后
{
  "name": "动作片",
  "categoryId": 1
}
```

**标签列表 - 响应变化**
```json
// 变更前
{
  "id": 1,
  "tagId": "TAG001",
  "name": "动作片",
  "identifier": "action",  // ❌ 不再返回
  "categoryId": 1
}

// 变更后
{
  "id": 1,
  "tagId": "TAG001",
  "name": "动作片",
  "categoryId": 1
}
```

#### 3. 关键词搜索行为

**变更前:**
```sql
WHERE (name LIKE '%keyword%' OR identifier LIKE '%keyword%' OR description LIKE '%keyword%')
```

**变更后:**
```sql
WHERE (name LIKE '%keyword%' OR description LIKE '%keyword%')
```

**影响:**
- 搜索不再匹配 identifier，但实际使用中影响很小
- 用户主要通过 name 搜索标签

---

## 四、数据库变更

### 当前数据库状态

**t_video_tag 表:**
- 包含 8 条标签数据
- identifier 字段有值（如: high-rated, new-2025, classic, trending, action）
- identifier 字段有唯一索引 `uk_identifier`

**t_video_tag_category 表:**
- 包含 6 条分类数据
- identifier 字段有值
- identifier 字段有唯一索引 `uk_identifier`

### 需要执行的 SQL

**迁移脚本:** `migrate/009_drop_identifier_fields.sql`

```sql
-- 1. 删除 t_video_tag 表的 identifier
ALTER TABLE `t_video_tag` DROP INDEX `uk_identifier`;
ALTER TABLE `t_video_tag` DROP COLUMN `identifier`;

-- 2. 删除 t_video_tag_category 表的 identifier
ALTER TABLE `t_video_tag_category` DROP INDEX `uk_identifier`;
ALTER TABLE `t_video_tag_category` DROP COLUMN `identifier`;
```

### 执行注意事项

1. ⚠️ **不可逆操作** - 删除列后数据无法恢复
2. ✅ **应用代码已就绪** - 代码已不再使用 identifier 字段
3. 📅 **建议执行时机** - 在应用部署完成后执行
4. 🔒 **数据备份** - 建议执行前备份数据库

---

## 五、前端影响

### 需要前端配合的变更

#### 1. 创建标签接口调整

**接口:** `POST /api/v1/site-svc-video-mgmt/tag`

**请求参数变化:**
```typescript
// 变更前
interface CreateTagRequest {
  name: string;
  identifier: string;  // ❌ 移除
  categoryId: number;
  tagType?: number;
  status?: number;
  description?: string;
}

// 变更后
interface CreateTagRequest {
  name: string;
  categoryId: number;
  tagType?: number;
  status?: number;
  description?: string;
}
```

#### 2. 更新标签接口调整

**接口:** `PUT /api/v1/site-svc-video-mgmt/tag`

**请求参数变化:**
```typescript
// 变更前
interface UpdateTagRequest {
  name?: string;
  identifier?: string;  // ❌ 移除
  categoryId?: number;
  tagType?: number;
  status?: number;
  description?: string;
}

// 变更后
interface UpdateTagRequest {
  name?: string;
  categoryId?: number;
  tagType?: number;
  status?: number;
  description?: string;
}
```

#### 3. 标签响应数据变化

**所有返回标签数据的接口:**
- `GET /api/v1/site-svc-video-mgmt/tags` - 标签列表
- `GET /api/v1/site-svc-video-mgmt/tag` - 标签详情

```typescript
// 变更前
interface Tag {
  id: number;
  tagId: string;
  name: string;
  identifier: string;  // ❌ 移除
  categoryId?: number;
  categoryName?: string;
  // ... 其他字段
}

// 变更后
interface Tag {
  id: number;
  tagId: string;
  name: string;
  categoryId?: number;
  categoryName?: string;
  // ... 其他字段
}
```

#### 4. 标签分类接口类似变更

创建和更新标签分类的接口也移除了 `identifier` 参数。

---

## 六、测试建议

### 后端测试

1. **单元测试**
   - ✅ 代码编译通过
   - ✅ 无 identifier 字段引用

2. **集成测试**
   - [ ] 测试标签 CRUD 操作
   - [ ] 测试标签分类 CRUD 操作
   - [ ] 测试标签搜索功能
   - [ ] 测试标签唯一性验证（仅 name）

3. **数据迁移测试**
   - [ ] 在测试环境执行 009_drop_identifier_fields.sql
   - [ ] 验证表结构正确
   - [ ] 验证应用功能正常

### 前端测试

1. **接口兼容性测试**
   - [ ] 创建标签（不传 identifier）
   - [ ] 更新标签（不传 identifier）
   - [ ] 标签列表展示（不依赖 identifier）
   - [ ] 标签搜索功能

2. **UI 测试**
   - [ ] 标签管理页面功能正常
   - [ ] 标签分类管理页面功能正常
   - [ ] 视频编辑页面的标签选择正常

---

## 七、部署计划

### 部署步骤

1. **代码部署**
   ```bash
   # 代码已推送到 dev 分支
   git push origin dev

   # CI/CD 自动构建部署
   # 等待 GitHub Actions 完成
   ```

2. **验证应用启动**
   ```bash
   # 检查 pod 状态
   kubectl get pods -n x-dev | grep cms-site-svc-video-mgmt

   # 检查应用日志
   kubectl logs -n x-dev <pod-name>
   ```

3. **执行数据库迁移**
   ```bash
   # 连接数据库
   kubectl exec -it -n x-dev <mysql-pod> -- mysql -u root -p

   # 切换数据库
   USE cms_video;

   # 执行迁移脚本
   SOURCE /path/to/migrate/009_drop_identifier_fields.sql;

   # 验证表结构
   DESCRIBE t_video_tag;
   DESCRIBE t_video_tag_category;
   ```

4. **功能验证**
   - 测试标签创建
   - 测试标签列表
   - 测试标签搜索
   - 测试标签编辑

### 回滚方案

如果发现问题需要回滚：

1. **代码回滚**
   ```bash
   # 回滚到上一个版本
   git revert d3ab43a
   git push origin dev
   ```

2. **数据库回滚**（较复杂，因为已删除列）
   ```sql
   -- 重新添加 identifier 列（需要从备份恢复数据）
   ALTER TABLE t_video_tag ADD COLUMN identifier varchar(100);
   ALTER TABLE t_video_tag ADD UNIQUE KEY uk_identifier (identifier);

   -- 需要从备份恢复 identifier 数据
   ```

---

## 八、总结

### ✅ 变更优势

1. **降低复杂度** - 减少一个唯一性约束字段
2. **简化代码** - 减少验证逻辑和查询方法
3. **提升性能** - 减少一个索引，略微提升写入性能
4. **易于使用** - 前端创建标签时少传一个参数

### ⚠️ 注意事项

1. **前后端协调** - 需要前端同步更新接口调用
2. **数据不可逆** - 删除列后原有 identifier 数据丢失
3. **搜索范围缩小** - 关键词搜索不再匹配 identifier

### 📊 风险评估

- **风险等级:** 🟢 低
- **影响范围:** 标签管理模块
- **回滚难度:** 🟡 中等（需要恢复数据）
- **建议执行:** ✅ 可以执行

---

**审核人员签字区:**
- [ ] 后端负责人
- [ ] 前端负责人
- [ ] DBA
- [ ] 测试负责人
