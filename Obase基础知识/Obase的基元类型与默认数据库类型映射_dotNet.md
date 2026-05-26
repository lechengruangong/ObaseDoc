Obase统一将一些语言内置类型定义为Obase基元类型,基元类型一般被用作实体型或关联型的属性类型,在数据源中一般映射为字段进行存储.

这些基元类型作为属性或者反持久化构造函数参数且数据库类型为默认映射时无需进行额外的配置(即无需配置属性的设值器和取值器,构造函数参数的转换函数).

在dotNet中Obase基元类型为内置基元类型+基元类型的Nullable<>包装结构+string(字符串)类型+decimal(精确十进制数)+decimal的Nullable<>包装结构+datetime(日期时间)+datetime的Nullable<>包装结构+timespan(时间) +timespan的Nullable<>包装结构+guid+GUID的Nullable<>包装结构+枚举.

以下是这些Obase基元类型在各个数据库的默认映射:

| Obase基元类型    | MySql数据类型             | Sqlite数据类型 | SqlServer数据类型 | PostgreSQL数据类型   |
|:-------------|:----------------------|:-----------|:--------------|:-----------------|
| bool         | tinyint,int           | INTEGER    | tinyint,int   | boolean          |
| byte/sbyte   | tinyint,int           | INTEGER    | tinyint,int   | smallint,integer |
| char         | char,varchar          | TEXT       | char          | char,varchar     |
| DateTime     | datetime,date         | TEXT       | datetime      | timestamp        |
| decimal      | decimal               | REAL(可能有损) | decimal       | decimal          |
| double       | double,decimal        | REAL       | real          | double precision |
| Enum         | tinyint,int           | INTEGER    | tinyint,int   | smallint,integer |
| float        | float,decimal         | REAL       | float         | real             |
| guid         | varchar,text,longtext | TEXT       | nvarchar      | varchar,text     |
| int/uint     | int,bigint            | INTEGER    | int,bigint    | integer,bigint   |
| long/ulong   | bigint                | INTEGER    | bigint        | bigint           |
| short/ushort | smallint,int          | INTEGER    | smallint,int  | smallint,integer |
| String       | varchar,text,longtext | TEXT       | nvarchar      | varchar,text     |
| TimeSpan     | time                  | TEXT       | time          | time             |