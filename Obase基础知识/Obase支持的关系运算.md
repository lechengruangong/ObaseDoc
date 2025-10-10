
关系运算指的是在使用Sql数据源时,使用Sql语言实现的查询运算.

在6.0+,Obase支持的关系运算有以下这些:

## dotNet

- All运算,判断对象集内对象是否全部满足条件.

有以下重载方法


```
//判断对象是否全部满足条件
All<TSource>(Expression<Func<TSource, bool>> predicate);
```

- Any运算,判断对象集内是否存在满足条件的对象.

有以下重载方法


```
//是否存在满足条件的对象
Any<TSource>(Expression<Func<TSource, bool>> predicate);
//是否存在对象
Any<TSource>();
```

- Average运算,获取对象集中对象某个属性的平均数.

有以下重载方法


```
//以下重载均为求平均值
Average<TSource>(Expression<Func<TSource, short>> selector); 
Average<TSource>(Expression<Func<TSource, int>> selector);
Average<TSource>(Expression<Func<TSource, long>> selector);
Average<TSource>(Expression<Func<TSource, float>> selector);
Average<TSource>(Expression<Func<TSource, double>> selector);
Average<TSource>(Expression<Func<TSource, decimal>> selector);
```

- Contains运算,判断对象集中对象的某个属性是否被包含于某个集合中或某个属性是否包含某个值.

可以支持字符串属性包含(如set.Where(p=>p.Name.Contains("123")))和集合包含某个属性(如set.Where(p=>list.Contains(p.Id)))

此外字符串属性还支持StartWith和EndWith,如set.Where(p>p.Name.StartWith("123"))

- Count运算,获取对象集中符合条件的对象个数.

有以下重载方法


```
//获取符合条件的对象个数
Count<TSource>(Expression<Func<TSource, bool>> predicate);
//获取对象个数
Count<TSource>();
```

- Distinct运算,获取对象集中不重复的对象.

有以下重载方法


```
//设置为获取不重复的
Distinct<TSource>();
```

- ElementAt运算,获取对象集中位于某个索引的对象,从0开始.

有以下重载方法


```
//获取某个索引的对象
ElementAt<TSource>(int index);
```

- First运算,获取对象集中符合条件的第一个对象,不存在对象时会抛出异常.

有以下重载方法


```
//获取符合条件的第一个对象
First<TSource>(Expression<Func<TSource, bool>> predicate);
//获取第一个对象
First<TSource>();
```

- FirstOrDefault运算,获取对象集中符合条件的第一个对象,不存在对象时会返回空.

有以下重载方法


```
//获取符合条件的第一个对象或默认值
FirstOrDefault<TSource>(Expression<Func<TSource, bool>> predicate);
//获取第一个对象或默认值
FirstOrDefault<TSource>();
```

- Group运算,获取对象集中将对象分组投影后的结果.

有以下重载方法


```
//获取根据某个属性TKey分组后的IGrouping<TKey, TSource>对象
GroupBy<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector));
//获取根据某个属性TKey分组后 投影到TElement的<IGrouping<TKey, TElement>对象
GroupBy<TSource, TKey, TElement>(Expression<Func<TSource, TKey>> keySelector, Expression<Func<TSource, TElement>> elementSelector);
//获取根据某个属性TKey分组后 投影到TElement 再根据resultSelector构造的对象
GroupBy<TSource, TKey, TElement>(Expression<Func<TSource, TKey>> keySelector, Expression<Func<TSource, TElement>> elementSelector, Expression<Func<TKey, TElement, TResult>> resultSelector));
```

- Include运算,指示查询时要一并加载的关联对象.

有以下重载方法


```
//指示要一并加载的对象路径
Include<T, TProperty>(Expression<Func<T, TProperty>> path);
//指示要一并加载的对象路径的字符串表示形式
Include<T>(string path);
```

第一个重载参数可以表示为set.Include(p=>p.B.Cs.Select(q=>q.D),即表示查询A时加载B,C集合和D.

请注意此方法只支持引用元素(关联引用或关联端)成员访问表达式和在一对多的时候使用Select调用引用元素(关联引用或关联端)成员访问表达式,如果传入其他的表达式会在构造对象路径时抛出异常.

第二个重载参数可以表示为set.Include("B.Cs.D").

- Last运算,获取对象集中符合条件的最后一个对象,不存在对象时会抛出异常.

有以下重载方法


```
//获取符合条件的最后一个对象
Last<TSource>(Expression<Func<TSource, bool>> predicate);
//获取最后一个对象
Last<TSource>();
```

- LastOrDefault获取对象集中符合条件的最后一个对象,不存在对象时会返回空.

有以下重载方法


```
//获取符合条件的最后一个对象
LastOrDefault<TSource>(Expression<Func<TSource, bool>> predicate);
//获取最后一个对象
LastOrDefault<TSource>();
```

- Max运算,获取对象集中对象某个属性的最大值.

有以下重载方法


```
//以下重载均为求最大值
Max<TSource>(Expression<Func<TSource, short>> selector); 
Max<TSource>(Expression<Func<TSource, int>> selector);
Max<TSource>(Expression<Func<TSource, long>> selector);
Max<TSource>(Expression<Func<TSource, float>> selector);
Max<TSource>(Expression<Func<TSource, double>> selector);
Max<TSource>(Expression<Func<TSource, decimal>> selector);
```

- Min运算,获取对象集中对象某个属性的最小值.

有以下重载方法


```
//以下重载均为求最小值
Min<TSource>(Expression<Func<TSource, short>> selector); 
Min<TSource>(Expression<Func<TSource, int>> selector);
Min<TSource>(Expression<Func<TSource, long>> selector);
Min<TSource>(Expression<Func<TSource, float>> selector);
Min<TSource>(Expression<Func<TSource, double>> selector);
Min<TSource>(Expression<Func<TSource, decimal>> selector);
```

- Order运算,指示查询时的排序.

有以下重载方法


```
//指定查询时按照哪个字段正序排序 注意重复调用此方法会清除之前的排序条件
OrderBy<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector);
//指定查询时按照哪个字段倒序排序 注意重复调用此方法会清除之前的排序条件
OrderByDescending<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector);
//指定查询时按照哪个字段正序排序 重复调用此方法不会清除之前的排序条件 而是按照顺讯组合为联合排序
ThenBy<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector);
//指定查询时按照哪个字段倒序排序 重复调用此方法不会清除之前的排序条件 而是按照顺讯组合为联合排序
ThenByDescending<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector);
```

- Reverse运算,指示查询结果是否逆序返回.

有以下重载方法


```
//将查询结果反序
Reverse<TSource>();
```

- Select运算,将对象集中的对象投影至某个结果.

有以下重载方法


```
//投影到对象的某个属性,关联对象或者构造一个新对象
Select<TSource, TResult>(Expression<Func<TSource, TResult>> selector);
```

- SelectMany运算,将对象集中对象平展投影至关联对象.

有以下重载方法


```
//平展投影到某个关联引用集合
SelectMany<TSource, TResult>(Expression<Func<TSource, IEnumerable<TResult>>> selector);
//平展投影到某个关联引用集合 然后根据此集合构建结果对象
SelectMany<TSource, TResult>(IEnumerable<TCollection>>> collectionSelector, Expression<Func<TSource, TCollection, TResult>> resultSelector);
```

- Single运算,获取对象集中符合条件的某个单一元素,如果不存在符合条件的元素会抛出异常.

有以下重载方法


```
//获取符合条件的单一对象
Single<TSource>(Expression<Func<TSource, bool>> predicate);
//获取单一对象
Single<TSource>();
```

- SingleOrDefault运算,获取对象集中符合条件的某个单一元素,如果不存在符合条件的元素会返回空.

有以下重载方法


```
//获取符合条件的单一对象
SingleOrDefault<TSource>(Expression<Func<TSource, bool>> predicate);
//获取单一对象
SingleOrDefault<TSource>();
```

- Skip运算,指示查询时跳过多少个对象.

有以下重载方法


```
//跳过多少个对象
Skip<TSource>(int count);
```

- Sum运算,获取对象集中对象某个属性的和.

有以下重载方法


```
//以下重载均为求和
Sum<TSource>(Expression<Func<TSource, short>> selector); 
Sum<TSource>(Expression<Func<TSource, int>> selector);
Sum<TSource>(Expression<Func<TSource, long>> selector);
Sum<TSource>(Expression<Func<TSource, float>> selector);
Sum<TSource>(Expression<Func<TSource, double>> selector);
Sum<TSource>(Expression<Func<TSource, decimal>> selector);
```

- Take运算,指示查询时提取多少个对象.

有以下重载方法


```
//提取多少个对象
Take<TSource>(int count);
```

- Where运算,指示查询时的筛选条件.

有以下重载方法


```
//按什么条件筛选
Where<TSource>(Expression<Func<TSource, bool>> predicate);
```

## Java
待重写