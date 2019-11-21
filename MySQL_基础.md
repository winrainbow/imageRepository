# MySQL

## SQL基础

### 数据类型：

1. 数值类型
   1. 整型：TINYINT(1),SMALLINT(2),MEDIUMINT(3),INT(4),BIGINT(8) *()中为字节数*
   2. 定点数：DECIMAL 、NUMERIC，小数点后的位数是固定的 DECIMAL(14,5)其中总长度是14位，小数点后是5位
   3. 浮点数：FLOAT DOUBLE ,DOUBLE(M,D):一共显示M个整数，其中D个整数位于小数点后面
2. 日期/时间类型
   * DATETIME
   * DATE
   * TIMESTAMP
   * TIME
   * YEAR
3. 字符串类型
   * CHAR VARCHAR CHAR(30)其中30是字符个数，不是字节个数
   * BINARY VARBINARY  二进制串
   * BLOB 二进制大对象
   * TEXT 长字符串
   * ENUM 枚举类型

### 函数

1. 数值函数：
   1. 算数操作符（+ - * / DIV(整除)）
   2. 数学函数
      * ABS(X) 取绝对值
      * CEILI(x) 返加不小于X的最小整数值
      * FLOOR(X)：不于于x的最大整数
      * CRC32(X)
      * RAND() RAND(N)
      * SIGN(X)
      * TRUNCATE(X,D)：
      * ROUND(X) ROUND(X,D):返回x最近似的整数，如果入参为两个，则返回x，保留D位小数，四舍五入
   3. 字符串类型处理函数
      * CHAR_LENGTH(str) 返回字符长度
      * LENGTH(str) 返回str长度，单位为字节，
      * CONCAT(str1,str2,...)返回连接后的结果
      * LEFT(str,len) 从字符串str开始，最左的len个字符
      * RIGHT(str,len),同上，返回最右len个字符
      * SUBSTRING(str,pos)  返回子串，从pos 到 end
      * SUBSTRING(str,pos，len) 返回子串，从pos 到 pos+len
      * LOWER(str), UPPER(str)
   4. 时间及日期函数
      * NOW():返回YYYY-MM-DD HH:mm:ss
      * CURTIME(): 返回HH:MM:SS
      * CURDATE() 返回 YYYY-MM-DD
      * DATEDIFF(expr1,expr2): 返回expr1 与expr2 之间相差的天数，为正数或负数
      * DATE_ADD(date,INTERVAL expr type) :select DATE_ADD('1997-12-31 23:59:59',INTERVAL 1 SECOND); 结果：1998-01-01 00：00：00
      * DATE_SUB(date,INTERVAL expr type) 同上，减，type取值：SECOND MINUTE HOUR DAY WEEK MONTH YEAR
      * DATE_FORMAT（date,format）:根据格式，格式化时间显示样式
      * STR_TO_DATE(str,format): 把字符串转成 date
   
2. DML

   1. DISTINCT 获取不重复的唯一值：select distinct name from students 去重

   2. 聚集函数：

      * COUNT
      * MIN select min(money) from account;
      * MAX
      * AVG
      * SUM

   3. 分组统计 group by :  select name ,sum(money)  from account group by name order by sum(money) desc;

      HAVING 语句，对聚集进行筛选 select name ,sum(money) as  total  from account group by name having total>200 order by total desc;
      
   4. UNION / UNIOＮ ALL 将多个查询结果合并到一起，UNIONALL 更快，因UNION增加了 DISTINCT功能
   
   5. JOIN（连接）：JOIN、LEFT JOIN、RIGHT JOIN;
   
      左连接where只影向右表，右连接where只影响左表。
   
      Left Join
   
      select * from tbl1 Left Join tbl2 where tbl1.ID = tbl2.ID
   
      左连接后的检索结果是显示tbl1的所有数据和tbl2中满足where 条件的数据。
   
      简言之 Left Join影响到的是右边的表
   
      Right Join
   
      select * from tbl1 Right Join tbl2 where tbl1.ID = tbl2.ID
   
      检索结果是tbl2的所有数据和tbl1中满足where 条件的数据。
   
      简言之 Right Join影响到的是左边的表。  

### 索引：

MySql 主要支持B树索引、散列索引、空间索引和全文索引，由于是在存储引擎层实现的索引，所以不同的存储引擎会有一些差异

逻辑上可以分为：单列索引、复合索引、唯一索引和非唯一索引

> 如果索引键值的逻辑顺序和索引服务的表中相应行的物理顺序相同，则称为簇索引也称为聚集索引
>
> 也就是说索引的叶子节点中，保存了真实的数据

**EXPLAIN工具：可以确认sql语句执行计划是否良好，查询是否走了合理的索引**



