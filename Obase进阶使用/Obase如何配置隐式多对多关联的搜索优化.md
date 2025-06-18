Obase默认情况下隐式的多对多关联想要根据关联的一端属性筛选另外一端时,只能查询有筛选属性的一端对象使用Include加载另外一端来查询.

如果希望直接查询另外一端的对象,就要进行特殊的配置.

## dotNet

有如下类Product表示产品,Category表示分类,他们之间的关系是多对多.

现在需要根据分类名称查询这个分类包含的产品.

类定义如下:
```
/// <summary>
///     产品
/// </summary>
public class Product
{
    /// <summary>
    ///     所属的分类
    /// </summary>
    private List<Category> _categories;

    /// <summary>
    ///     产品Code
    /// </summary>
    private string _code;

    /// <summary>
    ///     产品名称
    /// </summary>
    private string _name;

    /// <summary>
    ///     产品Code
    /// </summary>
    public string Code
    {
        get => _code;
        set => _code = value;
    }

    /// <summary>
    ///     产品名称
    /// </summary>
    public string Name
    {
        get => _name;
        set => _name = value;
    }

    /// <summary>
    ///     所属的分类
    /// </summary>
    public List<Category> Categories
    {
        get => _categories;
        set => _categories = value;
    }
}

/// <summary>
///     产品分类
/// </summary>
public class Category
{
    /// <summary>
    ///     产品分类ID
    /// </summary>
    private int _categoryId;

    /// <summary>
    ///     产品分类名称
    /// </summary>
    private string _name;

    /// <summary>
    ///     分类下的产品
    /// </summary>
    private List<Product> _products;

    /// <summary>
    ///     产品分类ID
    /// </summary>
    public int CategoryId
    {
        get => _categoryId;
        set => _categoryId = value;
    }

    /// <summary>
    ///     产品分类名称
    /// </summary>
    public string Name
    {
        get => _name;
        set => _name = value;
    }

    /// <summary>
    ///     分类下的产品
    /// </summary>
    public List<Product> Products
    {
        get => _products;
        set => _products = value;
    }
}
```
此时需要定义一个实体化的显式关联型来表示产品和分类之间的关系,取名为ProductCategory,以下为显式化的ProductCategory类

```
/// <summary>
///     显式化的产品分类隐式关联型
/// </summary>
public class ProductCategory
{
    /// <summary>
    ///     分类
    /// </summary>
    private Category _category;

    /// <summary>
    ///     分类ID
    /// </summary>
    private int _categoryId;

    /// <summary>
    ///     分类名称
    /// </summary>
    private string _categoryName;

    /// <summary>
    ///     产品
    /// </summary>
    private Product _product;

    /// <summary>
    ///     产品Code
    /// </summary>
    private string _productCode;

    /// <summary>
    ///     产品Code
    /// </summary>
    public string ProductCode
    {
        get => _productCode;
        set => _productCode = value;
    }

    /// <summary>
    ///     分类ID
    /// </summary>
    public int CategoryId
    {
        get => _categoryId;
        set => _categoryId = value;
    }

    /// <summary>
    ///     分类名称
    /// </summary>
    public string CategoryName
    {
        get => _categoryName;
        set => _categoryName = value;
    }

    /// <summary>
    ///     产品
    /// </summary>
    public Product Product
    {
        get => _product;
        set => _product = value;
    }

    /// <summary>
    ///     分类
    /// </summary>
    public Category Category
    {
        get => _category;
        set => _category = value;
    }
}
```
这个类上保存了分类名称这个冗余属性,在配置Obase时将这个类映射到关联表ProductCategory那么就可以在查询的时候直接筛选这个类,然后Include产品Product即可.

此时查询就从Category->ProductCategory->Product变成了ProductCategory->Product减少了一次联表操作.

那么配置方法如下:

```
//将产品配置为实体型
var productEntity = modelBuilder.Entity<Product>();
productEntity.HasKeyAttribute(p => p.Code).HasKeyIsSelfIncreased(false);
productEntity.ToTable("Product");

//将分类配置为实体型
var categoryEntity = modelBuilder.Entity<Category>();
categoryEntity.HasKeyAttribute(p => p.CategoryId).HasKeyIsSelfIncreased(true);
categoryEntity.ToTable("Category");

//配置显式化的隐式多对多关联型 注意此处不需要设置HasVisible 这里就应该是显式的 因为只有显式的关联型才能增加属性用于查询优化
var implicitMultiAssociation = modelBuilder.Association<ProductCategory>();
//配置产品关联端 在关联表中映射为主键Code->字段ProductCode
implicitMultiAssociation.AssociationEnd(p => p.Product).HasMapping("Code", "ProductCode");
//配置分类关联端 在关联表中映射为主键CategoryId->字段CategoryId
implicitMultiAssociation.AssociationEnd(p => p.Category).HasMapping("CategoryId", "CategoryId");
//多对多 独立关联表
implicitMultiAssociation.ToTable("ProductCategory");

//如果在概念建模阶段就注意到需要此种查询 域类已将此关联设置为显示关联时 类内会直接定义关联引用为List<ProductCategory> 则此处不需要做此转换
//此下的配置为领域模型未将关联引用显式化时的配置方式
//配置关联引用 注意此处使用的配置方法 指定的泛型参数为关联型类型
productEntity.AssociationReference<ProductCategory>("Categories", true)
    //配置取值器 即从对象中取值的方法 此处即为从关联型转换为List<Category>
    .HasValueGetter(new DelegateValueGetter<Product, List<ProductCategory>>(p =>
    {
        if (p.Categories == null || p.Categories.Count == 0)
            return null;
        return p.Categories.Select(q => new ProductCategory
        {
            Category = q,
            CategoryId = q.CategoryId,
            CategoryName = q.Name,
            Product = p,
            ProductCode = p.Code
        }).ToList();
    }))
    //配置设值器 即为对象设置值 此处即为从关联型转换为List<Category>
    .HasValueSetter<ProductCategory>((p, impValue) =>
    {
        if (impValue != null)
        {
            if (p.Categories == null)
                p.Categories = new List<Category>();
            //检查是否为自己的关联 以及去重
            if (p.Code == impValue.ProductCode
                && p.Categories.All(q => q != null && q.CategoryId != impValue.CategoryId))
                p.Categories.Add(impValue.Category);
        }
    }, eValueSettingMode.Appending)
    //左端为自己 产品Product 右端分类Category
    .HasLeftEnd("Product").HasRightEnd("Category");

//配置关联引用 注意此处使用的配置方法 指定的泛型参数为关联型类型
categoryEntity.AssociationReference<ProductCategory>("Products", true)
    //配置取值器 即从对象中取值的方法 此处即为从关联型转换为List<Product>
    .HasValueGetter(new DelegateValueGetter<Category, List<ProductCategory>>(p =>
    {
        if (p.Products == null || p.Products.Count == 0)
            return null;
        return p.Products.Select(q => new ProductCategory
        {
            Category = p,
            CategoryId = p.CategoryId,
            CategoryName = p.Name,
            Product = q,
            ProductCode = q.Code
        }).ToList();
    }))
    //配置设值器 即为对象设置值 此处即为从关联型转换为List<Product>
    .HasValueSetter<ProductCategory>((p, impValue) =>
    {
        if (impValue != null)
        {
            if (p.Products == null)
                p.Products = new List<Product>();
            //检查是否为自己的关联 以及去重
            if (p.CategoryId == impValue.CategoryId
                && p.Products.All(q => q != null && q.Code != impValue.ProductCode))
                p.Products.Add(impValue.Product);
        }
    }, eValueSettingMode.Appending)
    //左端为自己 分类Category 右端产品Product
    .HasLeftEnd("Category").HasRightEnd("Product");
```

以下为如何查询:

```
//根据分类名称查询下属的产品 此处需要借助冗余属性CategoryName
var productCategories = context.CreateSet<ProductCategory>().Where(p => p.CategoryName == "产品分类B")
    .Include(p => p.Product).ToList();
```

## Java

Java版待重写