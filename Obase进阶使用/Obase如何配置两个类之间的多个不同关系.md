Obase默认认为两个类之间只存在一种关系,那么当两个类之间有不同的多种隐式关联时就需要进行特殊的配置.

员工和房间之间有两种关系 管理的房间和所在的房间 此时需要自定义两个类来分别作为这两种关系的关联型


## dotNet
我们定义Employee表示员工,OfficeRoom表示办公室房间.员工和办公室房间有两种关系,一种是工作的房间,此关系为一对一;一种是管理的房间,此关系为一对多.

类定义如下:
```
/// <summary>
///     表示员工
/// </summary>
public class Employee
{
    /// <summary>
    ///     员工编码
    /// </summary>
    private string _employeeCode;

    /// <summary>
    ///     管理的房间
    /// </summary>
    private List<OfficeRoom> _manageRooms;

    /// <summary>
    ///     名称
    /// </summary>
    private string _name;

    /// <summary>
    ///     工作的房间
    /// </summary>
    private OfficeRoom _workRoom;

    /// <summary>
    ///     工作的房间编码
    /// </summary>
    private string _workRoomCode;

    /// <summary>
    ///     员工编码
    /// </summary>
    public string EmployeeCode
    {
        get => _employeeCode;
        set => _employeeCode = value;
    }

    /// <summary>
    ///     名称
    /// </summary>
    public string Name
    {
        get => _name;
        set => _name = value;
    }

    /// <summary>
    ///     管理的房间
    /// </summary>
    public virtual List<OfficeRoom> ManageRooms
    {
        get => _manageRooms;
        set => _manageRooms = value;
    }

    /// <summary>
    ///     工作的房间
    /// </summary>
    public virtual OfficeRoom WorkRoom
    {
        get => _workRoom;
        set => _workRoom = value;
    }

    /// <summary>
    ///     工作的房间编码
    /// </summary>
    public string WorkRoomCode
    {
        get => _workRoomCode;
        set => _workRoomCode = value;
    }

    /// <summary>
    ///     转换为字符串形式
    /// </summary>
    /// <returns></returns>
    public override string ToString()
    {
        return $"Employee;{{EmployeeCode-{_employeeCode},Name-{_name}}}";
    }
}

/// <summary>
///     表示办公室房间
/// </summary>
public class OfficeRoom
{
    /// <summary>
    ///     房间名称
    /// </summary>
    private string _name;

    /// <summary>
    ///     房间号
    /// </summary>
    private string _roomCode;

    /// <summary>
    ///     房间号
    /// </summary>
    public string RoomCode
    {
        get => _roomCode;
        set => _roomCode = value;
    }

    /// <summary>
    ///     房间名称
    /// </summary>
    public string Name
    {
        get => _name;
        set => _name = value;
    }

    /// <summary>
    ///     转换为字符串形式
    /// </summary>
    /// <returns></returns>
    public override string ToString()
    {
        return $"Room;{{RoomCode-{_roomCode},RoomName-{_name}}}";
    }
}
```
那么在Obase的配置中,就需要为这两种关系各自配置,做类似如下配置即可:

```
//配置Employee
var employeeConfig = modelBuilder.Entity<Employee>();
employeeConfig.HasKeyAttribute(p => p.EmployeeCode).HasKeyIsSelfIncreased(false);
employeeConfig.ToTable("Employee");

//配置OfficeRoom
var officeRoomConfig = modelBuilder.Entity<OfficeRoom>();
officeRoomConfig.HasKeyAttribute(p => p.RoomCode).HasKeyIsSelfIncreased(false);
officeRoomConfig.ToTable("OfficeRoom");

//开启一个新的隐式关联配置 配置ManageRooms关系
var manageAssociationTypeConfiguration = modelBuilder.Association();
//配置第一个端
var manageEndConfig = manageAssociationTypeConfiguration.AssociationEnd<Employee>();
//配置映射
manageEndConfig.HasMapping("EmployeeCode", "EmployeeCode");
//对于当两个类间有多种关系时 必须配置关联引用以保证不同的引用使用不同的隐式关联型
manageEndConfig.AssociationReference(p => p.ManageRooms);
//另外一个端
manageAssociationTypeConfiguration.AssociationEnd<OfficeRoom>().HasMapping("RoomCode", "WorkRoomCode");
//映射表
manageAssociationTypeConfiguration.ToTable("ManageRoom");

//开启一个新的隐式关联配置 配置WorkRoom关系
var workAssociationTypeConfiguration = modelBuilder.Association();
//配置第一个端
var workEndConfig = workAssociationTypeConfiguration.AssociationEnd<Employee>();
//配置映射
workEndConfig.HasMapping("EmployeeCode", "EmployeeCode");
//对于当两个类间有多种关系时 必须配置关联引用以保证不同的引用使用不同的隐式关联型
workEndConfig.AssociationReference(p => p.WorkRoom);
//另外一个端
workAssociationTypeConfiguration.AssociationEnd<OfficeRoom>().HasMapping("RoomCode", "WorkRoomCode");
//映射表
workAssociationTypeConfiguration.ToTable("Employee");
```

## Java

Java版待重写
