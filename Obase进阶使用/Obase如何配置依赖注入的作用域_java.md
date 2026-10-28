Obase的依赖注入管理的服务分为单例和多例,单例由容器持有,多例每次获取都新建.但**多例创建出来的实例也一直由容器持有,直到容器关闭**——容器如果不关闭,这些实例就一直累积.

在Obase本体内部这不是问题,因为Obase自己用的服务都是单例.但在Obase所在的领域设施层里,多例往往是"每次请求一份"的对象(仓储、工作单元、会话状态),如果它们实现了`AutoCloseable`,长期运行的宿主里就会持续累积内存和未释放的资源.

作用域(ServiceScope)解决两件事:

- **给一批多例实例指定一个明确的归属者和释放时机**;
- **让同一个作用域内的依赖共享同一批实例**,这由第三种生命周期`Scoped`(作用域内单例)提供.

## 创建与使用

作用域的创建方法在容器上:

```java
//建造容器
ServiceContainer container = ObaseDependencyInjection.createBuilder(SampleContext.class)
        .addScoped(IUnitOfWork.class, UnitOfWork.class)     //作用域内单例:一次作用域内共享
        .addTransient(IRepository.class, Repository.class)  //多例:每次获取都是新对象
        .build();

//每一次请求 创建一个作用域
try (ServiceScope scope = container.createScope()) {
    IUnitOfWork unitOfWork = scope.getService(IUnitOfWork.class);
    IRepository repository = scope.getService(IRepository.class);
}//离开作用域时 逆序 close repository、unitOfWork
```

`scope.getService(...)`与容器的用法完全一致,获取某服务类型的全部实现使用`scope.getServices(...)`.

## API速查

| 用途 | API |
|---|---|
| 生命周期枚举 | `EServiceLifetime.Singleton` / `Transient` / `Scoped` |
| 注册单例 | `builder.addSingleton(T.class)`、`ServiceDefinition.singletonService(T.class)` |
| 注册多例 | `builder.addTransient(T.class)`、`ServiceDefinition.transientService(T.class)` |
| 注册作用域内单例 | `builder.addScoped(T.class)`、`ServiceDefinition.scopedService(T.class)` |
| 创建作用域 | `container.createScope()` |
| 创建子作用域 | `scope.createScope()` |
| 按类型解析 | `scope.getService(Class<T>)` |
| 获取某类型的全部实现 | `scope.getServices(Class<T>)` |
| 关闭作用域 | `scope.close()`(配合 try-with-resources) |
| 是否已关闭 | `scope.isClosed()` |
| 已关闭后再获取 | 抛出`IllegalStateException` |
| 单例依赖作用域内单例 | 抛出`IllegalStateException`,异常信息含解析链 |
| 关闭时单个实例释放失败 | 不影响其余实例的释放 |

> `EServiceLifetime.Transients`在6.6.0改名为`Transient`,`addTransients`改名为`addTransient`;`ServiceDefinition.singleton`/`transients`改名为`singletonService`/`transientService`(Java的`transient`是保留字,不能作为方法名).

## 三种生命周期在作用域中的行为

- **单例(`addSingleton`)**:无论从容器还是从作用域获取,拿到的都是同一个实例;作用域关闭**不会**关闭单例.
- **多例(`addTransient`)**:从作用域获取的多例归该作用域所有,作用域关闭时释放;从容器获取的多例仍归容器所有,与以前一样在容器关闭时释放.
- **作用域内单例(`addScoped`)**:同一个作用域内多次获取都是同一个实例,不同作用域各自持有自己的实例,随自己的作用域关闭;从容器获取时归容器所有,随容器关闭.

同一作用域内的释放顺序是**创建顺序的逆序**,因此依赖总是先于依赖方创建、后于依赖方释放.

| 获取方式 | 实例归属 | 释放时机 |
|---|---|---|
| `container.getService(T.class)`(单例) | 容器 | 容器关闭时 |
| `container.getService(T.class)`(多例) | 容器 | 容器关闭时 |
| `container.getService(T.class)`(作用域内单例) | 容器 | 容器关闭时 |
| `scope.getService(T.class)`(单例) | 容器 | 容器关闭时 |
| `scope.getService(T.class)`(多例) | 作用域 | 作用域关闭时 |
| `scope.getService(T.class)`(作用域内单例) | 作用域 | 作用域关闭时 |

## 作用域的语义

1. **作用域只是释放边界与作用域内单例的缓存边界,不是可见性边界.** 作用域内能解析到的服务与根容器完全相同,不存在"只在作用域内可见"的注册.
2. **单例永远属于容器.** 无论从根容器还是从任意作用域获取,单例都是同一个实例;作用域关闭不影响单例.
3. **多例的归属取决于从谁那里获取.** 从根容器获取归容器(与改造前完全一致);从作用域获取归该作用域.
4. **作用域内单例在获取它的那个作用域内缓存.** 同一作用域内多次获取是同一个实例;不同作用域各自持有自己的实例;从根容器获取时归容器所有(容器本身被当成一个作用域).
5. **单例不能依赖作用域内单例.** 一旦允许,单例就会把某个作用域的实例长期持有,造成实例与作用域生命周期错位,因此在获取单例时直接抛出`IllegalStateException`,异常信息里带有解析链.
6. **同一作用域内的释放顺序是创建顺序的逆序.** 作用域内单例与它的多例依赖同样遵循这一顺序:作用域内单例先释放,它的依赖后释放.
7. **作用域可以嵌套.** 子作用域由父作用域创建(`scope.createScope()`),其内获取的多例与作用域内单例归子作用域;关闭父作用域时,先关闭仍然存活的子作用域(后创建的先关闭),再释放父作用域自己的实例.
8. **已关闭的作用域不能再获取服务**,抛出`IllegalStateException`;**重复关闭是幂等的**,不抛异常.
9. **容器关闭时先关闭所有仍然存活的作用域,再释放容器自身的实例**(含从容器获取的作用域内单例);容器关闭后不能再创建作用域,也不能再获取服务.
10. **解析失败时不额外回滚.** 获取过程中抛异常时,这一次已经创建并登记到作用域的实例仍然归该作用域,由作用域关闭时统一释放(与容器现有行为一致).
11. **作用域不是线程安全的.** 一个作用域应当只在一个逻辑执行流内使用,跨线程共享需调用方自行同步;容器与单例缓存的线程安全性与改造前一致.

## 明确不做的事

| 不做 | 原因 |
|---|---|
| 不引入"当前作用域"(`ThreadLocal`) | `ThreadLocal`不能跨线程、更不能跨响应式链路传播,隐式作用域会让"从哪取到实例"变得不可见;取服务统一走显式传入的作用域 |
| 不把"作用域内单例注入到单例"放宽为可用 | 那会让单例持有某个作用域的实例,释放时机错位(通常表现为内存泄漏与"已关闭对象仍被使用"),宁可获取时直接报错 |
| 不改变`Utils`的取服务方式 | `Utils`仍从根容器解析. **Obase本体内部使用的服务应注册为单例**;需要"每次请求一份"的服务由宿主(领域设施层)显式从作用域取 |
| 不做"首次使用时才校验依赖" | 依赖关系在获取时一次性校验,报错位置更接近注册错误 |

## 嵌套作用域

作用域可以再创建作用域:用`scope.createScope()`从父作用域创建子作用域.子作用域中的多例与作用域内单例归子作用域所有;关闭父作用域时,会先关闭仍然存活的子作用域,再释放父作用域自己的实例.

```java
ServiceScope outer = container.createScope();
try {
    IRepository repository = outer.getService(IRepository.class);
    try (ServiceScope inner = outer.createScope()) {
        IUnitOfWork unitOfWork = inner.getService(IUnitOfWork.class);
    }//此处 close unitOfWork
} finally {
    outer.close();//此处 close repository
}
```

> 子作用域也可以从容器直接创建(`container.createScope()`),那样的作用域与当前作用域没有父子关系,各自独立关闭;只有从父作用域创建的才会随父作用域一起关闭.

## 注意事项

- **作用域不是线程安全的**,应当只在一个逻辑执行流内使用;跨线程共享需要调用方自己同步.
- 作用域关闭后再获取服务会抛出`IllegalStateException`;重复关闭不会有任何影响.
- 容器关闭时,会先关闭所有仍然存活的作用域,再释放容器自身的实例;容器关闭后不能再创建作用域,也不能再获取服务.
- **单例不能依赖作用域内单例**:遇到`IllegalStateException`时应把被依赖的服务改为多例或单例,而不是绕过这项检查.
- Obase本体内部使用的服务(日志工厂、变更通知发送器、租户ID读取器、连接池配置)仍从容器获取,**这些服务应当注册为单例**,不要把它们的实现改成从作用域获取.
- 需要"每次请求一份"的服务,由调用方(通常是领域设施层)显式地从作用域获取,不要依赖`Utils`.
- 从根容器获取多例仍会累积到容器关闭为止——这是原有行为,保留不变. **宿主生命周期长、且服务的生命周期短于宿主时,改用作用域获取.**
