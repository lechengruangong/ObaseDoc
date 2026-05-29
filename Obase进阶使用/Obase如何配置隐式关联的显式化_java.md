一般的情况下,对于一个多对多的隐式关联,我们会使用一张独立的关联表来存储这个关系.

Obase默认情况下隐式的多对多关联想要根据关联的一端属性筛选另外一端时,只能查询有筛选属性的一端对象使用Include加载另外一端来查询.

如果希望直接查询另外一端的对象,就要进行特殊的配置,将常用的查询字段从关联的端转移至关联型上以减少联表操作,此时就需要隐式关联的显式化.

考虑一个这样的例子,一个产品可以属于不同的分类,一个分类下有不同的产品,我们需要根据分类名称查询这个分类包含的产品.

那么可以这样定义,类Product表示产品,Category表示分类,他们之间的关系是多对多,类定义如下:
```
/**
 * 产品
 */
public class Product {

    /**
     * 所属的分类
     */
    private List<Category> categories;

    /**
     * 产品Code
     */
    private String code;

    /**
     * 产品名称
     */
    private String name;

    /**
     * 获取所属的分类
     *
     * @return 所属的分类
     */
    public List<Category> getCategories() {
        return this.categories;
    }

    /**
     * 设置所属的分类
     *
     * @param categories 所属的分类
     */
    public void setCategories(List<Category> categories) {
        this.categories = categories;
    }

    /**
     * 获取产品Code
     *
     * @return 产品Code
     */
    public String getCode() {
        return this.code;
    }

    /**
     * 设置产品Code
     *
     * @param code 产品Code
     */
    public void setCode(String code) {
        this.code = code;
    }

    /**
     * 获取产品名称
     *
     * @return 产品名称
     */
    public String getName() {
        return this.name;
    }

    /**
     * 设置产品名称
     *
     * @param name 产品名称
     */
    public void setName(String name) {
        this.name = name;
    }
}

/**
 * 产品分类
 */
public class Category {

    /**
     * 产品分类ID
     */
    private int categoryId;

    /**
     * 产品分类名称
     */
    private String name;

    /**
     * 分类下的产品
     */
    private List<Product> products;

    /**
     * 获取产品分类ID
     *
     * @return 产品分类ID
     */
    public int getCategoryId() {
        return this.categoryId;
    }

    /**
     * 设置产品分类ID
     *
     * @param categoryId 产品分类ID
     */
    public void setCategoryId(int categoryId) {
        this.categoryId = categoryId;
    }

    /**
     * 获取产品分类名称
     *
     * @return 产品分类名称
     */
    public String getName() {
        return this.name;
    }

    /**
     * 设置产品分类名称
     *
     * @param name 产品分类名称
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取分类下的产品
     *
     * @return 分类下的产品
     */
    public List<Product> getProducts() {
        return this.products;
    }

    /**
     * 设置分类下的产品
     *
     * @param products 分类下的产品
     */
    public void setProducts(List<Product> products) {
        this.products = products;
    }
}
```
此时需要定义一个实体化的显式关联型来表示产品和分类之间的关系,取名为ProductCategory,以下为显式化的ProductCategory类

```
/**
 * 显式化的产品分类隐式关联型
 */
public class ProductCategory {

    /**
     * 分类
     */
    private Category category;

    /**
     * 分类ID
     */
    private int categoryId;

    /**
     * 分类名称
     */
    private String categoryName;

    /**
     * 产品
     */
    private Product product;

    /**
     * 产品Code
     */
    private String productCode;

    /**
     * 获取分类
     *
     * @return 分类
     */
    public Category getCategory() {
        return this.category;
    }

    /**
     * 设置分类
     *
     * @param category 分类
     */
    public void setCategory(Category category) {
        this.category = category;
    }

    /**
     * 获取分类ID
     *
     * @return 分类ID
     */
    public int getCategoryId() {
        return this.categoryId;
    }

    /**
     * 设置分类ID
     *
     * @param categoryId 分类ID
     */
    public void setCategoryId(int categoryId) {
        this.categoryId = categoryId;
    }

    /**
     * 获取分类名称
     *
     * @return 分类名称
     */
    public String getCategoryName() {
        return this.categoryName;
    }

    /**
     * 设置分类名称
     *
     * @param categoryName 分类名称
     */
    public void setCategoryName(String categoryName) {
        this.categoryName = categoryName;
    }

    /**
     * 获取产品
     *
     * @return 产品
     */
    public Product getProduct() {
        return this.product;
    }

    /**
     * 设置产品
     *
     * @param product 产品
     */
    public void setProduct(Product product) {
        this.product = product;
    }

    /**
     * 获取产品Code
     *
     * @return 产品Code
     */
    public String getProductCode() {
        return this.productCode;
    }

    /**
     * 设置产品Code
     *
     * @param productCode 产品Code
     */
    public void setProductCode(String productCode) {
        this.productCode = productCode;
    }
}

```
这个类上保存了分类名称这个冗余属性,在配置Obase时将这个类映射到关联表ProductCategory那么就可以在查询的时候直接筛选这个类,然后Include产品Product即可.

此时查询就从Category->ProductCategory->Product变成了ProductCategory->Product减少了一次联表操作.

那么配置方法如下:

```
//将产品配置为实体型
EntityTypeConfiguration<Product> productEntity = modelBuilder.entity(Product.class);
productEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
productEntity.toTable("Product");

//将分类配置为实体型
EntityTypeConfiguration<Category> categoryEntity = modelBuilder.entity(Category.class);
categoryEntity.hasKeyAttribute(p -> p.getCategoryId()).hasKeyIsSelfIncreased(true);
categoryEntity.toTable("Category");

//配置显式化的隐式多对多关联型 注意此处不需要设置HasVisible 这里就应该是显式的 因为只有显式的关联型才能增加属性用于查询优化
var implicitMultiAssociation = modelBuilder.Association<ProductCategory>();
//配置产品关联端 在关联表中映射为主键Code->字段ProductCode
implicitMultiAssociation.AssociationEnd(p => p.Product).HasMapping("Code", "ProductCode");
//配置分类关联端 在关联表中映射为主键CategoryId->字段CategoryId
implicitMultiAssociation.AssociationEnd(p => p.Category).HasMapping("CategoryId", "CategoryId");
//多对多 独立关联表
implicitMultiAssociation.ToTable("ProductCategory");

//配置显式化的隐式多对多关联型
AssociationTypeConfiguration<ProductCategory> implicitMultiAssociation = modelBuilder.association(ProductCategory.class);
//配置产品关联端 在关联表中映射为主键Code->字段ProductCode 映射符合推断
AssociationEndConfigurationGeneric<ProductCategory,Product> productEnd = implicitMultiAssociation.associationEnd(p -> p.getProduct());
//配置分类关联端 在关联表中映射为主键CategoryId->字段CategoryId 映射符合推断
AssociationEndConfigurationGeneric<ProductCategory,Category> categoryEnd = implicitMultiAssociation.associationEnd(p -> p.getCategory());
//多对多 独立关联表 默认的关联表名会被推断为ProductAssCategory
implicitMultiAssociation.toTable("ProductCategory");

//如果在概念建模阶段就注意到需要此种查询 域类已将此关联设置为显示关联时 类内会直接定义关联引用为List<ProductCategory> 则此处不需要做此转换
//此下的配置为领域模型未将关联引用显式化时的配置方式
//配置关联引用 注意此处使用的配置方法 使用的是手动配置方法
productEnd.associationReference("Categories", true)
        //配置取值器 即从对象中取值的方法 此处即为从关联型转换为List<Category>
        .hasValueGetter(new DelegateValueGetter<Product, List<ProductCategory>>(p ->
        {
            if (p.getCategories() == null || p.getCategories().size() == 0)
                return null;
            List<ProductCategory> productCategories = new ArrayList<>();
            for (Category category : p.getCategories()) {
                generateProductCategory(p, productCategories, category);
            }
            return productCategories;
        }))
        //配置设值器 即为对象设置值 此处即为从关联型转换为List<Category>
        .hasValueSetter((Product p, ProductCategory impValue) ->
        {
            if (impValue != null) {
                if (p.getCategories() == null)
                    p.setCategories(new ArrayList<>());
                //检查是否为自己的关联 以及去重
                if (p.getCode().equals(impValue.getProductCode()) && p.getCategories().stream().allMatch(q -> q != null && q.getCategoryId() != impValue.getCategoryId()))
                    p.getCategories().add(impValue.getCategory());
            }
        }, EValueSettingMode.Appending).hasEnableLazyLoading(true);

//配置关联引用 注意此处使用的配置方法 指定的泛型参数为关联型类型
categoryEnd.associationReference("Products", true)
        //配置取值器 即从对象中取值的方法 此处即为从关联型转换为List<Product>
        .hasValueGetter(new DelegateValueGetter<Category, List<ProductCategory>>(p ->
        {
            if (p.getProducts() == null || p.getProducts().size() == 0)
                return null;
            List<ProductCategory> productCategories = new ArrayList<>();
            for (Product product : p.getProducts()) {
                generateProductCategory(product, productCategories, p);
            }
            return productCategories;
        }))
        //配置设值器 即为对象设置值 此处即为从关联型转换为List<Product>
        .hasValueSetter((Category p, ProductCategory impValue) ->
        {
            if (impValue != null) {
                if (p.getProducts() == null)
                    p.setProducts(new ArrayList<>());
                //检查是否为自己的关联 以及去重
                if (p.getCategoryId() == impValue.getCategoryId()
                        && p.getProducts().stream().allMatch(q -> q != null && !Objects.equals(q.getCode(), impValue.getProductCode())))
                    p.getProducts().add(impValue.getProduct());
            }
        }, EValueSettingMode.Appending);
```

以下为如何查询:

```
//根据分类名称查询下属的产品 此处需要借助冗余属性CategoryName
ProductCategory productCategories = context.createSet(ProductCategory.class).filter(p -> p.getCategoryName() == "产品分类B")
                .include(p -> p.getProduct()).toList();
```