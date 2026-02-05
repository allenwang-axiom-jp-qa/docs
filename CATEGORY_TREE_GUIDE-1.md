# 分类层级结构实现指南

## 📋 目录
1. [层级设计原理](#层级设计原理)
2. [API使用说明](#api使用说明)
3. [前端实现示例](#前端实现示例)
4. [常见场景](#常见场景)

---

## 层级设计原理

### 数据库结构

```sql
CREATE TABLE video_category (
    id BIGINT UNSIGNED PRIMARY KEY,
    name VARCHAR(64) NOT NULL,
    parent_id BIGINT UNSIGNED NOT NULL DEFAULT 0,  -- 核心字段
    sort INT NOT NULL DEFAULT 0,
    status TINYINT NOT NULL DEFAULT 1,
    ...
);
```

**关键字段 `parent_id`**：
- `parent_id = 0` → 一级分类（顶级）
- `parent_id = 某ID` → 该ID的子分类

### 层级示例

```
影视 (id=12, parent_id=0, level=1)          ← 一级分类
├── 动作片 (id=13, parent_id=12, level=2)    ← 二级分类
│   ├── 枪战 (id=15, parent_id=13, level=3)  ← 三级分类
│   └── 武术 (id=16, parent_id=13, level=3)  ← 三级分类
└── 科幻片 (id=14, parent_id=12, level=2)    ← 二级分类
```

---

## API使用说明

### 1. 查询所有一级分类

**请求**：
```bash
GET /api/v1/x/categories?parentId=0&status=1
```

**响应**：
```json
{
  "total": 2,
  "page": 1,
  "list": [
    {
      "id": 12,
      "name": "影视",
      "nameCn": "影视",
      "nameEn": "Video",
      "parentId": 0,        ← 注意：parentId=0表示一级
      "sort": 100,
      "status": 1
    }
  ]
}
```

### 2. 查询某个分类的直接子分类

**查询"影视"(id=12)的子分类**：
```bash
GET /api/v1/x/categories?parentId=12&status=1
```

**响应**：
```json
{
  "total": 2,
  "page": 1,
  "list": [
    {
      "id": 13,
      "name": "动作片",
      "parentId": 12,      ← 父分类是"影视"
      "sort": 90,
      "status": 1
    },
    {
      "id": 14,
      "name": "科幻片",
      "parentId": 12,      ← 父分类是"影视"
      "sort": 85,
      "status": 1
    }
  ]
}
```

### 3. 查询某个三级分类

**查询"动作片"(id=13)的子分类**：
```bash
GET /api/v1/x/categories?parentId=13&status=1
```

**响应**：
```json
{
  "total": 2,
  "page": 1,
  "list": [
    {
      "id": 15,
      "name": "枪战",
      "parentId": 13,      ← 父分类是"动作片"
      "sort": 80,
      "status": 1
    },
    {
      "id": 16,
      "name": "武术",
      "parentId": 13,      ← 父分类是"动作片"
      "sort": 75,
      "status": 1
    }
  ]
}
```

### 4. 创建子分类

**在"影视"下创建新的二级分类**：
```bash
POST /api/v1/x/category
Content-Type: application/json

{
  "name": "喜剧片",
  "nameCn": "喜剧片",
  "nameEn": "Comedy",
  "parentId": 12,        ← 设置父分类ID
  "sort": 80,
  "status": 1,
  "description": "喜剧类影片"
}
```

---

## 前端实现示例

### 方案1：递归构建树形结构（推荐）

```javascript
// 1. 获取所有分类
const response = await fetch('/api/v1/x/categories?pageSize=1000&status=1');
const data = await response.json();
const allCategories = data.list;

// 2. 构建树形结构函数
function buildTree(categories, parentId = 0) {
  return categories
    .filter(cat => cat.parentId === parentId)
    .map(cat => ({
      ...cat,
      children: buildTree(categories, cat.id),
      level: parentId === 0 ? 1 : calculateLevel(categories, cat.id)
    }))
    .sort((a, b) => b.sort - a.sort); // 按sort降序排序
}

// 3. 计算层级深度
function calculateLevel(categories, categoryId) {
  let level = 1;
  let current = categories.find(c => c.id === categoryId);
  while (current && current.parentId !== 0) {
    level++;
    current = categories.find(c => c.id === current.parentId);
  }
  return level;
}

// 4. 使用
const tree = buildTree(allCategories);
console.log(tree);
```

**输出结构**：
```javascript
[
  {
    id: 12,
    name: "影视",
    parentId: 0,
    level: 1,
    children: [
      {
        id: 13,
        name: "动作片",
        parentId: 12,
        level: 2,
        children: [
          {
            id: 15,
            name: "枪战",
            parentId: 13,
            level: 3,
            children: []
          },
          {
            id: 16,
            name: "武术",
            parentId: 13,
            level: 3,
            children: []
          }
        ]
      },
      {
        id: 14,
        name: "科幻片",
        parentId: 12,
        level: 2,
        children: []
      }
    ]
  }
]
```

### 方案2：按需加载（适合大量数据）

```javascript
// 1. 初始加载一级分类
async function loadRootCategories() {
  const response = await fetch('/api/v1/x/categories?parentId=0&status=1');
  const data = await response.json();
  return data.list;
}

// 2. 点击展开时加载子分类
async function loadChildren(parentId) {
  const response = await fetch(`/api/v1/x/categories?parentId=${parentId}&status=1`);
  const data = await response.json();
  return data.list;
}

// 3. React示例
function CategoryTree() {
  const [categories, setCategories] = useState([]);

  useEffect(() => {
    loadRootCategories().then(setCategories);
  }, []);

  const handleExpand = async (categoryId) => {
    const children = await loadChildren(categoryId);
    // 更新状态，添加children到对应的category
  };

  return (
    <Tree data={categories} onExpand={handleExpand} />
  );
}
```

### 方案3：面包屑导航

```javascript
// 构建面包屑路径
function getBreadcrumb(categories, targetId) {
  const path = [];
  let current = categories.find(c => c.id === targetId);

  while (current) {
    path.unshift(current); // 添加到开头
    if (current.parentId === 0) break;
    current = categories.find(c => c.id === current.parentId);
  }

  return path;
}

// 使用
const breadcrumb = getBreadcrumb(allCategories, 15);
// 结果: [影视, 动作片, 枪战]
```

---

## 常见场景

### 场景1：级联选择器

```vue
<template>
  <el-cascader
    v-model="selectedCategory"
    :options="categoryTree"
    :props="{
      label: 'name',
      value: 'id',
      children: 'children',
      checkStrictly: true
    }"
    clearable
  />
</template>

<script>
export default {
  data() {
    return {
      selectedCategory: [],
      categoryTree: []
    };
  },
  async mounted() {
    // 加载所有分类并构建树
    const response = await fetch('/api/v1/x/categories?pageSize=1000');
    const data = await response.json();
    this.categoryTree = this.buildTree(data.list);
  },
  methods: {
    buildTree(categories, parentId = 0) {
      return categories
        .filter(cat => cat.parentId === parentId)
        .map(cat => ({
          ...cat,
          children: this.buildTree(categories, cat.id)
        }));
    }
  }
};
</script>
```

### 场景2：树形表格展示

```vue
<template>
  <el-table
    :data="categoryTree"
    row-key="id"
    :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
  >
    <el-table-column prop="name" label="分类名称" width="200" />
    <el-table-column prop="nameEn" label="英文名称" width="150" />
    <el-table-column prop="level" label="层级" width="80">
      <template #default="{ row }">
        <el-tag :type="getLevelType(row.level)">
          {{ row.level }}级
        </el-tag>
      </template>
    </el-table-column>
    <el-table-column prop="sort" label="排序" width="80" />
    <el-table-column prop="status" label="状态" width="80">
      <template #default="{ row }">
        <el-tag :type="row.status === 1 ? 'success' : 'info'">
          {{ row.status === 1 ? '启用' : '禁用' }}
        </el-tag>
      </template>
    </el-table-column>
    <el-table-column label="操作" width="200">
      <template #default="{ row }">
        <el-button size="small" @click="addChild(row)">
          添加子分类
        </el-button>
        <el-button size="small" type="primary" @click="edit(row)">
          编辑
        </el-button>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
export default {
  methods: {
    getLevelType(level) {
      const types = ['', 'primary', 'success', 'warning', 'danger'];
      return types[level] || 'info';
    },
    addChild(parent) {
      // 添加子分类时，设置parentId为当前分类的ID
      this.editForm = {
        name: '',
        parentId: parent.id,  // 关键：设置父分类
        sort: 0,
        status: 1
      };
      this.dialogVisible = true;
    }
  }
};
</script>
```

### 场景3：筛选某个分类下的所有后代

```javascript
// 递归获取所有后代ID
function getAllDescendantIds(categories, parentId) {
  const result = [];

  function traverse(pid) {
    const children = categories.filter(c => c.parentId === pid);
    children.forEach(child => {
      result.push(child.id);
      traverse(child.id); // 递归查找子分类
    });
  }

  traverse(parentId);
  return result;
}

// 使用
const actionCategoryId = 13; // 动作片
const allActionIds = getAllDescendantIds(allCategories, actionCategoryId);
// 结果: [15, 16] (枪战、武术)

// 查询所有动作片分类下的视频
const videos = await fetch(`/api/videos?categoryIds=${[actionCategoryId, ...allActionIds].join(',')}`);
```

---

## 层级管理最佳实践

### 1. 限制层级深度

```javascript
const MAX_LEVEL = 3; // 最多3级

async function createCategory(data) {
  if (data.parentId !== 0) {
    // 检查父分类的层级
    const parent = await fetchCategory(data.parentId);
    const parentLevel = await getLevel(parent.id);

    if (parentLevel >= MAX_LEVEL) {
      throw new Error(`最多只能创建${MAX_LEVEL}级分类`);
    }
  }

  // 创建分类...
}
```

### 2. 防止循环引用

```javascript
async function updateCategory(id, data) {
  if (data.parentId) {
    // 检查是否会造成循环
    const descendants = await getAllDescendantIds(id);
    if (descendants.includes(data.parentId)) {
      throw new Error('不能将分类移动到自己的子分类下');
    }
  }

  // 更新分类...
}
```

### 3. 删除检查

```javascript
async function deleteCategory(id) {
  // 检查是否有子分类
  const children = await fetch(`/api/v1/x/categories?parentId=${id}`);
  if (children.total > 0) {
    throw new Error('请先删除所有子分类');
  }

  // 检查是否有关联的视频
  const videos = await fetch(`/api/videos?categoryId=${id}`);
  if (videos.total > 0) {
    throw new Error('该分类下还有视频，无法删除');
  }

  // 删除分类...
}
```

---

## 性能优化建议

### 1. 使用缓存

```javascript
// 缓存树形结构
let categoryTreeCache = null;
let cacheTime = null;
const CACHE_DURATION = 5 * 60 * 1000; // 5分钟

async function getCategoryTree(forceRefresh = false) {
  const now = Date.now();

  if (!forceRefresh && categoryTreeCache && (now - cacheTime < CACHE_DURATION)) {
    return categoryTreeCache;
  }

  const response = await fetch('/api/v1/x/categories?pageSize=1000');
  const data = await response.json();
  categoryTreeCache = buildTree(data.list);
  cacheTime = now;

  return categoryTreeCache;
}
```

### 2. 懒加载

对于层级很深或分类很多的情况，使用按需加载：

```javascript
// 只在需要时加载子分类
async function loadTreeNode(node, resolve) {
  if (node.level === 0) {
    // 加载一级分类
    const response = await fetch('/api/v1/x/categories?parentId=0');
    const data = await response.json();
    resolve(data.list);
  } else {
    // 加载子分类
    const response = await fetch(`/api/v1/x/categories?parentId=${node.data.id}`);
    const data = await response.json();
    resolve(data.list);
  }
}
```

---

## 总结

**关键点**：
1. ✅ 使用 `parent_id` 字段建立父子关系
2. ✅ `parent_id = 0` 表示顶级分类
3. ✅ 通过递归构建完整的树形结构
4. ✅ 可以按需加载减少一次性数据量
5. ✅ 前端需要处理层级深度和循环引用
6. ✅ 删除时需要检查子分类

**现有API支持**：
- ✅ 查询指定父分类下的子分类：`GET /categories?parentId={id}`
- ✅ 查询所有分类后前端构建树
- ✅ 创建分类时指定 `parentId`
- ✅ 更新分类时可以修改 `parentId`（移动分类）

---

**文档版本**: v1.0
**创建时间**: 2024-12-12
