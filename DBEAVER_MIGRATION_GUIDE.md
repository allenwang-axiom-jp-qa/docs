# DBeaver 数据库迁移操作指南

## 一、确认当前服务数据库连接

根据代码分析，服务的数据库配置通过以下方式加载：

### 配置加载优先级
1. **etcd 配置中心**（最高优先级）
   - etcd endpoints: 从 `conf/env.json` 的 `etcd_addresses` 字段获取
   - etcd key: `/cms-site-svc-video-mgmt/mysql`
   - 配置结构:
     ```json
     {
       "mysql": {
         "host": "数据库主机",
         "port": 3306,
         "user": "用户名",
         "pwd": "密码",
         "name": "数据库名",
         "conn": 最大连接数
       }
     }
     ```

2. **本地配置文件**（作为备用）
   - 默认路径: `conf/env.json`

### 查看当前配置方法

**方法 1: 查看 etcd 配置（推荐）**
```bash
# 连接到 etcd pod
kubectl get pods -n x-dev -l app=etcd

# 查看视频管理服务的MySQL配置
kubectl exec -n x-dev <etcd-pod-name> -- etcdctl \
  --endpoints=http://127.0.0.1:2379 \
  get /cms-site-svc-video-mgmt/mysql

# 查看所有视频管理服务配置
kubectl exec -n x-dev <etcd-pod-name> -- etcdctl \
  --endpoints=http://127.0.0.1:2379 \
  get /cms-site-svc-video-mgmt/ --prefix
```

**方法 2: 查看服务日志**
```bash
# 获取服务 pod
kubectl get pods -n x-dev | grep cms-site-svc-video-mgmt

# 查看启动日志（会输出配置信息）
kubectl logs -n x-dev <pod-name> | grep -i mysql
kubectl logs -n x-dev <pod-name> | grep -i "etcd"
```

**方法 3: 查看本地配置文件**
```bash
cat conf/env.json | jq '.mysql'
```

### 根据您之前提供的信息

您之前提供的数据库配置是：
- **Host**: 10.1.160.41
- **Port**: 3306
- **User**: root
- **Password**: TestRoot@123456
- **Database**: cms_video

这很可能就是服务使用的数据库。但建议通过上述方法再次确认。

---

## 二、使用 DBeaver 连接数据库

### 1. 创建新连接

1. 打开 DBeaver
2. 点击菜单：**Database** → **New Database Connection**
3. 选择 **MySQL**
4. 点击 **Next**

### 2. 配置连接参数

在 **Main** 标签页填写：

| 参数 | 值 |
|------|-----|
| **Host** | `10.1.160.41` |
| **Port** | `3306` |
| **Database** | `cms_video` |
| **Username** | `root` |
| **Password** | `TestRoot@123456` |
| **Connection name** | `CMS Video (Dev)` |

### 3. 测试连接

1. 点击 **Test Connection** 按钮
2. 如果是首次连接，DBeaver 会提示下载 MySQL 驱动，点击 **下载**
3. 看到 "Connected" 提示表示连接成功

### 4. 保存连接

- 点击 **Finish** 保存连接
- 连接会出现在左侧导航树中

---

## 三、执行数据库迁移脚本

### 准备工作

1. **备份数据库**（重要！）
   ```sql
   -- 在 DBeaver SQL 编辑器中执行
   mysqldump -h 10.1.160.41 -u root -pTestRoot@123456 cms_video > cms_video_backup_20260108.sql
   ```

   或使用 DBeaver 的导出功能：
   - 右键数据库 → **Tools** → **Backup**

2. **确认当前表结构**
   ```sql
   -- 查看 t_video_tag 表结构
   SHOW CREATE TABLE t_video_tag;

   -- 查看 t_video_tag_category 表结构
   SHOW CREATE TABLE t_video_tag_category;

   -- 检查 identifier 字段是否存在
   DESC t_video_tag;
   DESC t_video_tag_category;
   ```

### 执行迁移脚本的步骤

#### 方法 1: 使用 SQL 编辑器（推荐）

1. **打开 SQL 编辑器**
   - 在左侧导航树中，右键点击数据库 `cms_video`
   - 选择 **SQL Editor** → **Open SQL Script**
   - 或按快捷键 `Ctrl+]` (Windows/Linux) 或 `Cmd+]` (Mac)

2. **打开迁移脚本**
   - 方式 A: 直接粘贴脚本内容到编辑器
   - 方式 B: 在编辑器中点击 **File** → **Open** → 选择 `migrate/009_drop_identifier_fields.sql`

3. **查看脚本内容**
   ```sql
   -- 确认脚本内容如下

   -- 1. 删除 t_video_tag 表的 identifier 字段和索引
   ALTER TABLE `t_video_tag` DROP INDEX `uk_identifier`;
   ALTER TABLE `t_video_tag` DROP COLUMN `identifier`;

   -- 2. 删除 t_video_tag_category 表的 identifier 字段和索引
   ALTER TABLE `t_video_tag_category` DROP INDEX `uk_identifier`;
   ALTER TABLE `t_video_tag_category` DROP COLUMN `identifier`;
   ```

4. **执行脚本**

   **方式 A: 执行整个脚本**
   - 确保整个脚本被选中（或不选中任何内容）
   - 点击工具栏的 **Execute SQL Script** 按钮（▶️图标）
   - 或按快捷键 `Ctrl+Alt+X` (Windows/Linux) 或 `Cmd+Option+X` (Mac)
   - 或右键 → **Execute** → **Execute SQL Script**

   **方式 B: 逐条执行（推荐用于测试）**
   - 选中第一条 ALTER 语句
   - 点击 **Execute SQL Statement** 按钮（带有选中标记的▶️图标）
   - 或按快捷键 `Ctrl+Enter` (Windows/Linux) 或 `Cmd+Enter` (Mac)
   - 检查执行结果
   - 依次执行剩余语句

5. **查看执行结果**
   - 在底部的 **Results** 面板查看执行状态
   - 成功信息: `Query OK, 0 rows affected`
   - 如果报错，检查错误信息并处理

#### 方法 2: 使用脚本执行工具

1. **打开脚本执行工具**
   - 右键点击数据库 `cms_video`
   - 选择 **Tools** → **Execute script**

2. **选择脚本文件**
   - 点击 **Select script file**
   - 选择 `migrate/009_drop_identifier_fields.sql`

3. **配置执行选项**
   - ✅ **Stop on error** - 遇到错误时停止
   - ✅ **Show results** - 显示执行结果
   - ⬜ **Ignore errors** - 根据需要选择

4. **执行脚本**
   - 点击 **Start** 开始执行
   - 等待完成并查看结果

### 验证迁移结果

在 SQL 编辑器中执行以下验证语句：

```sql
-- 1. 验证 t_video_tag 表结构
DESC t_video_tag;
-- 应该看不到 identifier 字段

-- 2. 验证 t_video_tag_category 表结构
DESC t_video_tag_category;
-- 应该看不到 identifier 字段

-- 3. 查看表的索引
SHOW INDEX FROM t_video_tag;
-- 应该看不到 uk_identifier 索引

SHOW INDEX FROM t_video_tag_category;
-- 应该看不到 uk_identifier 索引

-- 4. 测试查询功能（确保表仍然可用）
SELECT COUNT(*) FROM t_video_tag;
SELECT COUNT(*) FROM t_video_tag_category;

-- 5. 查看标签数据（确保数据完整）
SELECT id, tag_id, name, category_id FROM t_video_tag LIMIT 5;
SELECT id, category_id, name FROM t_video_tag_category LIMIT 5;
```

---

## 四、执行迁移的最佳实践

### 执行时机

1. ✅ **在部署新代码之后执行**
   - 确保应用代码已经不再使用 identifier 字段
   - 当前代码已经推送到 dev 分支并部署

2. ✅ **在低峰期执行**
   - 选择业务访问量较低的时间段
   - 减少对用户的影响

3. ✅ **准备回滚方案**
   - 保存数据库备份
   - 记录 identifier 字段的原始数据

### 执行前检查清单

- [ ] 已备份数据库
- [ ] 已确认新代码已部署并运行正常
- [ ] 已在测试环境验证过迁移脚本
- [ ] 已通知相关人员执行时间
- [ ] 已准备回滚脚本（如需要）

### 执行步骤

```sql
-- Step 1: 开始事务（可选，但推荐）
START TRANSACTION;

-- Step 2: 备份 identifier 数据到临时表（用于回滚）
CREATE TABLE t_video_tag_backup_identifier AS
SELECT id, identifier FROM t_video_tag;

CREATE TABLE t_video_tag_category_backup_identifier AS
SELECT id, identifier FROM t_video_tag_category;

-- Step 3: 执行迁移
ALTER TABLE `t_video_tag` DROP INDEX `uk_identifier`;
ALTER TABLE `t_video_tag` DROP COLUMN `identifier`;

ALTER TABLE `t_video_tag_category` DROP INDEX `uk_identifier`;
ALTER TABLE `t_video_tag_category` DROP COLUMN `identifier`;

-- Step 4: 验证结果
DESC t_video_tag;
DESC t_video_tag_category;

-- Step 5: 如果一切正常，提交事务
COMMIT;

-- 或者，如果有问题，回滚
-- ROLLBACK;
```

### 监控执行

在执行过程中，可以开启另一个 SQL 编辑器窗口监控：

```sql
-- 监控当前正在执行的查询
SHOW PROCESSLIST;

-- 监控表锁
SHOW OPEN TABLES WHERE In_use > 0;

-- 检查慢查询
SHOW VARIABLES LIKE 'long_query_time';
```

---

## 五、回滚方案（如果需要）

如果执行后发现问题需要回滚：

### 方法 1: 从备份恢复（如果有备份表）

```sql
-- 1. 重新添加列
ALTER TABLE t_video_tag
ADD COLUMN identifier VARCHAR(100) AFTER name;

ALTER TABLE t_video_tag_category
ADD COLUMN identifier VARCHAR(64) AFTER name;

-- 2. 从备份表恢复数据
UPDATE t_video_tag t
INNER JOIN t_video_tag_backup_identifier b ON t.id = b.id
SET t.identifier = b.identifier;

UPDATE t_video_tag_category t
INNER JOIN t_video_tag_category_backup_identifier b ON t.id = b.id
SET t.identifier = b.identifier;

-- 3. 重新添加唯一索引
ALTER TABLE t_video_tag ADD UNIQUE KEY uk_identifier (identifier);
ALTER TABLE t_video_tag_category ADD UNIQUE KEY uk_identifier (identifier);

-- 4. 清理备份表
DROP TABLE IF EXISTS t_video_tag_backup_identifier;
DROP TABLE IF EXISTS t_video_tag_category_backup_identifier;
```

### 方法 2: 从完整备份恢复

如果有 mysqldump 备份文件：

```bash
# 恢复整个数据库
mysql -h 10.1.160.41 -u root -pTestRoot@123456 cms_video < cms_video_backup_20260108.sql

# 或只恢复特定表
mysql -h 10.1.160.41 -u root -pTestRoot@123456 cms_video --one-database < backup.sql
```

---

## 六、常见问题处理

### 问题 1: 无法连接到数据库

**错误信息**: `Communications link failure` 或 `Connection refused`

**解决方案**:
1. 检查数据库服务器是否运行
2. 检查防火墙是否允许 3306 端口
3. 检查网络连接
4. 确认 IP 地址和端口号正确

### 问题 2: 索引不存在

**错误信息**: `Can't DROP 'uk_identifier'; check that column/key exists`

**解决方案**:
```sql
-- 先检查索引是否存在
SHOW INDEX FROM t_video_tag WHERE Key_name = 'uk_identifier';

-- 如果不存在，跳过删除索引步骤，直接删除列
ALTER TABLE `t_video_tag` DROP COLUMN `identifier`;
```

### 问题 3: 列不存在

**错误信息**: `Unknown column 'identifier' in 't_video_tag'`

**解决方案**:
- 这表示 identifier 字段已经被删除，无需再次执行迁移
- 验证当前表结构: `DESC t_video_tag;`

### 问题 4: 表被锁定

**错误信息**: `Waiting for table metadata lock`

**解决方案**:
```sql
-- 查找锁定的进程
SHOW PROCESSLIST;

-- 找到 State 为 'Waiting for table metadata lock' 的进程
-- 杀掉长时间运行的查询
KILL <process_id>;
```

### 问题 5: 权限不足

**错误信息**: `Access denied` 或 `ALTER command denied`

**解决方案**:
```sql
-- 检查当前用户权限
SHOW GRANTS FOR 'root'@'%';

-- 如果需要，授予 ALTER 权限
GRANT ALTER ON cms_video.* TO 'root'@'%';
FLUSH PRIVILEGES;
```

---

## 七、DBeaver 使用技巧

### 快捷键

| 功能 | Windows/Linux | Mac |
|------|--------------|-----|
| 打开 SQL 编辑器 | Ctrl+] | Cmd+] |
| 执行当前语句 | Ctrl+Enter | Cmd+Enter |
| 执行脚本 | Ctrl+Alt+X | Cmd+Option+X |
| 格式化 SQL | Ctrl+Shift+F | Cmd+Shift+F |
| 代码补全 | Ctrl+Space | Ctrl+Space |
| 查看表结构 | F4 | F4 |

### SQL 编辑器功能

1. **语法高亮**: 自动高亮 SQL 关键字
2. **自动补全**: 输入时自动提示表名、列名
3. **错误提示**: 实时检查 SQL 语法错误
4. **结果导出**: 可以导出查询结果为 CSV、JSON 等格式
5. **多标签页**: 可以同时打开多个 SQL 编辑器

### 查看执行计划

在执行 ALTER 语句前，可以查看影响范围：

```sql
-- 查看表数据量
SELECT
    TABLE_NAME,
    TABLE_ROWS,
    DATA_LENGTH,
    INDEX_LENGTH
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'cms_video'
AND TABLE_NAME IN ('t_video_tag', 't_video_tag_category');
```

---

## 八、总结

### 迁移脚本路径
```
migrate/009_drop_identifier_fields.sql
```

### 执行顺序
1. ✅ 确认数据库连接信息
2. ✅ 连接到 DBeaver
3. ✅ 备份数据库
4. ✅ 在测试环境验证（如有）
5. ✅ 执行迁移脚本
6. ✅ 验证结果
7. ✅ 测试应用功能

### 关键提醒
- ⚠️ 迁移是不可逆操作（删除列）
- ⚠️ 执行前务必备份
- ⚠️ 确保新代码已部署
- ⚠️ 在低峰期执行
- ⚠️ 准备好回滚方案

### 验证清单
- [ ] identifier 列已删除
- [ ] uk_identifier 索引已删除
- [ ] 表数据完整
- [ ] 应用功能正常
- [ ] 无报错日志

---

**如果遇到任何问题，请参考本文档的"常见问题处理"章节，或联系 DBA 协助。**
