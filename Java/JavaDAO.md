# 组成：
- ### 前台业务层 
 - #### 显示层
 - #### 控制层
- ### 后台业务层
  - #### 业务层
  - #### 数据层

## 业务层：
为整个程序提供操作功能，而一个业务层的操作需要多个数据库的操作数据库层只做原子性数据库操作
复杂情况可能有一个总业务层，多个子业务层
## 数据层：
又被称为数据访问层(Data Access Object,DAO)，专门进行数据库的原子性操作，主要为JDBC中ps接口的使用
业务层：又被称为业务对象(Bussiness Object,BO)，但现在又有一部分成为服务层(Service)，核心目的是调用多个数据层的操作以完成整体项目的业务设计

**数据层 ⇌ 业务层 ⇌ 控制层 间的交互由简单Java类完成，其操作保存在DAO包下**
## 命名规范

- 数据层接口：IEmpDAO（I：接口、Emp：表、DAO：DAO）
- 数据更新：以doXxx命名，如doCreate(),doUpdate()
- 数据查询：
	- 查表：findById(),findByName(),findAll()
	- 统计：getAllCount()
	> 全为接口内方法（不实现）

**业务层实现：EmpDAOImpl类实现**
## 工厂类：
