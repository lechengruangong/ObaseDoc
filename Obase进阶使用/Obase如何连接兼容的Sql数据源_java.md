通常情况下,使用Obase都是引用对应数据库的提供器,即io.obase.providers.mysql,io.obase.providers.oracle,io.obase.providers.sqlserver,io.obase.providers.sqlite,io.obase.providers.postgresql这四个软件包,并继承其中提供的上下文配置抽象类来实现自己需要的上下文配置.

这些提供器都引用了相应的官方驱动包来作为相应的连接提供程序,但有些时候我们会遇到需要使用非官方的驱动连接数据源或者连接的数据源是兼容某些数据源语法但要使用自己的连接提供程序的场景.

为保证灵活性,Obase并没有在这些对应数据库的提供器进行Sql语句的处理而是有一个单独的抽象层io.obase.providers.sql诸如io.obase.providers.mysql这些提供器软件包的特定数据源提供器其实是派生自SqlContextConfigProvider的.

所以,我们可以直接引用io.obase.providers.sql并自行派生SqlContextConfigProvider来定义满足我们需要求的上下文配置.

接下来以连接兼容MySql语法的达梦数据库做例子来讲解如何扩展.

首先,我们要引用达梦数据库的驱动和io.obase.providers.sql,以下都以引用了达梦和io.obase.providers.sql的maven包为前提.

然后,定义一个SqlContextConfigProvider的实现类.

```
/**
 * 达梦数据库配置
 */
public class DmConfiguration extends SqlContextConfigProvider {
    /**
     * 由派生类实现 获取数据库驱动名称字符串
     *
     * @return 数据库驱动名称字符串
     */
    @Override
    protected String getDbDriverClass() {
        return "dm.jdbc.driver.DmDriver";
    }

    /**
     * 由派生类实现，获取数据库连接字符串
     *
     * @return 数据库连接字符串
     */
    @Override
    protected String getConnectionString() {
        return "数据库连接字符串";
    }

    /**
     * 由派生类实现 获取数据库用户名
     *
     * @return 数据库用户名
     */
    @Override
    protected String getConnectionUserName() {
        return "数据库用户名";
    }

    /**
     * 由派生类实现 获取数据库密码
     *
     * @return 数据库密码
     */
    @Override
    protected String getConnectionPassWord() {
        return "数据库密码";
    }

    /**
     * 获取数据源类型
     *
     * @return 数据源类型
     */
    @Override
    protected EDataSource getSourceType() {
        return EDataSource.MySql;
    }

    /**
     * 使用指定的建模器创建对象数据模型
     *
     * @param modelBuilder 建模器
     */
    @Override
    protected void createModel(ModelBuilder modelBuilder) {
        //配置模型
    }
}

```
除了普通地创建模型方法和数据库连接字符串账号密码之外,还要重写两个方法:

- 数据库驱动名称字符串
- 数据源类型

这两个方法就是扩展点,数据库驱动名称字符串就是相应驱动的java.sql.Driver名称,用于创建连接;数据源类型需要提供EDataSource枚举,用于指定Sql语句的具体语法类型.

之后只需要正常配置对象数据模型即可,与一般情况下的Obase的使用方法相同.