### 设置默认时间为插入时间
default datetime(CURRENT_TIMESTAMP,'localtime')

### 没有直接对时间的减法，但是可以直接比较得出范围

1	date(timestring, modifier, modifier, ...)	以 YYYY-MM-DD 格式返回日期。
2	time(timestring, modifier, modifier, ...)	以 HH:MM:SS 格式返回时间。
3	datetime(timestring, modifier, modifier, ...)	以 YYYY-MM-DD HH:MM:SS 格式返回。
4	julianday(timestring, modifier, modifier, ...)	这将返回从格林尼治时间的公元前 4714 年 11 月 24 日正午算起的天数。
5	strftime(format, timestring, modifier, modifier, ...)	这将根据第一个参数指定的格式字符串返回格式化的日期。

时间字符串
一个时间字符串可以采用下面任何一种格式：
序号	时间字符串	实例
1	YYYY-MM-DD	2010-12-30
2	YYYY-MM-DD HH:MM	2010-12-30 12:10
3	YYYY-MM-DD HH:MM:SS.SSS	2010-12-30 12:10:04.100
4	MM-DD-YYYY HH:MM	30-12-2010 12:10
5	HH:MM	12:10
6	YYYY-MM-DDTHH:MM	2010-12-30 12:10
7	HH:MM:SS	12:10:01
8	YYYYMMDD HHMMSS	20101230 121001
9	now	2013-05-07
您可以使用 "T" 作为分隔日期和时间的文字字符。
修饰符（Modifier）
时间字符串后边可跟着零个或多个的修饰符，这将改变有上述五个函数返回的日期和/或时间。任何上述五大功能返回时间。修饰符应从左到右使用，下面列出了可在 SQLite 中使用的修饰符：
NNN days
NNN hours
NNN minutes
NNN.NNNN seconds
NNN months
NNN years
start of month
start of year
start of day
weekday N
unixepoch
localtime
utc
格式化
SQLite 提供了非常方便的函数 strftime() 来格式化任何日期和时间。您可以使用以下的替换来格式化日期和时间：
替换	描述
%d	一月中的第几天，01-31
%f	带小数部分的秒，SS.SSS
%H	小时，00-23
%j	一年中的第几天，001-366
%J	儒略日数，DDDD.DDDD
%m	月，00-12
%M	分，00-59
%s	从 1970-01-01 算起的秒数
%S	秒，00-59
%w	一周中的第几天，0-6 (0 is Sunday)
%W	一年中的第几周，01-53
%Y	年，YYYY
%%	% symbol

下面是计算给定 UNIX 时间戳 1092941466 相对本地时区的日期和时间：
sqlite> SELECT datetime(1092941466, 'unixepoch', 'localtime');
2004-08-19 11:51:06

sqlite> SELECT julianday('now') - julianday('1776-07-04');
86504.4775830326