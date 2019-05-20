---
layout:     post
title:      MySQL函数
subtitle:   MySQL Function
date:       2019-05-20
author:     Monk
header-img: img/Basic_operation_of_MySQL.jpg
catalog: true
tags:
    - MySQL
---

## MySQL函数

### 数学函数
1. 绝对值函数 **ABS(x)** `SELECT ABS(-2); // 2`;
2. 圆周率函数 **PI()** `SELECT PI(); // 3.141593`;
3. 平方根函数 **SQRT(x)** `SELECT SQRT(16); // 4`;
4. 求余函数 **MOD (x, y)** `SELECT MOD(10, 3); // 1`;
5. 获取整数的函数：
   * CEIL(x),CEILING(x): 返回不小于 x 的最小整数
   ```sql
    CEIL(-3.35); // -3
    CEILING(3.35); // 4
   ```
   * FLOOR(x): 获取不大于 x 的最大整数
   ```sql
    FLOOR(-3.35); // -4
    FLOOR(3.35); // 3
   ```
6. 随机数函数 RAND(), RAND(x);
7. 四舍五入函数 ROUND(x), ROUND(x,y), TRUNCATE(x,y);
### 字符串处理函数
1. 字符串截取：**LEFT(str, length)** `SELECT LEFT('sqlstudy.com', 3); // sql`;
2. 字符串截取：**RIGHT(str, length)** `SELECT RIGHT('sqlstudy.com', 3); // com`;
3. 字符串截取：**SUBSTRING(str, pos);** **substring(str, pos, len)**：pos 可以是负值，但 len 不能取负值。  
   3.1 从字符串的第 4 个字符位置开始取，直到结束。
   ```sql
   mysql> SELECT SUBSTRING('sqlstudy.com', 4);
   +------------------------------+
   | SUBSTRING('sqlstudy.com', 4) |
   +------------------------------+
   | study.com                    |
   +------------------------------+
   ```
   3.2 从字符串的第 4 个字符位置开始取，只取 2 个字符。  
   ```sql
   mysql> SELECT SUBSTRING('sqlstudy.com', 4, 2);
   +---------------------------------+
   | SUBSTRING('sqlstudy.com', 4, 2) |
   +---------------------------------+
   | st                              |
   +---------------------------------+
   ```
   3.3 从字符串的第 4 个字符位置（倒数）开始取，直到结束。  
   ```sql
   mysql> SELECT SUBSTRING('sqlstudy.com', -4);
   +-------------------------------+
   | SUBSTRING('sqlstudy.com', -4) |
   +-------------------------------+
   | .com                          |
   +-------------------------------+
   ```
   3.4 从字符串的第 4 个字符位置（倒数）开始取，只取 2 个字符。
   ```sql
   mysql> SELECT SUBSTRING('sqlstudy.com', -4, 2);
   +----------------------------------+
   | SUBSTRING('sqlstudy.com', -4, 2) |
   +----------------------------------+
   | .c                               |
   +----------------------------------+
   ```
4. 字符串截取：**SUBSTRING_INDEX(str,delim,count)**
   4.1 截取第二个 '.' 之前的所有字符。  
   ```sql
   mysql> SELECT SUBSTRING_INDEX('www.sqlstudy.com.cn', '.', 2);
   +------------------------------------------------+
   | SUBSTRING_INDEX('www.sqlstudy.com.cn', '.', 2) |
   +------------------------------------------------+
   | www.sqlstudy                                   |
   +------------------------------------------------+
   ```
   4.2 截取第二个 '.' （倒数）之后的所有字符。
   ```sql
   mysql> SELECT SUBSTRING_INDEX('www.sqlstudy.com.cn', '.', -2);
   +-------------------------------------------------+
   | SUBSTRING_INDEX('www.sqlstudy.com.cn', '.', -2) |
   +-------------------------------------------------+
   | com.cn                                          |
   +-------------------------------------------------+
   ```
   4.3 如果在字符串中找不到 delim 参数指定的值，就返回整个字符串
   ```sql
   mysql> SELECT SUBSTRING_INDEX('www.sqlstudy.com.cn', '.coc', 1);
   +---------------------------------------------------+
   | SUBSTRING_INDEX('www.sqlstudy.com.cn', '.coc', 1) |
   +---------------------------------------------------+
   | www.sqlstudy.com.cn                               |
   +---------------------------------------------------+
   ```
5. **ASCII(str)**：返回字符串str的最左面字符的ASCII代码值。如果str是空字符串，返回0。如果str是NULL，返回NULL。
   * SELECT ASCII('') -> 0
   * SELECT ASCII('1') -> 49
   * SELECT ASCII('123') -> 49（只取最左边的字符）
   * SELECT ASCII(NULL) -> NULL
6. **CONCAT(str1,str2,...)**：返回来自于参数连结的字符串。如果任何参数是NULL，返回NULL。可以有超过2个的参数。一个数字参数被变换为等价的字符串形式。
   SELECT CONCAT('12',NULL,'3'); -> NULL
   SELECT CONCAT('12','NULL','3'); -> 12NULL3
   SELECT CONCAT('My', 'S', 'QL'); -> MySQL
7. **LENGTH(str)**：计算str的长度
   SELECT LENGTH('text'); -> 4
8. **LTRIM(str)**:返回删除了其前置空格字符的字符串str, RTRIM(str):返回删除了其拖后空格字符的字符串str, TRIM([[BOTH | LEADING | TRAILING] [remstr] FROM] str):返回字符串str，其所有remstr前缀或后缀被删除了。如果没有修饰符BOTH、LEADING或TRAILING给出，BOTH被假定。如果remstr没被指定，空格被删除。
9. **REPLACE(str,from_str,to_str)**:返回字符串str，其字符串from_str的所有出现由字符串to_str代替。
```
mysql> SELECT REPLACE('www.mysql.com', 'w', 'Ww'); -> 'WwWwWw.mysql.com'
```
10. 综合实例：
```
address : XX门牌号, XX楼, XX号, XX街道, XX市, XX省, 中国
UPDATE table SET address = LEFT(REPLACE(address, substring_index(address, ',', -2), ''), CHAR_LENGTH(REPLACE(address, substring_index(address, ',', -2), '')) - 1);
```