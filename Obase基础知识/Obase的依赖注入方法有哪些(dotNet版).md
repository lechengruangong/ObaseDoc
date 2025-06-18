Obase提供了一个小型的依赖注入容器,用于在基础设施层来获取由应用层提供的服务实现.

依赖注入方法定义于ServiceContainerBuilder服务容器建造器类中,要获得服务容器建造器可以使用ObaseDependencyInjection的静态方法或者自行构造.


```
public static ServiceContainerBuilder CreateBuilder(Type contextType);
public static ServiceContainerBuilder CreateBuilder<TContext>() where TContext : ObjectContext;
```

这两个方法均用于获取一个特定于某种上下文的服务容器建造器.

服务容器建造器有以下方法可用:


```
public ServiceContainerBuilder AddSingleton(Type serviceType);
public ServiceContainerBuilder AddSingleton(Type serviceType, Type implementType);
public ServiceContainerBuilder AddSingleton<TService>();
public ServiceContainerBuilder AddSingleton<TService, TServiceImplement>() where TServiceImplement : TService;
public ServiceContainerBuilder AddSingleton<TService>(Func<ServiceContainer, object> factory);
public ServiceContainerBuilder AddSingleton<TService, TServiceImplement>(Func<ServiceContainer, object> factory);
```

这些方法分别为:

- 添加一个单例的服务定义,此服务的创建方式为根据反射获取到的第一个公开或非公开构造函数创建.此服务的注册类型和构造类型均为参数类型.

- 添加一个单例的服务定义,此服务的创建方式为根据反射获取到的第一个公开或非公开构造函数创建.此服务的注册类型第一个参数的类型和构造类型为第二个参数类型.

- 添加一个单例的服务定义,此服务的创建方式为根据反射获取到的第一个公开或非公开构造函数创建.此服务的注册类型和构造类型均为泛型参数类型.

- 添加一个单例的服务定义,此服务的创建方式为根据反射获取到的第一个公开或非公开构造函数创建.此服务的注册类型第一个泛型参数的类型和构造类型为第二个泛型参数类型.

- 添加一个单例的服务定义,此服务的创建方式为根据的委托构造.此服务的注册类型和构造类型均为泛型参数类型.

- 添加一个单例的服务定义,此服务的创建方式为根据的委托构造.此服务的注册类型第一个参数的类型和构造类型为第二个参数类型.


```
public ServiceContainerBuilder AddTransient(Type serviceType);
public ServiceContainerBuilder AddTransient(Type serviceType, Type implementType);
public ServiceContainerBuilder AddTransient<TService>();
public ServiceContainerBuilder AddTransient<TService, TServiceImplement>() where TServiceImplement : TService;
public ServiceContainerBuilder AddTransient<TService>(Func<ServiceContainer, object> factory);
public ServiceContainerBuilder AddTransient<TService, TServiceImplement>(Func<ServiceContainer, object> factory);
```

这些方法分别为:

- 添加一个多例的服务定义,此服务的创建方式为根据反射获取到的第一个公开或非公开构造函数创建.此服务的注册类型和构造类型均为参数类型.

- 添加一个多例的服务定义,此服务的创建方式为根据反射获取到的第一个公开或非公开构造函数创建.此服务的注册类型第一个参数的类型和构造类型为第二个参数类型.

- 添加一个多例的服务定义,此服务的创建方式为根据反射获取到的第一个公开或非公开构造函数创建.此服务的注册类型和构造类型均为泛型参数类型.

- 添加一个多例的服务定义,此服务的创建方式为根据反射获取到的第一个公开或非公开构造函数创建.此服务的注册类型第一个泛型参数的类型和构造类型为第二个泛型参数类型.

- 添加一个多例的服务定义,此服务的创建方式为根据的委托构造.此服务的注册类型和构造类型均为泛型参数类型.

- 添加一个多例的服务定义,此服务的创建方式为根据的委托构造.此服务的注册类型第一个参数的类型和构造类型为第二个参数类型.

```
public ServiceContainer Build();
```

此方法用于根据服务容器建造器重注册的服务建造特定于某种上下文的服务容器,注意此方法应在应用程序域中仅调用一次.
