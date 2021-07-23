# Controler Service Manager Dao

## Storey

![](C:\Users\Administrator\Desktop\secret\img\storey.png)

**1.开放接口层：** 可直接封装 Service 方法暴露成 RPC 接口；通过 Web 封装成 http 接口；进行 网关安全控制、流量控制等。
**2.终端显示层：** 各个端的模板渲染并执行显示的层。当前主要是 velocity 渲染，JS 渲染， JSP 渲染，移动端展示等。
**3.Web 层：** 主要是对访问控制进行转发，各类基本参数校验，或者不复用的业务简单处理等。
**4.Service 层：** 相对具体的业务逻辑服务层。
**5.Manager 层：** 通用业务处理层，它有如下特征：
A）对第三方平台封装的层，预处理返回结果及转化异常信息；
B）对 Service 层通用能力的下沉，如缓存方案、中间件通用处理；
C）与 DAO 层交互，对多个 DAO 的组合复用。
**6.DAO 层：** 数据访问层，与底层 MySQL、Oracle、Hbase 等进行数据交互。
**7.外部接口或第三方平台：** 包括其它部门 RPC 开放接口，基础平台，其它公司的 HTTP 接口

## Dao Dto

DTO：数据传输对象，一般是把数据库表封装成对象，表的各个字段就是该对象的各个变量。

![](C:\Users\Administrator\Desktop\secret\img\dto.png)

![](C:\Users\Administrator\Desktop\secret\img\dto2.png)

Dao：数据访问对象，负责封装对数据库的CRUD操作，一般是mapper写接口，xml文件写sql语句的形式。

![](C:\Users\Administrator\Desktop\secret\img\baseMapper.png)

![](C:\Users\Administrator\Desktop\secret\img\subMapper.png)

## Manager

**前提：** 如果是小应用，而且后续扩展的可能性不高，只需要Dao——service——controller的

**manager层：** 负责将Dao层中的数据库操作组合复用，主要是一些缓存方案，中间件的处理，以及对第三方平台封装的层。

![](C:\Users\Administrator\Desktop\secret\img\manager.png)

> 一个manager里面一般只引入一个对应的dao或者同包下的dao，禁止引入非同包下的多个dao

## Service

**service层：** 更加关注业务逻辑，是业务处理层，将manager组合过的操作和业务逻辑组合在一起，再封装成业务操作。

1. service里引入manager

   ![](C:\Users\Administrator\Desktop\secret\img\service_manager.png)

2. service里引入mapper

   ![](C:\Users\Administrator\Desktop\secret\img\service_mapper.png)

> 注：
>
> 一个service里可以注入多个其他service或manager，但只能注入一个对应的dao（即mapper），或者多个同包下面的dao（即mapper）；
>
> 一个manager里可以注入多个其他manager，但也只能注入一个对应的dao（即mapper），或者多个同包下面的dao（即mapper）；

## Controller

**controller层**:主要负责接受前台的数据和请求，并且在底层处理完之后把结果返回回去，一般不能写业务逻辑在这一层，因为第一造成了不可复用，第二以后的维护困难，第三这一层没有上层，如果给用户返回了奇怪的错误信息将会非常丑陋。

![](C:\Users\Administrator\Desktop\secret\img\controller.png)