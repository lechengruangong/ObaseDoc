事务是为了处理数据的并发而定义最小单元,在Obase中提供了以下几种事务处理模式用于不同的场景.

- 自动事务,指的是Obase自动附加的事务.在Obase中,每次saveChanges时会把此次保存和上次保存之间的所有上下文管理的对象修改操作包含在一个事务块内.

  如果是首次saveChanges则是从构造上下文开始的所有上下文管理的对象修改(此处的上下文管理的对象指的是附加到上下文的新对象和由上下文查询得到的旧对象).

- 手动事务,指的是调用Obase的手动事务方法自己控制事务.

  Obase的手动事务方法遵循JDBC的try-setAutoCommit(false)-commit-catch-rollBack-finally-close模式.

- 已有连接事务,指的是由调用方提供连接来构造上下文配置提供者并由调用方来管理连接的事务的模式.

  此种事务模式通常用于与旧系统的事务进行对接,比如旧系统中已存在了一个业务场景,原有逻辑为有一个事务先执行A语句再执行B语句最后一起提交,此时新的C部分由Obase来负责,需要将旧的逻辑改为先执行A再执行新逻辑C最后执行B再提交.

  此时可以使用已有连接事务来处理此问题,Obase提供了JDBC层的兼容方案,令Obase也使用旧系统提供的连接来处理事务.

## 事务相关方法

Obase的事务由上下文负责管理,提供了以下几个方法:

- 自动事务不需要调用任何方法.
- 手动事务使用以下方法,代码中的context即对象上下文:
    ```
    //手动开启事务
    context.beginTransaction();
    //提交修改
    context.commit();
    //回滚事务
    context.rollbackTransaction();
    //释放资源
    context.release();
    ```
- 已有连接事务不使用额外的方法控制事务,仅使用JDBC的事务方法即可,但需要继承SqlContextConfigurator类实现使用已存在的连接上下文配置.

## 具体示例

1. 以下代码为自动事务的示例: 
   ```
   //构造一个上下文
   Context context = new Context();
   //查询对象
   List<DomainClass> list = context.createSet(DomainClass.class).toList();
   //将前三个的主键修改 其中第二个主键修改为空
   list.get(0).setId(11);
   list.get(1).setId(null);
   list.get(2).setId(13);
   //在保存时 会由数据源抛出异常 此时这三个对象的修改都处于同一事务块内 所以这三个的修改都不会保存
   context.saveChanges();
   ```
   
   在自动事务中,连接和事务控制都由Obase负责.

2. 以下代码为手动事务的示例:
   ```
   //构造一个上下文
   Context context = new Context();
   //查询对象
   List<DomainClass> list = context.createSet(DomainClass.class).toList();
   try
   {
      //手动开启事务
      context.beginTransaction();

      //修改前三个的LongNumber
      list.get(0).setLongNumber(11L);
      list.get(1).setLongNumber(12L);
      list.get(2).setLongNumber(13L);
   
      //保存之前的修改
      context.saveChanges();
      //调用模拟的外部方法 
      //假定此方法会抛异常 那么就会在下面的Catch中导致从beginTransaction开始的事务块回滚
      //假定此方法不抛异常 那么就会就会进入commit将从beginTransaction开始的事务块提交
      outerMethod();
      
      //提交修改
      context.commit();
   }
   catch (Exception)
   {
      //发生异常 回滚
      context.rollbackTransaction();
   }
   finally
   {
      //最后释放资源
      context.release();
   }
   ```

   在手动事务中,连接由Obase负责,事务控制由调用方负责.

3. 以下代码为旧系统的事务示例,此处使用的是JDBC的事务基础代码作为示例,实际使用时可能有所包装,但只要仍使用JDBC模型就可以兼容:

   ```
   //假设此时已经有了一个连接 且是打开的
   Connection connection = getConnection();
   //此处开始是原有的代码 使用try代码块包裹的事务
   try
   {
      //设置为不自动提交 开启一个事务
      connection.setAutoCommit(false);
      //原有的执行逻辑A
      //此处做了一些数据库操作XXX
   
      //原有的执行逻辑B
      //此处做了一些数据库操作XXX
   
      //原有的提交事务
      connection.commit();
   
      //原有的其它代码XXX
   }
   catch(Exception ex)
   {
      //原有的回滚事务
      connection.rollback();
   }
   finally
   {
       //原有的关闭连接
       connection.close();
   }
   ```
   此时引入Obase并且想要保留之前的事务处理逻辑,那么就可以使用已有连接事务来做兼容,首先需要定义一个新的上下文配置提供者(此处以MySql为例):
   ```
   /**
   * 使用已存在的MySql连接上下文配置
     */
     public class MySqlExistingConnectionContextConfig extends SqlContextConfigurator {

     /**
       * 使用指定的Sql执行器初始化SqlContextConfigurator的新实例
       *
       * @param sqlExecutor         Sql语句执行器
       */
       public MySqlExistingConnectionContextConfig(ISqlExecutor sqlExecutor) {
          super(sqlExecutor, enableStructMapping);
       }

     /**
       * 使用指定的建模器创建对象数据模型
       *
       * @param modelBuilder 建模器
       */
       @Override
       protected void createModel(ModelBuilder modelBuilder) {
         //注册模型代码
       }
     }

     ```
     这个上下文配置提供者继承的配置提供器是SqlContextConfigurator,对于已有连接事务构造函数需要传入一个特定的ExistingConnectionSqlExecutor执行器来接收已有的连接作为参数,那么对应的上下文如下:
     ```
      /**
       * 使用已存在的MySql连接上下文
      */
      public class MySqlExistingConnectionContext extends ObjectContext {
         /**
         * 初始化使用已存在的MySql连接上下文
         *
         * @param connection 数据库连接
         public MySqlExistingConnectionContext(Connection connection) {
            super(new MySqlExistingConnectionContextConfig(new ExistingConnectionSqlExecutor(connection, EDataSource.MySql)));
         }
      }
      ```
      ExistingConnectionSqlExecutor执行器需要这样几个参数:数据源连接和数据源类型.

      使用这些参数构造的ExistingConnectionSqlExecutor会将已有的连接和事务作为Obase的Sql执行者使用,这样就可以复用原有的事务处理了.

      接下来只要修改旧逻辑即可:
      ```
      //假设此时已经有了一个连接 且是打开的
      Connection connection = getConnection();
      //此处开始是原有的代码 使用try代码块包裹的事务
      try
      {
         //原有的开启了一个事务
         connection.setAutoCommit(false);
         //原有的执行逻辑A
         //此处做了一些数据库操作XXX
   
         //构造一个上下文
         MySqlExistingConnectionContext exContext =  new MySqlExistingConnectionContext(connection);
   
         //执行一些Obase的逻辑
         //Obase的保存
         exContext.saveChanges();
   
         //原有的执行逻辑B
         //此处做了一些数据库操作XXX
   
         //原有的提交事务
         connection.commit();
   
         //原有的其它代码XXX
      }
      catch(Exception ex)
      {
         //原有的回滚事务
         connection.rollback();
      }
      finally
      {
          //原有的关闭连接
          connection.close();
      }
      ```
      实际上就是在原有的逻辑A和B之间使用现有的连接和事务构造上下文,之后再增加Obase的逻辑即可.

      在已有连接事务中,连接和事务控制都由调用方负责.

      当然,如果你需要特定的事务处理逻辑,你可以自己实现一个ISqlExecutor的接口实现,之后传入你继承SqlContextConfigurator的提供器构造函数即可.