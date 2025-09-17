Obase的结构映射是用于数据迁移的工具,会根据当前配置的对象数据模型在数据源内建立和修改表结构.

结构映射的配置项为配置提供器中的可选重写方法protected override bool EnableStructMapping { get; },当此属性的返回值设置为true时,在建造对象数据模型之后就会执行结构映射.

由于某个上下文的对象数据模型只会建造一次,所以结构映射也只会执行一次.此外结构映射需要对应数据源的管理权限,如果提供的凭据没有此权限会抛出相应的异常.

以下为Obase进行结构映射时建立表的字段类型:

| Obase基元类型    | MySql数据类型 | Sqlite数据类型 | SqlServer数据类型 | PostgreSQL数据类型   |
|:-------------|:----------|:-----------|:--------------|:-----------------|
| bool         | tinyint   | INTEGER    | tinyint       | boolean          |
| byte/sbyte   | tinyint   | INTEGER    | tinyint       | smallint         |
| char         | char      | TEXT       | char          | char             |
| DateTime     | datetime  | TEXT       | datetime      | timestamp        |
| decimal      | decimal   | REAL       | decimal       | decimal          |
| double       | double    | REAL       | real          | double precision |
| Enum         | tinyint   | INTEGER    | tinyint       | smallint         |
| float        | float     | REAL       | float         | real             |
| guid         | varchar   | TEXT       | nvarchar      | varchar          |
| int/uint     | int       | INTEGER    | int           | integer,         |
| long/ulong   | bigint    | INTEGER    | bigint        | bigint           |
| short/ushort | smallint  | INTEGER    | smallint      | smallint         |
| String       | varchar   | TEXT       | nvarchar      | varchar          |
| TimeSpan     | time      | TEXT       | time          | time             |

以下为Obase进行结构映射时建立表的默认的字段长度,可以在属性的配置上调用HasMaxLength方法来设定长度.
由于映射为string的类型和decimal类型需要长度,所以表内只有这两种类型:

| Obase基元类型 | MySql数据长度 | Sqlite数据长度 | SqlServer数据长度 | PostgreSQL数据长度 |
|:----------|:----------|:-----------|:--------------|:---------------|
| decimal   | (65,4)    | 不适用        | (38,4)        | (65,4)         |
| String    | 40        | 不适用        | 40            | 40             |

在属性配置上使用HasMaxLength方法设定的长度会改变表中String的长度,使用HasPrecision会该表Decimal类型中第二个数值的长度,最大为30.

如果属性的映射字段类型时字符串类型,使用HasMaxLength方法设定的长度设置超过255(SqlServer为500)时,会被升级为Text字段用于保存长文本.

值得注意的是,结构映射需要多次探测数据源的表结构,性能开销很大,应当将结构映射作为开发时迁移数据库或者建立数据库时的工具使用,不要在生产环境中使用.