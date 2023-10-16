# 查询表属性信息

要查询 MySQL 数据库中表的属性信息，你可以使用以下 SQL 查询语句：

```sql
-- 显示所有表信息(如存储引擎、数据编码)
SELECT *
FROM information_schema.tables
WHERE table_schema = 'your_database_name';

-- 也可以指定仅显示表名以及注释信息
SELECT table_name, table_comment
FROM information_schema.tables
WHERE table_schema = 'your_database_name';
```

请确保替换上述查询中的 `'your_database_name'` 为你要查询的实际数据库名称。此查询会检索指定数据库中的所有表，并返回表的名称和注释（如果有的话）。注释通常是在创建表时使用 `COMMENT` 子句添加的。如果表没有注释，`table_comment` 列将为 `NULL`。

下面是一个完整输出示例：

```
+---------------+--------------+---------------------------+------------+--------+---------+------------+------------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+--------------------+----------+----------------+-----------------------+
| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME                | TABLE_TYPE | ENGINE | VERSION | ROW_FORMAT | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH | MAX_DATA_LENGTH | INDEX_LENGTH | DATA_FREE | AUTO_INCREMENT | CREATE_TIME         | UPDATE_TIME | CHECK_TIME | TABLE_COLLATION    | CHECKSUM | CREATE_OPTIONS | TABLE_COMMENT         |
+---------------+--------------+---------------------------+------------+--------+---------+------------+------------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+--------------------+----------+----------------+-----------------------+
| def           | vertx        | sys_administrative_region | BASE TABLE | InnoDB |      10 | Dynamic    |      45760 |            242 |    11075584 |               0 |      2637824 |         0 |           NULL | 2023-09-02 21:03:07 | NULL        | NULL       | utf8mb4_0900_ai_ci |     NULL |                | 系统行政区划表        |
+---------------+--------------+---------------------------+------------+--------+---------+------------+------------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+--------------------+----------+----------------+-----------------------+
```


如果你只想查询特定表的信息和注释，可以在 `SELECT` 语句中添加适当的条件来限定表的名称。例如：

```sql
SELECT table_name, table_comment
FROM information_schema.tables
WHERE table_schema = 'your_database_name' AND table_name = 'your_table_name'; -- 替换成你的表名称
```

这将只返回指定表的信息和注释。

# 查询表结构以及字段信息

要查询MySQL数据库中表的结构信息，你可以使用以下SQL命令：

```sql
DESCRIBE table_name;
-- 或者
DESC table_name;
```

输出示例：

```
+-------------+---------------+------+-----+---------+-----------------------------+
| Field       | Type          | Null | Key | Default | Extra                       |
+-------------+---------------+------+-----+---------+-----------------------------+
| id          | varchar(32)   | NO   | PRI | NULL    |                             |
| name        | varchar(32)   | NO   |     | NULL    |                             |
| code        | bigint        | NO   | UNI | NULL    |                             |
| parents     | varchar(64)   | NO   |     | NULL    |                             |
| level       | int           | YES  |     | NULL    |                             |
| longitude   | double(20,15) | YES  |     | NULL    |                             |
| latitude    | double(20,15) | YES  |     | NULL    |                             |
| status      | int           | NO   |     | 1       |                             |
| deleted     | int           | NO   |     | 1       |                             |
| add_time    | datetime      | NO   |     | NULL    | on update CURRENT_TIMESTAMP |
| update_time | datetime      | NO   |     | NULL    | on update CURRENT_TIMESTAMP |
| description | text          | YES  |     | NULL    |                             |
+-------------+---------------+------+-----+---------+-----------------------------+
```

MySQL 的 DESCRIBE 或 DESC 命令不直接显示字段的注释。要查询字段的注释，你可以使用以下方法之一：

1. 使用 `SHOW FULL COLUMNS` 命令，它可以显示字段的注释信息：

    ```sql
    SHOW FULL COLUMNS FROM table_name;
    ```

    输出示例：

    ```
    +-------------+---------------+--------------------+------+-----+---------+-----------------------------+---------------------------------+-------------------------------------------+
    | Field       | Type          | Collation          | Null | Key | Default | Extra                       | Privileges                      | Comment                                   |
    +-------------+---------------+--------------------+------+-----+---------+-----------------------------+---------------------------------+-------------------------------------------+
    | id          | varchar(32)   | utf8mb4_0900_ai_ci | NO   | PRI | NULL    |                             | select,insert,update,references | 主键                                      |
    | name        | varchar(32)   | utf8mb4_0900_ai_ci | NO   |     | NULL    |                             | select,insert,update,references | 区域名称                                  |
    | code        | bigint        | NULL               | NO   | UNI | NULL    |                             | select,insert,update,references | 区域代码                                  |
    | parents     | varchar(64)   | utf8mb4_0900_ai_ci | NO   |     | NULL    |                             | select,insert,update,references | 父级行政区域                              |
    | level       | int           | NULL               | YES  |     | NULL    |                             | select,insert,update,references | 区域级别                                  |
    | longitude   | double(20,15) | NULL               | YES  |     | NULL    |                             | select,insert,update,references | 经度                                      |
    | latitude    | double(20,15) | NULL               | YES  |     | NULL    |                             | select,insert,update,references | 纬度                                      |
    | status      | int           | NULL               | NO   |     | 1       |                             | select,insert,update,references | 状态。1：启用，2禁用                      |
    | deleted     | int           | NULL               | NO   |     | 1       |                             | select,insert,update,references | 删除状态。1：正常，2：已删除              |
    | add_time    | datetime      | NULL               | NO   |     | NULL    | on update CURRENT_TIMESTAMP | select,insert,update,references | 创建时间                                  |
    | update_time | datetime      | NULL               | NO   |     | NULL    | on update CURRENT_TIMESTAMP | select,insert,update,references | 修改时间                                  |
    | description | text          | utf8mb4_0900_ai_ci | YES  |     | NULL    |                             | select,insert,update,references | 描述说明                                  |
    +-------------+---------------+--------------------+------+-----+---------+-----------------------------+---------------------------------+-------------------------------------------+
    ```

2. 使用 INFORMATION_SCHEMA 数据库中的查询，这是一个 MySQL 系统数据库，包含了有关数据库对象的元数据信息。以下是使用 INFORMATION_SCHEMA 来查询字段注释的示例：

    ```sql
    SELECT *
    FROM INFORMATION_SCHEMA.COLUMNS
    WHERE TABLE_SCHEMA = 'your_database_name'
    AND TABLE_NAME = 'table_name';
    ```

    这会将字段的全部属性信息全部显示出来，通常我们可能仅需要字段名、注释、索引以及数据类型等信息：

    ```sql
    select
    column_name, data_type, column_type, column_key, column_comment
    WHERE TABLE_SCHEMA = 'your_database_name'
    AND TABLE_NAME = 'table_name';
    ```

    下面是一个输出示例：

    ```
    +-------------+-----------+---------------+------------+-------------------------------------------+
    | COLUMN_NAME | DATA_TYPE | COLUMN_TYPE   | COLUMN_KEY | COLUMN_COMMENT                            |
    +-------------+-----------+---------------+------------+-------------------------------------------+
    | id          | varchar   | varchar(32)   | PRI        | 主键                                      |
    | name        | varchar   | varchar(32)   |            | 区域名称                                   |
    | code        | bigint    | bigint        | UNI        | 区域代码                                   |
    | parents     | varchar   | varchar(64)   |            | 父级行政区域                               |
    | level       | int       | int           |            | 区域级别                                   |
    | longitude   | double    | double(20,15) |            | 经度                                      |
    | latitude    | double    | double(20,15) |            | 纬度                                      |
    | status      | int       | int           |            | 状态。1：启用，2禁用                        |
    | deleted     | int       | int           |            | 删除状态。1：正常，2：已删除                 |
    | add_time    | datetime  | datetime      |            | 创建时间                                  |
    | update_time | datetime  | datetime      |            | 修改时间                                  |
    | description | text      | text          |            | 描述说明                                  |
    +-------------+-----------+---------------+------------+-------------------------------------------+
    ```