Obase支持多种对象查询操作,这些查询方法在Sql数据源中被映射为特定的关系运算,用户在查询时可以组合这些查询操作来实现查询需求.

## 查询运算

接下来介绍这些查询操作和对应的关系运算:

- All方法,判断对象集内对象是否全部满足条件.

映射为获取满足条件取反的数据个数操作后返回个数是否等于0的运算.

```
All<TSource>(Expression<Func<TSource, bool>> predicate);
```

- Any方法,判断对象集内是否存在满足条件的对象.

映射为获取满足条件的数据个数操作后返回个数是否大于0的运算.

```
Any<TSource>(Expression<Func<TSource, bool>> predicate);
Any<TSource>();
```

- Average方法,获取对象集中对象某个属性的平均数.

映射为取目标字段平均数运算.

```
Average<TSource>(Expression<Func<TSource, short>> selector); 
Average<TSource>(Expression<Func<TSource, int>> selector);
Average<TSource>(Expression<Func<TSource, long>> selector);
Average<TSource>(Expression<Func<TSource, float>> selector);
Average<TSource>(Expression<Func<TSource, double>> selector);
Average<TSource>(Expression<Func<TSource, decimal>> selector);
```

- Contains方法,判断对象集中对象的某个属性是否被包含于某个集合中或某个属性是否包含某个值.

映射为获取满足条件的数据个数操作后返回个数是否大于0的运算.

可以支持字符串属性包含(如set.Where(p=>p.Name.Contains("123")))和集合包含某个属性(如set.Where(p=>list.Contains(p.Id)))

此外字符串属性还支持StartWith和EndWith,如set.Where(p=>p.Name.StartWith("123"))

- Count方法,获取对象集中符合条件的对象个数.

映射为获取满足条件的数据个数运算.

```
Count<TSource>(Expression<Func<TSource, bool>> predicate);
Count<TSource>();
```

- Distinct方法,获取对象集中不重复的对象.

映射为去除查询结果中的重复值运算.

```
Distinct<TSource>();
```

- ElementAt方法,获取对象集中位于某个索引的对象,从0开始.

映射为提取传入的索引值+1并跳过索引值条数据的运算.

```
ElementAt<TSource>(int index);
```

- First方法,获取对象集中符合条件的第一个对象,不存在对象时会抛出异常.

映射为获取满足条件的首个数据运算.

```
First<TSource>(Expression<Func<TSource, bool>> predicate);
First<TSource>();
```

- FirstOrDefault方法,获取对象集中符合条件的第一个对象,不存在对象时会返回空.

映射为获取满足条件的首个数据运算.

```
FirstOrDefault<TSource>(Expression<Func<TSource, bool>> predicate);
FirstOrDefault<TSource>();
```

- Group方法,获取对象集中将对象分组投影后的结果.

映射为分组运算.

```
GroupBy<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector));
GroupBy<TSource, TKey, TElement>(Expression<Func<TSource, TKey>> keySelector, Expression<Func<TSource, TElement>> elementSelector);
GroupBy<TSource, TKey, TElement>(Expression<Func<TSource, TKey>> keySelector, Expression<Func<TSource, TElement>> elementSelector, Expression<Func<TKey, TElement, TResult>> resultSelector));
```

- Include方法,指示查询时要一并加载的关联对象.

映射为联表查询运算.

```
Include<T, TProperty>(Expression<Func<T, TProperty>> path);
Include<T>(string path);
```

第一个重载参数可以表示为set.Include(p=>p.B.Cs.Select(q=>q.D)),即表示查询A时加载B,C集合和D.

请注意此方法只支持引用元素(关联引用或关联端)成员访问表达式和在一对多的时候使用Select调用引用元素(关联引用或关联端)成员访问表达式,如果传入其他的表达式会在构造对象路径时抛出异常.

第二个重载参数可以表示为set.Include("B.Cs.D").

- Last方法,获取对象集中符合条件的最后一个对象,不存在对象时会抛出异常.

映射为获取满足条件的按照排序键倒序排序的首个数据运算.

```
Last<TSource>(Expression<Func<TSource, bool>> predicate);
Last<TSource>();
```

- LastOrDefault获取对象集中符合条件的最后一个对象,不存在对象时会返回空.

映射为获取满足条件的按照排序键倒序排序的首个数据运算.

```
LastOrDefault<TSource>(Expression<Func<TSource, bool>> predicate);
LastOrDefault<TSource>();
```

- Max运算,获取对象集中对象某个属性的最大值.

映射为取目标字段的最大值运算.

```
Max<TSource>(Expression<Func<TSource, short>> selector); 
Max<TSource>(Expression<Func<TSource, int>> selector);
Max<TSource>(Expression<Func<TSource, long>> selector);
Max<TSource>(Expression<Func<TSource, float>> selector);
Max<TSource>(Expression<Func<TSource, double>> selector);
Max<TSource>(Expression<Func<TSource, decimal>> selector);
```

- Min运算,获取对象集中对象某个属性的最小值.

映射为取目标字段的最小值运算.

```
Min<TSource>(Expression<Func<TSource, short>> selector); 
Min<TSource>(Expression<Func<TSource, int>> selector);
Min<TSource>(Expression<Func<TSource, long>> selector);
Min<TSource>(Expression<Func<TSource, float>> selector);
Min<TSource>(Expression<Func<TSource, double>> selector);
Min<TSource>(Expression<Func<TSource, decimal>> selector);
```

- Order运算,指示查询时的排序.

映射为按目标字段排序运算.

```
OrderBy<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector);
OrderByDescending<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector);
ThenBy<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector);
ThenByDescending<TSource, TKey>(Expression<Func<TSource, TKey>> keySelector);
```

这些方法中,OrderBy是覆盖之前条件的正序排序,OrderByDescending是覆盖之前排序条件的倒序排序,ThenBy是保留之前排序的正序排序,ThenByDescending是保留之前排序的倒序排序.

- Reverse运算,指示查询结果是否逆序返回.

映射为按照排序键倒序排序运算.

```
Reverse<TSource>();
```

- Select运算,将对象集中的对象投影至某个结果.

如果目标为字段,映射为取指定字段运算.如果目标为关联对象,映射为联表查询运算.

```
Select<TSource, TResult>(Expression<Func<TSource, TResult>> selector);
```

- SelectMany运算,将对象集中对象平展投影至关联对象.

如果目标为字段,映射为取指定字段运算.如果目标为关联对象,映射为联表查询运算.

```
SelectMany<TSource, TResult>(Expression<Func<TSource, IEnumerable<TResult>>> selector);
SelectMany<TSource, TResult>(IEnumerable<TCollection>>> collectionSelector, Expression<Func<TSource, TCollection, TResult>> resultSelector);
```

- Single运算,获取对象集中符合条件的某个单一元素,如果不存在符合条件的元素会抛出异常.

映射为获取满足条件的数据后,判断数据是否只有一条的运算.

```
Single<TSource>(Expression<Func<TSource, bool>> predicate);
Single<TSource>();
```

- SingleOrDefault运算,获取对象集中符合条件的某个单一元素,如果不存在符合条件的元素会返回空.

映射为获取满足条件的数据后,判断数据是否只有一条的运算.

```
SingleOrDefault<TSource>(Expression<Func<TSource, bool>> predicate);
SingleOrDefault<TSource>();
```

- Skip运算,指示查询时跳过多少个对象.

映射为跳过指定条数数据的运算.

```
Skip<TSource>(int count);
```

- Sum运算,获取对象集中对象某个属性的和.

映射为取目标字段的和运算.

```
Sum<TSource>(Expression<Func<TSource, short>> selector); 
Sum<TSource>(Expression<Func<TSource, int>> selector);
Sum<TSource>(Expression<Func<TSource, long>> selector);
Sum<TSource>(Expression<Func<TSource, float>> selector);
Sum<TSource>(Expression<Func<TSource, double>> selector);
Sum<TSource>(Expression<Func<TSource, decimal>> selector);
```

- Take运算,指示查询时提取多少个对象.

映射为提取指定条数数据的运算.

```
Take<TSource>(int count);
```

- Where运算,指示查询时的筛选条件.

映射为获取满足条件的数据运算.

```
Where<TSource>(Expression<Func<TSource, bool>> predicate);
```

- 获取结果方法

这些方法是用来执行终结操作,将结果转换为指定类型的方法.

通常会使用ToList或者ToArray方法来获取结果.

## 筛选条件拼合器

日常使用中,最常用的查询操作就是筛选(Where)操作,为了动态的拼接筛选条件往往会直接操作Lambda表达式来拼接条件.

Obase提供了两个Expression<Func<T, bool>>的扩展方法And和Or用于直接拼接同层级条件的与和或.