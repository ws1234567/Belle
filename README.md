[Uploading spec-2026-05-17-user-center.md…]()
# 普通用户个人中心设计文档

**日期：** 2026-05-17

## 概述

为系统添加普通用户个人中心功能，允许普通用户上传作品并由管理员审核。

## 需求确认

1. **注册方式：** 两者都要（管理员可创建 + 普通用户可用注册码注册）
2. **功能范围：** 完整版（头像上传、个人资料、修改密码、作品管理、审核状态查看）
3. **审核流程：** 详细审核（通过/拒绝 + 拒绝原因）

## 架构设计

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   主页(index)   │     │ 作品页(works)  │     │ 用户中心(user) │
│   轮播图展示    │     │   展示审核通过  │     │   普通用户使用  │
└────────┬────────┘     │     的作品     │     └────────┬────────┘
         │              └────────┬────────┘              │
         └───────────────────────┼───────────────────────┘
                    ┌────────────┴────────────┐
                    │      服务器(server.js)   │
                    └─────────────────────────┘
```

## 数据表设计

### 新增 `users` 表（普通用户）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | INT | 主键 |
| username | VARCHAR(50) | 用户名（唯一） |
| password_hash | VARCHAR(255) | 密码哈希 |
| avatar_url | VARCHAR(500) | 头像URL |
| email | VARCHAR(100) | 邮箱 |
| created_at | TIMESTAMP | 创建时间 |

### 修改 `works` 表

| 字段 | 类型 | 说明 |
|------|------|------|
| user_id | INT | 上传用户ID（nullable，null为管理员添加） |
| status | ENUM | pending/approved/rejected |
| reject_reason | TEXT | 拒绝原因 |
| submitted_at | TIMESTAMP | 提交时间 |

### 修改 `register_codes` 表

| 字段 | 类型 | 说明 |
|------|------|------|
| code_type | ENUM | admin/user |

## API 设计

### 用户认证

- `POST /api/user/register` - 用户注册（需注册码）
- `POST /api/user/login` - 用户登录
- `GET /api/user/verify` - 验证token
- `GET /api/user/profile` - 获取用户资料
- `PUT /api/user/profile` - 更新用户资料
- `PUT /api/user/password` - 修改密码

### 作品管理（用户端）

- `GET /api/user/works` - 获取我的作品（按状态筛选）
- `POST /api/user/works` - 提交新作品
- `PUT /api/user/works/:id` - 编辑我的作品
- `DELETE /api/user/works/:id` - 删除我的作品

### 管理员端

- `GET /api/admin/users` - 获取所有普通用户
- `PUT /api/admin/users/:id/status` - 禁用/启用用户
- `DELETE /api/admin/users/:id` - 删除用户
- `GET /api/admin/pending-works` - 获取待审核作品
- `PUT /api/admin/works/:id/approve` - 审核通过
- `PUT /api/admin/works/:id/reject` - 审核拒绝（带原因）

## 页面设计

### 用户中心 (user.html)

1. **登录/注册区域**
   - 用户名、密码输入
   - 注册链接（需注册码）

2. **个人资料卡片**
   - 头像上传
   - 用户名、邮箱显示/编辑

3. **作品管理区域**
   - 状态标签筛选（全部/待审核/已通过/已拒绝）
   - 作品卡片列表
   - 上传作品表单

4. **审核状态显示**
   - 待审核：黄色标签
   - 已通过：绿色标签
   - 已拒绝：红色标签 + 拒绝原因

### 管理员后台增强

1. **用户管理面板**
   - 用户列表
   - 禁用/启用
   - 删除（非admin）

2. **作品审核面板**
   - 待审核作品列表
   - 审核操作：通过/拒绝
   - 拒绝时填写原因

## 页面样式

- 保持现有深色主题风格
- 卡片式布局
- 状态标签颜色：
  - 待审核：#fbbf24（黄色）
  - 已通过：#4ade80（绿色）
  - 已拒绝：#ff5c6c（红色）
