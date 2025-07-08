下面是基于您提供的 `download_bp` 和 `auth_bp` 两个模块生成的接口文档，Markdown 格式。

---

## 目录

* [认证说明](#认证说明)
* [文件下载模块 (`/download`)](#文件下载模块-download)

  * [上传文件 `POST /download/upload`](#上传文件-post-downloaddownloadupload)
  * [列出用户文件 `GET /download/files`](#列出用户文件-get-downloaddownloadfiles)
  * [下载文件 `GET /download/download/{file_id}`](#下载文件-get-downloaddownloaddownloadfile_id)
  * [更新文件元数据 `PUT /download/file/{file_id}`](#更新文件元数据-put-downloaddownloadfilefile_id)
  * [删除文件 `DELETE /download/file/{file_id}`](#删除文件-delete-downloaddownloadfilefile_id)
* [用户认证模块 (`/auth`)](#用户认证模块-auth)

  * [注册 `POST /auth/register`](#注册-post-authregister)
  * [登录 `POST /auth/login`](#登录-post-authlogin)
  * [登出 `POST /auth/logout`](#登出-post-authlogout)
  * [更新个人信息 `PATCH /auth/user`](#更新个人信息-patch-authuser)
  * [获取安全问题 `GET /auth/questions`](#获取安全问题-get-authquestions)
  * [校验安全答案 `POST /auth/questions/verify`](#校验安全答案-post-authquestionsverify)
  * [重置密码 `POST /auth/reset_password`](#重置密码-post-authreset_password)
  * [设置／更新安全问题 `PUT /auth/questions`](#设置更新安全问题-put-authquestions)
  * [修改密码 `POST /auth/change_password`](#修改密码-post-authchangepassword)
  * [删除账号 `POST /auth/delete_account`](#删除账号-post-authdelete_account)

---

## 认证说明

所有受保护接口（即需要用户已登录的接口）都要求请求中包含有效的 Session（`session['user_id']`）。未登录时返回 `401 Unauthorized`。

---

## 文件下载模块 (`/download`)

### 上传文件 `POST /download/upload`

* **URL**： `/download/upload`
* **方法**： `POST`
* **权限**：已登录用户
* **请求参数**（表单/`multipart/form-data`）：

  | 字段                | 类型     | 必填 | 说明                                     |
  | ----------------- | ------ | -- | -------------------------------------- |
  | `file`            | file   | 是  | 上传的文件                                  |
  | `file_permission` | string | 否  | 文件权限，`public` 或 `private`，默认 `private` |
  | `description`     | string | 否  | 文件描述                                   |
* **响应示例**（`201 Created`）：

  ```json
  {
    "message": "Upload successful",
    "file_id": 123,
    "file_name": "example.txt",
    "file_permission": "private",
    "description": "示例文件",
    "file_hash": "abcdef123456...",
    "file_size": 1024,
    "uploaded_at": "2025-07-08T09:00:00Z"
  }
  ```
* **错误响应**：

  * `400 Bad Request`：缺少文件、空文件名、重复上传、无效权限等
  * `401 Unauthorized`：未登录

---

### 列出用户文件 `GET /download/files`

* **URL**： `/download/files`
* **方法**： `GET`
* **权限**：已登录用户
* **响应示例**（`200 OK`）：

  ```json
  [
    {
      "file_id": 123,
      "file_name": "example.txt",
      "updated_at": "2025-07-08T08:59:00",
      "description": "示例文件",
      "file_permission": "private",
      "file_hash": "abcdef123456...",
      "file_size": 1024
    },
    ...
  ]
  ```
* **错误响应**：

  * `401 Unauthorized`：未登录

---

### 下载文件 `GET /download/download/{file_id}`

* **URL**： `/download/download/{file_id}`
* **方法**： `GET`
* **权限**：已登录用户
* **URL 参数**：

  | 参数        | 类型  | 说明       |
  | --------- | --- | -------- |
  | `file_id` | int | 文件的唯一 ID |
* **响应**：以附件形式返回文件流。
* **错误响应**：

  * `401 Unauthorized`：未登录
  * `404 Not Found`：文件不存在或磁盘丢失

---

### 更新文件元数据 `PUT /download/file/{file_id}`

* **URL**： `/download/file/{file_id}`
* **方法**： `PUT`
* **权限**：已登录用户
* **URL 参数**：

  | 参数        | 类型  | 说明       |
  | --------- | --- | -------- |
  | `file_id` | int | 文件的唯一 ID |
* **请求体**（JSON）：

  | 字段                | 类型     | 必填 | 说明                       |
  | ----------------- | ------ | -- | ------------------------ |
  | `file_name`       | string | 否  | 新文件名（会重命名磁盘文件）           |
  | `file_permission` | string | 否  | 新权限，`public` 或 `private` |
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Update successful" }
  ```
* **错误响应**：

  * `400 Bad Request`：无更新字段或权限非法
  * `401 Unauthorized`：未登录
  * `404 Not Found`：文件不存在

---

### 删除文件 `DELETE /download/file/{file_id}`

* **URL**： `/download/file/{file_id}`
* **方法**： `DELETE`
* **权限**：已登录用户
* **URL 参数**：

  | 参数        | 类型  | 说明       |
  | --------- | --- | -------- |
  | `file_id` | int | 文件的唯一 ID |
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Delete successful" }
  ```
* **错误响应**：

  * `401 Unauthorized`：未登录
  * `404 Not Found`：文件不存在

---

## 用户认证模块 (`/auth`)

### 注册 `POST /auth/register`

* **URL**： `/auth/register`
* **方法**： `POST`
* **请求体**（JSON）：

  | 字段         | 类型     | 必填 | 说明        |
  | ---------- | ------ | -- | --------- |
  | `username` | string | 是  | 用户名       |
  | `password` | string | 是  | 原始密码      |
  | `email`    | string | 否  | 邮箱（唯一校验）  |
  | `phone`    | string | 否  | 手机号（唯一校验） |
  | `qq`       | string | 否  | QQ        |
  | `wechat`   | string | 否  | 微信号       |
* **响应示例**（`201 Created`）：

  ```json
  { "message": "Registration successful", "user_id": 42 }
  ```
* **错误响应**：

  * `400 Bad Request`：缺少必填字段
  * `409 Conflict`：用户名/邮箱/手机号已存在

---

### 登录 `POST /auth/login`

* **URL**： `/auth/login`
* **方法**： `POST`
* **请求体**（JSON）：

  | 字段         | 类型     | 必填 | 说明   |
  | ---------- | ------ | -- | ---- |
  | `username` | string | 是  | 用户名  |
  | `password` | string | 是  | 原始密码 |
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Login successful", "user_id": 42 }
  ```
* **错误响应**：

  * `400 Bad Request`：缺少用户名或密码
  * `401 Unauthorized`：凭据无效

---

### 登出 `POST /auth/logout`

* **URL**： `/auth/logout`
* **方法**： `POST`
* **权限**：已登录用户
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Logged out successfully", "user_id": 42 }
  ```

---

### 更新个人信息 `PATCH /auth/user`

* **URL**： `/auth/user`
* **方法**： `PATCH`
* **权限**：已登录用户
* **请求体**（JSON，可选字段）：

  | 字段         | 类型     | 说明         |
  | ---------- | ------ | ---------- |
  | `email`    | string | 新邮箱（唯一校验）  |
  | `phone`    | string | 新手机号（唯一校验） |
  | `qq`       | string | QQ         |
  | `wechat`   | string | 微信号        |
  | `password` | string | 新密码（会哈希存储） |
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Profile updated successfully", "updated": ["email","qq"] }
  ```
* **错误响应**：

  * `400 Bad Request`：无可更新字段
  * `401 Unauthorized`：未登录
  * `409 Conflict`：邮箱/手机号已被占用

---

### 获取安全问题 `GET /auth/questions`

* **URL**： `/auth/questions`
* **方法**： `GET`
* **权限**：已登录用户
* **响应示例**（`200 OK`）：

  ```json
  { "question1": "您的母亲姓名？", "question2": "您的第一辆车？" }
  ```

  或（未设置时）：

  ```json
  { "message": "Security questions not set" }
  ```

---

### 校验安全答案 `POST /auth/questions/verify`

* **URL**： `/auth/questions/verify`
* **方法**： `POST`
* **权限**：已登录用户
* **请求体**（JSON）：

  | 字段        | 类型     | 必填 | 说明                 |
  | --------- | ------ | -- | ------------------ |
  | `answer1` | string | 是  | 对应 `question1` 的答案 |
  | `answer2` | string | 条件 | 如果设置了第二题则必填        |
* **响应示例**（`200 OK`）：

  ```json
  { "match": true }
  ```
* **错误响应**：

  * `400 Bad Request`：缺少 `answer1` 或需要 `answer2`
  * `401 Unauthorized`：未登录
  * `400 Bad Request`：未设置安全问题

---

### 重置密码 `POST /auth/reset_password`

* **URL**： `/auth/reset_password`
* **方法**： `POST`
* **权限**：已登录用户
* **请求体**（JSON）：

  | 字段             | 类型     | 必填 | 说明          |
  | -------------- | ------ | -- | ----------- |
  | `new_password` | string | 是  | 新密码         |
  | `answer1`      | string | 是  | 安全问题1 的答案   |
  | `answer2`      | string | 条件 | 如果设置了第二题则必填 |
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Password reset successful" }
  ```
* **错误响应**：

  * `400 Bad Request`：缺少参数或未设置安全问题
  * `401 Unauthorized`：未登录
  * `403 Forbidden`：安全答案错误

---

### 设置／更新安全问题 `PUT /auth/questions`

* **URL**： `/auth/questions`
* **方法**： `PUT`
* **权限**：已登录用户
* **请求体**（JSON）：

  * **首次设置**：必须包含 `question1`/`answer1` （可附加 `question2`/`answer2`）
  * **更新已有**：需提供 `old_answer1` 或 `old_answer2` 验证旧答案后，才能更新新 `question`/`answer`
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Security questions set" }
  ```

  或

  ```json
  { "message": "Security questions updated" }
  ```
* **错误响应**：

  * `400 Bad Request`：参数不全或验证失败
  * `401 Unauthorized`：未登录
  * `403 Forbidden`：旧答案验证失败

---

### 修改密码 `POST /auth/change_password`

* **URL**： `/auth/change_password`
* **方法**： `POST`
* **权限**：已登录用户
* **请求体**（JSON）：

  | 字段                 | 类型     | 必填 | 说明     |
  | ------------------ | ------ | -- | ------ |
  | `current_password` | string | 是  | 当前登录密码 |
  | `new_password`     | string | 是  | 新密码    |
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Password changed successfully" }
  ```
* **错误响应**：

  * `400 Bad Request`：缺少字段
  * `401 Unauthorized`：未登录
  * `403 Forbidden`：当前密码错误

---

### 删除账号 `POST /auth/delete_account`

* **URL**： `/auth/delete_account`
* **方法**： `POST`
* **权限**：已登录用户
* **请求体**（JSON）：

  | 字段         | 类型     | 必填 | 说明   |
  | ---------- | ------ | -- | ---- |
  | `password` | string | 是  | 当前密码 |
* **响应示例**（`200 OK`）：

  ```json
  { "message": "Account deleted successfully" }
  ```
* **错误响应**：

  * `400 Bad Request`：缺少密码
  * `401 Unauthorized`：未登录
  * `403 Forbidden`：密码错误

---

> **注意**：所有时间戳均为 UTC。文件路径等敏感信息由后端保持，不建议前端暴露。
