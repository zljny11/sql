
# <center><font face="Arial" color = 'green' size ='55'> mySQL_Note (MOSH ver)</font></center>

# <center><font face="Arial" color = 'orange' size='5' >zljny</font></center>
## CATLOGUE
- [==L2== Basic](#l2-basic)
- [==L3== Join](#l3-join)
- [==L4== CRUD](#l4-crud)
- [==L5== Summarizing Data](#l5-summarizing-data)
- [==L6== Complex Quiry](#l6-complex-quiry)
- [==L7== Data Manipulation Functions](#l7-data-manipulation-functions)
- [==L11== Transaction and Concurrency](#l11-transaction-and-concurrency)
- [==L12== Data Types](#l12-data-types)
- [==L14== Indexing](#l14-indexing)

### ==L2== Basic

#### 1. SELECT STATEMENT

- ***basic statement***

```sql
USE sql_store;   #(';'is a must)

SELECT 1, 2
```

- ***conditional select***

```sql
SELECT *  #('*' will return all columns)
FROM customers
WHERE customer_id = 1
ORDER BY first_name #(in ascending order)
-- ORDER BY last_name DESC 
#(in descending order)
```

#### 2.SELECT CLAUSE

- ***create new column and rename***

```sql
SELECT 
    last_name, 
    first_name, 
    points, 
    points*10 AS 'discount factor'
    #(rename the column)
```

- ***no-duplicate select***

```sql
  SELECT DISTINCT state
```

#### 3. WHERE CLAUSE

- ***add constraint operator***

```sql
WHERE points >= 3000
WHERE state = 'VA'
```

- ***date in SQL***

```sql
WHERE birth_date > '1990-01-01'
```

#### 4. BOOLEAN OPERATOR

- ***AND/OR/NOT***

```sql
WHERE 
order_id = 6 
AND ( (quantity*unit_price > 30) 
OR discount_value < 30)
```

- ***IN OPERATOR***

```sql
WHERE state = 'VA' OR 'GA' OR 'FL'
==
WHERE state in ('VA', 'GA', 'FL')
```

#### 5. BETWEEN OPERATOR

```sql
WHERE birth_date 
BETWEEN '1990-01-01' AND '2000-01-01'
```

#### 6. LIKE OPERATOR

- ***_ : single char***

```sql
WHERE last_name LIKE '___y'
```

- ***% : any num of char***

```sql
WHERE customer LIKE '%y' or '%b%'
```

#### 7. REGEXP (regular expression)

- ***To find objects including at leastone of certain word***

```sql
WHERE customer REGEXP 'field|mac|Rose'
```

- ***End or beginning constraint***

```sql
WHERE customer REGEXP '^field'
#(start by field)

WHERE customer REGEXP 'Rose$'
#(end by Rose)
```

- ***[a-z]***

```sql
WHERE customer REGEXP '[a-h]e'
```

```sql
WHERE customer REGEXP 'br|bu'
==
WHERE customer REGEXP 'b[ru]'
```

### ==L3== Join 

#### 1. Inner Join

```sql
USE sql_store;
SELECT order_id, first_name, last_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
-- use alias 
```

#### 2. Outer Join

```sql
USE sql_inventory;
Select order_id, p.product_id, name as product_name
FROM sql_store.order_items oi
JOIN products p ON oi.product_id = p.product_id
-- need to identify which item come from which table
```

#### 3. Self Join

```sql
USE sql_hr;
SELECT 
 e.employee_id, 
 e.first_name, 
    m.first_name AS manager
FROM employees e
JOIN employees m
 ON e.reports_to = m.employee_id
```

#### 4. Multi-Table Join

```sql
USE sql_invoicing;
SELECT 
 p.date,
 p.invoice_id,
 p.amount,
 c.name,
 pm.name as payment
FROM payments p
JOIN clients c
 ON p.client_id = c.client_id
JOIN payment_methods pm 
 ON p.payment_method = pm.payment_method_id
 
```

#### 5. Implicit Join

```sql
SELECT *
FROM orders o 
JOIN customers c
 ON o.customer_id = c.customer_id
```

**equals to：**

```sql
SELECT *
FROM orders o ,customers c
WHERE o.customer_id = c.customer_id
-- better not to use this cuz crossjoin without where
```

#### 6. Outer Join

- ***Right join means the item in joined chart have to be kept***
eg: ( all customer will be joined even without order )

```sql
SELECT
    c.customer_id,
    c.first_name,
    o.order_id
FROM orders o
RIGHT JOIN customers c
    ON c.customer_id = o.customer_id
ORDER BY c.customer_id;
```

- ***Left join means the joined chart is less important（Used More）FROM的表是主表***
  
```sql
SELECT
    o.order_id,
    o.order_date,
    o.total_amount,
    s.shipment_date,
    s.tracking_number
FROM orders o
LEFT JOIN shipments s
    ON o.order_id = s.order_id
ORDER BY o.order_date;
```

#### 7. Self Outer Join

- ***Use left join to ensure the integrity of the original columns***
  
```sql
USE sql_hr;

SELECT
    e.employee_id,
    e.first_name,
    m.first_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.reports_to = m.employee_id;
```

#### 8. "Using" Statement

- ***use for same column name***

```sql
SELECT *
FROM order_items oi
JOIN order_item_notes oin
    ON oi.order_id = oin.order_id 
    AND oi.product_id = oin.product_id
    // equal to
JOIN order_item_notes oin
    USING (order_id, product_id)
```

#### 9. Natural Join

- ***not recommended***

```sql
NATURAL JOIN order_item_notes
```

#### 10. Cross Join

- ***Cartesian product***

```sql
SELECT
    c.first_name AS customer,
    p.name AS product
FROM customers c
CROSS JOIN products p
ORDER BY c.first_name, product
```

- ***Implicit way***

```sql
SELECT
    c.first_name AS customer,
    p.name AS product
FROM customers c, products p
ORDER BY c.first_name, product
```

#### 11.  Union

- ***directly connect two tables together***

```sql
SELECT
    order_id,
    order_date,
    'Active' AS status
FROM orders
WHERE order_date >= '2019-01-01'
// set the standard to be 'Active'

UNION

SELECT
    order_id,
    order_date,
    'Archived' AS status
FROM orders
WHERE order_date < '2019-01-01'
```

### ==L4== CRUD

#### 1. Column Attributes

![tool section](https://img.cdn1.vip/i/68e09f41c1297_1759551297.webp)

- **VARCHAR: immutable char length**
- **PK: primary key**
- **NN: not Null**
- **AI: auto increment** (eg:Int(10)->Int(11))

#### 2. Insert Single Row

```sql
INSERT INTO customers (
    customer_id, 
    first_name, 
    last_name, 
    btd, phone, 
    street_address, 
    city, 
    state, 
    points)
VALUES (
    DEFAULT,
    -- Auto Increment
    'John',
    'Smith',
    '1990-01-01',
    NULL,
    'address',
    'city',
    'CA',
    DEFAULT
);
```

#### 3. Insert Multiple Rows

```sql
INSERT INTO customers (customer_id, first_name, last_name, email)
VALUES 
(
    DEFAULT,            -- customer_id 字段通常设置为 DEFAULT，让自增主键自动生成
    'Frank',            -- first_name
    'Taylor',           -- last_name
    'frank.t@example.com'
),
(
    DEFAULT, 
    'Grace', 
    'Lee', 
    'grace.lee@example.com'
),
(
    DEFAULT, 
    'Henry', 
    'Clark', 
    'h.clark@example.com'
);
```

#### 4. Insert Hierachic Rows

- **Use LAST_INSERT_ID when child table don't have auto increment tool**

```sql
-- 先插入主表orders（比如订单表），order_id为自增列
INSERT INTO orders (customer_id, order_date) VALUES (201, '2025-10-26');

-- 再插入子表order_items，order_id使用刚插入的主表自增ID
INSERT INTO order_items (order_id, product_id, quantity)
VALUES (LAST_INSERT_ID(), 501, 2),
       (LAST_INSERT_ID(), 502, 1);
```

#### 5. Create A Copy Of Table

```sql
CREATE TABLE new_table LIKE old_table;
-- 只复制结构

CREATE TABLE new_table AS -- AS：把 SELECT 查询出来的结果作为新表的内容
SELECT * FROM old_table; -- subquery
-- 复制所有结构和数据（没有PK和AI）

INSERT INTO new_table (col1, col2)
SELECT col1, col2 FROM old_table WHERE order_date <'2019-01-01';
-- 新建表之后有条件插入
```

#### 6. Update A Single Row

```sql
UPDATE users --更新已有的表格
SET name = '张三', email = 'zhangsan@example.com'
WHERE id = 1; -- for mutli-row updatinbg, just refine the WHERE statement
```

#### 7. Subquery in UPDATE

- **UPDATE and SET should always show up together**

- **Use IN instead of '=' for updating multi-rows by subquery in WHERE**
  
```sql
UPDATE product
SET price = 'good price'
WHERE price IN ( --IN is used to select multiple items simultaneously
    SELECT new_price
    FROM price_list
    WHERE 15 < price <50);
```

#### 8. Delete

```sql
DELETE FROM invoices
WHERE client_id = (
    SELECT client_id
    FROM clients
    WHERE name = 'Myworks'
)
```

### ==L5== Summarizing Data

#### 1. Aggregation Function

```sql
SELECT
    MAX(invoice_total) AS highest,           -- 发票金额最大值
    MIN(invoice_total) AS lowest,            -- 发票金额最小值
    AVG(invoice_total) AS average,           -- 发票金额平均值
    SUM(invoice_total) AS total,             -- 发票金额总和
    COUNT(invoice_total) AS number_of_invoices,  -- 有金额的发票总数
    COUNT(payment_date) AS count_of_payments,    -- 有付款日期的行数
    COUNT(*) AS total_records                    -- 表中总行数
    COUNT(DISTINCT client_id) AS id  --  不重复的id数量
FROM invoices;
```

#### 2. GROUP BY

- **不用 GROUP BY，聚合函数是在全表基础上计算**

- **用 GROUP BY，则是对每个分组分别计算聚合值**
  
- **used to divide and conclude**

```sql
SELECT
    client_id,                              -- 客户编号
    SUM(invoice_total) AS total_sales       -- 该客户的发票总金额
FROM invoices
GROUP BY client_id                         -- 按客户端分组
ORDER BY total_sales DESC                  -- 按销售总额降序排列
-- ORDER BY放在最后避免影响前面的运算逻辑
```

```sql
GROUP BY state, city          -- 按照state和city的组合来分组
```

#### 3. HAVING

- HAVING is used after GROUP BY ==//== WHERE can only use before GROUP BY
  
```sql
SELECT
    client_id,                             
    SUM(invoice_total) AS total_sales      
GROUP BY client_id 
HAVING total_sales > 500
```

- HAVING can only use SELECT columns as the condition ==//== WHERE can use any column in the table as conditions
  
```sql
    SELECT
    client_id,                             
    SUM(invoice_total) AS total_sales   
    COUNT(*) AS number_of_invoices   
GROUP BY client_id 
HAVING total_sales > 500 AND number_of_invoices > 5
```

#### 4. WITH ROLLUP

- 计算所有group的总和

```sql
SELECT payment_method,
       SUM (amount) AS total
FROM payments
GROUP BY payment_method WITH ROULLUP
```

### ==L6== Complex Quiry

#### 1. Sub quiry

```sql
USE sql_hr;
SELECT *
FROM employees
WHERE salary >=(
    SELECT AVG(salary)
    FROM employees
)
```

#### 2. IN / = ANY

```sql
SELECT *
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
)
```

==

```sql
SELECT *
FROM employees
WHERE department_id = ANY (
    SELECT department_id
    FROM departments
)
```

#### 3. Subquiry vs Join

- Choose more readable and intuitative when  execution time is equal
- example --- when JOIN is more readable: find customers who have ordered lettuce(id = 3)
  
  
```sql
SELECT DISTINCT customer_id, first_name, last_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders o
    WHERE o.order_id IN (
        SELECT order_id
        FROM order_items oi
        WHERE oi.product_id = 3
    )
)
```

==

```sql
SELECT DISTINCT customer_id, first_name, last_name
FROM customers c
JOIN orders o USING (customer_id)
JOIN order_items oi USING (order_id)
WHERE oi.product_id = 3
```

#### 4. ALL

- ALL can be used to get the iterated result of a list, resembling MAX in selection
  
```sql
SELECT *
FROM invoices
WHERE invoice_total > (
    SELECT MAX(invoice-total)
    FROM invoices
    WHERE client_id = 3
)
```

==

```sql
SELECT *
FROM invoices
WHERE invoices_total > ALL(
    SELECT invoice_total
    FROM invoices
    WHERE client_id = 3
)
```

#### 5. Correlated Subquery

- 当需要逐行比较，且比较的标准随着每一行变化时，就用关联子查询

```sql
// 找出每个客户的开票金额大于平均值的发票
SELECT *
FROM invoices i
// alias在关联子查询中指的是外部表格的单独行（迭代的元素）
WHERE invoice_total > (
    SELECT AVG(invoice_total)
    FROM invoices
    WHERE client_id = i.client_id 
    // i.client_id是迭代的元素
)
```

==WINDOW FUNCTION==
```sql
// 窗口函数改写版本：找出高于客户平均金额的发票
SELECT
    i.invoice_id,
    i.client_id,
    i.invoice_total
FROM (
    -- 这是一个子查询，用于计算窗口函数的结果
    SELECT
        *,
        -- 核心：计算每个客户的平均发票金额，并将其作为一列返回
        AVG(invoice_total) OVER (PARTITION BY client_id) AS client_avg_invoice
    FROM
        invoices
) AS i
WHERE
    -- 在外部查询中，对每一行的发票金额与其客户平均值进行比较
    i.invoice_total > i.client_avg_invoice;
```

#### 6. EXISTS

- used to optimize the performance
```sql
// original one:
// overflow when subquery is large with many rows
SELECT *
FROM customers 
WHERE customer_id IN (
    SELECT DISTINCT customer_id
    FROM orders
)

// optimized one: using EXISTS
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT customer_id
    FROM orders o
    WHERE o.customer_id = c.customer_id
)
``` 

- NOT EXISTS
  
```sql
// return products that havent been ordered

SELECT *
FROM products p
WHERE NOT EXISTS (
    SELECT product_id
    FROM order_items oi
    WHERE oi.product_id = p.product_id
)   
``` 

#### 7. Subquiries in SELECT

```sql
SELECT
    product_id,
    product_name,
    (SELECT COUNT(*) FROM order_items WHERE order_items.product_id = products.product_id) AS number_of_orders 
FROM products;
``` 

#### 8. Subquieries in FROM 

```sql
SELECT *
FROM (
    SELECT product_id,
    product_name,
    (SELECT COUNT(*) FROM order_items WHERE order_items.product_id = products.product_id) AS number_of_orders) AS product_summary
WHERE number_of_orders > 0;
``` 

#### 9. Window Functions

- 排名函数：
   > `ROW_NUMBER()` (客户内部排名)此功能用于在不折叠原始行的情况下，为每个组（客户）内的行分配一个唯一的、有序的从1开始的编号。
   
```sql
//找出每个客户金额最高的 3 张发票，
//并在每张发票旁标明其在客户内部的金额排名
SELECT
    invoice_id,
    client_id,
    invoice_total,
    -- 核心逻辑：按客户分区，按金额降序排序，分配行号
    ROW_NUMBER() OVER (
        PARTITION BY client_id 
        ORDER BY invoice_total DESC) 
        AS client_rank
FROM
    invoices
-- 外部查询
WHERE client_rank <= 3; 
```
- 偏移函数：
    > `LAG(column, N)` (前后行对比)此功能用于获取窗口中当前行之前（或之后）某一行的数据，常用于时间序列分析和比较。
    
```sql
//将当前发票的金额与其上一次发票的金额进行对比,
//以分析客户消费金额的变化。
SELECT
    invoice_id,
    client_id,
    invoice_date,
    invoice_total,
    -- 核心逻辑：获取同一客户内、按日期排序后的前一行金额
    LAG(invoice_total, 1) OVER (PARTITION BY client_id ORDER BY invoice_date) AS previous_invoice_total,
    -- 通过计算来查看消费金额的增减
    invoice_total - LAG(invoice_total, 1) 
    OVER (PARTITION BY client_id 
    ORDER BY invoice_date) 
    AS amount_difference 
    //这里不可以直接减去previous_invoice_total，
    //因为SELECT子句几乎是同时执行的

FROM
    invoices;

// LAG(column, N)获取当前行向前数第 N 行的 column 值
//这里的 N=1 表示上一行
//OVER()窗口定义启用窗口计算模式
//PARTITION BY client_id窗口边界：确保只在同一个客户的发票之间进行比较
//ORDER BY invoice_date窗口排序：定义“上一次”的标准，即按日期排序
```
- 实用总结记住窗口函数的三要素：做什么 (Function): ROW_NUMBER(), LAG(), AVG(), SUM() 等。分界线 (PARTITION BY): 划定组的范围（例如 client_id）。内排序 (ORDER BY): 决定组内数据的顺序，这对排名和偏移尤其重要。

### ==L7== Data Manipulation Functions

#### 1. Numeric Functions
- ROUND
  
```sql
SELECT ROUND(5.729,2) //5.73
```

- round to the specified decimal places

```sql
SELECT ROUND(column_name, decimal_places) AS new_column_name
FROM table_name;
```

- CEILING

```sql
SELECT CEILING(5.729) //6
```

- FLOOR

```sql
SELECT FLOOR(5.729) //5
```

- ABS

```sql
SELECT ABS(-5.729) //5.729
```

- RAND

```sql
SELECT RAND() //0.729 in 0 to 1
```

#### 2. String Functions

- LENGTH

```sql
SELECT LENGTH('Hello World') //11
```

- SUBSTRING

```sql
SELECT SUBSTRING('Hello World', 1, 5) //Hello
```

- REPLACE

```sql
SELECT REPLACE('Hello World', 'World', 'MySQL') 
//Hello MySQL                               
```

- TRIM（去掉前后的空格）

```sql              
SELECT TRIM('Hello World') //Hello World
```

- UPPER

```sql
SELECT UPPER('Hello World') //HELLO WORLD
```

- LOWER

```sql
SELECT LOWER('Hello World') //hello world
```

- LOCATE (0 as default value if not found)   
  
```sql
SELECT LOCATE('World', 'Hello World') //6
```
#### 3. DATE Functions

- NOW

```sql
SELECT NOW() //2025-12-01 17:35:33
```
- YEAR

```sql
SELECT * FROM orders 
WHERE YEAR(order_date) >= YEAR(NOW())
``` 
- CURDATE

```sql
SELECT CURDATE() //2025-12-01
```

- CURTIME

```sql
SELECT CURTIME() //17:35:33
```

#### 4. Formatting Functions

- DATE_FORMAT

```sql
SELECT DATE_FORMAT('2025-12-01', '%Y-%m-%d') //2025-12-01
``` 

- TIME_FORMAT

```sql
SELECT TIME_FORMAT('17:35:33', '%H:%i:%s') //17:35:33
``` 

#### 5. Calculating Time
- DATE_ADD

```sql
SELECT DATE_ADD('2025-12-01', INTERVAL 1 DAY) //2025-12-02
```

- DATE_SUB

```sql
SELECT DATE_SUB('2025-12-01', INTERVAL 1 DAY) //2025-11-30
``` 

- TIMESTAMPDIFF

```sql
SELECT TIMESTAMPDIFF(DAY, '2025-12-01', '2025-12-02') //1
```     

#### 6. IFNULL and COALESCE

- IFNULL: return the message if the column is null
- COALESCE: return the first non-null value and can take multiple arguments

```sql
SELECT 
    IFNULL(column_name, 'message') AS default_value
    COALESCE(column_name, comment, 'message') AS default_value
    //here comments is another column in the table
FROM table_name;
```

#### 7. CONCAT

- 将同一行的多个列连接成一个字符串
- 如果有其中一个参数为null则这一行结果为null
- 如果不想因为一个null参数返回null，使用CONCAT_WS(separator, str1, str2, ...)
  
```sql
SELECT CONCAT(str1, str2, ...)AS new_column_name
    CONCAT_WS(separator, str1, str2, ...) AS new_column_name
```

#### 8. IF

```sql
SELECT 
    IF(condition, value_if_true, value_if_false) AS new_column_name
FROM table_name;
```

#### 9. CASE

```sql
SELECT 
    CASE
        WHEN condition1 THEN result1
        WHEN condition2 THEN result2
        ELSE result3
    END AS new_column_name 
    //column name
FROM table_name;
```

### ==L11== Transaction and Concurrency

#### 1. Transaction 
- ACID properties: Atomicity, Consistency, Isolation, Durability
  
#### 2. Creating Transaction

- **START TRANSACTION** / **BEGIN**
  - Start a new transaction 
- **关闭自动提交**
```sql
-- Check autocommit status (default is ON)
SHOW VARIABLES LIKE 'autocommit';

-- Disable autocommit
SET autocommit = 0;
```
- **COMMIT**
  - Save changes to the database (提交修改)
- **ROLLBACK**
  - Undo changes if something goes wrong (回滚/撤销修改)

```sql
USE sql_store;

START TRANSACTION; 

INSERT INTO orders (customer_id, order_date, status)
VALUES (1, '2019-01-01', 1);

INSERT INTO order_items
VALUES (LAST_INSERT_ID(), 1, 1, 1);
// LAST_INSERT_ID() returns the ID of the last inserted row
COMMIT; 
// ROLLBACK;
```

#### 3. Concurrency

- **Concurrency Problems** (并发问题)
  - **Lost Updates**: Two transactions update the same row, last one wins.
  - **Dirty Reads**: Reading uncommitted data. 
  //例如事务B读取了事务A回滚的无效数据
  - **Non-repeating Reads**: Reading different values for the same row in one transaction.
  //例如事务B读取了事务A修改后的数据 和修改前读取的数据不一致
  - **Phantom Reads**: Missing or seeing new rows in a range query.
  //例如事务B读取了事务A插入的新的数据 发现和原数据不一致 产生幻觉

- Higher isolation level, lower concurrency problems, 
  but lower concurrency and worse performance.

#### 4. Isolation Level
![Isolation level](https://img.cdn1.vip/i/692fa634d75e4_1764730420.webp)

- **Isolation Levels** (隔离级别)
  - **READ UNCOMMITTED**: Transactions can see changes made by others even before they are committed.
  - **READ COMMITTED**: Transactions only see changes that have been committed by other transactions.
  - **REPEATABLE READ**: (Default in MySQL) Establishes a consistent snapshot at the first read, ignoring changes committed by other transactions to ensure consistent results.
  - **SERIALIZABLE**: Forces transactions to run sequentially, as if one after another, by locking the accessed data.

```sql
-- Check isolation level
SHOW VARIABLES LIKE 'transaction_isolation';

-- Set isolation level for the next transaction
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set isolation level for the current session
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set isolation level for all sessions
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```
#### 5. Locking
- **Locking** (锁)
  - **Implicit Locking**: MySQL automatically locks rows during UPDATE/DELETE.
  - **Explicit Locking**: Manually locking rows to prevent others from modifying them.

```sql
-- Shared Lock (S Lock): Allows read, blocks write
SELECT * FROM customers 
WHERE customer_id = 1 LOCK IN SHARE MODE;

-- Exclusive Lock (X Lock): Blocks read and write (FOR UPDATE)
SELECT * FROM customers 
WHERE customer_id = 1 FOR UPDATE;
```

- **Deadlock**: Two transactions wait for each other to release locks.
  - *Example*: 
    - Transaction A holds Lock 1, waits for Lock 2.
    - Transaction B holds Lock 2, waits for Lock 1.
  - *Solution*: Update records in the same order in all transactions.

```sql
-- Transaction A
START TRANSACTION;
UPDATE customers SET state = 'VA' WHERE customer_id = 1;
-- ... pause ...
UPDATE orders SET status = 1 WHERE order_id = 1; 
-- Waits for B to release order 1

-- Transaction B
START TRANSACTION;
UPDATE orders SET status = 1 WHERE order_id = 1;
-- ... pause ...
UPDATE customers SET state = 'VA' WHERE customer_id = 1; 
-- Waits for A to release customer 1 -> DEADLOCK
```

### ==L12== Data Types

#### 1. String Types
- **CHAR(x)**: Fixed length string.
  - Use for fixed-length data like state abbreviations (e.g., 'CA', 'NY').
  - More efficient than VARCHAR for fixed length.
- **VARCHAR(x)**: Variable length string.
  - Max 65,535 characters (approx).
  - Use for names, addresses, emails.
- **TEXT**: For long strings.
  - TINYTEXT (255 bytes), TEXT (64KB), MEDIUMTEXT (16MB), LONGTEXT (4GB).
  - Use for blog posts, descriptions.

#### 2. Integer Types
- **TINYINT**: 1 byte [-128, 127]
- **SMALLINT**: 2 bytes [-32k, 32k]
- **MEDIUMINT**: 3 bytes [-8M, 8M]
- **INT**: 4 bytes [-2B, 2B]
- **BIGINT**: 8 bytes
- **UNSIGNED**: Disallow negative values, doubling the positive range.
  - e.g., `TINYINT UNSIGNED` [0, 255].
- **ZEROFILL**: Pads with zeros (e.g., 0001).

#### 3. Fixed-Point vs Floating-Point
- **DECIMAL(p, s)**: Fixed-point (Exact).
  - `p`: Precision (total digits).
  - `s`: Scale (digits after decimal).
  - Use for **Currency/Money** to avoid rounding errors.
  - e.g., `DECIMAL(9, 2)` -> 1234567.89
- **FLOAT / DOUBLE**: Floating-point (Approximate).
  - Use for scientific calculations where slight errors are acceptable.

#### 4. Boolean
- **BOOL / BOOLEAN**: Synonym for `TINYINT(1)`.
  - `TRUE` = 1, `FALSE` = 0.

#### 5. Enums and Sets
- **ENUM('small', 'medium', 'large')**:
  - Restricts a column to a single value from a list.
  - *Not recommended* (changing the list requires rebuilding the table).
- **SET(...)**:
  - Stores multiple values from a list.

#### 6. Date and Time Types
- **DATE**: 'YYYY-MM-DD'
- **TIME**: 'HH:MM:SS'
- **DATETIME**: 8 bytes. 'YYYY-MM-DD HH:MM:SS'.
  - Range: 1000 to 9999.
- **TIMESTAMP**: 4 bytes.
  - Range: 1970 to 2038 (Y2K38 problem).
  - Often used for `created_at` or `updated_at`.
- **YEAR**: 4-digit year.

#### 7. JSON Type
- Store JSON documents efficiently.
- Validate JSON syntax automatically.

```sql
UPDATE products
SET properties = '
{
  "dimensions": [1, 2, 3],
  "weight": 10,
  "manufacturer": {"name": "sony"}
}
'
WHERE product_id = 1;

-- Extracting data
SELECT 
    product_id, 
    JSON_EXTRACT(properties, '$.weight') AS weight,
    properties -> '$.dimensions[0]' AS width, -- '->' is shorthand for JSON_EXTRACT
    properties ->> '$.manufacturer.name' AS manufacturer_name -- '->>' removes quotes
FROM products;
```
### ==L13== Designing Database

### ==L14== Indexing

#### 1. Creating Index
- **Purpose**: Speed up queries by reducing scanned rows (from Full Table Scan to Index Scan).
- **Cost**: Slows down WRITE operations (INSERT/UPDATE/DELETE) and consumes disk space.

```sql
EXPLAIN SELECT customer_id FROM customers WHERE points > 1000;
-- type: ALL (Full Table Scan), rows: 1010

CREATE INDEX idx_points ON customers (points);
-- 创建一个名叫 idx_points 的索引，
-- 它是基于 customers 表里的 points 这一列生成的
EXPLAIN SELECT customer_id FROM customers WHERE points > 1000;
-- type: range (Index Range Scan), rows: 528 (Scanned fewer rows)
```

#### 2. Index Types
- **Primary Key Index**: Automatically created for primary key columns.
- **Unique Index**: Ensures column values are unique.
- **Normal Index**: Fastest for equality and range queries.
- **Full-Text Index**: For full-text search (MyISAM, InnoDB).

#### 3. View Index

- **Collation**: Sorting rule (A=Ascending, NULL=Unsorted).
- **Cardinality**: Estimated number of unique values. 
  - Higher (more unique values) is better for index efficiency.
- **Sub-part**: Number of indexed characters for Prefix Index (NULL for full column).

```sql
SHOW INDEXES IN customers; 
-- 显示 customers 表的索引信息

ANALYZE TABLE customers;
-- 更新表的统计信息

OPTIMIZE TABLE customers;
-- 重建表，回收未使用的空间

DROP INDEX idx_points ON customers;
-- 删除索引
```

#### 4. Prefix Index
- **Purpose**: Index only the first N characters of a string column (e.g., CHAR, VARCHAR, TEXT) to save space and improve performance.
- **Use Case**: Long strings (e.g., Description, Blog Content) where the full string is too large to index efficiently.

```sql
-- How to decide the length? (Check uniqueness)
SELECT 
    COUNT(DISTINCT LEFT(last_name, 1)), -- 1 char
    COUNT(DISTINCT LEFT(last_name, 5)), -- 5 chars (Optimal if close to total count)
    COUNT(DISTINCT last_name)           -- Total unique full names
FROM customers;

-- Then create an index on the first 5 characters of the last_name column
CREATE INDEX idx_lastname ON customers (last_name(5));
```
#### 4. Full-Text Index (Google)
- **Purpose**: Efficiently search for keywords in long text columns (e.g., Title, Body).
- **Mechanism**: Uses `MATCH(...) AGAINST(...)` instead of `LIKE` to calculate relevance scores.

```sql
-- 1. Create Full-Text Index
CREATE FULLTEXT INDEX idx_title_body ON posts (title, body);

-- 2. Search for 'react' or 'redux'
SELECT *, MATCH(title, body) AGAINST('react redux') AS relevance
FROM posts 
WHERE MATCH(title, body) AGAINST('react redux');
-- Returns rows containing 'react' or 'redux', sorted by relevance.
WHERE MATCH(title, body) AGAINST('"react redux"' IN BOOLEAN MODE);
-- Returns rows containing 'react' and 'redux', sorted by relevance.
WHERE MATCH(title, body) AGAINST('+react +redux' IN BOOLEAN MODE);
-- Returns rows containing 'react' and 'redux', sorted by relevance.
WHERE MATCH(title, body) AGAINST('+react -redux' IN BOOLEAN MODE);
-- Returns rows containing 'react' but not 'redux', sorted by relevance.
```

#### 5. Composite Index (dictionary)
- **Purpose**: Optimize queries that filter on **multiple columns** (e.g., State AND Points).
- **Mechanism**: Sorts by the first column, then by the second column (like a phone book: Last Name -> First Name).
- **Order Matters**: Put the most frequently used or most selective column first.

```sql
-- Create a composite index on state and points
CREATE INDEX idx_state_points ON customers (state, points);

-- Efficiently handles: WHERE state = 'CA' AND points > 1000
EXPLAIN SELECT customer_id FROM customers 
WHERE state = 'CA' AND points > 1000;
-- Uses idx_state_points because it can jump to 'CA' and scan points > 1000 directly.
```
- **Order Priority**:
  1. **High Cardinality First**: Put the most selective column first.(With higer cardinality) 选独特性高的
  2. **Most Frequently Used**: Put the column that is always in WHERE clause first.(Used more in query)
  3. **Query Dependent**: Optimize for your specific queries (e.g., sorting, range).
   
> get cardinality:
```sql
SELECT COUNT(DISTINCT state) FROM customers;
```

#### 6. Index Maintenance

- **Duplicate Index**: Same columns, same order, same type.
  - *Example*: `INDEX (A)` and `INDEX (A)`.
  - *Action*: Delete one.
- **Redundant Index**: One index is a prefix of another.
  - *Example*: `INDEX (A, B)` already covers `INDEX (A)`.
  - *Action*: Delete `INDEX (A)` because `(A, B)` can handle queries for `A` alone.
  - *Note*: `INDEX (B, A)` is NOT redundant to `(A, B)`.

#### 7. When Indexes are Ignored
  - **Expression on Column**: `WHERE points + 10 > 2000` (Bad) -> `WHERE points > 1990` (Good).
  - **Function on Column**: `WHERE YEAR(date) = 2019` (Bad) -> `WHERE date BETWEEN '2019-01-01' AND '2019-12-31'` (Good).
  - **Implicit Type Conversion**: Comparing string column with number.
- **Rule**: Always keep the indexed column **alone** on one side of the comparison operator.

#### 8. Covering Index
- **Definition**: An index that contains (covers) **all** the columns needed for a query.
- **Benefit**: Extremely fast because MySQL can satisfy the query using **only the index** without reading the actual table (No "Table Access").
- **Indicator**: `Extra: Using index` in EXPLAIN output.

```sql
-- Index: idx_state_points (state, points) + implicitly (customer_id)
// customer_id is not in the index, 
//but it is auto included in secondary index as the primary key
EXPLAIN SELECT customer_id, state FROM customers ORDER BY state;
-- Result: Extra = Using index (All data is in the index)
```
#### 9. Using Indexes for Sorting
- **Goal**: Avoid expensive `Filesort` by using the index's natural order.
- **Index**: `(state, points)`

| Query                                | Status     | Reason                                               |
| :----------------------------------- | :--------- | :--------------------------------------------------- |
| `ORDER BY state`                     | ✅ Index    | Matches 1st column.                                  |
| `ORDER BY state, points`             | ✅ Index    | Matches full index order.                            |
| `WHERE state = 'CA' ORDER BY points` | ✅ Index    | `state` is constant, `points` is sorted within 'CA'. |
| `ORDER BY points`                    | ❌ Filesort | Skips 1st column (`state`).                          |
| `ORDER BY state, id`                 | ❌ Filesort | `id` is not in index definition (explicitly).        |
| `ORDER BY state ASC, points DESC`    | ❌ Filesort | Mixed direction (unless MySQL 8+ Descending Index).  |

- **Check**: `EXPLAIN` -> `Extra`:
  - `Using filesort`: Bad (Sorting in memory/disk).
  - `NULL` / `Using index`: Good (Using pre-sorted index).
