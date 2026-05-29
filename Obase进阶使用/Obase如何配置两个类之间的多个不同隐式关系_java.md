Obase默认认为两个类之间只存在一种隐式关系,那么当两个类之间有不同的多种隐式关联时就需要进行特殊的配置.

显然,只有隐式关联才可能存在多个看上去一模一样的关系,两个相同类之间不同的显式关联自然是不同的类来表示.

那么对于这种情况,只需要为这些不同的关联引用各自定义隐式关联型即可.

考虑一个例子:员工和房间之间有两种关系,管理的房间和所在的房间,要如何为这种情况进行定义实体类和配置Obase?

我们定义Employee表示员工,OfficeRoom表示办公室房间.员工和办公室房间有两种关系,一种是工作的房间,此关系为一对一;一种是管理的房间,此关系为一对多.

考虑到工作房间关系是一对一的,这个关联就和员工存储在员工表内,而管理的房间关系是一对多的,就单独的存储在另外一张关联表内.

类定义如下:
```
/**
 * 表示员工
 */
public class Employee {

    /**
     * 员工编码
     */
    private String employeeCode;

    /**
     * 管理的房间
     */
    private List<OfficeRoom> manageRooms;

    /**
     * 名称
     */
    private String name;

    /**
     * 工作的房间
     */
    private OfficeRoom workRoom;

    /**
     * 工作的房间编码
     */
    private String workRoomCode;

    /**
     * 获取员工编码
     *
     * @return 员工编码
     */
    public String getEmployeeCode() {
        return this.employeeCode;
    }

    /**
     * 设置员工编码
     *
     * @param employeeCode 员工编码
     */
    public void setEmployeeCode(String employeeCode) {
        this.employeeCode = employeeCode;
    }

    /**
     * 获取名称
     *
     * @return 名称
     */
    public String getName() {
        return this.name;
    }

    /**
     * 设置名称
     *
     * @param name 名称
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取管理的房间
     *
     * @return 管理的房间
     */
    public List<OfficeRoom> getManageRooms() {
        return this.manageRooms;
    }

    /**
     * 设置管理的房间
     *
     * @param manageRooms 管理的房间
     */
    public void setManageRooms(List<OfficeRoom> manageRooms) {
        this.manageRooms = manageRooms;
    }

    /**
     * 获取工作的房间
     *
     * @return 工作的房间
     */
    public OfficeRoom getWorkRoom() {
        return this.workRoom;
    }

    /**
     * 设置工作的房间
     *
     * @param workRoom 工作的房间
     */
    public void setWorkRoom(OfficeRoom workRoom) {
        this.workRoom = workRoom;
    }

    /**
     * 获取工作的房间编码
     *
     * @return 工作的房间编码
     */
    public String getWorkRoomCode() {
        return this.workRoomCode;
    }

    /**
     * 设置工作的房间编码
     *
     * @param workRoomCode 工作的房间编码
     */
    public void setWorkRoomCode(String workRoomCode) {
        this.workRoomCode = workRoomCode;
    }

    /**
     * 重写字符串表示形式
     *
     * @return 字符串表示形式
     */
    @Override
    public String toString() {
        return "Employee{" +
                "employeeCode='" + this.employeeCode + '\'' +
                ", manageRooms=" + this.manageRooms +
                ", name='" + this.name + '\'' +
                ", workRoom=" + this.workRoom +
                ", workRoomCode='" + this.workRoomCode + '\'' +
                '}';
    }
}

/**
 * 表示办公室房间
 */
public class OfficeRoom {

    /**
     * 房间名称
     */
    private String name;

    /**
     * 房间号
     */
    private String roomCode;

    /**
     * 获取房间名称
     *
     * @return 房间名称
     */
    public String getName() {
        return this.name;
    }

    /**
     * 设置房间名称
     *
     * @param name 房间名称
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取房间名称
     *
     * @return 房间名称
     */
    public String getRoomCode() {
        return this.roomCode;
    }

    /**
     * 设置房间名称
     *
     * @param roomCode 房间名称
     */
    public void setRoomCode(String roomCode) {
        this.roomCode = roomCode;
    }

    /**
     * 重写字符串表示形式
     *
     * @return 字符串表示形式
     */
    @Override
    public String toString() {
        return "OfficeRoom{" +
                "name='" + this.name + '\'' +
                ", roomCode='" + this.roomCode + '\'' +
                '}';
    }
}

```
那么在Obase的配置中,就需要为这两种关系各自配置,做类似如下配置即可:

```
//配置Employee
EntityTypeConfiguration<Employee> employeeConfig = modelBuilder.entity(Employee.class);
employeeConfig.hasKeyAttribute(p -> p.getEmployeeCode()).hasKeyIsSelfIncreased(false);
employeeConfig.toTable("Employee");

//配置OfficeRoom
EntityTypeConfiguration<OfficeRoom> officeRoomConfig = modelBuilder.entity(OfficeRoom.class);
officeRoomConfig.hasKeyAttribute(p -> p.getRoomCode()).hasKeyIsSelfIncreased(false);
officeRoomConfig.toTable("OfficeRoom");

//开启一个新的隐式关联配置
AssociationConfiguratorBuilder manageAssociationTypeConfiguration = modelBuilder.association();
//配置第一个端
manageAssociationTypeConfiguration.associationEnd(Employee.class)
        //对于当两个类间有多种关系时 必须配置关联引用以保证不同的引用使用不同的隐式关联型
        .associationReference(p -> p.getManageRooms());
//另外一个端
manageAssociationTypeConfiguration.associationEnd(OfficeRoom.class);
//映射表
manageAssociationTypeConfiguration.toTable("ManageRoom");

//开启一个新的隐式关联配置
AssociationConfiguratorBuilder workAssociationTypeConfiguration = modelBuilder.association();
//配置第一个端
workAssociationTypeConfiguration.associationEnd(Employee.class)
        //对于当两个类间有多种关系时 必须配置关联引用以保证不同的引用使用不同的隐式关联型
        .associationReference(p -> p.getWorkRoom());
//另外一个端
workAssociationTypeConfiguration.associationEnd(OfficeRoom.class)
        .hasMapping("RoomCode", "WorkRoomCode");
//映射表
workAssociationTypeConfiguration.toTable("Employee");
```
