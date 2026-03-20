# API接口文档

## 1. 目的
约定前后端通信的规范，提供清晰的请求头、路径、参数、响应及状态码，确保前后端协同开发顺畅。系统基于 FastAPI 自动生成 OpenAPI (Swagger) 规范。

## 2. 基础与认证接口

### 2.1 用户登录
- **路径**: `POST /api/auth/login`
- **说明**: 用户凭证校验，获取 JWT Token。
- **请求体**: `{ "username": "1001", "password": "..." }`
- **响应 (200 OK)**: `{ "access_token": "ey...", "token_type": "bearer", "user": { ... } }`

### 2.2 系统健康检查
- **路径**: `GET /api/health`
- **说明**: 运维监控探针，检查数据库连接与服务状态。
- **响应 (200 OK)**: `{ "status": "ok", "db": "connected" }`

## 3. 问卷管理接口 (Admin/User)

### 2.1 新增问卷 (Admin)
- **路径**: `POST /api/surveys/`
- **说明**: 教导主任新建一份问卷，初始状态为草稿 (draft)。
- **请求体 (application/json)**:
  ```json
  {
    "title": "2023秋季学期教学进度摸底",
    "deadline": "2023-10-31T23:59:59Z",
    "survey_schema": [
      { "key": "progress", "label": "当前章节", "type": "text" },
      { "key": "advice", "label": "意见反馈", "type": "textarea" },
      { "key": "date", "label": "填报日期", "type": "date" }
    ]
  }
  ```
- **响应 (201 Created)**:
  ```json
  { "id": 1, "title": "...", "status": "draft", "created_at": "2023-10-01T10:00:00Z" }
  ```

### 2.2 编辑问卷 (Admin)
- **路径**: `PUT /api/surveys/{survey_id}`
- **说明**: 教导主任在“问卷可视化编辑器”中修改问卷的标题或字段（仅限草稿状态）。
- **请求体 (application/json)**: 同 2.1。
- **响应 (200 OK)**: 返回更新后的问卷详情。

### 2.3 下发问卷 (Admin)
- **路径**: `POST /api/surveys/{survey_id}/publish`
- **说明**: 将问卷状态从草稿改为已下发 (published)，允许教师填报。
- **响应 (200 OK)**: `{ "status": "published" }`

### 2.4 撤回问卷 (Admin)
- **路径**: `POST /api/surveys/{survey_id}/withdraw`
- **说明**: 撤回已下发的问卷，将其变回草稿状态，清空或保留已有填报数据（取决于具体参数）。
- **响应 (200 OK)**: `{ "status": "draft" }`

### 2.5 截止问卷 (Admin)
- **路径**: `POST /api/surveys/{survey_id}/close`
- **说明**: 终止问卷收集，将状态改为已结束 (closed)。
- **响应 (200 OK)**: `{ "status": "closed" }`

### 2.6 删除问卷及数据 (Admin)
- **路径**: `DELETE /api/surveys/{survey_id}`
- **说明**: 级联删除问卷及其下属的所有答卷记录。
- **响应 (204 No Content)**: 删除成功。

### 2.7 获取问卷详情 (User/Admin)
- **路径**: `GET /api/surveys/{survey_id}`
- **说明**: 前端获取问卷元数据，用于渲染动态填写界面。
- **响应 (200 OK)**: 包含 `survey_schema` 和 `status` 等信息。
- **状态码**: `404 Not Found` (问卷不存在)

## 4. 表单模板管理接口 (Admin)

### 4.1 创建表单模板
- **路径**: `POST /api/form-templates/`
- **说明**: 将问卷中的某一部分字段保存为可复用的表单模板。
- **请求体**: `{ "name": "基本信息模块", "template_schema": [...] }`
- **响应 (201 Created)**: 返回模板详情。

### 4.2 获取表单模板列表
- **路径**: `GET /api/form-templates/`
- **响应 (200 OK)**: 返回模板数组。

### 4.3 删除表单模板
- **路径**: `DELETE /api/form-templates/{template_id}`
- **响应 (204 No Content)**: 删除成功。

## 5. 问卷答卷接口 (User/Admin)

### 5.1 提交问卷答卷 (User)
- **路径**: `POST /api/surveys/{survey_id}/responses/`
- **说明**: 教师首次填写问卷并提交数据。系统需校验问卷是否处于已下发状态及该用户是否已提交。
- **请求体 (application/json)**:
  ```json
  {
    "dynamic_data": {
      "progress": "第三章 第二节"
    }
  }
  ```
- **响应 (201 Created)**: 返回保存成功的答卷详情。

### 5.2 修改问卷答卷 (User)
- **路径**: `PUT /api/surveys/{survey_id}/responses/{response_id}`
- **说明**: 教师修改已提交的问卷数据。
- **请求体**: 同 4.1。
- **响应 (200 OK)**: 返回更新后的答卷详情。

### 5.3 撤回/删除问卷答卷 (User)
- **路径**: `DELETE /api/surveys/{survey_id}/responses/{response_id}`
- **说明**: 教师主动撤回已填写的答卷（如果允许）。
- **响应 (204 No Content)**: 删除成功。

### 5.4 获取问卷答卷列表/统计 (User/Admin)
- **路径**: `GET /api/surveys/{survey_id}/responses/`
- **说明**: 
  - Admin: 获取完整数据进行管理与导出。
  - User (教师): 获取该问卷下所有人的填报数据（可能包含脱敏处理），用于“全员实时看板”展示。
- **响应 (200 OK)**: 返回数组格式，包含合并后的静态信息与动态 JSON 信息。

### 5.5 导出报表 (Admin)
- **路径**: `GET /api/surveys/{survey_id}/export`
- **说明**: 导出包含身份信息与业务数据的 Excel 文件。
- **响应**: `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` (文件流)

## 6. 用户管理接口 (Admin)
- **获取用户列表**: `GET /api/users/` (支持分页与学科筛选)
- **新增用户**: `POST /api/users/` (入参包含：工号、姓名、学科、角色)
- **修改用户**: `PUT /api/users/{user_id}` 
- **删除用户**: `DELETE /api/users/{user_id}`

### 6.1 批量导入用户
- **路径**: `POST /api/users/import`
- **说明**: 批量导入教师信息 (Excel/CSV)。
- **请求体**: `multipart/form-data` (file)
- **响应 (200 OK)**: `{ "success_count": 50, "fail_count": 0 }`

## 7. 实时通信接口 (WebSocket)

### 7.1 问卷数据实时广播
- **路径**: `WS /api/ws/surveys/{survey_id}`
- **说明**: 客户端（Admin大屏/User实时看板）建立长连接。当有新答卷提交或更新时，服务器广播变更消息。
- **消息格式 (Server -> Client)**:
  ```json
  {
    "type": "new_response",
    "data": {
      "filler_name": "张三",
      "dynamic_data": { ... },
      "created_at": "..."
    }
  }
  ```

*(注：开发阶段可通过访问 `http://localhost:8000/docs` 查看实时互动的 Swagger UI 文档)*
