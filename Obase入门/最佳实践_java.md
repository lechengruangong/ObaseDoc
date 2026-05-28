在阅读了[快速开始](./快速开始_java.md)之后,我已经可以使用Obase对Article进行持久化操作了,但现在我又新增了一张表Category表示文章的分类,并且我定义了Category数据模型类:
```
/**
 * 文章分类
 */
public class Category {

    /**
     * 分类标识
     */
    private Integer id;

    /**
     * 分类名称
     */
    private String name;

    /**
     * 分类下文章集合
     */
    private List<Article> articles;

    /**
     * 获取分类标识
     * @return 分类标识
     */
    public Integer getId() {
        return id;
    }

    /**
     * 设置分类标识
     * @param id 分类标识
     */
    public void setId(Integer id) {
        this.id = id;
    }

    /**
     * 获取分类名称
     * @return 分类名称
     */
    public String getName() {
        return name;
    }

    /**
     * 设置分类名称
     * @param name 分类名称
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取分类下文章集合
     * @return 分类下文章集合
     */
    public List<Article> getArticles() {
        return articles;
    }

    /**
     * 设置分类下文章集合
     * @param articles 分类下文章集合
     */
    public void setArticles(List<Article> articles) {
        this.articles = articles;
    }
}
```
为了表示某个文章属于某个文章分类,我还修改了Article数据模型类

```
/**
 * 博客文章
 */
public class Article {

    /**
     * 文章代码
     */
    private String code;

    /**
     * 文章标题
     */
    private String title;

    /**
     * 文章所属的分类ID
     */
    private Integer categoryId;

    /**
     * 文章所属的分类
     */
    private Category category;

    /**
     * 获取文章代码
     * @return 文章代码
     */
    public String getCode() {
        return code;
    }

    /**
     * 设置文章代码
     * @param code 文章代码
     */
    public void setCode(String code) {
        this.code = code;
    }

    /**
     * 获取文章标题
     * @return 文章标题
     */
    public String getTitle() {
        return title;
    }

    /**
     * 设置文章标题
     * @param title 文章标题
     */
    public void setTitle(String title) {
        this.title = title;
    }

    /**
     * 获取文章所属的分类ID
     * @return 所属分类ID
     */
    public Integer getCategoryId() {
        return categoryId;
    }

    /**
     * 设置文章所属的分类ID
     * @param categoryId 所属分类ID
     */
    public void setCategoryId(Integer categoryId) {
        this.categoryId = categoryId;
    }

    /**
     * 获取文章所属的分类
     * @return 所属分类
     */
    public Category getCategory() {
        return category;
    }

    /**
     * 设置文章所属的分类
     * @param category 所属分类
     */
    public void setCategory(Category category) {
        this.category = category;
    }
}
```

和Article表和数据模型类,新增了CategoryId来表示文章所属的分类ID和Category对象,现在我要怎么使用这两个类来操作数据库?

### 修改配置

首先,我们要修改一下原有的配置代码,原有的代码仅有Article的配置,显然我们需要增加Category的配置.

此外,我们还需要配置Article和Category之间的关系,关系配置是Obase引入的全新概念,引入了关系之后Obase可以更准确的追踪对象之间的关系变更,以实现更多的和关系相关的功能.

```
 /// <summary>使用指定的建模器创建对象数据模型。</summary>
 /// <param name="modelBuilder">对象数据模型建造器</param>
 protected override void CreateModel(ModelBuilder modelBuilder)
 {
        //对Article数据模型类进行配置
        EntityTypeConfiguration<Article> articleEntity = modelBuilder.entity(Article.class);
        //配置主键映射 和 主键是否自增
        articleEntity.hasKeyAttribute(p -> p.getCode()).hasKeyIsSelfIncreased(false);
        //配置映射表
        articleEntity.toTable("Article");

        //对Category数据模型类进行配置
        EntityTypeConfiguration<Category> categoryEntity = modelBuilder.entity(Category.class);
        //配置主键映射 和 主键是否自增
        categoryEntity.hasKeyAttribute(p -> p.getId()).hasKeyIsSelfIncreased(true);
        //配置映射表
        categoryEntity.toTable("Category");

        //配置Article和Category之间的关系
        AssociationConfiguratorBuilder categoryAssArticle = modelBuilder.association();
        //Article和Category之间的关系中 一端是Article 在关联表中 Article的主键Code就是Code字段
        categoryAssArticle.associationEnd(Article.class).hasMapping("Code", "Code");
        //Article和Category之间的关系中 一端是Category 在关联表中 Category的主键Id就是CategoryId字段
        categoryAssArticle.associationEnd(Category.class).hasMapping("Id", "CategoryId");
        //Article和Category之间的关系存储于Article表
        categoryAssArticle.toTable("Article");
 }
```
这里首先新增了Category的映射关系,首先将Category类注册为Entity,然后设置Category的主键和主键是否自增,最后配置Category在数据库所存储的表.

然后是新增Article和Category之间关系,首先声明一个关系,然后这个关系有两个关联端,一个是Article,另外一个是Category,这个关系是存储于Article表的.

这里的关系是一个全新的概念,一个关系至少需要两个实体参与,所以这里配置了参与此关系的关联端,此外还要知道在这个关系中,端实体的主键存储于数据库的哪个字段.

这个Article和Category之间关系是存储在Article表里的,所以Article的Mapping就是Article的主键Code在Article表里映射为Code字段,而Category的主键IdArticle表里映射为CategoryId字段.

更多的Obase对象数据建模概念,请参考[深入理解](./深入理解_java.md)这篇文档.

在新增了这些配置之后,我们就可以开始了.

### 一并保存

我们这次要新建一个分类,叫做默认分类并且在此分类下新增我们的第一篇文章,那么就可以写作如下代码:

```
//新增分类和文章
Category category = new Category();
category.setName("默认分类");
//构造文章时将分类赋值
Article article = new Article();
article.setCode("A0001");
article.setTitle("第一篇文章");
article.setCategory(category);
//附加至上下文
context.attach(category);
context.attach(article);
//保存至数据库
context.saveChanges();
```
当然,我们在之后添加文章时可能使用的是已有的分类,那么就可以写作如下的代码:

```
//先查询出之前的默认分类
Category category = context.createSet(Category.class).findFirst(p -> p.getName() == "默认分类").orElse(null);
//构造文章时将分类赋值
Article article = new Article();
article.setCode("A0002");
article.setTitle("第二篇文章");
article.setCategory(category);
//附加至上下文
context.attach(category);
context.attach(article);
//保存至数据库
context.saveChanges();
```

这里的引用赋值实质上是建立了Category和Article之间的关系.

Obase会根据配置的实体型和关联型来侦测对象和对象间的关系,如果想要更详细的了解Obase的配置,请阅读[深入理解](./深入理解_java.md).

### 关联查询

以下介绍几种关联查询的方法.

由于我们在文章上定义了分类的ID,所以我们可以很轻易的写出根据分类ID查询文章的查询:
```
//查询分类ID为1的所有文章
List<Article> article = context.createSet(Article.class).filter(p -> p.getategoryId() == 1).toList();
```
我们在Article数据模型类里有引用Category,那么当然也可以在查询文章时同时查询文章的分类信息:
```
//查询Code为A0001的Article 并且 同时加载关联的Category
Article article = context.createSet(Article.class).include(p -> p.getCategory()).findFirst(p -> p.getCode() == "A0001").orElse(null);
```
这里使用了一个Obase的方法Include,此方法会在查询时同时加载Category对象.

当然我们也可以在查询Category时同时查询Article,
```
//查询分类ID为1的分类 并且 同时加载关联的Article
List<Article> articles = context.createSet(Category.class).include(p -> p.getArticles()).filter(p -> p.getId() == 1).toList();
```
Include的参数可以进行延展,比如再定义一个User对象并且在Article上定义Creator的关联引用,想要在查询分类时同时加载关联的Article和User,

就可以写作Include("Articles.Creator");

Obase还可以对关联对象使用投影,分组等操作,比如查询某个文章所属的分类可以使用这样的查询:

```
//查询Code为A0002的Article关联的Category
Category category = context.createSet(Article.class).filter(p -> p.getCode() == "A0002").map(p -> p.getCategory(), Category.class).findFirst().orElse(null);
```

自然可以从Category投影至Article,比如查询某个分类下所有的文章就可以使用平展投影:

```
//查询分类ID为1的分类下所有文章
List<Article> articles = context.createSet(Category.class).filter(p -> p.getId() == 1).flatMap(p -> p.getArticles(), Article.class).toList();
```
更多的查询操作可以参考[Obase如何进行查询](Obase如何进行查询_java.md).

### 解除关联

我们在之前一并保存里建立的关系自然是可以解除的:

```
//查询出Article
Article article = context.createSet(Article.class).include(p -> p.getCategory()).findFirst(p -> p.getCode() == "A0001").orElse(null);
//将Category置空
article.setCategory(null);
//保存至数据库
context.saveChanges();
```
保存后可以发现Article的CategoryId被置空了但Category表内的数据并没有删除,因为此操作只解除了关联没有删除对象.