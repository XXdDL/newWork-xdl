# GitHub 仓库设置指南

## 1. 创建GitHub仓库

### 1.1 在GitHub上创建新仓库
1. 登录 GitHub 账户
2. 点击右上角 "+" → "New repository"
3. 填写仓库信息：
   - **Repository name**: smart-transport-system
   - **Description**: 智能运输管理系统 - 基于6A工作流的现代化运输调度平台
   - **Visibility**: Public (或 Private，根据需求选择)
   - **Initialize with README**: 不勾选（我们已经有了README）
   - **Add .gitignore**: 选择 Python
   - **Choose a license**: 选择 MIT License

### 1.2 本地仓库配置

#### 设置远程仓库地址
```bash
# 添加远程仓库（将YOUR_USERNAME替换为您的GitHub用户名）
git remote add origin https://github.com/YOUR_USERNAME/smart-transport-system.git

# 验证远程仓库设置
git remote -v
```

#### 推送代码到GitHub
```bash
# 首次推送
git push -u origin master

# 或者如果使用main分支
git branch -M main
git push -u origin main
```

## 2. GitHub Actions 配置

### 2.1 设置 Secrets（如果需要）
如果CI/CD流程需要访问密钥，请在GitHub仓库中设置：
1. 进入仓库 → Settings → Secrets and variables → Actions
2. 点击 "New repository secret"
3. 添加必要的密钥（如API密钥、部署密钥等）

### 2.2 验证工作流
推送代码后，GitHub Actions会自动运行：
1. 进入仓库 → Actions
2. 查看 "Architecture Guard Check" 工作流状态
3. 确保所有检查通过

## 3. 分支策略

### 3.1 推荐分支结构
```
master/main     - 生产环境分支
develop        - 开发主分支
feature/*      - 功能开发分支
hotfix/*       - 紧急修复分支
release/*      - 发布准备分支
```

### 3.2 创建开发分支
```bash
# 创建并切换到develop分支
git checkout -b develop

# 推送develop分支到远程
git push -u origin develop
```

## 4. 保护分支设置

### 4.1 设置分支保护规则
1. 进入仓库 → Settings → Branches
2. 点击 "Add branch protection rule"
3. 配置保护规则：
   - **Branch name pattern**: master
   - ☑ Require a pull request before merging
   - ☑ Require status checks to pass before merging
   - ☑ Require conversation resolution before merging
   - ☑ Include administrators

### 4.2 设置Required Status Checks
在分支保护规则中：
1. 找到 "Status checks that must pass before merging"
2. 添加以下检查：
   - architecture-check
   - 其他CI检查（后续添加）

## 5. 协作设置

### 5.1 添加协作者
1. 进入仓库 → Settings → Collaborators
2. 点击 "Add people"
3. 输入协作者的GitHub用户名
4. 设置适当的权限级别

### 5.2 设置团队权限
如果使用GitHub Organizations：
1. 进入仓库 → Settings → Manage access
2. 添加团队并设置权限

## 6. 项目管理

### 6.1 启用项目管理功能
1. **Projects**: 启用项目管理看板
2. **Wiki**: 启用Wiki文档
3. **Issues**: 启用Issue跟踪
4. **Discussions**: 启用讨论区（可选）

### 6.2 配置Labels
创建标准的问题标签：
- `bug` - 缺陷修复
- `enhancement` - 功能增强
- `documentation` - 文档更新
- `architecture` - 架构相关
- `ci/cd` - 持续集成

## 7. 代码审查设置

### 7.1 Pull Request 模板
在 `.github/PULL_REQUEST_TEMPLATE.md` 创建PR模板：
```markdown
## 变更描述

## 相关Issue

## 测试验证
- [ ] 本地测试通过
- [ ] 架构检查通过
- [ ] CI/CD通过

## 截图（如适用）
```

### 7.2 Code Owners
创建 `.github/CODEOWNERS` 文件：
```
# 核心架构文件
/docs/SMART_TRANSPORT_SYSTEM/ @team-architecture

# 后端代码
/src/backend/ @team-backend

# 前端代码
/src/frontend/ @team-frontend
```

## 8. 监控和统计

### 8.1 启用Insights
1. **Code frequency**: 代码提交频率
2. **Dependency graph**: 依赖关系图
3. **Network**: 分支网络图
4. **Traffic**: 访问统计

### 8.2 设置Webhooks（可选）
用于集成其他服务：
1. CI/CD系统通知
2. 聊天工具通知
3. 监控告警

## 9. 故障排除

### 9.1 常见问题

#### 推送权限错误
```bash
# 检查远程地址
git remote -v

# 更新远程地址
git remote set-url origin https://github.com/USERNAME/REPO.git
```

#### 分支保护冲突
- 确保本地分支与远程同步
- 检查所需的status checks是否通过

### 9.2 获取帮助
- GitHub Docs: https://docs.github.com
- GitHub Community: https://github.com/community

## 10. 下一步行动

1. ✅ 本地Git仓库初始化完成
2. 🔄 按照本指南在GitHub创建远程仓库
3. 📤 推送代码到GitHub
4. ✅ 验证CI/CD工作流
5. 👥 设置团队协作权限
6. 🛡️ 配置分支保护规则
7. 📊 启用项目监控功能

完成以上步骤后，您的智能运输系统项目就完全配置好了GitHub仓库，可以开始团队协作了！