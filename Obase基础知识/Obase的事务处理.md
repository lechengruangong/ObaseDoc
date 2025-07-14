在Obase中,根据不同的平台特性提供了不同的事务处理.
## dotNet
在dotNet平台,Obase提供了如下的事务处理模式:

1. 自动事务

   Obase的自动事务,指的是Obase自动附加的事务.在Obase中,每次SaveChanges时会把此次保存和上次保存之间的所有上下文管理的对象修改操作包含在一个事务块内.

   如果是首次SaveChanges则是从构造上下文开始的所有上下文管理的对象修改(此处的上下文管理的对象指的是附加到上下文的新对象和由上下文查询得到的旧对象).
   
   以下代码为自动事务的示例: 
   ```
   //构造一个上下文
   var context = new Context();
   //查询对象
   var list = context.CreateSet<DomainClass>().ToList();
   //将前三个的主键修改 其中第二个主键修改为空
   list[0].Id = 11;
   list[1].Id = null;
   list[2].Id = 13;
   //在保存时 会由数据源抛出异常 此时这三个对象的修改都处于同一事务块内 所以这三个的修改都不会保存
   context.SaveChanges();
   ```
2. 手动事务

   Obase的手动事务,指的是调用Obase的手动事务方法自己控制事务.

   Obase的手动事务方法遵循ADO.NET的try-Begin-Commit-Catch-RollBack-Finally-Release模式.

   以下代码为手动事务的示例:
   ```
   //构造一个上下文
   var context = new Context();
   //查询对象
   var list = context.CreateSet<DomainClass>().ToList();
   try
   {
      //手动开启事务
      context.BeginTransaction();

      //修改前三个的LongNumber
      list[0].LongNumber = 14;
      list[1].LongNumber = 15;
      list[2].LongNumber = 16;
   
      //保存之前的修改
      context.SaveChanges();
      //调用模拟的外部方法 
      //假定此方法会抛异常 那么就会在下面的Catch中导致从BeginTransaction开始的事务块回滚
      //假定此方法不抛异常 那么就会就会进入Commit将从BeginTransaction开始的事务块提交
      OuterMethod();
      
      //提交修改
      context.Commit();
   }
   catch (Exception)
   {
      //发生异常 回滚
      context.RollbackTransaction();
   }
   finally
   {
      //最后释放资源
      context.Release();
   }
   ```

## java

java版待重写