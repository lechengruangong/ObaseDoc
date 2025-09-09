Obase的对象模型配置中,关联端配置有两个较为特殊的配置,分别是关联端是否聚合于关联和关联端是否作为新对象附加至上下文.

这两个配置的配置方法位于关联端配置上,在关联型上使用AssociationEnd<>方法获得关联端配置后,在关联端配置上调用IsAggregated(bool isAggregated)就可以指定关联端是否聚合于关联,默认值是false;
调用HasDefaultAsNew(bool defaultAsNew)就可以指定关联端是否作为新对象附加至上下文,默认值为false.

其中,关联端是否聚合于关联影响Obase保存旧对象时的行为,默认情况下(即关联端是否聚合于关联设置为false)Obase在解除关联时,只会将关联的映射置空,而将关联端是否聚合于关联设置为true时,会在解除关联时将次关联端对象删除.

关联端是否作为新对象附加则会影响Obase新对象时的行为,默认情况下(即关联端是否作为新对象附加设置为false)时,在没有将关联端对象Attach至上下文时,是不会将此对象保存至数据源的,而将关联端是否作为新对象附加设置为true时,会将关联端对象默认作为新对象Attach至上下文保存至数据源.

接下来就以两个示例来介绍这两个配置.

## dotNet

### 关联端是否聚合于关联

考虑一个这样的场景,WorkOrder(工单)包含了多个工单明细(WorkOrderDetail)用于详细的描述工单内容,在工单上定义了AddDetail和RemoveDetail方法来添加和移除工单明细.
当工单与工单详细解除关联时,工单明细就不再需要了,这时就可以使用关联端是否聚合于关联配置来处理;

以下是类定义:

```
/// <summary>
///     工单
/// </summary>
public class WorkOrder
{
    /// <summary>
    ///     工单标识
    /// </summary>
    public int Id { get; protected internal set; }

    /// <summary>
    ///     标题
    /// </summary>
    public string Title { get; set; }

    /// <summary>
    ///     内容
    /// </summary>
    public string Content { get; set; }

    /// <summary>
    ///     工单明细
    /// </summary>
    public IReadOnlyCollection<WorkOrderDetail> Details { get; protected internal set; }

    /// <summary>
    ///     添加明细
    /// </summary>
    /// <param name="detail">明细</param>
    public void AddDetail(WorkOrderDetail detail)
    {
        Details ??= new List<WorkOrderDetail>();
        ((List<WorkOrderDetail>)Details).Add(detail);
    }

    /// <summary>
    ///     移除明细
    /// </summary>
    /// <param name="detail">明细</param>
    public void RemoveDetail(WorkOrderDetail detail)
    {
        Details ??= new List<WorkOrderDetail>();
        ((List<WorkOrderDetail>)Details).Remove(detail);
    }
}

/// <summary>
///     工单明细 
/// </summary>
public class WorkOrderDetail
{
    /// <summary>
    ///     明细标识
    /// </summary>
    public int Id { get; protected internal set; }

    /// <summary>
    ///     工单标识
    /// </summary>
    public int WorkOrderId { get; protected internal set; }

    /// <summary>
    ///     详细内容
    /// </summary>
    public string Detail { get; set; }
}
```

以下是配置代码:

```
//配置工单
var workOrderEntity = modelBuilder.Entity<WorkOrder>();
workOrderEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
workOrderEntity.ToTable("WorkOrder");

//配置工单明细
var workOrderDetailEntity = modelBuilder.Entity<WorkOrderDetail>();
workOrderDetailEntity.HasKeyAttribute(p => p.Id).HasKeyIsSelfIncreased(true);
workOrderDetailEntity.ToTable("WorkOrderDetail");

//配置工单和工单明细的关系
var workOrderDetail = modelBuilder.Association();
workOrderDetail.AssociationEnd<WorkOrder>().HasMapping("Id", "WorkOrderId");
workOrderDetail.AssociationEnd<WorkOrderDetail>().HasMapping("Id", "Id").IsAggregated(true);
workOrderDetail.ToTable("WorkOrderDetail");
```
这里将WorkOrderDetail这个关联端配置为关联端聚合于关联,那么在解除关联的时候,被移除的WorkOrderDetail对象会从数据源内删除,类似于如下的调用:

```
//首先 查询工单并加载工单明细 假设此时已经有一个工单明细
var order = context.CreateSet<WorkOrder>().Include(p => p.Details).First();

//记录下此时的工单明细Id
var detailIds = order.Details.Select(p => p.Id).ToList();

//然后删除现有的工单明细
order.RemoveDetail(0);

//新增一个工单明细
var newDetail = new WorkOrderDetail()
{
    Detail = "新的工单明细"
};

order.AddDetail(newDetail);

context.Attach(newDetail);

//保存更改
context.SaveChanges();

//再次查询工单并加载工单明细
order = context.CreateSet<WorkOrder>().Include(p => p.Details).First();

//可以发现工单明细的ID和之前的不一样了，说明之前的工单明细已经被删除了
var newDetailIds = order.Details.Select(p => p.Id).ToList();
Console.WriteLine($"旧的工单明细ID: {string.Join(", ", detailIds)}");
Console.WriteLine($"新的工单明细ID: {string.Join(", ", newDetailIds)}");
```

### 关联端是否作为新对象

考虑一个这样的场景,Agency(代理商)可以发放自己的VIPCard(贵宾卡),贵宾卡每个代理商只能发三张且只能在发放贵宾卡的代理商使用,所以在代理商定义了AddVipCard方法且在方法内部初始化VipCard,由于新增的VipCard无法直接在上下文中附加,此时就需要将代理商和贵宾卡之间的关系配置为贵宾卡默认作为新对象附加.

以下是类定义:

```
   /// <summary>
   ///     代理商
   /// </summary>
   public class Agency
   {
       /// <summary>
       ///     代码
       /// </summary>
       public string Code { get; protected internal set; }

       /// <summary>
       ///     名称
       /// </summary>
       public string Name { get; set; }

       /// <summary>
       ///     VIP卡
       /// </summary>
       public IReadOnlyCollection<VipCard> VipCards { get; protected internal set; }

       /// <summary>
       ///     添加VIP卡
       /// </summary>
       /// <exception cref="InvalidOperationException"></exception>
       public void AddVipCard()
       {
           VipCards ??= new List<VipCard>();
           if(VipCards.Count >= 3)
           {
               throw new InvalidOperationException("每个代理商最多只能有3张VIP卡");
           }
           ((List<VipCard>)VipCards).Add(new VipCard());
       }

       /// <summary>
       ///     移除VIP卡
       /// </summary>
       /// <param name="vipCardIndex">VIP卡索引</param>
       public void RemoveVipCard(int vipCardIndex)
       {
           VipCards ??= new List<VipCard>();
           ((List<VipCard>)VipCards).RemoveAt(vipCardIndex);
       }
   }

   /// <summary>
   ///     VIP卡
   /// </summary>
   public class VipCard
   {
       /// <summary>
       ///     代码
       /// </summary>
       public string Code { get; protected internal set; }

       /// <summary>
       ///     代理商代码
       /// </summary>
       public string AgencyCode { get; protected internal set; }

       /// <summary>
       ///     代理商
       /// </summary>
       public Agency Agency { get; set; }

   }
```
以下是配置代码:
```
 //配置代理商
 var agencyEntity = modelBuilder.Entity<Agency>();
 agencyEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
 agencyEntity.ToTable("Agency");

 //配置VIP卡
 var vipCardEntity = modelBuilder.Entity<VipCard>();
 vipCardEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
 vipCardEntity.ToTable("VipCard");

 //配置代理商和VIP卡的关系
 var agencyVipCard = modelBuilder.Association();
 agencyVipCard.AssociationEnd<Agency>().HasMapping("Code", "AgencyCode");
 agencyVipCard.AssociationEnd<VipCard>().HasMapping("Code", "Code").HasDefaultAsNew(true);
 agencyVipCard.ToTable("VipCard");
```

这里将VipCard这个关联端配置为默认作为新对象附加,那么在VipCards里有新增的对象时,就会作为新对象附加至上下文进行保存,类似于如下的调用:

```
 //查询已有的代理商及其VIP卡 
 var agency = context.CreateSet<Agency>().Include(p => p.VipCards).First();
 //目前没有VIP卡
 var cardCount = agency.VipCards?.Count ?? 0;
 //为该代理商添加一张VIP卡
 agency.AddVipCard();
 //保存到数据库
 context.SaveChanges();
 //再次查询该代理商及其VIP卡
 agency = context.CreateSet<Agency>().Include(p => p.VipCards).First();
 //有一张VIP卡
 var newCardCount = agency.VipCards?.Count ?? 0;

 Console.WriteLine($"未添加前有{cardCount}张VIP卡");
 Console.WriteLine($"添加后有{newCardCount}张VIP卡");
```

