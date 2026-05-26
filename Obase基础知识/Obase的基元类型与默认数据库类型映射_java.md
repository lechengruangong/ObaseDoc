Obase统一将一些语言内置类型定义为Obase基元类型,基元类型一般被用作实体型或关联型的属性类型,在数据源中一般映射为字段进行存储.

这些基元类型作为属性或者反持久化构造函数参数且数据库类型为默认映射时无需进行额外的配置(即无需配置属性的设值器和取值器,构造函数参数的转换函数).

在java中Obase基元类型为内置基元类型+基元类型的包装类+string(字符串)类型+bigdecimal(精确十进制数)+ date(早期版本日期时间)+ localdate(本地日期)+localdatetime(本地日期时间)+localtime(本地时间) + uuid(6.1新增)+枚举

以下是这些Obase基元类型在各个数据库的默认映射:


| Obase基元类型     | MySql数据类型             | Sqlite数据类型 | SqlServer数据类型 | PostgreSQL数据类型   |
|:--------------|:----------------------|:-----------|:--------------|:-----------------|
| bigdecimal    | decimal               | REAL(可能有损) | decimal       | decimal          |
| boolean       | tinyint,int           | INTEGER    | tinyint,int   | boolean          |
| byte          | tinyint,int           | INTEGER    | tinyint,int   | smallint,integer |
| char          | char,varchar          | TEXT       | char          | char,varchar     |
| date          | datetime              | TEXT       | datetime      | timestamp        |
| double        | double,decimal        | REAL       | real          | double precision |
| Enum          | tinyint,int           | INTEGER    | tinyint,int   | smallint,integer |
| float         | float,decimal         | REAL       | float         | real             |
| int           | int,bigint            | INTEGER    | int,bigint    | integer,bigint   |
| long          | bigint                | INTEGER    | bigint        | bigint           |
| localdatetime | datetime              | TEXT       | datetime      | timestamp        |
| localdate     | date                  | TEXT       | date          | date             |
| localTime     | time                  | TEXT       | time          | time             |
| short         | smallint,int          | INTEGER    | smallint      | smallint,int     |
| String        | varchar,text,longtext | TEXT       | nvarchar      | varchar,text     |
| uuid          | varchar,text,longtext | TEXT       | nvarchar      | varchar,text     |
