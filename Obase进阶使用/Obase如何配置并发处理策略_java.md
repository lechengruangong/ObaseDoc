在软件开发中,当多个进程或线程并发的对同一个数据源进行操作时,就有可能出现并发冲突.

并发冲突可以归类为以下三种:
1. 重复创建,即尝试创建主键相同的对象.
2. 版本冲突,修改对象时对象已被其他线程/进程修改.
3. 更新幻影,修改对象时对象已被其他线程/进程删除.

三种并发冲突中,重复创建和更新幻影可以会根据数据源返回的信息(如重复插入相同主键异常和未能更新任何数据)进行判断,版本冲突则一般表现为数据错误,不能由数据源返回的信息进行判断,考虑一个如下的典型的版本冲突场景:
> SQL数据源中有两张表,其中A表有一个属性sum记录B表中的行数总和,此时有两个线程T1和T2分别查询了B表的行数,并且要插入一条新的B表记录.
> T1和T2获取到的行数都是b,并且都将b+1的值更新至A表的sum字段,但此时B表其实插入了两条数据,此时A表的sum应该是b+2.

为了应对这些并发冲突,Obase提供了乐观并发冲突处理模型,并引入了以下的概念:

- 版本键,用于标识对象数据版本来应对版本冲突这种并发冲突.

  版本键可以配置多个,可以使用会发生并发冲突的属性或者使用时间戳标识最后的修改时间来作为版本键,能区分对象最后被谁修改的属性都可以作为版本键.

  对于上面的例子就可以建一个LastModifyTime作为版本键或者直接将Sum作为版本键.
- 并发冲突处理策略,用于指定如何处理发生的并发冲突.Obase提供了5种并发冲突处理策略,分别是:
  1. 忽略策略,当发生并发时不做任何处理,本策略可以处理所有类型的并发冲突.
  2. 抛出异常策略,当发生并发异常,会抛出特定的异常告知调用方处理,本策略可以处理所有类型的并发冲突,此策略是默认的并发冲突处理策略.
  3. 强制覆盖策略,当发生并发时,用当前对象覆盖原有对象,本策略可以处理重复创建和版本冲突两种并发情况.
  4. 重建对象策略,当发生异常时,将当前对象做为新对象进行创建,本策略可以处理更新幻影这种并发情况.
  5. 版本合并策略,当发生异常时,将当前对象与旧对象的属性进行合并,本策略可以处理重复创建和版本冲突这两种并发情况.
- 属性合并策略,在使用版本合并策略时用于指定如何合并发生冲突的属性.Obase提供了3种处理版本冲突时的属性合并策略,分别是
  1. 覆盖,强制覆盖对方版本的值,此策略是默认版本合并策略.
  2. 忽略,忽略当前属性,即承认冲突对方版本的值.
  3. 累加,将当前版本中属性值的增量累加到对方版本,此种合并策略只能用于数值型的属性上.

## 模型内配置并发处理策略

并发处理策略是包含在Obase本体中的,因此不需要额外引用软件包,通常情况下,由于引用传递只需要引用对应的数据源提供器即可.

在模型配置中有以下几个方法用于配置对象变更通知,这些方法可以由实体型配置对象或者关联型配置对象调用,,以下代码中所有的entityConfig就是要处理并发冲突的实体型或者关联型配置对象,attrConfig就是对象上的要进行版本合并的属性配置:

```
//配置并发处理策略
entityConfig.hasConcurrentConflictHandlingStrategy(EConcurrentConflictHandlingStrategy.Combine);
//配置版本键 最简单的方式就是将会出现并发冲突的属性作为版本键
entityConfig.hasVersionAttribute(p -> p.getAmount());
//如果之前并发处理策略是版本合并策略 还需要设置会出现并发冲突的属性的版本合并策略
attrConfig.hasCombinationHandler(EAttributeCombinationHandlingStrategy.Accumulate)
```

这里的配置就是配置具体的对象并发冲突合并策略,如果已经预见到不会发生版本冲突,就不需要配置版本键;如果对象并发冲突合并策略不是版本合并,就不需要配置并发冲突的属性的版本合并策略.

## 具体示例

考虑一个如下的场景,进行团购时需要参与团购的人报名,团购需要记录已报名的人数,那么团购报名时就会出现多个线程同时修改团购的已报名属性的情形.

首先,定义团购类和团购报名类:
```
/**
 * 团购
 */
public class TeamBuy {
    /**
     * 团购ID
     */
    private int teamBuyId;

    /**
     * 已经报名的人数
     */
    private int signed;

    /**
     * 团购报名列表
     */
    private List<TeamBySignUp> signUpList;

    /**
     * 团购ID
     */
    public int getTeamBuyId() {
        return teamBuyId;
    }
    
    /**
     * 团购ID
     */
    public void setTeamBuyId(int teamBuyId) {
        this.teamBuyId = teamBuyId;
    }

    /**
     * 已经报名的人数
     */
    public int getSigned() {
        return signed;
    }

    /**
     * 已经报名的人数
     */
    public void setSigned(int signed) {
        this.signed = signed;
    }

    /**
     * 团购报名列表
     */
    public List<TeamBySignUp> getSignUpList() {
        return signUpList;
    }

    /**
     * 团购报名列表
     */
    public void setSignUpList(List<TeamBySignUp> signUpList) {
        this.signUpList = signUpList;
    }
}

/**
 * 团购报名
 */
public class TeamBySignUp {
    /**
     * 团购ID
     */
    private int teamBuyId;

    /**
     * 报名人数
     */
    private int signUpNum;


    /**
    * 团购ID
     */
    public int getTeamBuyId() {
        return teamBuyId;
    }

    /**
     * 团购ID
     */
    public void setTeamBuyId(int teamBuyId) {
        this.teamBuyId = teamBuyId;
    }

    /**
     * 报名人数
     */
    public int getSignUpNum() {
        return signUpNum;
    }

    /**
     * 报名人数
     */
    public void setSignUpNum(int signUpNum) {
        this.signUpNum = signUpNum;
    }
}

```

根据分析,团购的已报名人数属性会出现并发冲突,那么就需要将已报名人数配置为版本键,合并策略是版本合并,已报名人数的合并策略是累加.

那么配置如下:

```
//团购实体型
EntityTypeConfiguration<TeamBuy> teamBuyConfiguration = modelBuilder.entity(TeamBuy.class);
//配置团购中的并发策略为版本合并策略
teamBuyConfiguration.hasConcurrentConflictHandlingStrategy(EConcurrentConflictHandlingStrategy.Combine);
//Signed会因为并发导致出现错误 故将Signed设置为版本键
teamBuyConfiguration.hasVersionAttribute(p -> p.getSigned());
//为Signed设置版本合并策略为累加
teamBuyConfiguration.attribute(p -> p.getSigned())
    //使用累加策略
    .hasCombinationHandler(EAttributeCombinationHandlingStrategy.Accumulate);
//省略掉团购报名配置 团购和团购报名之间关系的配置
```

调用时只要正常执行新增团购报名和修改团购的已报名人数然后一起保存即可,Obase会根据配置来处理并发冲突,将已报名人数累加至正确值.