在新建或者修改了领域模型之后,往往还需要对存储数据的数据源进行修改,过程繁琐且容易出错,为提高效率Obase提供了结构映射功能.

Obase的结构映射会根据当前配置的对象数据模型在数据源内建立和修改表结构.

**需要注意的是,结构映射需要多次探测数据源的表结构,性能开销很大,应当将结构映射作为开发时迁移数据库或者建立数据库时的工具使用,不要在生产环境中使用.**

由于某个上下文的对象数据模型只会建造一次,所以结构映射也只会执行一次.此外结构映射需要对应数据源的管理权限,如果提供的凭据没有此权限会抛出相应的异常.

## 模型内的结构映射相关方法

在模型中的属性配置上有以下几个方法用于配置结构映射,以下代码中的attrConfig就是属性配置项:
```
//最大字符数 仅限字符串类型
attrConfig.hasMaxcharNumber(500);
//精度 只支持为映射类型decimal设置精度
attrConfig.hasPrecision(10);
//设置是否可空 对主键不生效
attrConfig.hasNullable(true);
```

这些方法配置的结果会影响进行结构映射时的字段定义.

## 启用结构映射

在上下文配置提供器中,重写protected boolean getEnableStructMapping()方法,当此方法的返回值设置为true时,在建造对象数据模型之后就会执行结构映射.

## 结构映射的数据库字段

以下为Obase进行结构映射时建立表的字段类型:

| Obase基元类型       | MySql数据类型 | Sqlite数据类型 | SqlServer数据类型 | PostgreSQL数据类型   |
|:----------------|:----------|:-----------|:--------------|:-----------------|
| boolean/Boolean | tinyint   | INTEGER    | tinyint       | boolean          |
| byte/Byte       | tinyint   | INTEGER    | tinyint       | smallint         |
| char/Character  | char      | TEXT       | char          | char             |
| LocalDateTime   | datetime  | TEXT       | datetime      | timestamp        |
| BigDecimal      | decimal   | REAL       | decimal       | decimal          |
| double/Double   | double    | REAL       | real          | double precision |
| Enum            | tinyint   | INTEGER    | tinyint       | smallint         |
| float/Float     | float     | REAL       | float         | real             |
| UUID            | varchar   | TEXT       | nvarchar      | varchar          |
| int/Integer     | int       | INTEGER    | int           | integer,         |
| long/Long       | bigint    | INTEGER    | bigint        | bigint           |
| short/Short     | smallint  | INTEGER    | smallint      | smallint         |
| String          | varchar   | TEXT       | nvarchar      | varchar          |
| LocalTime       | time      | TEXT       | time          | time             |

对于字符串类型,使用hasMaxCharNumber可以指定字段的长度,如果长度设置超过255(SqlServer为500)时,会被升级为Text字段用于保存长文本.

下表为默认情况下,字符串类型的字段长度:

| MySql数据长度 | Sqlite数据长度 | SqlServer数据长度 | PostgreSQL数据长度 |
|:----------|:-----------|:--------------|:---------------|
| 40        | 不适用        | 40            | 40             |

对于BigDecimal类型,使用hasPrecision可以指定字段的精度,设置的值对应字段中小数部分的位数,应当在0-28之间.

下表为默认情况下,十进制数类型(BigDecimal)的字段精度:

| MySql数据精度 | Sqlite数据精度 | SqlServer数据精度 | PostgreSQL数据精度 |
|:----------|:-----------|:--------------|:---------------|
| (65,0)    | 不适用        | (38,0)        | (65,0)         |