在6.4.5中,Obase为属性配置增加了序列化器相关的配置方法,用于处理某些属性实际上是序列化后进行存储的场景.

配置了序列化器的属性会在取值后进行序列化,在设值前进行反序列化,序列化器的相关行为由接口ISerializer和ITextSerializer指定,Obase同时提供了抽象基类TextSerializer用于处理一般情况下的序列化,用户只需要实现TextSerializer提供的两个对象与字符串转换的方法即可.

当然,如果需要精细的控制序列化中流的行为,也可以实现ITextSerializer来自定义序列化器.

接下来以具体的场景介绍如何配置序列化器.

## dotNet

假设我们有一个Goods类,内有一个属性GoodsPhotos表示商品的图片集合,类型是字符串数组.

我们想要将此GoodsPhotos属性存储于数据库内的单个字段中,用Json进行存储,那么就可以为这个属性配置序列化器.

序列化器的配置方法为属性配置上的UseSerializer方法,有四个重载,其中两个参数为TextSerializer,两个参数为ITextSerializer,对于使用Json存储这种通常的场景,我们使用TextSerializer参数的配置方法即可.

```
/// <summary>
///     商品
/// </summary>
public class Goods
{
    /// <summary>
    ///     商品ID
    /// </summary>
    public long GoodsId { get; set; }

    /// <summary>
    ///     商品照片集合
    /// </summary>
    public string[] GoodsPhotos { get; set; }
}
```
以上为商品类,接下来定义一个继承TextSerializer的类.
```
/// <summary>
///     普通的JSON序列化器
///     通过TextSerializer简化实现 只需要处理字符串和对象的转换
///     大多数场景下推荐使用此类
/// </summary>
public class JsonSerializer : TextSerializer
{
    /// <summary>
    ///     对给定的文本（以UTF-8编码）实施反序列化，以重建对象（图）。
    /// </summary>
    /// <param name="serializationText">序列化文本。</param>
    /// <param name="objType">要反序列化的对象的类型。</param>
    protected override object DoDeserialize(string serializationText, Type objType)
    {
        return JsonConvert.DeserializeObject(serializationText, objType);
    }

    /// <summary>
    ///     对指定的对象或以该对象为根的对象图实施文本序列化（以UTF-8编码）。
    /// </summary>
    /// <param name="obj">要序列化的对象。</param>
    protected override string DoSerialize(object obj)
    {
        return JsonConvert.SerializeObject(obj);
    }
}
```
实现很简单,使用了JsonConvert来处理Json的序列化,当然也可以加入一些诸如日期格式等Json序列化的参数来自定义Json的序列化结果,不过此处就仅使用默认配置.

最后在配置内加入如下的配置即可:
```
//配置商品实体型
var goods = modelBuilder.Entity<Goods>();
//主键
goods.HasKeyAttribute(p => p.GoodsId).HasKeyIsSelfIncreased(true);
//自定义GoodsPhotos属性
goods.Attribute(p => p.GoodsPhotos)
    //使用Json序列化器
    .UseSerializer<string[]>(new JsonSerializer())
    //设置为255长 超过255会令数据库建表类型变为Text 方便存储
    .HasMaxcharNumber(255);
//映射表
goods.ToTable("Goods");
```
注意UseSerializer的类型参数,是GoodsPhotos的原始类型,会在JsonSerializer的DoDeserialize第二个参数传入,用于辅助反序列化.

此外UseSerializer方法还有两个使用ITextSerializer的重载,这两个重载当然也可以使用之前的JsonSerializer,但如果你想精细的控制序列化中流的行为,也可以直接实现ITextSerializer,以下是一个示例的逗号分隔的序列化器,可以参考其中的方法内注释来确定你的实现.

```
/// <summary>
///     逗号分隔的序列化器
///     实现ITextSerializer接口 可以较为精细的控制流的读写
///     在需要对流进行特殊处理的场景下可以使用 本类仅展示了实现方法 未对流进行处理
/// </summary>
public class CommaSplitSerializer : ITextSerializer
{
    /// <summary>
    ///     对给定的数据实施反序列化，以重建对象（图）。
    /// </summary>
    /// <param name="serializationStream">提供序列化数据的流，它可以指代多种后备存储区，如内存、文件、网络等。</param>
    /// <param name="objType">反序列化的对象的类型。</param>
    public object Deserialize(Stream serializationStream, Type objType)
    {
        //此处使用的是Utils内提供的流转字符串方法 此方法会将流全部读入内存 适用于数据量较小的场景
        //如果需要自定义处理 可以直接操作流
        return Utils.GetUtf8StringFromStream(serializationStream).Split(",");
    }

    /// <summary>
    ///     对指定的对象或以该对象为根的对象图实施序列化。
    /// </summary>
    /// <param name="obj">要序列化的对象。</param>
    /// <param name="serializationStream">存储序列化数据的流，它可以指代多种后备存储区，如内存、文件、网络等。</param>
    public void Serialize(object obj, Stream serializationStream)
    {
        //此处使用的是Utils内提供的字符串写入流的方法 此方法会将流全部读入内存 适用于数据量较小的场景
        //如果需要自定义处理 可以直接操作流
        //此处obj肯定是string[]
        Utils.WriteUtf8StringToStream(string.Join(",", (string[])obj), serializationStream);
    }

    /// <summary>
    ///     对给定的文本（以UTF-8编码）实施反序列化，以重建对象（图）。
    /// </summary>
    /// <param name="serializationText">序列化文本。</param>
    /// <param name="objType">要反序列化的对象的类型。</param>
    public object Deserialize(string serializationText, Type objType)
    {
        //直接分隔即可 不需要考虑objType
        return serializationText.Split(",");
    }

    /// <summary>
    ///     对指定的对象或以该对象为根的对象图实施文本序列化（以UTF-8编码）。
    /// </summary>
    /// <param name="obj">要序列化的对象。</param>
    public string Serialize(object obj)
    {
        //此处obj肯定是string[]
        return string.Join(",", (string[])obj);
    }
}
```

