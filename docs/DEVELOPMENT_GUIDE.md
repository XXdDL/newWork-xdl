# 智能运输系统开发指南

## 🏗️ 四层架构实施指南

### 1. 架构原则

#### 1.1 分层职责
- **数据层 (Data Layer)**: 纯数据存储和访问
- **服务层 (Service Layer)**: 核心业务逻辑实现
- **应用层 (Application Layer)**: API端点和请求处理
- **表现层 (Presentation Layer)**: 用户界面和交互

#### 1.2 依赖方向
```
表现层 → 应用层 → 服务层 → 数据层
```

### 2. 代码组织规范

#### 2.1 后端代码结构
```
src/
├── backend/
│   ├── models/          # 数据层 - 数据库模型
│   │   ├── user.py
│   │   ├── vehicle.py
│   │   └── task.py
│   ├── services/        # 服务层 - 业务逻辑
│   │   ├── user_service.py
│   │   ├── vehicle_service.py
│   │   └── task_service.py
│   ├── api/             # 应用层 - API端点
│   │   ├── auth.py
│   │   ├── users.py
│   │   └── tasks.py
│   └── core/            # 核心组件
│       ├── database.py
│       ├── config.py
│       └── security.py
```

#### 2.2 前端代码结构
```
src/
├── components/          # 可复用UI组件
├── views/               # 页面级组件
├── stores/              # 状态管理
├── services/            # API调用服务
├── models/              # 数据类型定义
└── utils/               # 工具函数
```

### 3. 开发最佳实践

#### 3.1 数据层开发规范
```python
# ✅ 正确示例 - 纯数据模型
class User(BaseModel):
    id: int
    username: str
    email: str
    
    # ❌ 错误 - 不要在模型中包含业务逻辑
    # def calculate_something(self):
    #     return some_business_logic()
```

#### 3.2 服务层开发规范
```python
# ✅ 正确示例 - 纯业务逻辑
class UserService:
    def create_user(self, user_data: dict):
        # 业务验证逻辑
        self._validate_user_data(user_data)
        
        # 创建用户
        user = User(**user_data)
        db.session.add(user)
        db.session.commit()
        
        return user
    
    def _validate_user_data(self, data: dict):
        # 业务规则验证
        if not data.get('username'):
            raise ValueError("用户名不能为空")
```

#### 3.3 应用层开发规范
```python
# ✅ 正确示例 - 精简的API处理
@auth_bp.route('/users', methods=['POST'])
def create_user():
    try:
        # 请求数据验证
        data = request.get_json()
        validate_user_create_data(data)
        
        # 调用服务层
        user = user_service.create_user(data)
        
        # 返回响应
        return jsonify({
            'success': True,
            'data': user.to_dict()
        }), 201
        
    except ValueError as e:
        return jsonify({'error': str(e)}), 400
```

#### 3.4 表现层开发规范
```vue
<!-- ✅ 正确示例 - 纯UI组件 -->
<template>
  <div>
    <user-form @submit="handleSubmit" />
    <user-list :users="users" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { userService } from '@/services/userService'

const users = ref([])

// ❌ 错误 - 不要在组件中直接写业务逻辑
// const validateBusinessRules = () => { ... }

// ✅ 正确 - 调用服务层处理业务
const handleSubmit = async (userData) => {
  try {
    const newUser = await userService.createUser(userData)
    users.value.push(newUser)
  } catch (error) {
    console.error('创建用户失败:', error)
  }
}
</script>
```

### 4. 架构检查流程

#### 4.1 本地开发检查
```bash
# 运行架构检查
make guard

# 严格模式检查
make guard-strict

# 检查特定模块
make guard-module module=dispatch
```

#### 4.2 CI/CD集成
- 每次push到main分支自动运行架构检查
- PR合并前必须通过架构检查
- 架构违规会阻止部署

#### 4.3 常见架构问题

1. **数据层违规**: 
   - 在模型中包含业务方法
   - 直接处理HTTP请求

2. **服务层违规**:
   - 直接返回HTTP响应
   - 包含界面渲染逻辑

3. **应用层违规**:
   - 编写复杂业务逻辑
   - 直接操作数据库

4. **表现层违规**:
   - 直接调用数据库
   - 包含业务验证逻辑

### 5. 代码审查要点

#### 5.1 架构审查清单
- [ ] 各层职责是否清晰分离
- [ ] 是否存在跨层直接调用
- [ ] 业务逻辑是否集中在服务层
- [ ] API层是否保持精简
- [ ] UI组件是否纯粹

#### 5.2 技术债务识别
- 架构违规代码
- 混合职责的组件
- 过长的函数和方法
- 复杂的条件判断链

### 6. 重构指南

#### 6.1 架构重构模式
1. **提取服务层**: 从API层提取业务逻辑到服务层
2. **纯化数据模型**: 移除模型中的业务方法
3. **精简API层**: 将复杂逻辑移到服务层
4. **组件拆分**: 将混合职责的组件拆分为纯UI组件和业务逻辑

#### 6.2 重构示例
```python
# 重构前 - 混合架构
@api.route('/task')
def create_task():
    data = request.json
    
    # ❌ 业务逻辑在API层
    if not data.get('title'):
        return {'error': '标题不能为空'}, 400
    
    # ❌ 直接数据库操作
    task = Task(title=data['title'])
    db.session.add(task)
    db.session.commit()
    
    return {'data': task.to_dict()}

# 重构后 - 清晰分层
@api.route('/task')
def create_task():
    try:
        data = request.json
        
        # ✅ API层只做验证和转发
        validate_task_data(data)
        
        # ✅ 调用服务层处理业务
        task = task_service.create_task(data)
        
        return {'data': task.to_dict()}
    except ValidationError as e:
        return {'error': str(e)}, 400
```

### 7. 工具和支持

#### 7.1 开发工具
- **架构检查**: `scripts/architecture_guard.py`
- **代码模板**: `templates/` 目录
- **CI/CD**: GitHub Actions工作流

#### 7.2 学习资源
- 架构保障文档: `docs/ARCHITECTURE_GUARDRAILS.md`
- 正反例代码: `examples/architecture/`
- 培训材料: `docs/training/`

通过遵循本指南，您可以确保代码始终符合四层架构规范，提高项目的可维护性和可扩展性。