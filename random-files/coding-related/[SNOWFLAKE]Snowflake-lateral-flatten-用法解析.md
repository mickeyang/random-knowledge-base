> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0Q8mLBOf0am

# you asked

message time: 2026-09-04 00:38:02

snowflake lateral flatten

---

# yuanbao response

`LATERAL FLATTEN` 是 Snowflake 里处理半结构化数据（VARIANT / OBJECT / ARRAY）最常用的组合：`FLATTEN` 负责把数组或对象"炸"成多行，`LATERAL` 让这个炸裂过程能逐行引用左边表的列，从而保留原行的上下文（如主键、客户名等）。

## 基本语法

```sql
SELECT 原表.列, f.VALUE, f.INDEX, f.KEY
FROM 原表,
     LATERAL FLATTEN(INPUT => 原表.半结构化列) f;
```

`FLATTEN` 是一个表函数，必填参数 `INPUT` 必须是 VARIANT / OBJECT / ARRAY 类型；可选参数有：

- `PATH`：指定 VARIANT 中要展开的深层路径，默认空（展开最外层）
- `OUTER => TRUE | FALSE`：默认 FALSE，**数组为空或路径不存在的那一行会被整个丢弃**；设 TRUE 则会保留原行，炸裂列填 NULL
- `RECURSIVE => TRUE | FALSE`：True 时递归展开所有子层
- `MODE => 'OBJECT' | 'ARRAY' | 'BOTH'`：控制展开对象、数组还是两者，默认 BOTH

`FLATTEN` 输出固定 6 列：`SEQ`、`KEY`、`PATH`、`INDEX`、`VALUE`、`THIS`。日常用得最多的是 `VALUE`（元素值）、`INDEX`（数组下标）、`KEY`（对象的 key）。

## 典型场景示例

**1. 展开单层数组（如订单里的 items）**

```sql
SELECT o.order_id,
       f.VALUE:product::VARCHAR AS product,
       f.VALUE:price::NUMERIC  AS price
FROM orders o,
     LATERAL FLATTEN(INPUT => o.order_data:items) f;
```

一行订单如果 items 有 2 个元素，结果会变成 2 行，原订单的 `order_id` 自动复制下来。这种**单层数组**场景下，`LATERAL` 其实可以省略，`TABLE(FLATTEN(...))` 效果一样。

**2. 嵌套数组必须链式 LATERAL**

当数组里还套数组，第二个 FLATTEN 要引用第一个的输出，这时 **LATERAL 是强制的**：

```sql
SELECT e.event_id,
       c.VALUE:type::VARCHAR AS contact_type,
       i.VALUE::VARCHAR      AS contact_value
FROM events e,
     LATERAL FLATTEN(INPUT => e.event_data:contacts) c,
     LATERAL FLATTEN(INPUT => c.VALUE:items) i;
```

> 💡 关键点：第二个 FLATTEN 的 `INPUT` 用的是 `c.VALUE:items`——也就是第一个 FLATTEN 的输出。没有 LATERAL 做不到这种"链式引用"。

**3. 保留空数组的原行**

```sql
SELECT p.product_id,
       t.VALUE::STRING AS tag
FROM products p,
     LATERAL FLATTEN(INPUT => p.data:tags, OUTER => TRUE) t;
```

不加 `OUTER => TRUE`，tags 为空数组的产品行会直接从结果里消失；加上后，这行仍在，tag 为 NULL。

## LATERAL 和普通 JOIN 的区别

| 维度 | LATERAL | 普通 JOIN |
|---|---|---|
| 右侧能否引用左侧列 | ✅ 可以 | ❌ 不行 |
| 典型用途 | 调用表函数（如 FLATTEN）逐行处理 | 两表按条件关联 |
| 是否产生笛卡尔积 | ❌ 不会 | 交叉 JOIN 会 |

简单说：**LATERAL 的本质是"对左边每一行，带着这一行的值去执行右边的表函数/子查询"**。

## 一个实用技巧：用 RECURSIVE 探明 JSON 结构

如果你拿到一段不熟悉的大 JSON，不知道里面有几层、字段叫什么，可以用递归展开快速看全貌：

```sql
SELECT f.PATH, f.KEY, f.VALUE
FROM raw_table,
     LATERAL FLATTEN(INPUT => raw_table.json_col, RECURSIVE => TRUE) f
WHERE f.PATH NOT LIKE '%[%]';
```

这会把所有叶子节点的路径、key、value 都打出来，方便你决定后面该怎么正式展开。

> ⚠️ 注意：单行数据被 FLATTEN 成多行后，原行的其他列会自动"复制"到每一行上匹配。如果你后面还要做聚合，留意行数膨胀带来的影响。

实际工作中 90% 的 `LATERAL FLATTEN` 都是场景 1 和 2——把 JSON 里的数组转成行项目，再拿来 JOIN 其他表或做聚合分析。如果你的 JSON 结构是固定的单层数组，其实连 LATERAL 都可以省；只有嵌套结构才真正离不开它。