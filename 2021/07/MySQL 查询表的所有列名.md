# MySQL 查询information_schema信息

```sql
# 获取表
SELECT * FROM information_schema.TABLES WHERE TABLE_SCHEMA = (SELECT DATABASE());

# 获取字段
SELECT * FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = (SELECT DATABASE()) AND TABLE_NAME = "table_name";

# 获取列以 ，分割
SELECT GROUP_CONCAT(COLUMN_NAME SEPARATOR ",") FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = 'db_name' AND TABLE_NAME = 'table_name'
```
