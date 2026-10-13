Obase统一将一些语言内置类型定义为Obase基元类型,基元类型一般被用作实体型或关联型的属性类型,在数据源中一般映射为字段进行存储.

这些基元类型作为属性或者反持久化构造函数参数且数据库类型为默认映射时无需进行额外的配置(即无需配置属性的设值器和取值器,构造函数参数的转换函数).

在java中Obase基元类型为内置基元类型+基元类型的包装类+string(字符串)类型+bigdecimal(精确十进制数)+ date(早期版本日期时间)+ localdate(本地日期)+localdatetime(本地日期时间)+localtime(本地时间) + uuid+枚举

以下是这些Obase基元类型在各个数据库的默认映射:


| Obase基元类型     | MySql数据类型             | Sqlite数据类型 | SqlServer数据类型 | PostgreSQL数据类型   |
|:--------------|:----------------------|:-----------|:--------------|:-----------------|
| bigdecimal    | decimal               | REAL(可能有损) | decimal       | decimal          |
| boolean       | tinyint               | INTEGER    | tinyint       | boolean          |
| byte          | tinyint               | INTEGER    | tinyint       | character        |
| char          | char                  | INTEGER    | char          | character        |
| date          | datetime              | TEXT       | datetime      | timestamp        |
| localdate     | datetime              | TEXT       | datetime      | date             |
| localdatetime | datetime              | TEXT       | datetime      | timestamp        |
| double        | double                | REAL       | real          | double precision |
| Enum          | tinyint               | INTEGER    | tinyint       | smallint         |
| float         | float                 | REAL       | float         | real             |
| int           | int                   | INTEGER    | int           | int              |
| long          | bigint                | INTEGER    | bigint        | bigint           |
| localtime     | time                  | TEXT       | time          | time             |
| short         | smallint              | INTEGER    | smallint      | smallint         |
| String        | varchar               | TEXT       | nvarchar      | varchar          |
| uuid          | varchar               | TEXT       | nvarchar      | varchar          |

需要注意几点:

1. 上表是仅按类型得到的默认映射.对于需要长度或精度的类型,Obase还会在类型后面补上长度或精度,具体规则见下面几条.

2. 对于String和uuid类型,Obase会给字段设置长度,长度取属性值的长度除以8,如果结果为0则按40处理.当长度超过255时,MySql和PostgreSQL会映射为Text,SqlServer的阈值则是500(超过500映射为Text),Sqlite则忽略长度直接映射为TEXT.

3. 对于bigdecimal类型,MySql和PostgreSQL映射为decimal(65,精度),SqlServer映射为decimal(38,精度),精度由属性的精度配置决定,Sqlite忽略精度直接映射为REAL,因此Sqlite下bigdecimal的数据可能会有精度损失.

4. 对于byte和char类型,只有PostgreSQL会带上长度映射为character(长度,长度取属性值的长度除以8,如果结果为0则按2处理),MySql、Sqlite和SqlServer都不带长度.

5. 对于自增的主键,PostgreSQL会把类型换成smallserial(对应short)、serial(对应int)或bigserial(对应long),其他数据源则在建表语句里追加各自的自动增长语法.
