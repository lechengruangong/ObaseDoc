序列化是一种常见的数据持久化的手段,我们在定义类时也往往有一些属性虽然定义为某个类型但不需要进行独立存储只需要存储在字段里的情形.

为应对此需求,Obase为属性配置增加了序列化器相关的配置方法,用于处理某些属性实际上是序列化后进行存储的场景.

配置了序列化器的属性会在取值后进行序列化,在设值前进行反序列化,序列化器的相关行为由接口ISerializer和ITextSerializer指定,Obase同时提供了抽象基类TextSerializer用于处理一般情况下的序列化,用户只需要实现TextSerializer提供的两个对象与字符串转换的方法即可.

当然,如果需要精细的控制序列化中流的行为,也可以实现ITextSerializer来自定义序列化器.

## 属性序列化配置方法

序列化器的配置方法为属性配置上的useSerializer方法,有四个重载,其中两个参数为TextSerializer,两个参数为ITextSerializer,对于使用Json存储这种通常的场景,我们使用TextSerializer参数的配置方法即可.

```
//比较常用的配置序列器方法
attrConfig.useSerializer(new CommaSplitSerializer(), String[].class)
```
注意第二个参数,此处的类型应当为属性的定义类型.

## 实现序列化器

通常实现抽象类TextSerializer即可,在DoDeserialize方法中进行反序列化,在DoSerialize方法中进行序列化.

如果有特定的需求,也可以实现ITextSerializer接口,此接口相较TextSerializer多了两个针对流的处理方法需要实现.

## 具体示例

假设我们有一个Goods类,内有一个属性GoodsPhotos表示商品的图片集合,类型是字符串数组.

我们想要将此GoodsPhotos属性存储于数据库内的单个字段中,用Json进行存储,那么就可以为这个属性配置序列化器.

```
/**
 * 商品
 */
public class Goods {
    /**
     * 商品ID
     */
    private long goodsId;

    /**
     * 商品照片集合
     */
    private String[] goodsPhotos;

    /**
     * 获取商品ID
     * @return 商品ID
     */
    public long getGoodsId() {
        return goodsId;
    }

    /**
     * 设置商品ID
     * @param goodsId 商品ID
     */
    public void setGoodsId(long goodsId) {
        this.goodsId = goodsId;
    }

    /**
     * 获取商品照片集合
     * @return 商品照片集合
     */
    public String[] getGoodsPhotos() {
        return goodsPhotos;
    }

    /**
     * 设置商品照片集合
     * @param goodsPhotos 商品照片集合
     */
    public void setGoodsPhotos(String[] goodsPhotos) {
        this.goodsPhotos = goodsPhotos;
    }
}
```
以上为商品类,接下来定义一个继承TextSerializer的类.
```
/**
 * 普通的JSON序列化器
 */
public class JsonSerializer extends TextSerializer {
    /**
     * 对给定的文本（以UTF-8编码）实施反序列化，以重建对象（图）
     * 重写此方法进行反序列化
     *
     * @param serializationText 序列化文本
     * @param objType           要反序列化的对象的类型
     * @return 反序列化的对象
     */
    @Override
    public Object doDeserialize(String serializationText, Class<?> objType) {
        return JSON.parseObject(serializationText, objType);
    }

    /**
     * 对指定的对象或以该对象为根的对象图实施文本序列化（以UTF-8编码）
     * 重写此方法进行序列化
     *
     * @param obj 要序列化的对象
     * @return 序列化的结果
     */
    @Override
    public String doSerialize(Object obj) {
        //指定日期格式
        return JSON.toJSONString(obj, "yyyy-MM-dd HH:mm:ss.SSS");
    }
}
```
实现很简单,使用了FastJson2来处理Json的序列化,当然也可以加入一些诸如日期格式等Json序列化的参数来自定义Json的序列化结果,不过此处就仅使用默认配置.

最后在配置内加入如下的配置即可:
```
//配置商品实体型
EntityTypeConfiguration<Goods> goods = modelBuilder.entity(Goods.class);
//主键
goods.hasKeyAttribute(p -> p.getGoodsId()).hasKeyIsSelfIncreased(true);
//自定义GoodsPhotos属性
goods.attribute(p -> p.getGoodsPhotos(), String.class)
    //使用Json序列化器
    .useSerializer(new JsonSerializer(), String[].class)
    //设置为255长 超过255会令数据库建表类型变为Text 方便存储
    .hasMaxCharNumber(255)
//映射表
goods.toTable("Goods");
```
注意UseSerializer的类型参数,是GoodsPhotos的原始类型,会在JsonSerializer的doDeserialize第二个参数传入,用于辅助反序列化.

此外UseSerializer方法还有两个使用ITextSerializer的重载,这两个重载当然也可以使用之前的JsonSerializer,但如果你想精细的控制序列化中流的行为,也可以直接实现ITextSerializer,以下是一个示例的逗号分隔的序列化器,可以参考其中的方法内注释来确定你的实现.

```
/**
 * 逗号分隔的序列化器
 */
public class CommaSplitSerializer implements ITextSerializer {
    /**
     * 对给定的数据实施反序列化，以重建对象（图）。
     *
     * @param serializationStream 提供序列化数据的流，它可以指代多种后备存储区，如内存、文件、网络等
     * @param objType             反序列化的对象的类型
     * @return 对象
     */
    @Override
    public Object deserialize(InputStream serializationStream, Class<?> objType) {
        return Utils.readUtf8StringFromInputStream(serializationStream).split(",");
    }

    /**
     * 对指定的对象或以该对象为根的对象图实施序列化
     *
     * @param obj                 要序列化的对象
     * @param serializationStream 存储序列化数据的流，它可以指代多种后备存储区，如内存、文件、网络等
     */
    @Override
    public void serialize(Object obj, OutputStream serializationStream) {
        //此处传入的Obj肯定为string[]
        Utils.writeUtf8StringToOutputStream(String.join(",", (String[]) obj), serializationStream);
    }

    /**
     * 对给定的文本（以UTF-8编码）实施反序列化，以重建对象（图）。
     *
     * @param serializationText 序列化文本。
     * @param objType           要反序列化的对象的类型。
     * @return 重建对象
     */
    @Override
    public Object deserialize(String serializationText, Class<?> objType) {
        return serializationText.split(",");
    }

    /**
     * 对指定的对象或以该对象为根的对象图实施文本序列化（以UTF-8编码）。
     *
     * @param obj 要序列化的对象
     * @return 文本序列化
     */
    @Override
    public String serialize(Object obj) {
        return String.join(",", (String[]) obj);
    }
}
```