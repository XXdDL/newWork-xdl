# 智能运力系统 - API设计文档

## 1. RESTful API接口体系概述

智能运力系统采用RESTful API设计风格，提供了完整的派车任务管理、审核流程、状态管理、公司管理和车辆管理等功能接口。系统支持双轨派车流程（轨道A和轨道B），并实现了基于角色的访问控制机制。

## 2. 双轨派车流程

### 2.1 轨道A流程（需要审核）
1. **任务创建**：车间地调或区域调度员创建任务
2. **待调度员审核**：任务进入审核状态
3. **审核通过**：区域调度员审核通过后，任务进入待供应商响应状态
4. **供应商响应**：供应商确认并填写车辆信息
5. **任务完成**：任务执行完成

### 2.2 轨道B流程（直接派车）
1. **任务创建**：区域调度员或超级管理员创建任务
2. **待供应商响应**：任务直接进入待供应商响应状态
3. **供应商响应**：供应商确认并填写车辆信息
4. **任务完成**：任务执行完成

## 3. 状态流转

### 3.1 任务状态列表
- `待调度员审核`: 任务创建后需要区域调度员审核
- `待供应商响应`: 任务审核通过后等待供应商确认
- `供应商已响应`: 供应商确认响应并填写车辆信息
- `任务完成`: 任务执行完成
- `审核拒绝`: 任务审核未通过

### 3.2 状态流转规则
- 车间地调创建的任务只能进入轨道A，初始状态为`待调度员审核`
- 区域调度员创建的任务可选择轨道A或轨道B：
  - 轨道A：初始状态为`待调度员审核`
  - 轨道B：初始状态为`待供应商响应`
- 超级管理员创建的任务可选择轨道A或轨道B：
  - 轨道A：初始状态为`待调度员审核`
  - 轨道B：初始状态为`待供应商响应`

## 4. API接口详细设计

### 4.1 任务管理接口

#### 创建派车任务
- **HTTP方法**: POST
- **路径**: `/api/dispatch/tasks`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "required_time": "2025-08-20T10:00:00",
    "start_location": "站点A",
    "end_location": "站点B",
    "carrier_company": "运输公司A",
    "transport_type": "公路运输",
    "requirement_type": "普通货物",
    "volume": 10.5,
    "weight": 5.0,
    "special_requirements": "轻拿轻放",
    "assigned_supplier_id": "1",
    "dispatch_track": "轨道A"
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "task_id": "T202508201000001",
      "status": "待调度员审核",
      "dispatch_track": "轨道A",
      "current_handler_role": "区域调度员"
    }
  }
  ```
- **业务逻辑**: 根据用户角色和选择的轨道确定任务初始状态和处理流程
  - 车间地调: 强制使用轨道A，初始状态为待调度员审核
  - 区域调度员: 可选择轨道A或B，轨道A初始状态为待调度员审核，轨道B初始状态为待供应商响应
  - 超级管理员: 可选择轨道A或B，轨道A初始状态为待调度员审核，轨道B初始状态为待供应商响应

#### 获取任务列表
- **HTTP方法**: GET
- **路径**: `/api/dispatch/tasks`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`, `供应商`
- **查询参数**: 
  - `page`: 页码，默认1
  - `limit`: 每页数量，默认20
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "list": [
        {
          "task_id": "T202508201000001",
          "required_date": "2025-08-20T10:00:00",
          "start_bureau": "站点A",
          "route_name": "站点B",
          "carrier_company": "运输公司A",
          "transport_type": "公路运输",
          "requirement_type": "普通货物",
          "volume": 10.5,
          "weight": 5.0,
          "status": "待调度员审核",
          "created_at": "2025-08-19T15:30:00",
          "updated_at": "2025-08-19T15:30:00",
          "special_requirements": "轻拿轻放"
        }
      ],
      "total": 1,
      "page": 1,
      "limit": 20
    }
  }
  ```
- **业务逻辑**: 根据用户角色返回不同范围的任务列表
  - 超级管理员/区域调度员: 可以看到所有任务
  - 供应商: 只能看到分配给自己的任务或与自己公司相关的任务
  - 车间地调: 可以看到所有状态为'供应商已响应'的任务

#### 获取任务详情
- **HTTP方法**: GET
- **路径**: `/api/dispatch/tasks/{task_id}`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`, `供应商`
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "task_id": "T202508201000001",
      "required_date": "2025-08-20T10:00:00",
      "start_bureau": "站点A",
      "route_name": "站点B",
      "carrier_company": "运输公司A",
      "transport_type": "公路运输",
      "requirement_type": "普通货物",
      "volume": 10.5,
      "weight": 5.0,
      "status": "待调度员审核",
      "created_at": "2025-08-19T15:30:00",
      "updated_at": "2025-08-19T15:30:00",
      "special_requirements": "轻拿轻放",
      "history": [
        {
          "status": "待调度员审核",
          "timestamp": "2025-08-19T15:30:00",
          "updated_by": "用户张三",
          "notes": "任务创建"
        }
      ]
    }
  }
  ```
- **业务逻辑**: 获取任务的详细信息，包括状态历史记录

#### 更新任务信息
- **HTTP方法**: PUT
- **路径**: `/api/dispatch/tasks/{task_id}`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "title": "紧急运输任务",
    "vehicle_type": "货车",
    "purpose": "原材料运输",
    "start_location": "站点A",
    "end_location": "站点B",
    "expected_start_time": "2025-08-20T10:00:00",
    "expected_end_time": "2025-08-20T16:00:00",
    "passenger_count": 0,
    "cargo_weight": 5.0,
    "cargo_volume": 10.5,
    "special_requirements": "轻拿轻放"
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "任务更新成功"
    }
  }
  ```
- **业务逻辑**: 更新任务的基本信息，不包括状态变更

### 4.2 审核流程接口

#### 提交审核
- **HTTP方法**: POST
- **路径**: `/api/dispatch/tasks/{task_id}/submit-audit`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "notes": "请尽快审核此任务"
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "任务已提交审核",
      "new_status": "待调度员审核"
    }
  }
  ```
- **业务逻辑**: 将任务提交审核，更新任务状态

#### 审核通过
- **HTTP方法**: POST
- **路径**: `/api/dispatch/tasks/{task_id}/approve`
- **权限**: `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "notes": "审核通过，同意派车",
    "assigned_supplier_id": "2"
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "任务审核通过",
      "new_status": "待供应商响应"
    }
  }
  ```
- **业务逻辑**: 审核通过任务，更新任务状态为待供应商响应，可指定分配的供应商

#### 审核拒绝
- **HTTP方法**: POST
- **路径**: `/api/dispatch/tasks/{task_id}/reject`
- **权限**: `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "notes": "任务信息不完整，需要补充"
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "任务审核拒绝",
      "new_status": "审核拒绝"
    }
  }
  ```
- **业务逻辑**: 拒绝任务审核，更新任务状态为审核拒绝

### 4.3 状态管理接口

#### 更新任务状态
- **HTTP方法**: PUT
- **路径**: `/api/dispatch/tasks/{task_id}/status`
- **权限**: `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "new_status": "任务完成",
    "notes": "任务已成功完成"
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "任务状态已更新",
      "new_status": "任务完成"
    }
  }
  ```
- **业务逻辑**: 更新任务状态，记录状态变更历史

#### 获取状态历史
- **HTTP方法**: GET
- **路径**: `/api/dispatch/tasks/{task_id}/history`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`, `供应商`
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": [
      {
        "status": "待调度员审核",
        "timestamp": "2025-08-19T15:30:00",
        "updated_by": "用户张三",
        "notes": "任务创建"
      },
      {
        "status": "待供应商响应",
        "timestamp": "2025-08-19T16:45:00",
        "updated_by": "用户李四",
        "notes": "审核通过"
      }
    ]
  }
  ```
- **业务逻辑**: 获取任务的完整状态变更历史

### 4.4 供应商响应接口

#### 供应商确认响应
- **HTTP方法**: POST
- **路径**: `/api/dispatch/tasks/{task_id}/confirm`
- **权限**: `供应商`
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "供应商响应成功",
      "task_id": "T202508201000001",
      "new_status": "供应商已响应"
    }
  }
  ```
- **业务逻辑**: 供应商确认响应任务，更新任务状态为供应商已响应
- **数据验证**: 
  - 任务必须存在
  - 任务状态必须为待供应商响应
  - 该任务不能已存在车辆信息

#### 供应商确认并填写车辆信息
- **HTTP方法**: POST
- **路径**: `/api/dispatch/tasks/{task_id}/confirm-with-vehicle`
- **权限**: `供应商`
- **请求参数**: JSON格式
  ```json
  {
    "vehicle_number": "京A12345",
    "vehicle_type": "货车",
    "driver_name": "王五",
    "driver_phone": "13800138000",
    "capacity_volume": 15.0,
    "capacity_weight": 8.0
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "供应商响应并填写车辆信息成功",
      "task_id": "T202508201000001",
      "new_status": "供应商已响应",
      "vehicle_id": "1"
    }
  }
  ```
- **业务逻辑**: 供应商确认响应任务并填写车辆信息，更新任务状态为供应商已响应
- **数据验证**: 
  - 任务必须存在
  - 任务状态必须为待供应商响应
  - 车辆信息必须完整

### 4.5 公司管理接口

#### 获取公司列表
- **HTTP方法**: GET
- **路径**: `/api/company/list`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`
- **查询参数**: 
  - `is_supplier`: 是否为供应商，可选
  - `page`: 页码，默认1
  - `limit`: 每页数量，默认20
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "list": [
        {
          "id": "1",
          "name": "运输公司A",
          "contact_person": "张三",
          "contact_phone": "13800138000",
          "address": "北京市朝阳区",
          "is_supplier": true
        }
      ],
      "total": 1,
      "page": 1,
      "limit": 20
    }
  }
  ```
- **业务逻辑**: 获取公司列表，可筛选是否为供应商

#### 添加公司
- **HTTP方法**: POST
- **路径**: `/api/company`
- **权限**: `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "name": "运输公司B",
    "contact_person": "李四",
    "contact_phone": "13900139000",
    "address": "上海市浦东新区",
    "is_supplier": true
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "公司添加成功",
      "company_id": "2"
    }
  }
  ```
- **业务逻辑**: 添加新的公司信息
- **数据验证**: 
  - 公司名称不能为空
  - 联系电话格式必须正确

#### 获取公司ID
- **HTTP方法**: GET
- **路径**: `/api/company/id`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`
- **查询参数**: 
  - `name`: 公司名称
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "company_id": "1"
    }
  }
  ```
- **业务逻辑**: 根据公司名称查询公司ID

### 4.6 车辆管理接口

#### 获取车辆容积参考数据
- **HTTP方法**: GET
- **路径**: `/api/vehicle/volume-reference`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": [
      {
        "id": "1",
        "vehicle_type": "小型货车",
        "min_volume": 5.0,
        "max_volume": 10.0,
        "avg_volume": 7.5
      },
      {
        "id": "2",
        "vehicle_type": "中型货车",
        "min_volume": 10.0,
        "max_volume": 20.0,
        "avg_volume": 15.0
      }
    ]
  }
  ```
- **业务逻辑**: 获取车辆类型对应的容积参考数据

#### 更新车辆容积参考数据
- **HTTP方法**: PUT
- **路径**: `/api/vehicle/volume-reference/{id}`
- **权限**: `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "min_volume": 6.0,
    "max_volume": 12.0,
    "avg_volume": 9.0
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "车辆容积参考数据更新成功"
    }
  }
  ```
- **业务逻辑**: 更新特定车辆类型的容积参考数据
- **数据验证**: 
  - 容积数据必须为正数
  - min_volume <= avg_volume <= max_volume

#### 插入车辆容积参考数据
- **HTTP方法**: POST
- **路径**: `/api/vehicle/volume-reference`
- **权限**: `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "vehicle_type": "大型货车",
    "min_volume": 20.0,
    "max_volume": 35.0,
    "avg_volume": 27.5
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "车辆容积参考数据插入成功",
      "id": "3"
    }
  }
  ```
- **业务逻辑**: 插入新的车辆类型容积参考数据
- **数据验证**: 
  - 车辆类型不能为空
  - 容积数据必须为正数
  - min_volume <= avg_volume <= max_volume

#### 删除车辆容积参考数据
- **HTTP方法**: DELETE
- **路径**: `/api/vehicle/volume-reference/{id}`
- **权限**: `区域调度员`, `超级管理员`
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "车辆容积参考数据删除成功"
    }
  }
  ```
- **业务逻辑**: 删除特定的车辆容积参考数据

#### 批量导入车辆容积参考数据
- **HTTP方法**: POST
- **路径**: `/api/vehicle/volume-reference/batch-import`
- **权限**: `区域调度员`, `超级管理员`
- **请求参数**: JSON格式
  ```json
  {
    "data": [
      {
        "vehicle_type": "小型货车",
        "min_volume": 5.0,
        "max_volume": 10.0,
        "avg_volume": 7.5
      },
      {
        "vehicle_type": "中型货车",
        "min_volume": 10.0,
        "max_volume": 20.0,
        "avg_volume": 15.0
      }
    ]
  }
  ```
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "message": "车辆容积参考数据批量导入成功",
      "imported_count": 2,
      "failed_count": 0
    }
  }
  ```
- **业务逻辑**: 批量导入车辆容积参考数据
- **数据验证**: 
  - 每个数据项必须符合单个插入的验证规则

#### 车辆信息搜索
- **HTTP方法**: GET
- **路径**: `/api/vehicle/search`
- **权限**: `车间地调`, `区域调度员`, `超级管理员`
- **查询参数**: 
  - `vehicle_number`: 车牌号（模糊搜索）
  - `driver_name`: 司机姓名（模糊搜索）
  - `company_id`: 所属公司ID
  - `page`: 页码，默认1
  - `limit`: 每页数量，默认20
- **响应示例**: 
  ```json
  {
    "success": true,
    "data": {
      "list": [
        {
          "id": "1",
          "vehicle_number": "京A12345",
          "vehicle_type": "货车",
          "driver_name": "王五",
          "driver_phone": "13800138000",
          "capacity_volume": 15.0,
          "capacity_weight": 8.0,
          "company_id": "1",
          "company_name": "运输公司A"
        }
      ],
      "total": 1,
      "page": 1,
      "limit": 20
    }
  }
  ```
- **业务逻辑**: 根据条件搜索车辆信息

## 5. API实现状态

| 接口类别 | 接口名称 | 实现状态 | 备注 |
|----------|----------|----------|------|
| 任务管理 | 创建派车任务 | 已实现 | 支持双轨派车流程 |
| 任务管理 | 获取任务列表 | 已实现 | 支持分页和角色过滤 |
| 任务管理 | 获取任务详情 | 已实现 | 包含状态历史 |
| 任务管理 | 更新任务信息 | 已实现 | 支持更新任务基本信息 |
| 审核流程 | 提交审核 | 待实现 | |
| 审核流程 | 审核通过 | 待实现 | |
| 审核流程 | 审核拒绝 | 待实现 | |
| 状态管理 | 更新任务状态 | 待实现 | |
| 状态管理 | 获取状态历史 | 已实现 | 包含在任务详情接口中 |
| 供应商响应 | 供应商确认响应 | 已实现 | 支持基本确认 |
| 供应商响应 | 供应商确认并填写车辆信息 | 部分实现 | 支持填写车辆信息 |
| 公司管理 | 获取公司列表 | 待实现 | |
| 公司管理 | 添加公司 | 待实现 | |
| 公司管理 | 获取公司ID | 待实现 | |
| 车辆管理 | 获取车辆容积参考数据 | 待实现 | |
| 车辆管理 | 更新车辆容积参考数据 | 待实现 | |
| 车辆管理 | 插入车辆容积参考数据 | 待实现 | |
| 车辆管理 | 删除车辆容积参考数据 | 待实现 | |
| 车辆管理 | 批量导入车辆容积参考数据 | 待实现 | |
| 车辆管理 | 车辆信息搜索 | 待实现 | |

## 6. API错误处理

### 6.1 错误类型
- **APIError**: 通用API错误
- **ValidationError**: 数据验证错误
- **PermissionError**: 权限错误
- **NotFoundError**: 资源不存在错误

### 6.2 错误响应格式
```json
{
  "success": false,
  "error": {
    "code": 4001,
    "message": "数据验证失败: 必填字段缺失"
  }
}
```

### 6.3 常见错误代码
| 错误代码 | 错误消息 | 说明 |
|----------|----------|------|
| 4001 | 数据验证失败 | 请求参数不符合要求 |
| 4002 | 无权限执行此操作 | 用户角色不满足权限要求 |
| 4041 | 任务不存在 | 找不到指定的任务 |
| 4042 | 任务状态不正确 | 当前任务状态不允许执行此操作 |
| 4043 | 该任务已存在车辆信息 | 任务已分配车辆，不能重复操作 |
| 5001 | 内部服务器错误 | 服务器处理请求时发生错误 |