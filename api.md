以下是基于现有 Flask Blueprint 的接口文档（Markdown 格式）：

---

## 认证模块 `/auth`

| 方法    | 路径                | 描述        | 参数（JSON Body）                                                                                                                                          | 返回示例                                                                                                                                          |
| ----- | ----------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| POST  | `/auth/register`  | 用户注册      | `username`<sup>*</sup>（string）<br>`password`<sup>*</sup>（string）<br>`email`（string，可选）<br>`phone`（string，可选）<br>`qq`（string，可选）<br>`wechat`（string，可选） | **成功**<br>HTTP 201<br>`json<br>{<br>  "message":"Registration successful",<br>  "user_id":123<br>}`                                           |
| POST  | `/auth/login`     | 用户登录      | `username`<sup>*</sup>（string）<br>`password`<sup>*</sup>（string）                                                                                       | **成功**<br>HTTP 200<br>`json<br>{<br>  "message":"Login successful",<br>  "user_id":123<br>}`                                                  |
| POST  | `/auth/logout`    | 用户登出      | —                                                                                                                                                      | **成功**<br>HTTP 200<br>`json<br>{<br>  "message":"Logged out successfully",<br>  "user_id":123<br>}`                                           |
| PATCH | `/auth/user`      | 更新用户资料    | `email`（string，可选）<br>`phone`（string，可选）<br>`qq`（string，可选）<br>`wechat`（string，可选）<br>`password`（string，可选，新密码）                                        | **成功**<br>HTTP 200<br>`json<br>{<br>  "message":"Profile updated successfully",<br>  "updated":["email","password"]<br>}`                     |
| PUT   | `/auth/questions` | 设置/更新安全问题 | `question1` & `answer1`<sup>†</sup>（string）<br>`question2` & `answer2`（string，可选）                                                                      | **成功**<br>HTTP 200<br>`json<br>{<br>  "message":"Security questions set/updated successfully",<br>  "questions":["question1","answer1"]<br>}` |

<sup>\*</sup> 必填； <sup>†</sup> 至少提供一对 `question`/`answer`。

---

## 文件管理模块 `/download`

| 方法     | 路径                                 | 描述             | 参数                                                                                                                            | 返回示例                                                                                                                                                                                                                                                          |
| ------ | ---------------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| POST   | `/download/upload`                 | 上传文件并去重        | **Form + File**<br>- `file`<br>- `file_permission`（string，可选，`public` 或 `private`，默认 `private`）<br>- `description`（string，可选） | **成功**<br>HTTP 201<br>`json<br>{<br>  "message":"Upload successful",<br>  "file_id":456,<br>  "file_name":"example.pdf",<br>  "file_permission":"private",<br>  "description":"测试文件",<br>  "file_hash":"...",<br>  "uploaded_at":"2025-07-07T15:30:00Z"<br>}` |
| GET    | `/download/files`                  | 列出当前用户所有文件     | —                                                                                                                             | **成功**<br>HTTP 200<br>`json<br>[{<br>  "file_id":456,<br>  "file_name":"example.pdf",<br>  "updated_at":"2025-07-07T15:30:00Z",<br>  "description":"测试",<br>  "file_permission":"private",<br>  "file_hash":"..."<br>}...]`                                   |
| GET    | `/download/download/<int:file_id>` | 下载指定文件         | URL path 参数：`file_id`                                                                                                         | **成功** 返回文件流，`Content-Disposition: attachment; filename="..."`                                                                                                                                                                                                |
| PUT    | `/download/file/<int:file_id>`     | 更新文件元数据／重命名／权限 | JSON Body:<br>- `file_name`（string，可选）<br>- `file_permission`（string，可选，`public` 或 `private`）                                 | **成功**<br>HTTP 200<br>`json<br>{<br>  "message":"Update successful"<br>}`                                                                                                                                                                                     |
| DELETE | `/download/file/<int:file_id>`     | 删除指定文件         | URL path 参数：`file_id`                                                                                                         | **成功**<br>HTTP 200<br>`json<br>{<br>  "message":"Delete successful"<br>}`                                                                                                                                                                                     |

---

### 全局错误响应示例

* **400 Bad Request**

  ```json
  {"error":"Missing field: username"}
  ```
* **401 Unauthorized**

  ```json
  {"error":"Authentication required"}
  ```
* **404 Not Found**

  ```json
  {"error":"File not found"}
  ```
* **409 Conflict**

  ```json
  {"error":"Username already taken"}
  ```

---

### 说明

1. 所有 `/auth/*` 和 `/download/*` 接口都依赖 session 验证，除了注册和登录外，请确保在请求头中携带正确的 Cookie。
2. 上传接口会对文件内容做 SHA-256 去重；同一内容上传会返回 400 错误。
3. 重命名文件时，物理文件会在服务器上同步重命名并更新对应路径。
4. `file_permission=public` 时，上传会额外向 `file_public` 表插入记录；修改为 `private` 时会删除对应记录。

---
