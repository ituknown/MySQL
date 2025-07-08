# 乐观锁

乐观锁认为，事务之间的冲突很少发生。它不依赖数据库的锁定机制，而是在更新数据时检查数据是否被其他事务修改过。

- 如果数据还未被修改，则能更新成功；
- 否则会更新失败，就需要重新获取最新数据再次尝试更新。

乐观锁说直接点就是使用一个字段做 WHERE 的条件，最主流的实现方法是通过版本号（version）或者时间戳（timestamp）字段，当然也可以使用任意业务字段！

## 使用版本号（version）实现乐观锁

在数据表中增加一个名为 `version` 的整数类型字段，初始值为 0。

1.  查询数据时，一并将 `version` 字段读取出来。
2.  更新数据时，将 `version` 字段的值加 1，并将其作为 `WHERE` 条件的一部分，与之前读取到的 `version` 值进行比较。

假设有一个 `products` 表：

```sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    stock INT,
    version INT DEFAULT 0
);
```

现在，需要更新 `id` 为 1 的库存：

```sql
-- 1. 查询当前数据和版本号
SELECT stock, version FROM products WHERE id = 1;
-- 假设查询结果：stock = 100, version = 1

-- 2. 执行更新操作，同时检查版本号
UPDATE products
SET stock = 99, version = version + 1
WHERE id = 1 AND version = 1;
```

如果这条 SQL 语句影响的行数为 1，说明更新成功。如果影响行数为 0，说明在更新之前，已经被其他事务修改过（`version` 值已经被改变，导致更新失败）。此时，就需要重新获取最新的数据和版本号，然后重新尝试更新。

## 使用时间戳（timestamp）实现乐观锁

时间戳的原理和版本号类似，只不过是用时间来标识数据的版本。

在数据表中增加一个名为 `last_updated_time` 的时间戳字段，每次数据更新时，也同时更新这个时间戳。

例如：

```sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    stock INT,
    last_updated_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

更新操作：

```sql
-- 1. 查询当前数据和时间戳
SELECT stock, last_updated_time FROM products WHERE id = 1;
-- 假设查询结果：stock = 100, last_updated_time = '2025-07-08 15:30:00'

-- 2. 执行更新操作，同时检查时间戳
UPDATE products
SET stock = 99, last_updated_time = NOW()
WHERE id = 1 AND last_updated_time = '2025-07-08 15:30:00';
```

和版本号一样，如果影响行数为 1，说明更新成功；影响行数为 0，说明数据被其他事务修改过。

## 乐观锁的适用场景

乐观锁适用于读多写少的场景，因为冲突发生的概率较低。在高并发写入的场景下，如果冲突频繁，可能导致大量的重试，反而降低了系统性能。

# 悲观锁

顾名思义，对并发操作持悲观态度。核心在于其名称所蕴含的“悲观”思想：它认为并发冲突总是会发生，所以在操作数据前就先将其锁定。

它在数据被修改之前就给它加上锁，确保在整个数据处理过程中，其他事务不能访问或修改这部分数据，直到当前事务完成并释放锁。这样可以有效避免数据冲突，但代价是降低了并发性能。

在 MySQL 中，悲观锁的实现主要依赖于数据库的锁机制，特别是 `SELECT ... FOR UPDATE` 语句。

## `SELECT ... FOR UPDATE`

当使用 `SELECT ... FOR UPDATE` 查询数据时，被查询的行会被加上排他锁（X锁）。这意味着：

  * 其他事务不能再对这些行进行 `UPDATE` 或 `DELETE` 操作。
  * 其他事务也不能使用 `SELECT ... FOR UPDATE` 再次锁定这些行。
  * 其他事务如果尝试对这些行进行上述操作，会被阻塞，直到当前事务提交（`COMMIT`）或回滚（`ROLLBACK`），释放锁。

## 示例

假设有一个 `accounts` 表，存储用户账户余额：

```sql
CREATE TABLE accounts (
    id INT PRIMARY KEY,
    user_id INT,
    balance DECIMAL(10, 2)
);

INSERT INTO accounts (id, user_id, balance) VALUES (1, 101, 1000.00);
```

现在，一个用户要从账户中取出 500 元。为了防止超卖或重复支付等并发问题，就可以这样使用悲观锁：

```sql
-- 开启事务
START TRANSACTION;

-- 1. 锁定账户余额行
-- 这条语句会锁定 id 为 1 的行，其他事务无法修改或再次锁定此行
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- 假设查询结果：balance = 1000.00

-- 2. 检查余额是否足够，并进行扣款操作
-- 假设业务逻辑判断余额充足
UPDATE accounts SET balance = balance - 500 WHERE id = 1;

-- 3. 提交事务，释放锁
COMMIT;
```

如果在 `SELECT ... FOR UPDATE` 执行之后、`COMMIT` 之前，有另一个事务也尝试修改 `id` 为 1 的行，它就会被阻塞。只有当前事务提交或回滚后，那个被阻塞的事务才能继续执行。

## 悲观锁的适用场景

悲观锁适用于写多读少、数据竞争激烈的场景，或者对数据一致性要求极高的业务场景，例如：

  * **库存扣减**：确保商品库存不会超卖。
  * **银行转账**：确保账户金额的正确性，避免数据不一致。

在这些场景下，宁愿牺牲一些并发性能，也要保证数据的绝对正确。但需要注意的是，过度使用悲观锁可能导致死锁和性能瓶颈，因此在使用时要仔细评估业务需求。
