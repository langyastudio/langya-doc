> 本文来自[https://www.begtut.com/sql](https://www.begtut.com/sql)，郎涯进行简单排版与补充

### MySQL 聚合函数

- [AVG](https://www.begtut.com/sql/func-mysql-avg.html) - 计算一组值或表达式的平均值。
- [COUNT](https://www.begtut.com/sql/func-mysql-count.html) - 计算表中的行数。
- [INSTR](https://www.begtut.com/sql/func-mysql-instr.html) - 返回字符串中第一次出现的子字符串的位置。
- [SUM ](https://www.begtut.com/sql/func-mysql-sum.html) - 计算一组值或表达式的总和。
- [MIN](https://www.begtut.com/sql/func-mysql-min.html) - 在一组值中找到最小值
- [MAX](https://www.begtut.com/sql/func-mysql-max.html) - 在一组值中找到最大值



### MySQL 字符串函数

- [CONCAT](https://www.begtut.com/sql/func-mysql-concat.html) - 将两个或多个字符串组合成一个字符串。
- [LENGTH＆CHAR_LENGTH ](https://www.begtut.com/sql/func-mysql-length.html) - 获取字符串的长度，以字节和字符为单位。
- [LEFT](https://www.begtut.com/sql/func-mysql-left.html) - 获取具有指定长度的字符串的左侧部分。
- [REPLACE](https://www.begtut.com/sql/func-mysql-replace.html) - 搜索并替换字符串中的子字符串。
- [SUBSTRING](https://www.begtut.com/sql/func-mysql-SUBSTRING.html) - 从具有特定长度的位置开始提取子字符串。
- [TRIM](https://www.begtut.com/sql/func-mysql-trim.html) - 从字符串中删除不需要的字符。
- [FIND_IN_SET](https://www.begtut.com/sql/func-mysql-find-in-set.html) - 在以逗号分隔的字符串列表中查找字符串。
- [FORMAT](https://www.begtut.com/sql/func-mysql-format.html) - 格式化具有特定区域设置的数字，四舍五入到小数位数



### MySQL 控制流功能

- [CASE](https://www.begtut.com/sql/func-mysql-case.html) - `THEN`如果`WHEN`满足分支中的条件，则返回分支中的相应结果，否则返回`ELSE`分支中的结果。
- [IF](https://www.begtut.com/sql/func-mysql-if.html) - 根据给定条件返回值。
- [IFNULL](https://www.begtut.com/sql/func-mysql-ifnull.html) - 如果它不是NULL则返回第一个参数，否则返回第二个参数。
- [NULLIF](https://www.begtut.com/sql/func-mysql-nullif.html) - 如果第一个参数等于第二个参数，[则](https://www.begtut.com/sql/func-mysql-nullif.html)返回NULL，否则返回第一个参数。



### MySQL 日期和时间函数

- [CURDATE](https://www.begtut.com/sql/func-mysql-curdate.html) - 返回当前日期。
- [DATEDIFF](https://www.begtut.com/sql/func-mysql-datediff.html) - 计算两个`DATE`值之间的天数  。
- [DAY](https://www.begtut.com/sql/func-mysql-day.html) - 获取指定日期的月份日期。
- [DATE_ADD](https://www.begtut.com/sql/func-mysql-date-add.html) - 将日期值添加到日期值。
- [DATE_SUB](https://www.begtut.com/sql/func-mysql-date-sub.html) - 从日期值中减去时间值。
- [DATE_FORMAT](https://www.begtut.com/sql/func-mysql-date-format.html) - 根据指定的日期格式格式化日期值。
- [DAYNAME](https://www.begtut.com/sql/func-mysql-dayname.html) - 获取指定日期的工作日名称。
- [DAYOFWEEK](https://www.begtut.com/sql/func-mysql-dayofweek.html) - 返回日期的工作日索引。
- [EXTRACT](https://www.begtut.com/sql/func-mysql-extract.html) - 提取日期的一部分。
- [NOW](https://www.begtut.com/sql/func-mysql-now.html) - 返回执行语句的当前日期和时间。
- [MONTH](https://www.begtut.com/mysql/mysql-month.html) - 返回表示指定日期月份的整数。
- [STR_TO_DATE](https://www.begtut.com/sql/func-mysql-str-to-date.html) - 根据指定的格式将字符串转换为日期和时间值。
- [SYSDATE](https://www.begtut.com/sql/func-mysql-sysdate.html) - 返回当前日期。
- [TIMEDIFF](https://www.begtut.com/sql/func-mysql-timediff.html) - 计算两个`TIME`或`DATETIME`值之间的差异。
- [TIMESTAMPDIFF](https://www.begtut.com/sql/func-mysql-timestampdiff.html) - 计算两个`DATE`或`DATETIME`值之间的差异。
- [WEEK](https://www.begtut.com/sql/func-mysql-week.html) - 返回一个星期的日期。
- [WEEKDAY](https://www.begtut.com/sql/func-mysql-weekday.html) - 返回日期的工作日索引。
- [YEAR](https://www.begtut.com/sql/func-mysql-year.html) -返回日期值的年份部分。



### MySQL 比较功能

- [COALESCE](https://www.begtut.com/sql/func-mysql-coalesce.html) - 返回第一个非null参数，这对于替换null非常方便。
- [GREATEST＆LEAST](https://www.begtut.com/sql/func-mysql-greatest.html) - 取n个参数并分别返回n个参数的最大值和最小值。
- [ISNULL](https://www.begtut.com/mysql/mysql-isnull-function.html) - 如果参数为null，则返回1，否则返回零。



### MySQL 数学函数

- [ABS](https://www.begtut.com/sql/func-mysql-abs.html) - 返回数字的绝对值。
- [CEIL](https://www.begtut.com/sql/func-mysql-ceil.html) - 返回大于或等于输入数字的最小整数值。
- [FLOOR](https://www.begtut.com/sql/func-mysql-floor.html) - 返回不大于参数的最大整数值。
- [MOD](https://www.begtut.com/sql/func-mysql-mod.html) - 返回数字的余数除以另一个。
- [ROUND](https://www.begtut.com/sql/func-mysql-round.html) - 将数字四舍五入到指定的小数位数。
- [TRUNCATE](https://www.begtut.com/sql/func-mysql-truncate.html) - 将数字截断为指定的小数位数。



### 其他 MySQL 功能

- [LAST_INSERT_ID](https://www.begtut.com/sql/func-mysql-last-insert-id.html) - 获取最后生成的最后一个插入记录的序列号。
- [CAST](https://www.begtut.com/sql/func-mysql-cast.html) - 将任何类型的值转换为具有指定类型的值。



### 更多 MySQL 函数

- 请查看访问：[MySQL 函数](https://www.begtut.com/sql/sql-ref-mysql.html)