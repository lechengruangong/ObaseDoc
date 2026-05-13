有些时候我们的系统中可能存在多个不同的用户,他们购买我们的服务支撑自己的业务,这些业务对象需要根据用户进行区分.

为了应对这个需求,Obase提供了多租户的功能.

## 配置方法

首先,需要在引用Obase本体及数据源提供器之外,引用多租户插件.通常情况下,由于引用传递只需要引用对应的数据源提供器和多租户插件即可.

引入的配置类似如下形式,版本改为你需要的版本即可:

```
<PackageReference Include="Obase.MultiTenant" Version="x.x.x" />
<PackageReference Include="Obase.Providers.MySql" Version="x.x.x" />
```