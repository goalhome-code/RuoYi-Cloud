# RuoYi-Cloud 微服务系统 REST API 接口开发文档

## 目录
- [1. 项目概览](#1-项目概览)
- [2. 认证和权限](#2-认证和权限)
- [3. API 接口文档](#3-api-接口文档)
- [4. 数据模型](#4-数据模型)
- [5. 错误码说明](#5-错误码说明)

## 1. 项目概览

### 1.1 项目架构说明

RuoYi-Cloud 是一个基于 Spring Cloud 的微服务架构系统，采用前后端分离的开发模式。

**微服务模块划分：**
```
com.ruoyi     
├── ruoyi-ui              // 前端框架 [80]
├── ruoyi-gateway         // 网关模块 [8080]
├── ruoyi-auth            // 认证中心 [9200]
├── ruoyi-api             // 接口模块
│   └── ruoyi-api-system  // 系统接口
├── ruoyi-common          // 通用模块
│   ├── ruoyi-common-core         // 核心模块
│   ├── ruoyi-common-datascope    // 权限范围
│   ├── ruoyi-common-datasource   // 多数据源
│   ├── ruoyi-common-log          // 日志记录
│   ├── ruoyi-common-redis        // 缓存服务
│   ├── ruoyi-common-seata        // 分布式事务
│   ├── ruoyi-common-security     // 安全模块
│   ├── ruoyi-common-sensitive    // 数据脱敏
│   └── ruoyi-common-swagger      // 系统接口
├── ruoyi-modules         // 业务模块
│   ├── ruoyi-system      // 系统模块 [9201]
│   ├── ruoyi-gen         // 代码生成 [9202]
│   ├── ruoyi-job         // 定时任务 [9203]
│   └── ruoyi-file        // 文件服务 [9300]
└── ruoyi-visual          // 图形化管理模块
    └── ruoyi-visual-monitor  // 监控中心 [9100]
```

### 1.2 技术栈和框架版本

| 技术栈 | 版本 | 说明 |
|--------|------|------|
| Spring Boot | 2.7.18 | 基础框架 |
| Spring Cloud | 2021.0.9 | 微服务框架 |
| Spring Cloud Alibaba | 2021.0.6.1 | 阿里巴巴微服务组件 |
| Nacos | - | 服务注册与配置中心 |
| Gateway | - | 服务网关 |
| JWT | 0.9.1 | JWT认证 |
| MyBatis | - | ORM框架 |
| Redis | - | 缓存数据库 |
| MySQL | - | 关系型数据库 |
| Druid | 1.2.23 | 数据库连接池 |
| FastJSON | 2.0.57 | JSON处理 |
| POI | 4.1.2 | Excel处理 |
| Quartz | - | 定时任务 |
| Vue.js | 3.x | 前端框架 |

### 1.3 各模块职责和功能描述

| 模块 | 端口 | 职责描述 |
|------|------|----------|
| ruoyi-gateway | 8080 | 统一网关入口，路由转发、负载均衡、限流熔断 |
| ruoyi-auth | 9200 | 认证授权中心，用户登录、JWT令牌管理 |
| ruoyi-system | 9201 | 系统管理模块，用户、角色、菜单、部门等基础功能 |
| ruoyi-gen | 9202 | 代码生成模块，根据数据库表自动生成代码 |
| ruoyi-job | 9203 | 定时任务模块，任务调度和执行 |
| ruoyi-file | 9300 | 文件服务模块，文件上传下载管理 |
| ruoyi-monitor | 9100 | 监控中心，系统监控和健康检查 |

## 2. 认证和权限

### 2.1 JWT 认证机制

系统采用 JWT（JSON Web Token）进行用户认证：

1. **登录流程**：用户提供用户名密码 → 验证成功后生成JWT令牌 → 返回令牌给客户端
2. **请求验证**：客户端在请求头中携带 `Authorization: Bearer {token}` → 网关验证令牌有效性 → 转发到具体服务
3. **令牌刷新**：提供令牌刷新接口，延长令牌有效期

### 2.2 权限控制注解

| 注解 | 说明 | 示例 |
|------|------|------|
| `@RequiresPermissions` | 需要特定权限才能访问 | `@RequiresPermissions("system:user:list")` |
| `@InnerAuth` | 内部服务调用认证 | `@InnerAuth` |
| `@Log` | 操作日志记录 | `@Log(title = "用户管理", businessType = BusinessType.INSERT)` |

### 2.3 统一响应格式

所有API接口都遵循统一的响应格式：

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {},
  "rows": [],
  "total": 0
}
```

**字段说明：**
- `code`: 响应状态码（200成功，其他为错误码）
- `msg`: 响应消息
- `data`: 单个对象数据
- `rows`: 列表数据
- `total`: 总记录数（分页查询时使用）

## 3. API 接口文档

### 3.1 认证模块 (ruoyi-auth)

#### 3.1.1 用户登录
- **接口路径**：`POST /auth/login`
- **功能说明**：用户登录认证
- **请求参数**：
```json
{
  "username": "admin",
  "password": "admin123"
}
```
- **响应格式**：
```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "access_token": "eyJhbGciOiJIUzUxMiJ9...",
    "expires_in": 720
  }
}
```

#### 3.1.2 用户登出
- **接口路径**：`DELETE /auth/logout`
- **功能说明**：用户退出登录
- **请求头**：`Authorization: Bearer {token}`
- **响应格式**：
```json
{
  "code": 200,
  "msg": "操作成功"
}
```

#### 3.1.3 刷新令牌
- **接口路径**：`POST /auth/refresh`
- **功能说明**：刷新JWT令牌
- **请求头**：`Authorization: Bearer {token}`
- **响应格式**：
```json
{
  "code": 200,
  "msg": "操作成功"
}
```

#### 3.1.4 用户注册
- **接口路径**：`POST /auth/register`
- **功能说明**：用户注册
- **请求参数**：
```json
{
  "username": "testuser",
  "password": "123456"
}
```

### 3.2 系统管理模块 (ruoyi-system)

#### 3.2.1 用户管理

##### 获取用户列表
- **接口路径**：`GET /system/user/list`
- **权限要求**：`system:user:list`
- **请求参数**：
  - `pageNum`: 页码
  - `pageSize`: 每页数量
  - `userName`: 用户名（可选）
  - `status`: 状态（可选）
- **响应格式**：
```json
{
  "code": 200,
  "msg": "查询成功",
  "rows": [
    {
      "userId": 1,
      "userName": "admin",
      "nickName": "管理员",
      "email": "ry@163.com",
      "phonenumber": "15888888888",
      "sex": "1",
      "status": "0",
      "createTime": "2023-01-01 00:00:00"
    }
  ],
  "total": 1
}
```

##### 获取用户详情
- **接口路径**：`GET /system/user/{userId}`
- **权限要求**：`system:user:query`
- **路径参数**：`userId` - 用户ID

##### 新增用户
- **接口路径**：`POST /system/user`
- **权限要求**：`system:user:add`
- **请求参数**：
```json
{
  "userName": "testuser",
  "nickName": "测试用户",
  "email": "test@example.com",
  "phonenumber": "13800138000",
  "sex": "0",
  "status": "0",
  "deptId": 103,
  "postIds": [1, 2],
  "roleIds": [2]
}
```

##### 修改用户
- **接口路径**：`PUT /system/user`
- **权限要求**：`system:user:edit`

##### 删除用户
- **接口路径**：`DELETE /system/user/{userIds}`
- **权限要求**：`system:user:remove`
- **路径参数**：`userIds` - 用户ID数组，多个用逗号分隔

##### 重置密码
- **接口路径**：`PUT /system/user/resetPwd`
- **权限要求**：`system:user:resetPwd`

##### 修改用户状态
- **接口路径**：`PUT /system/user/changeStatus`
- **权限要求**：`system:user:edit`

##### 获取当前用户信息
- **接口路径**：`GET /system/user/getInfo`
- **功能说明**：获取当前登录用户的详细信息
- **响应格式**：
```json
{
  "code": 200,
  "msg": "操作成功",
  "user": {
    "userId": 1,
    "userName": "admin",
    "nickName": "管理员"
  },
  "roles": ["admin"],
  "permissions": ["*:*:*"]
}
```

#### 3.2.2 角色管理

##### 获取角色列表
- **接口路径**：`GET /system/role/list`
- **权限要求**：`system:role:list`

##### 获取角色详情
- **接口路径**：`GET /system/role/{roleId}`
- **权限要求**：`system:role:query`

##### 新增角色
- **接口路径**：`POST /system/role`
- **权限要求**：`system:role:add`

##### 修改角色
- **接口路径**：`PUT /system/role`
- **权限要求**：`system:role:edit`

##### 删除角色
- **接口路径**：`DELETE /system/role/{roleIds}`
- **权限要求**：`system:role:remove`

##### 修改角色状态
- **接口路径**：`PUT /system/role/changeStatus`
- **权限要求**：`system:role:edit`

##### 分配数据权限
- **接口路径**：`PUT /system/role/dataScope`
- **权限要求**：`system:role:edit`

#### 3.2.3 菜单管理

##### 获取菜单列表
- **接口路径**：`GET /system/menu/list`
- **权限要求**：`system:menu:list`

##### 获取菜单详情
- **接口路径**：`GET /system/menu/{menuId}`
- **权限要求**：`system:menu:query`

##### 新增菜单
- **接口路径**：`POST /system/menu`
- **权限要求**：`system:menu:add`

##### 修改菜单
- **接口路径**：`PUT /system/menu`
- **权限要求**：`system:menu:edit`

##### 删除菜单
- **接口路径**：`DELETE /system/menu/{menuId}`
- **权限要求**：`system:menu:remove`

##### 获取路由信息
- **接口路径**：`GET /system/menu/getRouters`
- **功能说明**：获取当前用户的菜单路由信息

#### 3.2.4 部门管理

##### 获取部门列表
- **接口路径**：`GET /system/dept/list`
- **权限要求**：`system:dept:list`

##### 获取部门详情
- **接口路径**：`GET /system/dept/{deptId}`
- **权限要求**：`system:dept:query`

##### 新增部门
- **接口路径**：`POST /system/dept`
- **权限要求**：`system:dept:add`

##### 修改部门
- **接口路径**：`PUT /system/dept`
- **权限要求**：`system:dept:edit`

##### 删除部门
- **接口路径**：`DELETE /system/dept/{deptId}`
- **权限要求**：`system:dept:remove`

#### 3.2.5 岗位管理

##### 获取岗位列表
- **接口路径**：`GET /system/post/list`
- **权限要求**：`system:post:list`

##### 获取岗位详情
- **接口路径**：`GET /system/post/{postId}`
- **权限要求**：`system:post:query`

##### 新增岗位
- **接口路径**：`POST /system/post`
- **权限要求**：`system:post:add`

##### 修改岗位
- **接口路径**：`PUT /system/post`
- **权限要求**：`system:post:edit`

##### 删除岗位
- **接口路径**：`DELETE /system/post/{postIds}`
- **权限要求**：`system:post:remove`

#### 3.2.6 字典管理

##### 字典类型管理

###### 获取字典类型列表
- **接口路径**：`GET /system/dict/type/list`
- **权限要求**：`system:dict:list`

###### 获取字典类型详情
- **接口路径**：`GET /system/dict/type/{dictId}`
- **权限要求**：`system:dict:query`

###### 新增字典类型
- **接口路径**：`POST /system/dict/type`
- **权限要求**：`system:dict:add`

###### 修改字典类型
- **接口路径**：`PUT /system/dict/type`
- **权限要求**：`system:dict:edit`

###### 删除字典类型
- **接口路径**：`DELETE /system/dict/type/{dictIds}`
- **权限要求**：`system:dict:remove`

##### 字典数据管理

###### 获取字典数据列表
- **接口路径**：`GET /system/dict/data/list`
- **权限要求**：`system:dict:list`

###### 根据字典类型获取字典数据
- **接口路径**：`GET /system/dict/data/type/{dictType}`
- **功能说明**：根据字典类型查询字典数据信息

###### 获取字典数据详情
- **接口路径**：`GET /system/dict/data/{dictCode}`
- **权限要求**：`system:dict:query`

###### 新增字典数据
- **接口路径**：`POST /system/dict/data`
- **权限要求**：`system:dict:add`

###### 修改字典数据
- **接口路径**：`PUT /system/dict/data`
- **权限要求**：`system:dict:edit`

###### 删除字典数据
- **接口路径**：`DELETE /system/dict/data/{dictCodes}`
- **权限要求**：`system:dict:remove`

#### 3.2.7 参数配置管理

##### 获取参数配置列表
- **接口路径**：`GET /system/config/list`
- **权限要求**：`system:config:list`

##### 获取参数配置详情
- **接口路径**：`GET /system/config/{configId}`

##### 根据参数键名查询参数值
- **接口路径**：`GET /system/config/configKey/{configKey}`

##### 新增参数配置
- **接口路径**：`POST /system/config`
- **权限要求**：`system:config:add`

##### 修改参数配置
- **接口路径**：`PUT /system/config`
- **权限要求**：`system:config:edit`

##### 删除参数配置
- **接口路径**：`DELETE /system/config/{configIds}`
- **权限要求**：`system:config:remove`

##### 刷新参数缓存
- **接口路径**：`DELETE /system/config/refreshCache`
- **权限要求**：`system:config:remove`

#### 3.2.8 通知公告管理

##### 获取通知公告列表
- **接口路径**：`GET /system/notice/list`
- **权限要求**：`system:notice:list`

##### 获取通知公告详情
- **接口路径**：`GET /system/notice/{noticeId}`
- **权限要求**：`system:notice:query`

##### 新增通知公告
- **接口路径**：`POST /system/notice`
- **权限要求**：`system:notice:add`

##### 修改通知公告
- **接口路径**：`PUT /system/notice`
- **权限要求**：`system:notice:edit`

##### 删除通知公告
- **接口路径**：`DELETE /system/notice/{noticeIds}`
- **权限要求**：`system:notice:remove`

#### 3.2.9 日志管理

##### 操作日志

###### 获取操作日志列表
- **接口路径**：`GET /system/operlog/list`
- **权限要求**：`system:operlog:list`

###### 删除操作日志
- **接口路径**：`DELETE /system/operlog/{operIds}`
- **权限要求**：`system:operlog:remove`

###### 清空操作日志
- **接口路径**：`DELETE /system/operlog/clean`
- **权限要求**：`system:operlog:remove`

##### 登录日志

###### 获取登录日志列表
- **接口路径**：`GET /system/logininfor/list`
- **权限要求**：`system:logininfor:list`

###### 删除登录日志
- **接口路径**：`DELETE /system/logininfor/{infoIds}`
- **权限要求**：`system:logininfor:remove`

###### 清空登录日志
- **接口路径**：`DELETE /system/logininfor/clean`
- **权限要求**：`system:logininfor:remove`

###### 账户解锁
- **接口路径**：`GET /system/logininfor/unlock/{userName}`
- **权限要求**：`system:logininfor:unlock`

#### 3.2.10 在线用户管理

##### 获取在线用户列表
- **接口路径**：`GET /system/online/list`
- **权限要求**：`monitor:online:list`

##### 强退用户
- **接口路径**：`DELETE /system/online/{tokenId}`
- **权限要求**：`monitor:online:forceLogout`

### 3.3 代码生成模块 (ruoyi-gen)

#### 3.3.1 获取代码生成列表
- **接口路径**：`GET /code/gen/list`
- **权限要求**：`tool:gen:list`

#### 3.3.2 查询数据库列表
- **接口路径**：`GET /code/gen/db/list`
- **权限要求**：`tool:gen:list`

#### 3.3.3 获取代码生成信息
- **接口路径**：`GET /code/gen/{tableId}`
- **权限要求**：`tool:gen:query`

#### 3.3.4 导入表结构
- **接口路径**：`POST /code/gen/importTable`
- **权限要求**：`tool:gen:import`

#### 3.3.5 修改代码生成信息
- **接口路径**：`PUT /code/gen`
- **权限要求**：`tool:gen:edit`

#### 3.3.6 删除代码生成
- **接口路径**：`DELETE /code/gen/{tableIds}`
- **权限要求**：`tool:gen:remove`

#### 3.3.7 预览生成代码
- **接口路径**：`GET /code/gen/preview/{tableId}`
- **权限要求**：`tool:gen:preview`

#### 3.3.8 生成代码
- **接口路径**：`GET /code/gen/download/{tableName}`
- **权限要求**：`tool:gen:code`

### 3.4 定时任务模块 (ruoyi-job)

#### 3.4.1 获取定时任务列表
- **接口路径**：`GET /schedule/job/list`
- **权限要求**：`monitor:job:list`

#### 3.4.2 获取定时任务详情
- **接口路径**：`GET /schedule/job/{jobId}`
- **权限要求**：`monitor:job:query`

#### 3.4.3 新增定时任务
- **接口路径**：`POST /schedule/job`
- **权限要求**：`monitor:job:add`

#### 3.4.4 修改定时任务
- **接口路径**：`PUT /schedule/job`
- **权限要求**：`monitor:job:edit`

#### 3.4.5 删除定时任务
- **接口路径**：`DELETE /schedule/job/{jobIds}`
- **权限要求**：`monitor:job:remove`

#### 3.4.6 修改任务状态
- **接口路径**：`PUT /schedule/job/changeStatus`
- **权限要求**：`monitor:job:changeStatus`

#### 3.4.7 立即执行任务
- **接口路径**：`PUT /schedule/job/run`
- **权限要求**：`monitor:job:changeStatus`

#### 3.4.8 获取任务执行日志列表
- **接口路径**：`GET /schedule/job/log/list`
- **权限要求**：`monitor:job:list`

#### 3.4.9 删除任务执行日志
- **接口路径**：`DELETE /schedule/job/log/{jobLogIds}`
- **权限要求**：`monitor:job:remove`

### 3.5 文件服务模块 (ruoyi-file)

#### 3.5.1 文件上传
- **接口路径**：`POST /file/upload`
- **请求参数**：`file` - 上传的文件
- **响应格式**：
```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "name": "test.jpg",
    "url": "http://localhost:9300/statics/2023/01/01/test_20230101_001.jpg"
  }
}
```

#### 3.5.2 文件删除
- **接口路径**：`DELETE /file/delete`
- **请求参数**：`fileUrl` - 文件URL

## 4. 数据模型

### 4.1 用户相关实体

#### SysUser (用户信息)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| userId | Long | 用户ID |
| deptId | Long | 部门ID |
| userName | String | 用户账号 |
| nickName | String | 用户昵称 |
| userType | String | 用户类型（00系统用户） |
| email | String | 用户邮箱 |
| phonenumber | String | 手机号码 |
| sex | String | 用户性别（0男 1女 2未知） |
| avatar | String | 头像地址 |
| password | String | 密码 |
| status | String | 账号状态（0正常 1停用） |
| delFlag | String | 删除标志（0存在 2删除） |
| loginIp | String | 最后登录IP |
| loginDate | Date | 最后登录时间 |
| pwdUpdateDate | Date | 密码最后更新时间 |

#### SysRole (角色信息)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| roleId | Long | 角色ID |
| roleName | String | 角色名称 |
| roleKey | String | 角色权限字符串 |
| roleSort | Integer | 显示顺序 |
| dataScope | String | 数据范围 |
| menuCheckStrictly | Boolean | 菜单树选择项是否关联显示 |
| deptCheckStrictly | Boolean | 部门树选择项是否关联显示 |
| status | String | 角色状态（0正常 1停用） |
| delFlag | String | 删除标志（0存在 2删除） |

#### SysMenu (菜单权限)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| menuId | Long | 菜单ID |
| menuName | String | 菜单名称 |
| parentId | Long | 父菜单ID |
| orderNum | Integer | 显示顺序 |
| path | String | 路由地址 |
| component | String | 组件路径 |
| query | String | 路由参数 |
| isFrame | Integer | 是否为外链（0是 1否） |
| isCache | Integer | 是否缓存（0缓存 1不缓存） |
| menuType | String | 菜单类型（M目录 C菜单 F按钮） |
| visible | String | 菜单状态（0显示 1隐藏） |
| status | String | 菜单状态（0正常 1停用） |
| perms | String | 权限标识 |
| icon | String | 菜单图标 |

### 4.2 系统管理实体

#### SysDept (部门信息)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| deptId | Long | 部门ID |
| parentId | Long | 父部门ID |
| ancestors | String | 祖级列表 |
| deptName | String | 部门名称 |
| orderNum | Integer | 显示顺序 |
| leader | String | 负责人 |
| phone | String | 联系电话 |
| email | String | 邮箱 |
| status | String | 部门状态（0正常 1停用） |
| delFlag | String | 删除标志（0存在 2删除） |

#### SysPost (岗位信息)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| postId | Long | 岗位ID |
| postCode | String | 岗位编码 |
| postName | String | 岗位名称 |
| postSort | Integer | 显示顺序 |
| status | String | 状态（0正常 1停用） |

#### SysConfig (参数配置)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| configId | Long | 参数主键 |
| configName | String | 参数名称 |
| configKey | String | 参数键名 |
| configValue | String | 参数键值 |
| configType | String | 系统内置（Y是 N否） |

#### SysDictType (字典类型)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| dictId | Long | 字典主键 |
| dictName | String | 字典名称 |
| dictType | String | 字典类型 |
| status | String | 状态（0正常 1停用） |

#### SysDictData (字典数据)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| dictCode | Long | 字典编码 |
| dictSort | Long | 字典排序 |
| dictLabel | String | 字典标签 |
| dictValue | String | 字典键值 |
| dictType | String | 字典类型 |
| cssClass | String | 样式属性 |
| listClass | String | 表格字典样式 |
| isDefault | String | 是否默认（Y是 N否） |
| status | String | 状态（0正常 1停用） |

### 4.3 日志相关实体

#### SysOperLog (操作日志)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| operId | Long | 日志主键 |
| title | String | 操作模块 |
| businessType | Integer | 业务类型（0其它 1新增 2修改 3删除） |
| method | String | 请求方法 |
| requestMethod | String | 请求方式 |
| operatorType | Integer | 操作类别（0其它 1后台用户 2手机端用户） |
| operName | String | 操作人员 |
| deptName | String | 部门名称 |
| operUrl | String | 请求URL |
| operIp | String | 主机地址 |
| operLocation | String | 操作地点 |
| operParam | String | 请求参数 |
| jsonResult | String | 返回参数 |
| status | Integer | 操作状态（0正常 1异常） |
| errorMsg | String | 错误消息 |
| operTime | Date | 操作时间 |

#### SysLogininfor (登录日志)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| infoId | Long | 访问ID |
| userName | String | 用户账号 |
| ipaddr | String | 登录IP地址 |
| loginLocation | String | 登录地点 |
| browser | String | 浏览器类型 |
| os | String | 操作系统 |
| status | String | 登录状态（0成功 1失败） |
| msg | String | 提示消息 |
| loginTime | Date | 访问时间 |

### 4.4 任务相关实体

#### SysJob (定时任务)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| jobId | Long | 任务ID |
| jobName | String | 任务名称 |
| jobGroup | String | 任务组名 |
| invokeTarget | String | 调用目标字符串 |
| cronExpression | String | cron执行表达式 |
| misfirePolicy | String | 计划执行错误策略（1立即执行 2执行一次 3放弃执行） |
| concurrent | String | 是否并发执行（0允许 1禁止） |
| status | String | 状态（0正常 1暂停） |

#### SysJobLog (定时任务执行日志)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| jobLogId | Long | 任务日志ID |
| jobName | String | 任务名称 |
| jobGroup | String | 任务组名 |
| invokeTarget | String | 调用目标字符串 |
| jobMessage | String | 日志信息 |
| status | String | 执行状态（0正常 1失败） |
| exceptionInfo | String | 异常信息 |
| startTime | Date | 开始时间 |
| stopTime | Date | 停止时间 |

## 5. 错误码说明

### 5.1 通用错误码

| 错误码 | 说明 |
|--------|------|
| 200 | 操作成功 |
| 500 | 系统异常 |
| 401 | 认证失败 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 400 | 请求参数错误 |

### 5.2 业务错误码

| 错误码 | 说明 |
|--------|------|
| 601 | 用户名或密码错误 |
| 602 | 账号已被锁定 |
| 603 | 验证码错误 |
| 604 | 验证码已过期 |
| 605 | 用户不存在或已被删除 |
| 606 | 用户已被停用 |
| 607 | 当前用户无此操作权限 |

### 5.3 文件上传错误码

| 错误码 | 说明 |
|--------|------|
| 701 | 文件大小超出限制 |
| 702 | 文件类型不支持 |
| 703 | 文件上传失败 |
| 704 | 文件不存在 |

## 6. 接口调用示例

### 6.1 登录获取Token

```bash
curl -X POST "http://localhost:8080/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "admin123"
  }'
```

### 6.2 携带Token访问接口

```bash
curl -X GET "http://localhost:8080/system/user/list" \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..." \
  -H "Content-Type: application/json"
```

### 6.3 文件上传

```bash
curl -X POST "http://localhost:8080/file/upload" \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..." \
  -F "file=@/path/to/your/file.jpg"
```

## 7. 注意事项

1. **认证要求**：除了登录、注册等公开接口外，所有接口都需要在请求头中携带有效的JWT令牌
2. **权限控制**：每个接口都有对应的权限要求，用户必须具备相应权限才能访问
3. **分页查询**：列表查询接口支持分页，通过 `pageNum` 和 `pageSize` 参数控制
4. **数据格式**：所有接口都使用JSON格式进行数据交换
5. **时间格式**：时间字段统一使用 `yyyy-MM-dd HH:mm:ss` 格式
6. **状态码**：接口返回的状态码遵循HTTP标准，业务状态通过响应体中的 `code` 字段表示

## 8. 联系方式

- **项目地址**：https://gitee.com/y_project/RuoYi-Cloud
- **官方文档**：http://doc.ruoyi.vip/ruoyi-cloud/
- **技术交流**：QQ群 1389287
