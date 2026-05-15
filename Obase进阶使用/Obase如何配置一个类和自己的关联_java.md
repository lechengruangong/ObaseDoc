一般情况下,关系是存在于两个不同的类之间的,但有些时候关系的两个参与者是相同的,这种情况就称之为自关联.

自然的,自关联也分为两种,隐式自关联和显式自关联.接下来考虑一个如下的场景,Area(区域)表示行政区划,如省,市,区.Area上有两个关联,分别是当前区域的父级和子级区域以及当前区域的友好区域(如友好城市).

那么类定义如下:

```
/**
 * 表示一个区域
 */
public class Area {

    /**
     * 区域代码
     */
    private String code;

    /**
     * 友好区域
     */
    private List<FriendlyArea> friendlyAreas;

    /**
     * 名字
     */
    private String name;

    /**
     * 父级区域
     */
    private Area parentArea;

    /**
     * 父级区域代码
     */
    private String parentCode;

    /**
     * 子区域
     */
    private List<Area> subAreas;

    /**
     * 获取区域代码
     *
     * @return 区域代码
     */
    public String getCode() {
        return this.code;
    }

    /**
     * 设置区域代码
     *
     * @param code 区域代码
     */
    public void setCode(String code) {
        this.code = code;
    }

    /**
     * 获取友好区域
     *
     * @return 友好区域
     */
    public List<FriendlyArea> getFriendlyAreas() {
        return this.friendlyAreas;
    }

    /**
     * 设置友好区域
     *
     * @param friendlyAreas 友好区域
     */
    public void setFriendlyAreas(List<FriendlyArea> friendlyAreas) {
        this.friendlyAreas = friendlyAreas;
    }

    /**
     * 获取名字
     *
     * @return 名字
     */
    public String getName() {
        return this.name;
    }

    /**
     * 设置名字
     *
     * @param name 名字
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取父级区域
     *
     * @return 父级区域
     */
    public Area getParentArea() {
        return this.parentArea;
    }

    /**
     * 设置父级区域
     *
     * @param parentArea 父级区域
     */
    public void setParentArea(Area parentArea) {
        this.parentArea = parentArea;
    }

    /**
     * 获取父级区域代码
     *
     * @return 父级区域代码
     */
    public String getParentCode() {
        return this.parentCode;
    }

    /**
     * 设置父级区域代码
     *
     * @param parentCode 父级区域代码
     */
    public void setParentCode(String parentCode) {
        this.parentCode = parentCode;
    }

    /**
     * 获取子区域
     *
     * @return 子区域
     */
    public List<Area> getSubAreas() {
        return this.subAreas;
    }

    /**
     * 设置子区域
     *
     * @param subAreas 子区域
     */
    public void setSubAreas(List<Area> subAreas) {
        this.subAreas = subAreas;
    }
}

/**
 * 友好区域
 */
public class FriendlyArea {

    /**
     * 区域
     */
    private Area area;

    /**
     * 区域代码
     */
    private String areaCode;

    /**
     * 友好区域
     */
    private Area friend;

    /**
     * 友好区域代码
     */
    private String friendlyAreaCode;

    /**
     * 开始时间
     */
    private Date startTime;

    /**
     * 获取区域
     *
     * @return 区域
     */
    public Area getArea() {
        return this.area;
    }

    /**
     * 设置区域
     *
     * @param area 区域
     */
    public void setArea(Area area) {
        this.area = area;
    }

    /**
     * 获取区域代码
     *
     * @return 区域代码
     */
    public String getAreaCode() {
        return this.areaCode;
    }

    /**
     * 设置区域代码
     *
     * @param areaCode 区域代码
     */
    public void setAreaCode(String areaCode) {
        this.areaCode = areaCode;
    }

    /**
     * 获取友好区域
     *
     * @return 友好区域
     */
    public Area getFriend() {
        return this.friend;
    }

    /**
     * 设置友好区域
     *
     * @param friend 友好区域
     */
    public void setFriend(Area friend) {
        this.friend = friend;
    }

    /**
     * 获取友好区域代码
     *
     * @return 友好区域代码
     */
    public String getFriendlyAreaCode() {
        return this.friendlyAreaCode;
    }

    /**
     * 设置友好区域代码
     *
     * @param friendlyAreaCode 友好区域代码
     */
    public void setFriendlyAreaCode(String friendlyAreaCode) {
        this.friendlyAreaCode = friendlyAreaCode;
    }

    /**
     * 获取开始时间
     *
     * @return 开始时间
     */
    public Date getStartTime() {
        return this.startTime;
    }

    /**
     * 设置开始时间
     *
     * @param startTime 开始时间
     */
    public void setStartTime(Date startTime) {
        this.startTime = startTime;
    }
}
```

对于这两个关系,可以辨析出区域和区域之间的父级子级的自关联和友好区域自关联,分别是隐式自关联和显示自关联.那么相应的配置如下:

```
//注册区域
EntityTypeConfiguration<Area> areaEntity = modelBuilder.entity(Area.class);
areaEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
areaEntity.toTable("Area");

//区域的自关联
AssociationConfiguratorBuilder areaArea = modelBuilder.association();
//配置第一个Area端
AssociationEndConfigurationGeneric<Area> areaEnd1 = areaArea.associationEnd(Area.class);
//配置映射
areaEnd1.hasMapping("Code", "Code");
//配置是否启用延迟加载
areaEnd1.hasEnableLazyLoading(true);
//配置关联引用和关联引用的延迟加载
AssociationReferenceConfiguration<Area> subRef = areaEnd1.associationReference(p -> p.getParentArea());
subRef.hasEnableLazyLoading(true);
//配置第二个Area端
AssociationEndConfigurationGeneric<Area> areaEnd2 = areaArea.associationEnd(Area.class);
//配置映射
areaEnd2.hasMapping("Code", "ParentCode");
//配置是否启用延迟加载
areaEnd2.hasEnableLazyLoading(true);
//配置关联引用和关联引用的延迟加载
AssociationReferenceConfiguration<Area> parentRef = areaEnd2.associationReference(p -> p.getSubAreas());
parentRef.hasEnableLazyLoading(true);
//映射表
areaArea.toTable("Area");

//区域的显式自关联 友好区域
AssociationTypeConfiguration<FriendlyArea> friendlyArea = modelBuilder.association(FriendlyArea.class);
//配置第一个Area端
AssociationEndConfigurationGeneric<FriendlyArea,Area> friendlyAreaEnd1 = friendlyArea.associationEnd(p -> p.getArea());
//配置延迟加载 映射
friendlyAreaEnd1.hasMapping("Code", "AreaCode").hasEnableLazyLoading(true);
//配置关联引用 是否启用延迟加载 映射
friendlyAreaEnd1.associationReference("FriendlyAreas", true).hasEnableLazyLoading(true);
//配置第二个Area端 配置映射 是否启用延迟加载
friendlyArea.associationEnd(p -> p.getFriend()).hasMapping("Code", "FriendlyAreaCode").hasEnableLazyLoading(true);
//映射表
friendlyArea.toTable("FriendlyArea");
```

这里与普通的配置不同的地方是,在设置关联端的映射之后,需要指定此关联端上的关联引用,注意配置两个端的具体配置顺序即可.