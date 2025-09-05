# ARCHITECTURE_SMART_TRANSPORT_SYSTEM

## 1. 系统架构概述

### 1.1 整体架构设计（一期开发重点）
智能运力系统采用前后端分离的微服务架构，遵循分层架构原则，确保各层职责清晰、耦合度低。

**一期开发重点**:
- **用户管理模块**: 完整的用户认证、权限控制和用户信息管理
- **基础数据模块**: 车辆信息和公司信息的基础数据管理
- **调度管理模块**: 派车任务管理、审核流程和调度功能
- **核心架构**: 认证中间件、权限控制、数据模型和服务层

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           表现层 (Presentation Layer)                    │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │   Vue.js 3      │    │  路由管理       │    │  状态管理       │    │
│  │   TypeScript    │    │  Vue Router     │    │  Pinia          │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼ HTTP/HTTPS (RESTful API)
┌─────────────────────────────────────────────────────────────────────────┐
│                           应用层 (Application Layer)                     │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │   Flask RESTful  │    │  业务逻辑       │    │  数据验证       │    │
│  │   Blueprint      │    │  控制器         │    │  序列化         │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼ 服务调用
┌─────────────────────────────────────────────────────────────────────────┐
│                           服务层 (Service Layer)                        │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │  任务服务       │    │  用户服务       │    │  审核服务       │    │
│  │  TaskService    │    │  UserService    │    │  AuditService   │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │  车辆服务       │    │  公司服务       │    │  权限服务       │    │
│  │  VehicleService │    │  CompanyService │    │  AuthService    │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │  运费服务       │    │  核算服务       │    │  飞书服务       │    │
│  │  FreightService │    │  AccountingService│  │  FeishuService  │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │  利润服务       │    │  预测服务       │    │  优化服务       │    │
│  │  ProfitService  │    │  PredictService │    │  OptimizeService│    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼ 数据访问
┌─────────────────────────────────────────────────────────────────────────┐
│                           数据层 (Data Layer)                           │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │  SQLAlchemy     │    │  数据库适配器    │    │  数据模型       │    │
│  │  ORM            │    │  DB Adapter     │    │  Models         │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           存储层 (Storage Layer)                        │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │  SQLite         │    │  PostgreSQL     │    │  Redis          │    │
│  │  (开发环境)     │    │  (生产环境)     │    │  (缓存/会话)    │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 技术栈详细配置

#### 前端技术栈
- **框架**: Vue 3.3 + Composition API + TypeScript
- **路由**: Vue Router 4
- **状态管理**: Pinia
- **UI组件**: Element Plus
- **数据可视化**: VChart
- **构建工具**: Vite + Rollup

#### VChart 数据可视化配置
VChart 是 VisActor 可视化体系中的图表组件库，提供丰富的图表类型和交互能力：

**主要特性**：
- 支持多种图表类型：折线图、柱状图、饼图、散点图、雷达图等
- 响应式设计，支持移动端和桌面端
- 丰富的交互功能：数据筛选、缩放、拖拽等
- TypeScript 全面支持
- 与 Vue 3 深度集成

**安装配置**：
```bash
# 安装 VChart
npm install @visactor/vchart
```

**Vue 组件使用示例**：
```vue
<template>
  <VChart :option="chartOption" :autoresize="true" />
</template>

<script setup>
import VChart from '@visactor/vchart'
import { ref } from 'vue'

const chartOption = ref({
  title: {
    text: '任务统计图表'
  },
  tooltip: {
    trigger: 'axis'
  },
  xAxis: {
    type: 'category',
    data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
  },
  yAxis: {
    type: 'value'
  },
  series: [{
    data: [120, 200, 150, 80, 70, 110, 130],
    type: 'bar'
  }]
})
</script>
```

**图表类型应用场景**：
- **折线图**: 任务完成趋势分析
- **柱状图**: 各类型任务数量统计
- **饼图**: 任务状态分布比例
- **雷达图**: 多维度任务指标对比
- **HTTP客户端**: Axios
- **代码规范**: ESLint + Prettier
- **测试**: Vitest + Vue Test Utils

#### 后端技术栈
- **框架**: Flask 2.3 + Flask-RESTful
- **ORM**: SQLAlchemy 2.0
- **数据库**: SQLite (开发), PostgreSQL (生产)
- **认证**: JWT + Flask-Login
- **序列化**: Marshmallow
- **缓存**: Redis (可选)
- **任务队列**: Celery (可选)
- **测试**: Pytest + Factory Boy
- **文档**: Swagger/OpenAPI

## 2. 系统模块设计

### 2.1 前端模块结构（一期开发核心）
```
src/
├── components/           # 公共组件
│   ├── layout/          # 布局组件
│   ├── form/            # 表单组件
│   ├── table/           # 表格组件
│   ├── modal/           # 弹窗组件
│   └── charts/          # 图表组件
├── views/               # 页面视图（一期核心模块）
│   ├── dashboard/       # 仪表板
│   ├── basic-data/      # 基础数据管理（一期核心）
│   │   ├── vehicle/     # 车辆管理
│   │   ├── company/     # 公司管理
│   │   └── user/        # 用户管理
│   ├── dispatch/        # 调度管理（一期核心）
│   │   ├── task/        # 任务管理
│   │   └── audit/       # 审核管理
│   └── auth/            # 认证相关
├── stores/              # Pinia状态管理（一期核心）
│   ├── auth.store.ts    # 认证状态
│   ├── task.store.ts    # 任务状态
│   ├── user.store.ts    # 用户状态
│   ├── basic-data.store.ts # 基础数据状态
│   └── dispatch.store.ts # 调度状态
├── services/            # API服务（一期核心）
│   ├── api.client.ts    # HTTP客户端
│   ├── task.service.ts  # 任务服务
│   ├── auth.service.ts  # 认证服务
│   ├── basic-data.service.ts # 基础数据服务
│   ├── dispatch.service.ts # 调度服务
│   └── types/           # TypeScript类型定义
├── router/              # 路由配置
├── utils/               # 工具函数
└── assets/              # 静态资源
```

### 2.2 后端模块结构（一期开发核心模块）
```
backend/
├── app/
│   ├── __init__.py     # 应用初始化
│   ├── auth/           # 认证模块（一期核心）
│   │   ├── __init__.py
│   │   ├── models.py    # 用户模型
│   │   ├── services.py  # 用户服务
│   │   ├── api.py       # 认证API
│   │   └── schemas.py   # 序列化
│   ├── basic_data/     # 基础数据模块（一期核心）
│   │   ├── __init__.py
│   │   ├── models.py    # 车辆、公司模型
│   │   ├── services.py  # 基础数据服务
│   │   ├── api.py       # 基础数据API
│   │   └── schemas.py   # 序列化
│   ├── dispatch/       # 调度管理模块（一期核心）
│   │   ├── __init__.py
│   │   ├── models.py    # 任务模型
│   │   ├── services.py  # 任务服务
│   │   ├── api.py       # 调度API
│   │   └── schemas.py   # 序列化
│   ├── common/         # 公共模块
│   │   ├── __init__.py
│   │   ├── database.py  # 数据库配置
│   │   ├── utils.py     # 工具函数
│   │   └── exceptions.py # 异常处理
│   └── middleware/     # 中间件
│       ├── __init__.py
│       ├── auth.py      # 认证中间件
│       └── permission.py # 权限中间件
│   ├── config.py       # 配置文件
│   ├── extensions.py    # 扩展初始化
│   ├── models/         # 数据模型
│   │   ├── user.py     # 用户模型
│   │   ├── task.py     # 任务模型
│   │   ├── vehicle.py  # 车辆模型
│   │   ├── company.py  # 公司模型
│   │   └── audit.py    # 审核模型
│   ├── services/       # 服务层（按功能模块组织）
│   │   ├── auth_service.py      # 认证服务
│   │   ├── basic_data_service.py # 基础数据服务
│   │   │   ├── user_service.py   # 用户服务
│   │   │   ├── vehicle_service.py # 车辆服务
│   │   │   └── company_service.py # 公司服务
│   │   ├── plan_service.py      # 计划管理服务
│   │   │   ├── task_service.py   # 任务服务
│   │   │   └── audit_service.py  # 审核服务
│   │   ├── cost_service.py      # 成本分析服务
│   │   │   ├── freight_service.py # 运费服务
│   │   │   ├── accounting_service.py # 核算服务
│   │   │   └── report_service.py  # 报表服务
│   │   ├── reconciliation_service.py # 对账管理服务
│   │   │   ├── settlement_service.py # 结算单服务
│   │   │   └── bill_service.py    # 账单服务
│   │   ├── dispatch_service.py   # 调度管理服务
│   │   │   ├── realtime_service.py # 实时调度服务
│   │   │   └── history_service.py  # 调度历史服务
│   │   └── system_service.py     # 系统管理服务
│   │       ├── config_service.py  # 配置服务
│   │       ├── log_service.py    # 日志服务
│   │       └── monitor_service.py # 监控服务
│   ├── api/            # API接口（按功能模块组织）
│   │   ├── v1/         # API版本1
│   │   │   ├── __init__.py
│   │   │   ├── auth.py          # 认证API
│   │   │   ├── basic_data.py    # 基础数据API
│   │   │   │   ├── users.py     # 用户管理API
│   │   │   │   ├── vehicles.py  # 车辆管理API
│   │   │   │   └── companies.py # 公司管理API
│   │   │   ├── plan.py          # 计划管理API
│   │   │   │   ├── tasks.py     # 任务管理API
│   │   │   │   └── audits.py    # 审核管理API
│   │   │   ├── cost.py          # 成本分析API
│   │   │   │   ├── freight.py   # 运费计算API
│   │   │   │   ├── accounting.py # 单车核算API
│   │   │   │   └── reports.py   # 报表分析API
│   │   │   ├── reconciliation.py # 对账管理API
│   │   │   │   ├── settlement.py # 结算单API
│   │   │   │   └── bills.py      # 账单管理API
│   │   │   ├── dispatch.py       # 调度管理API
│   │   │   │   ├── realtime.py   # 实时调度API
│   │   │   │   └── history.py    # 调度历史API
│   │   │   ├── system.py        # 系统管理API
│   │   │   │   ├── config.py    # 系统配置API
│   │   │   │   ├── logs.py      # 操作日志API
│   │   │   │   └── monitor.py   # 系统监控API
│   │   │   └── predict.py       # 智能预测API（暂不实现）
│   │   └── __init__.py
│   ├── utils/          # 工具函数
│   │   ├── decorators.py # 装饰器
│   │   ├── validators.py # 数据验证
│   │   ├── pagination.py # 分页工具
│   │   └── response.py # 响应格式化
│   ├── middleware/     # 中间件
│   └── tasks/          # 后台任务
├── migrations/         # 数据库迁移
├── tests/              # 测试代码
│   ├── unit/          # 单元测试
│   ├── integration/   # 集成测试
│   └── fixtures/      # 测试数据
├── requirements.txt   # Python依赖
└── run.py             # 启动脚本
```

## 3. 数据模型设计

### 3.1 核心实体关系
```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│    User     │      │   Task      │      │  Vehicle    │
├─────────────┤      ├─────────────┤      ├─────────────┤
│ - id        │◆──┐  │ - id        │◆──┐  │ - id        │
│ - username  │   │  │ - title     │   │  │ - plate_no  │
│ - email     │   │  │ - status    │   │  │ - type      │
│ - password  │   │  │ - track     │   │  │ - capacity   │
│ - company_id│   └──│ - creator_id│   └──│ - task_id   │
└─────────────┘      │ - auditor_id│      └─────────────┘
         │           │ - supplier_id│
         │           └─────────────┘
         │                  │
         │                  │
┌─────────────┐      ┌─────────────┐
│  Company    │      │ AuditRecord │
├─────────────┤      ├─────────────┤
│ - id        │      │ - id        │
│ - name      │      │ - task_id   │
│ - type      │      │ - action    │
│ - contact   │      │ - user_id   │
│ - address   │      │ - timestamp │
└─────────────┘      │ - notes     │
                     └─────────────┘
```

### 3.2 详细数据模型设计

#### User 用户模型
```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)
    full_name = db.Column(db.String(100))
    phone = db.Column(db.String(20))
    company_id = db.Column(db.Integer, db.ForeignKey('company.id'))
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # 关系
    company = db.relationship('Company', backref=db.backref('users', lazy=True))
    roles = db.relationship('Role', secondary='user_role', backref=db.backref('users', lazy=True))
    created_tasks = db.relationship('Task', foreign_keys='Task.creator_id', backref='creator')
    audited_tasks = db.relationship('Task', foreign_keys='Task.auditor_id', backref='auditor')
```

#### Task 任务模型
```python
class Task(db.Model):
    id = db.Column(db.String(20), primary_key=True)  # 任务唯一ID
    required_date = db.Column(db.String(100))  # 需求日期
    start_bureau = db.Column(db.String(100))  # 起始站段
    route_direction = db.Column(db.String(100))  # 路线方向
    carrier_company = db.Column(db.String(100))  # 运输公司
    route_name = db.Column(db.String(100))  # 路线名称
    transport_type = db.Column(db.String(50))  # 运输类型
    requirement_type = db.Column(db.String(50))  # 需求类型
    volume = db.Column(db.Float)  # 需求容积
    weight = db.Column(db.Float)  # 需求重量
    special_requirements = db.Column(db.Text)  # 特殊要求
    status = db.Column(db.String(50))  # 任务状态
    dispatch_track = db.Column(db.String(10))  # 派车轨道
    initiator_role = db.Column(db.String(50))  # 发起人角色
    initiator_user_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 发起人用户ID
    initiator_department = db.Column(db.String(100))  # 发起人部门
    audit_required = db.Column(db.Boolean)  # 是否需要审核
    auditor_role = db.Column(db.String(50))  # 审核人角色
    auditor_user_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 审核人用户ID
    audit_status = db.Column(db.String(50))  # 审核状态
    audit_time = db.Column(db.String(100))  # 审核时间
    audit_note = db.Column(db.Text)  # 审核备注
    current_handler_role = db.Column(db.String(50))  # 当前处理人角色
    current_handler_user_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 当前处理人用户ID
    created_at = db.Column(db.String(100))  # 创建时间
    updated_at = db.Column(db.String(100))  # 更新时间
    assigned_supplier_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 分配供应商ID
    
    # 关系
    initiator = db.relationship('User', foreign_keys=[initiator_user_id])
    auditor = db.relationship('User', foreign_keys=[auditor_user_id])
    current_handler = db.relationship('User', foreign_keys=[current_handler_user_id])
    assigned_supplier = db.relationship('User', foreign_keys=[assigned_supplier_id])
    vehicles = db.relationship('Vehicle', backref='task', lazy=True)
    audit_records = db.relationship('AuditRecord', backref='task', lazy=True)
```

#### Vehicle 车辆模型
```python
class Vehicle(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    task_id = db.Column(db.String(20), db.ForeignKey('task.id'))  # 关联任务ID
    manifest_number = db.Column(db.String(100))  # 货票号
    dispatch_number = db.Column(db.String(100))  # 派车单号
    license_plate = db.Column(db.String(20))  # 车牌号
    carriage_number = db.Column(db.String(100))  # 车皮号
    created_at = db.Column(db.String(100))  # 创建时间
    notes = db.Column(db.Text)  # 备注
    actual_volume = db.Column(db.Float)  # 实际容积
    volume_photo_url = db.Column(db.String(200))  # 容积照片URL
    volume_modified_by = db.Column(db.Integer, db.ForeignKey('user.id'))  # 容积修改人
    required_volume = db.Column(db.Float)  # 需求容积
    confirmed_volume = db.Column(db.Float)  # 确认容积
    
    # 关系
    volume_modifier = db.relationship('User', foreign_keys=[volume_modified_by])
```

#### Company 公司模型
```python
class Company(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False, unique=True)
    type = db.Column(db.String(20), nullable=False)  # 承运商、供应商等
    contact_person = db.Column(db.String(50))
    contact_phone = db.Column(db.String(20))
    address = db.Column(db.String(200))
    bank_name = db.Column(db.String(100))
    account_number = db.Column(db.String(50))
    
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

#### AuditRecord 审核记录模型
```python
class AuditRecord(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    task_id = db.Column(db.String(20), db.ForeignKey('task.id'))  # 关联任务ID
    status_change = db.Column(db.String(100))  # 状态变更
    operator = db.Column(db.String(100))  # 操作人
    timestamp = db.Column(db.String(100))  # 时间戳
    note = db.Column(db.Text)  # 备注
    
    # 关系
    task = db.relationship('Task', backref=db.backref('status_history', lazy=True))
```

## 4. API接口设计

### 4.1 RESTful API规范
- **版本控制**: `/api/v1/`
- **资源命名**: 使用复数名词（如`/tasks`, `/users`）
- **HTTP方法**: 
  - GET: 获取资源
  - POST: 创建资源
  - PUT: 更新资源
  - DELETE: 删除资源
  - PATCH: 部分更新资源

### 4.2 核心API端点（按功能模块组织）

#### 4.2.1 认证模块
- `POST /api/v1/auth/login` - 用户登录
- `POST /api/v1/auth/logout` - 用户登出
- `POST /api/v1/auth/refresh` - 刷新Token
- `GET /api/v1/auth/me` - 获取当前用户信息

#### 4.2.2 基础数据管理模块
**用户管理**
- `GET /api/v1/users` - 获取用户列表
- `POST /api/v1/users` - 创建用户
- `GET /api/v1/users/{id}` - 获取用户详情
- `PUT /api/v1/users/{id}` - 更新用户信息
- `DELETE /api/v1/users/{id}` - 删除用户

**车辆管理**
- `GET /api/v1/vehicles` - 获取车辆列表
- `POST /api/v1/vehicles` - 创建车辆
- `GET /api/v1/vehicles/{id}` - 获取车辆详情
- `PUT /api/v1/vehicles/{id}` - 更新车辆信息
- `DELETE /api/v1/vehicles/{id}` - 删除车辆
- `GET /api/v1/vehicles/types` - 获取车型分类（5吨、8吨、12吨、20吨、30吨、40吨A、40吨B）

**公司管理**
- `GET /api/v1/companies` - 获取公司列表
- `POST /api/v1/companies` - 创建公司
- `GET /api/v1/companies/{id}` - 获取公司详情
- `PUT /api/v1/companies/{id}` - 更新公司信息
- `DELETE /api/v1/companies/{id}` - 删除公司

**邮路管理**
- `GET /api/v1/routes` - 获取邮路列表
- `POST /api/v1/routes` - 创建邮路
- `GET /api/v1/routes/{id}` - 获取邮路详情
- `PUT /api/v1/routes/{id}` - 更新邮路信息
- `DELETE /api/v1/routes/{id}` - 删除邮路
- `GET /api/v1/routes/types` - 获取路向类型列表

**参数设置**
- `GET /api/v1/settings/fuel-price` - 获取油价设置
- `PUT /api/v1/settings/fuel-price` - 更新油价设置
- `GET /api/v1/settings/labor-price` - 获取人工价格设置
- `PUT /api/v1/settings/labor-price` - 更新人工价格设置
- `GET /api/v1/settings/toll-fee` - 获取路桥费设置
- `PUT /api/v1/settings/toll-fee` - 更新路桥费设置
- `GET /api/v1/settings/maintenance` - 获取维修费设置
- `PUT /api/v1/settings/maintenance` - 更新维修费设置
- `GET /api/v1/settings/tyre-cost` - 获取轮胎费设置
- `PUT /api/v1/settings/tyre-cost` - 更新轮胎费设置
- `GET /api/v1/settings/insurance` - 获取保险费设置
- `PUT /api/v1/settings/insurance` - 更新保险费设置
- `GET /api/v1/settings/depreciation` - 获取折旧费设置
- `PUT /api/v1/settings/depreciation` - 更新折旧费设置
- `GET /api/v1/settings/fixed-cost` - 获取其他固定费用设置
- `PUT /api/v1/settings/fixed-cost` - 更新其他固定费用设置
- `GET /api/v1/settings/management-fee` - 获取管理费设置
- `PUT /api/v1/settings/management-fee` - 更新管理费设置

#### 4.2.3 计划管理模块
**任务管理**
- `GET /api/v1/tasks` - 获取任务列表（支持分页、过滤、排序）
- `POST /api/v1/tasks` - 创建新任务
- `GET /api/v1/tasks/{id}` - 获取任务详情
- `PUT /api/v1/tasks/{id}` - 更新任务信息
- `DELETE /api/v1/tasks/{id}` - 删除任务
- `POST /api/v1/tasks/{id}/submit-audit` - 提交审核
- `POST /api/v1/tasks/{id}/approve` - 审核通过
- `POST /api/v1/tasks/{id}/reject` - 审核拒绝
- `POST /api/v1/tasks/{id}/confirm` - 供应商确认响应
- `POST /api/v1/tasks/{id}/complete` - 标记任务完成

**审核管理**
- `GET /api/v1/audit-records` - 获取审核记录列表
- `GET /api/v1/tasks/{id}/audit-records` - 获取任务审核记录

#### 4.2.4 成本分析模块
**运费计算**
- `POST /api/v1/freight/calculate` - 自动计算运费
- `GET /api/v1/freight/rules` - 获取运费计算规则
- `PUT /api/v1/freight/rules` - 更新运费计算规则

**单车核算**
- `GET /api/v1/accounting/daily-report` - 生成日报总结
- `GET /api/v1/accounting/vehicle-cost` - 单车成本核算
- `POST /api/v1/accounting/warning` - 预警信息推送

**报表分析**
- `GET /api/v1/reports/freight` - 运费统计分析
- `GET /api/v1/reports/vehicle-utilization` - 车辆利用率分析
- `GET /api/v1/reports/company-performance` - 公司绩效分析

#### 4.2.5 对账管理模块
**结算单管理**
- `POST /api/v1/settlement/generate` - 生成电子结算单
- `GET /api/v1/settlement/{id}` - 获取结算单详情
- `POST /api/v1/settlement/{id}/confirm` - 供应商确认结算单
- `POST /api/v1/settlement/{id}/sign` - 电子签名确认

**账单管理**
- `GET /api/v1/bills` - 获取账单列表
- `GET /api/v1/bills/{id}` - 获取账单详情
- `POST /api/v1/bills/{id}/verify` - 账单核验
- `POST /api/v1/bills/{id}/export` - 导出账单

#### 4.2.6 调度管理模块
**实时调度**
- `GET /api/v1/dispatch/realtime-tasks` - 获取实时任务列表
- `POST /api/v1/dispatch/assign-vehicle` - 分配车辆
- `POST /api/v1/dispatch/update-status` - 更新调度状态
- `GET /api/v1/dispatch/vehicle-locations` - 获取车辆位置

**调度历史**
- `GET /api/v1/dispatch/history` - 获取调度历史记录
- `GET /api/v1/dispatch/statistics` - 调度统计分析
- `POST /api/v1/dispatch/export-history` - 导出调度历史

#### 4.2.7 系统管理模块
**系统配置**
- `GET /api/v1/system/config` - 获取系统配置
- `PUT /api/v1/system/config` - 更新系统配置
- `GET /api/v1/system/params` - 获取系统参数
- `PUT /api/v1/system/params` - 更新系统参数

**操作日志**
- `GET /api/v1/system/logs` - 获取操作日志
- `GET /api/v1/system/logs/{id}` - 获取日志详情
- `POST /api/v1/system/logs/export` - 导出操作日志

**系统监控**
- `GET /api/v1/system/monitor/status` - 获取系统状态
- `GET /api/v1/system/monitor/performance` - 获取性能指标
- `GET /api/v1/system/monitor/database` - 获取数据库状态

#### 4.2.8 智能预测模块（暂不实现）
- `POST /api/v1/predict/demand` - 预测各路向车型需求
- `GET /api/v1/predict/history` - 获取历史预测数据
- `POST /api/v1/predict/warning` - 发车数预警提醒

## 5. 权限控制设计

### 5.1 基于角色的访问控制（RBAC）
```python
# 角色定义
ROLES = {
    'super_admin': '超级管理员',
    'area_dispatcher': '区域调度员', 
    'workshop_dispatcher': '车间地调',
    'supplier': '供应商'
}

# 权限矩阵（按功能模块组织）
PERMISSIONS = {
    # 基础数据管理权限
'basic_data:user:manage': ['super_admin'],
'basic_data:user:view': ['super_admin', 'area_dispatcher'],
'basic_data:vehicle:manage': ['super_admin', 'area_dispatcher'],
'basic_data:vehicle:view': ['super_admin', 'area_dispatcher', 'workshop_dispatcher'],
'basic_data:company:manage': ['super_admin', 'area_dispatcher'],
'basic_data:company:view': ['super_admin', 'area_dispatcher', 'workshop_dispatcher', 'supplier'],
'basic_data:route:manage': ['super_admin', 'area_dispatcher'],
'basic_data:route:view': ['super_admin', 'area_dispatcher', 'workshop_dispatcher'],
'basic_data:settings:manage': ['super_admin', 'area_dispatcher'],
'basic_data:settings:view': ['super_admin', 'area_dispatcher']
    
    # 计划管理权限
    'plan:task:create': ['super_admin', 'area_dispatcher', 'workshop_dispatcher'],
    'plan:task:view_all': ['super_admin', 'area_dispatcher'],
    'plan:task:view_own': ['workshop_dispatcher', 'supplier'],
    'plan:task:update': ['super_admin', 'area_dispatcher'],
    'plan:task:delete': ['super_admin'],
    'plan:audit:approve': ['super_admin', 'area_dispatcher'],
    'plan:audit:reject': ['super_admin', 'area_dispatcher'],
    
    # 成本分析权限
'cost:freight:calculate': ['super_admin', 'area_dispatcher', 'workshop_dispatcher'],
'cost:freight:manage_rules': ['super_admin', 'area_dispatcher'],
'cost:accounting:view': ['super_admin', 'area_dispatcher', 'financial_staff'],
'cost:report:view': ['super_admin', 'area_dispatcher', 'financial_staff'],
'cost:report:export': ['super_admin', 'area_dispatcher'],
'cost:analysis:view': ['super_admin', 'area_dispatcher', 'financial_staff'],
'cost:analysis:export': ['super_admin', 'area_dispatcher']
    
    # 对账管理权限
    'reconciliation:settlement:generate': ['super_admin', 'area_dispatcher'],
    'reconciliation:settlement:confirm': ['supplier'],
    'reconciliation:settlement:sign': ['super_admin', 'area_dispatcher', 'financial_staff'],
    'reconciliation:bill:view': ['super_admin', 'area_dispatcher', 'financial_staff'],
    'reconciliation:bill:verify': ['super_admin', 'area_dispatcher'],
    'reconciliation:bill:export': ['super_admin', 'area_dispatcher'],
    
    # 调度管理权限
    'dispatch:realtime:view': ['super_admin', 'area_dispatcher'],
    'dispatch:realtime:manage': ['super_admin', 'area_dispatcher'],
    'dispatch:history:view': ['super_admin', 'area_dispatcher'],
    'dispatch:history:export': ['super_admin', 'area_dispatcher'],
    
    # 系统管理权限
    'system:config:manage': ['super_admin'],
    'system:config:view': ['super_admin', 'area_dispatcher'],
    'system:log:view': ['super_admin'],
    'system:log:export': ['super_admin'],
    'system:monitor:view': ['super_admin'],
    
    # 智能预测权限
'predict:view': ['super_admin', 'area_dispatcher'],
'predict:manage': ['super_admin', 'area_dispatcher'],
'dispatch:benchmark:manage': ['super_admin', 'area_dispatcher'],
'dispatch:benchmark:view': ['super_admin', 'area_dispatcher'],
'dispatch:demand:calculate': ['super_admin', 'area_dispatcher'],
'dispatch:response:confirm': ['super_admin', 'area_dispatcher', 'supplier'],
'dispatch:warnings:view': ['super_admin', 'area_dispatcher']
}
```

### 5.2 JWT认证流程
1. 用户登录获取Access Token和Refresh Token
2. Access Token用于API请求认证（有效期较短）
3. Refresh Token用于刷新Access Token（有效期较长）
4. Token包含用户ID、角色和权限信息

## 6. 状态流转规则

### 6.1 任务状态列表
- `待审核`: 任务创建后需要审核
- `审核通过`: 任务审核通过
- `待供应商响应`: 任务等待供应商确认
- `供应商已响应`: 供应商确认响应
- `任务完成`: 任务执行完成
- `审核拒绝`: 任务审核未通过

### 6.2 状态流转规则
- 车间地调创建的任务初始状态为`待审核`
- 区域调度员创建的任务可选择轨道A或轨道B：
  - 轨道A：初始状态为`待审核`
  - 轨道B：初始状态为`待供应商响应`
- 超级管理员创建的任务可选择轨道A或轨道B：
  - 轨道A：初始状态为`待审核`
  - 轨道B：初始状态为`待供应商响应`

## 7. 数据流设计（按功能模块组织）

### 7.1 基础数据管理流程
**用户管理流程**
```
前端 → 基础数据API → 用户服务 → 数据验证 → 数据库操作 → 返回结果
```

**车辆管理流程**
```
前端 → 基础数据API → 车辆服务 → 数据验证 → 数据库操作 → 返回结果
```

**公司管理流程**
```
前端 → 基础数据API → 公司服务 → 数据验证 → 数据库操作 → 返回结果
```

### 7.2 计划管理流程
**任务创建流程**
```
前端 → 计划管理API → 任务服务 → 数据验证 → 数据库操作 → 返回结果
```

**审核流程**
```
前端提交审核 → 计划管理API → 审核服务 → 状态验证 → 状态变更 → 记录审核历史 → 返回结果
```

**供应商响应流程**
```
前端确认响应 → 计划管理API → 任务服务 → 状态验证 → 车辆信息验证 → 状态变更 → 记录历史 → 返回结果
```

### 7.3 成本分析流程
**运费计算流程**
```
任务创建/更新 → 成本分析API → 运费服务 → 车型匹配 → 规则计算 → 运费核实 → 结果返回
```

**单车核算流程**
```
任务完成 → 成本分析API → 核算服务 → 数据汇总 → 成本计算 → 生成报表 → 返回结果
```

**报表分析流程**
```
前端请求 → 成本分析API → 报表服务 → 数据查询 → 统计分析 → 生成图表 → 返回结果
```

### 7.4 对账管理流程
**结算单生成流程**
```
任务完成 → 对账管理API → 结算单服务 → 数据汇总 → 模板生成 → 电子签名 → 飞书协同 → 供应商确认
```

**账单管理流程**
```
前端请求 → 对账管理API → 账单服务 → 数据查询 → 账单生成 → 核验确认 → 返回结果
```

### 7.5 调度管理流程
**实时调度流程**
```
前端请求 → 调度管理API → 实时调度服务 → 数据查询 → 状态更新 → 返回结果
```

**调度历史流程**
```
前端请求 → 调度管理API → 调度历史服务 → 数据查询 → 统计分析 → 生成报告 → 返回结果
```

### 7.6 系统管理流程
**系统配置流程**
```
前端请求 → 系统管理API → 配置服务 → 数据验证 → 配置更新 → 返回结果
```

**操作日志流程**
```
操作发生 → 中间件记录 → 日志服务 → 数据库存储 → 前端查询 → 返回结果
```

**系统监控流程**
```
定时任务 → 监控服务 → 系统状态检查 → 性能指标收集 → 数据库存储 → 前端展示
```

### 7.7 智能预测流程（暂不实现）
```
历史数据收集 → 预测服务 → 机器学习模型 → 车型需求预测 → 预警分析 → 派车指导方案
```

## 8. 安全设计

### 8.1 数据安全
- 密码使用bcrypt加密存储
- 敏感数据加密传输（HTTPS）
- SQL注入防护（使用ORM参数化查询）
- XSS防护（输入输出过滤）

### 8.2 接口安全
- JWT Token认证
- 基于角色的权限控制
- API速率限制
- 请求参数验证

### 8.3 日志和监控
- 操作日志记录
- 错误日志监控
- 性能监控
- 安全事件审计

## 9. 部署架构

### 9.1 开发环境
```
前端: Vite开发服务器 (localhost:3000)
后端: Flask开发服务器 (localhost:5000)  
数据库: SQLite
```

### 9.2 生产环境
```
前端: Nginx + Vue构建产物
后端: Gunicorn + Flask (多进程)
数据库: PostgreSQL (主从复制)
缓存: Redis (可选)
负载均衡: Nginx
监控: Prometheus + Grafana
日志: ELK Stack
```

## 10. 性能优化策略

### 10.1 数据库优化
- 合理的索引设计
- 查询优化
- 连接池管理
- 读写分离（生产环境）

### 10.2 API优化
- 接口缓存
- 分页查询
- 懒加载关联数据
- 批量操作支持

### 10.3 前端优化
- 组件懒加载
- 路由懒加载
- 图片和资源优化
- 代码分割和Tree Shaking

---

**架构师**: 
**审核时间**: 
**版本**: 1.0