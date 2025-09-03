# Obase框架
Obase是一款存储抽象层框架中间件,分别在dotNet和Java两种语言下提供了相同的功能,旨在为开发者提供一个统一的存储抽象层,使得开发者可以更专注于业务逻辑的实现,而不必过多关注底层存储的细节.
以下为开源仓库地址:
- [Obase4DotNet](https://github.com/lechengruangong/Obase4DotNet) Obase框架在dotNet平台的实现
- [Obase4Java](https://github.com/lechengruangong/Obase4Java) Obase框架在Java平台的实现

# Obase文档
本项目为Obase的相关文档,目录结构如下:
- 旧版Wiki:此目录下为Obase的旧版Wiki文档,内容较为陈旧,不建议使用.
- Obase入门:此目录下是Obase的入门知识,推荐将按顺序阅读其中的内容.
- Obase基础知识:此目录下为6.0+版本的Obase基础知识,包含如何配置,版本号说明,基元类型等内容.
- Obase进阶使用:此目录下为Obase的进阶使用文档,包含如何配置继承关系,如何配置相同类之间的多个关系等内容.

# 新手推荐阅读
如果是首次接触Obase,推荐按照以下顺序阅读这些文档:
1. 按照顺序阅读Obase入门中的四篇文档,即[快速开始](./Obase入门/快速开始.md),[最佳实践](./Obase入门/最佳实践.md),[深入理解](./Obase入门/深入理解.md),[代码优先](./Obase入门/代码优先.md)这四篇文档来入门.
2. 阅读Obase基础知识中的[Obase的总体配置思路](./Obase基础知识/Obase的总体配置思路.md)和[Obase对象数据模型配置示例](./Obase基础知识/Obase对象数据模型配置示例.md)这两篇文档来深入了解如何配置Obase对象数据模型.
3. 阅读Obase基础知识中[Obase的配置提供器有哪些可以重写的方法和属性](./Obase基础知识/Obase的配置提供器有哪些可以重写的方法和属性.md)和[Obase连接兼容的Sql数据源](./Obase基础知识/Obase连接兼容的Sql数据源.md)这两篇文档补充了解上下文配置提供者中的各个需要重写的方法和属性.