# API

## 通用错误

一些错误是所有 API 都可能会返回的。
具体说来，每个 API 都可能返回如下错误之一：

=== "找不到页面"

    > `404 Not Found`

    ```json
    {
        "code": 1000,
        "info": "NOT_FOUND",
        "data": {}
    }
    ```

=== "未认证"

    > `401 Unauthorized`

    ```json
    {
        "code": 1001,
        "info": "UNAUTHORIZED",
        "data": {}
    }
    ```

=== "内部错误"

    服务器内部错误。

    > `500 Internal Server Error`

    ```json
    {
        "code": 1003,
        "info": "INTERNAL_ERROR",
        "data": {}
    }
    ```

=== "请求体格式错误"

    解析 JSON 请求体时缺少必要参数或出现错误。

    > `400 Bad Request`

    ```json
    {
        "code": 1005,
        "info": "INVALID_FORMAT",
        "data": {}
    }
    ```

## 用户管理相关

### POST /user/register

注册一个用户。

=== "请求"

    请求正文样例：

    ```json
    {
        "user_name": "Alice",
        "password": "Hashed_Word",
        "salt": "secret_salt", 
        "mail": "mymail@163.com"
    }
    ```

    正文的 JSON 包含一个字典，字典的各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`user_name`|字符串|是|用户名|
    |`password`|字符串|是|用户密码|
    |`salt`|字符串|是|用户 salt|
    |`mail`|字符串|是|用户邮箱|

=== "行为"

    前端将用户名、加密后的密码与盐值传输到后端。

    后端接受请求后，首先验证用户名是否重复、合法。

    若用户名与密码都合法，则向所给邮箱发送一封邮件，并创建一个待验证用户对象等待邮件校验；否则返回错误信息。

=== "响应"

    - 成功发送校验邮件

        若成功发送用户注册校验邮件，则返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 用户名重复或用户名待验证

        > `400 Bad Request`

        ```json
        {
            "code": 1,
            "info": "USER_NAME_CONFLICT",
            "data": {}
        }
        ```

    - 用户名格式非法

        > `400 Bad Request`

        ```json
        {
            "code": 2,
            "info": "INVALID_USER_NAME_FORMAT",
            "data": {}
        }
        ```

    - 密码格式不合法

        > `400 Bad Request`

        ```json
        {
            "code": 3,
            "info": "INVALID_PASSWORD_FORMAT",
            "data": {}
        }
        ```

### POST /user/salt

返回一个用户注册时的 salt。

=== "请求"

    请求正文样例：

    ```json
    {
        "user_name": "Alice"
    }
    ```

    各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`user_name`|字符串|是|用户名|

=== "行为"

    后端接受请求后，首先验证用户名是否合法，合法则返回用户 salt。

=== "响应"

    - 成功返回 salt

        若成功注册用户，则返回如下响应并转入已登录状态：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "salt": "SECRET_SALT"
            }
        }
        ```

        各字段含义如下：

        |字段|类型|必选|含义|
        |-|-|-|-|
        |`salt`|字符串|是|用户 salt|

=== "错误"

    - 用户名或密码错误

        > `400 Bad Request`

        ```json
        {
            "code": 4,
            "info": "USER_NAME_NOT_EXISTS_OR_WRONG_PASSWORD",
            "data": {}
        }
        ```

### GET /user/verify/[token]

校验一个注册的用户。

=== "请求"

    无需请求正文。

=== "行为"

    后端检验 token 对应的待验证用户是否存在、未验证且处在待验证时间内。

    满足这三个条件，则注册用户、将密码进行哈希后存储并返回登录 Token，否则返回错误信息。

=== "响应"

    - 成功校验用户

        返回如下响应并转入已登录状态：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 1,
                "user_name": "Alice",
                "token": "SECRET_TOKEN"
            }
        }
        ```

        其中 `data` 是一个字典，各字段含义如下：

        |字段|类型|必选|含义|
        |-|-|-|-|
        |`id`|整数|是|用户 ID|
        |`user_name`|字符串|是|用户名|
        |`token`|字符串|是|用户 token|

=== "错误"

    - 验证用户名不存在

        > `400 Bad Request`

        ```json
        {
            "code": 15,
            "info": "INVALID_TOKEN",
            "data": {}
        }
        ```

    - 用户已经验证

        > `400 Bad Request`

        ```json
        {
            "code": 16,
            "info": "ALREADY_VERIFIED",
            "data": {}
        }
        ```

    - 长时间未验证需重新注册

    > `400 Bad Request`

    ```json
    {
        "code": 17,
        "info": "TOO_LONG_TIME",
        "data": {}
    }
    ```

### POST /user/login

用户登录。

=== "请求"

    请求正文样例：

    ```json
    {
        "user_name": "Alice",
        "password": "Bob19937"
    }
    ```

    各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`user_name`|字符串|是|用户名|
    |`password`|字符串|是|用户密码|

=== "行为"

    后端接受请求后，验证用户是否存在，若是则验证密码与存储的密码哈希值是否匹配。

    二者都通过则成功登录并返回 token，否则返回用户名或密码错误。

=== "响应"

    - 成功登录

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 1,
                "user_name": "Alice",
                "token": "SECRET_TOKEN"
            }
        }
        ```

        其中 `data` 是一个字典，各字段含义如下：

        |字段|类型|必选|含义|
        |-|-|-|-|
        |`id`|整数|是|用户 ID|
        |`user_name`|字符串|是|用户名|
        |`token`|字符串|是|用户 token|

=== "错误"

    - 用户名格式非法

        > `400 Bad Request`

        ```json
        {
            "code": 2,
            "info": "INVALID_USER_NAME_FORMAT",
            "data": {}
        }
        ```

    - 密码格式非法

        > `400 Bad Request`

        ```json
        {
            "code": 3,
            "info": "INVALID_PASSWORD_FORMAT",
            "data": {}
        }
        ```

    - 用户名或密码错误

        > `400 Bad Request`

        ```json
        {
            "code": 4,
            "info": "USER_NAME_NOT_EXISTS_OR_WRONG_PASSWORD",
            "data": {}
        }
        ```

### POST /user/modifypassword

用户修改密码。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。
    
    请求正文样例：

    ```json
    {
        "user_name": "Alice",
        "old_password": "Bob19937",
        "new_password": "Carol48271"
    }
    ```

    正文的 JSON 包含一个字典，字典的各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`user_name`|字符串|是|用户名|
    |`old_password`|字符串|是|用户旧密码|
    |`new_password`|字符串|是|用户新密码|

=== "行为"

    后端接受到请求之后，先检验 token 是否合理。

    如果合理，则判断 `old_password` 是否是该用户的原本密码。
    
    如果原密码正确，则检验新密码是否符合格式，最后进行修改。

=== "响应"

    - 修改成功

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 用户名或密码错误

        > `400 Bad Request`

        ```json
        {
            "code": 4,
            "info": "USER_NAME_NOT_EXISTS_OR_WRONG_PASSWORD",
            "data": {}
        }
        ```

### POST /user/avatar

修改用户头像。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求正文是包含头像文件的 form-data 表单：

    ```form-data
        file: avatar_file
    ```

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。
    
    接下来判断上传头像文件是否存在，文件存在且格式正确则进行修改，并返回头像的 base64 编码。

=== "响应"

    - 修改成功

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 1,
                "user_name": "Bob",
                "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/=="
            }
        }
        ```

=== "错误"

    - 图片不存在或格式非法

        > `400 Bad Request`

        ```json
        {
            "code": 18,
            "info": "AVATAR_FILE_NOT_FOUND",
            "data": {}
        }
        ```

### GET /user/avatar

获取用户头像。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，并返回相应用户头像的 base64 编码。

=== "响应"

    - 获取成功

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 1,
                "user_name": "Bob",
                "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/=="
            }
        }
        ```

=== "错误"

    本 API 不应该出错。


### POST /user/signature

修改用户头像。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文样例：

    ```json
    {
        "signature": "Hello world!"
    }
    ```

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，并修改用户签名。

=== "响应"

    - 修改成功

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 1,
                "user_name": "Bob",
                "signature": "Hello world!"
            }
        }
        ```

=== "错误"

    本 API 不应该出错。

### POST /user/logout

登出用户。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 `token` 是否有效。如有效，将该用户登出。

=== "响应"

    - 成功登出

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 用户未登录

        > `401 Unauthorized`

        ```json
        {
            "code": 1001,
            "info": "UNAUTHORIZED",
            "data": {}
        }
        ```

### POST /user/checklogin

检查用户登录状态。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，检验 `token` 是否有效，并返回登陆状态。

=== "响应"

    - 登录状态有效

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 登陆状态失效

        > `401 Unauthorized`

        ```json
        {
            "code": 1001,
            "info": "UNAUTHORIZED",
            "data": {}
        }
        ```

### GET /user/profile/[user_id]

返回用户主页信息。

=== "请求"

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验用户 id 是否有效，并返回用户相关信息。

=== "响应"

    - 用户存在

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 1,
                "user_name": "Bob",
                "signature": "This is my signature.",
                "mail": "Bob21@163.com",
                "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                "followers": 191,
                "following": 81,
                "data": [
                    {
                        "id": 1,
                        "title": "ILoveCakes",
                        "width": 400,
                        "height": 222,
                        "category": "food",
                        "tags": [
                            "happy",
                            "laugh",
                            "sad"
                        ],
                        "duration": 1.8,
                        "pub_time": "2023-04-18T04:11:07.785Z",
                        "like": 0
                    }
                ]
            }
        }
        ```

        其中 `data` 是一个数组，其中每个对象各字段含义如下：

        |字段|类型|必选|含义|
        |-|-|-|-|
        |`id`|整数|是|用户 ID|
        |`user_name`|字符串|是|用户名|
        |`signature`|字符串|是(可能为空)|用户签名|
        |`mail`|字符串|是|用户邮箱|
        |`avatar`|字符串|是(可能为空)|用户头像|
        |`followers`|字符串|是|用户粉丝数|
        |`following`|字符串|是|用户关注数|
        |`data`|数组|是|用户上传 Gif 信息|

=== "错误"

    - 用户不存在

        > `400 Bad Request`

        ```json
        {
            "code": 12,
            "info": "USER_NOT_FOUND",
            "data": {}
        }
        ```

### GET /user/info/[user_id]

返回简短的用户信息。

=== "请求"

    请求可以在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 和 id 是否有效，如果有效则返回简短的用户信息。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，包含用户信息。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": {
            "id": 1,
            "user_name": "Bob",
            "signature": "This is my signature.",
            "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
            "is_followed": true
        }
    }
    ```

    其中 `data` 是一个数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|用户 ID|
    |`user_name`|字符串|是|用户名|
    |`signature`|字符串|是(可能为空)|用户签名|
    |`avatar`|字符串|是(可能为空)|用户头像|
    |`is_followed`|布尔值|是|用户是否被关注|

=== "错误"

    - 用户不存在

        > `400 Bad Request`

        ```json
        {
            "code": 12,
            "info": "USER_NOT_FOUND",
            "data": {}
        }
        ```

## 用户泛社交相关

### POST /user/follow/[user_id]

关注用户。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，若该用户并非自身且该用户未被关注，则进行关注。

=== "响应"

    - 成功关注

        成功关注用户，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 用户不存在

        > `400 Bad Request`

        ```json
        {
            "code": 12,
            "info": "USER_NOT_FOUND",
            "data": {}
        }
        ```

    - 关注用户为自身

        > `400 Bad Request`

        ```json
        {
            "code": 13,
            "info": "CANNOT_FOLLOW_SELF",
            "data": {}
        }
        ```

    - 用户已被关注

        > `400 Bad Request`

        ```json
        {
            "code": 14,
            "info": "INVALID_FOLLOWS",
            "data": {}
        }
        ```

### POST /user/unfollow/[user_id]

取关用户。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，若该用户并非自身且该用户已被关注，则进行取关。

=== "响应"

    - 成功取关

        成功取关用户，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 用户不存在

        > `400 Bad Request`

        ```json
        {
            "code": 12,
            "info": "USER_NOT_FOUND",
            "data": {}
        }
        ```

    - 取关用户为自身

        > `400 Bad Request`

        ```json
        {
            "code": 13,
            "info": "CANNOT_FOLLOW_SELF",
            "data": {}
        }
        ```

    - 用户未被关注

        > `400 Bad Request`

        ```json
        {
            "code": 14,
            "info": "INVALID_FOLLOWS",
            "data": {}
        }
        ```

### GET /user/followers/[user_id]

获取用户的粉丝列表。

=== "请求"

    请求正文无需附带内容。请求需要携带 query 参数，参数 page 代表需要获取的记录的页码。
    
    示例：

    ```HTTP
    /user/followers/1?page=5
    ```

=== "行为"

    后端接收到请求后，返回指定页码的用户粉丝列表。一页定义为 10 个用户，页码从 1 开始计数。

    若 page 不为正整数则应当报错，错误响应在下面定义。

    若 page 为正整数则总是正常响应，即使对应的页码并没有记录也是如此，此时返回的用户列表为空。

=== "响应"

    - 获取成功

        成功获取用户粉丝列表，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "page_count": 15,
                "page_data": [
                    {
                        "id": 12,
                        "user_name": "Bob",
                        "signature": "It's Me!",
                        "mail": "superBob@163.com",
                        "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                        "followers": 29,
                        "following": 16,
                        "register_time": "2023-05-09T08:03:04.517Z"
                    },
                    {
                        "id": 15,
                        "user_name": "Bob",
                        "signature": "Hello World!",
                        "mail": "superBob@163.com",
                        "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                        "followers": 9,
                        "following": 6,
                        "register_time": "2023-05-12T08:03:04.517Z"
                    }
                ]
            }
        }
        ```

=== "错误"

    - 页码非正整数

        > `400 Bad Request`

        ```json
        {
            "code": 6,
            "info": "INVALID_PAGES",
            "data": {}
        }
        ```

    - 用户不存在

        > `400 Bad Request`

        ```json
        {
            "code": 12,
            "info": "USER_NOT_FOUND",
            "data": {}
        }
        ```

### GET /user/followings/[user_id]

获取用户的关注列表。

=== "请求"

    请求正文无需附带内容。请求需要携带 query 参数，参数 page 代表需要获取的记录的页码。
    
    示例：

    ```HTTP
    /user/followings/1?page=5
    ```

=== "行为"

    后端接收到请求后，返回指定页码的用户关注列表。一页定义为 10 个用户，页码从 1 开始计数。

    若 page 不为正整数则应当报错，错误响应在下面定义。

    若 page 为正整数则总是正常响应，即使对应的页码并没有记录也是如此，此时返回的用户列表为空。

=== "响应"

    - 获取成功

        成功获取用户关注列表，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "page_count": 15,
                "page_data": [
                    {
                        "id": 12,
                        "user_name": "Bob",
                        "signature": "It's Me!",
                        "mail": "superBob@163.com",
                        "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                        "followers": 29,
                        "following": 16,
                        "register_time": "2023-05-09T08:03:04.517Z"
                    },
                    {
                        "id": 15,
                        "user_name": "Bob",
                        "signature": "Hello World!",
                        "mail": "superBob@163.com",
                        "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                        "followers": 9,
                        "following": 6,
                        "register_time": "2023-05-12T08:03:04.517Z"
                    }
                ]
            }
        }
        ```

=== "错误"

    - 页码非正整数

        > `400 Bad Request`

        ```json
        {
            "code": 6,
            "info": "INVALID_PAGES",
            "data": {}
        }
        ```

    - 用户不存在

        > `400 Bad Request`

        ```json
        {
            "code": 12,
            "info": "USER_NOT_FOUND",
            "data": {}
        }
        ```

### POST /user/message/post

用户发送私信。

=== "请求"

    在请求头中携带 Authorization 字段来记录 token，可通过 token 来确定用户身份。

    请求正文样例：

    ```json
    {
        "user_id": 3,
        "message": "Hello!"
    }
    ```

    各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`user_id`|整数|是|私信用户 ID|
    |`message`|字符串|是|私信内容|

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，如果有效且该私信用户非自身则发送私信。

=== "响应"

    - 成功发送私信

        若成功发送私信，则返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "sender": 1,
                "receiver": 4,
                "message": "Hello!",
                "pub_time": "2023-04-25T17:13:55.648217Z"
            }
        }
        ```

        各字段含义如下：

        |字段|类型|必选|含义|
        |-|-|-|-|
        |`sender`|整数|是|发送用户 ID|
        |`receiver`|整数|是|接收用户 ID|
        |`message`|字符串|是|私信内容|
        |`pub_time`|字符串|是|私信时间|

=== "错误"

    - 用户不存在

        > `400 Bad Request`

        ```json
        {
            "code": 12,
            "info": "USER_NOT_FOUND",
            "data": {}
        }
        ```

    - 私信用户为自身

        > `400 Bad Request`

        ```json
        {
            "code": 22,
            "info": "CANNOT_MESSAGE_SELF",
            "data": {}
        }
        ```

### GET /user/message/list

获取用户指定页数的私信对象记录，每个私信用户返回元信息、最后一条信息的内容与发送时间。

=== "请求"

    在请求头中携带 Authorization 字段来记录 token，可通过 token 来确定用户身份。
    
    请求需要携带 query 参数，参数 page 代表需要获取的消息列表的页码。
    
    示例：

    ```HTTP
    /user/message/list?page=5
    ```

=== "行为"

    后端接收到请求后，返回指定页码的用户关注列表，返回顺序默认按照时间倒序排列。一页定义为 10 条私信，页码从 1 开始计数。

    若 page 不为正整数则应当报错，错误响应在下面定义。

    若 page 为正整数则总是正常响应，即使对应的页码并没有记录也是如此，此时返回的私信列表为空。

=== "响应"

    - 成功获取私信记录

        成功获取私信记录，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "page_count": 2,
                "page_data": [
                    {
                        "user": {
                            "id": 2,
                            "user_name": "Bob",
                            "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                            "signature": "This is my future"
                        },
                        "message": {
                            "message": "See you next time!",
                            "pub_time": "2023-05-17T02:09:00.167Z",
                            "is_read": false
                        }
                    },
                    {
                        "user": {
                            "id": 3,
                            "user_name": "Alice",
                            "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                            "signature": "Hello world"
                        },
                        "message": {
                            "message": "Long time no see...",
                            "pub_time": "2023-05-17T02:08:29.733Z",
                            "is_read": true
                        }
                    }
                ]
            }
        }
        ```

=== "错误"

    - 页码非正整数

        > `400 Bad Request`

        ```json
        {
            "code": 6,
            "info": "INVALID_PAGES",
            "data": {}
        }
        ```

### GET /user/message/read/[user_id]

获取与指定用户的私信记录。

=== "请求"

    在请求头中携带 Authorization 字段来记录 token，可通过 token 来确定用户身份。
    
    请求需要携带 query 参数，参数 page 代表需要获取的消息列表的页码。
    
    示例：

    ```HTTP
    /user/message/read/4?page=5
    ```

=== "行为"

    后端接收到请求后，返回指定页码的用户私信内容，返回顺序默认按照时间倒序排列。

    将用户对其的所有私信状态置为已阅读，一页定义为 50 条私信，页码从 1 开始计数。

    若 page 不为正整数则应当报错，错误响应在下面定义。

    若 page 为正整数则总是正常响应，即使对应的页码并没有记录也是如此，此时返回的私信列表为空。

=== "响应"

    - 成功获取消息记录

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "page_count": 7,
                "page_data": [
                    {
                        "sender": 3,
                        "receiver": 1,
                        "message": "See you next time!",
                        "pub_time": "2023-05-17T02:08:29.733Z"
                    },
                    {
                        "sender": 1,
                        "receiver": 3,
                        "message": "Nice to chat",
                        "pub_time": "2023-05-17T02:02:54.144Z"
                    },
                    {
                        "sender": 3,
                        "receiver": 1,
                        "message": "What's up man?",
                        "pub_time": "2023-05-17T01:01:53.578Z"
                    },
                    {
                        "sender": 1,
                        "receiver": 3,
                        "message": "Hi",
                        "pub_time": "2023-05-17T01:01:22.343Z"
                    }
                ]
            }
        }
        ```

=== "错误"

    - 页码非正整数

        > `400 Bad Request`

        ```json
        {
            "code": 6,
            "info": "INVALID_PAGES",
            "data": {}
        }
        ```

    - 用户不存在

        > `400 Bad Request`

        ```json
        {
            "code": 12,
            "info": "USER_NOT_FOUND",
            "data": {}
        }
        ```

    - 私信用户为自身

        > `400 Bad Request`

        ```json
        {
            "code": 22,
            "info": "CANNOT_MESSAGE_SELF",
            "data": {}
        }
        ```

### GET /user/readhistory

获取用户访问的历史记录。

=== "请求"

    在请求头中携带 Authorization 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 page 代表需要获取的历史记录的页码。

        示例：

    ```HTTP
    /user/message/readhistory?page=5
    ```

=== "行为"

    后端接收到请求后，返回指定页码的用户私信内容。一页定义为 20 个 Gif 历史记录，页码从 1 开始计数。

    若 page 不为正整数则应当报错，错误响应在下面定义。

    若 page 为正整数则总是正常响应，即使对应的页码并没有记录也是如此，此时返回的私信列表为空。

=== "响应"

    - 历史记录获取成功

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "page_count": 15,
                "page_data": [
                    {
                        "data": {
                            "id": 117,
                            "title": "This is a Pretty Gif",
                            "width": 400,
                            "height": 250,
                            "duration": 5.2,
                            "uploader": "Bob",
                            "uploader_id": 4,
                            "category": "beauty",
                            "tags": ["beauty", "fun"],
                            "like": 412,
                            "pub_time": "2023-04-25T17:13:55.648217Z"
                        },
                        "visit_time": "2023-04-25T17:13:55.648217Z"
                    }
                ]
            }
        }
        ```

=== "错误"

    - 页码非正整数

        > `400 Bad Request`

        ```json
        {
            "code": 6,
            "info": "INVALID_PAGES",
            "data": {}
        }
        ```

### POST /user/readhistory

记录用户访问 Gif 的记录。

=== "请求"

    在请求头中携带 Authorization 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 id 代表用户点击的 Gif 的 ID。

        示例：

    ```HTTP
    /user/readhistory?id=5
    ```

=== "行为"

    后端接收到请求后，根据用户点击的 Gif 来进行用户标签、阅读历史等功能的更新。

    在用户历史记录中更新并记录此 Gif 访问状态。

=== "响应"

    - 历史记录更改成功

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - Gif 不存在

        > `400 Bad Request`

        ```json
        {
            "code": 9,
            "info": "GIFS_NOT_FOUND",
            "data": {}
        }
        ```

### GET /user/personalize

返回用户偏好的 Gif 图组.

=== "请求"

    在请求头中携带 Authorization 字段来记录 token，可通过 token 来确定用户身份。无需附带请求正文。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，并向搜索后端发送用户 tags 作为查询关键词的搜索请求。

    每次返回的 Gif 最多不超过 10 个，并默认按照关匹配程序进行排序。

=== "响应"

    - 获取推荐成功

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "gif_ids": [
                    "221", "252", "253", "254", "255",
                    "256", "257", "226", "227", "228"
                ]
            }
        }
        ```

=== "错误"

    本 API 不应该出现错误。

### POST /image/like/[gif_id]

点赞 Gif。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，若 Gif 存在，且用户未对该 Gif 点赞，则进行点赞。

=== "响应"

    - 成功点赞

        成功点赞 Gif，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - Gif 已被点赞过

        > `400 Bad Request`

        ```json
        {
            "code": 5,
            "info": "INVALID_LIKES",
            "data": {}
        }
        ```

    - Gif 不存在

        > `400 Bad Request`

        ```json
        {
            "code": 9,
            "info": "GIFS_NOT_FOUND",
            "data": {}
        }
        ```

### POST /image/cancellike/[gif_id]

取消点赞 Gif。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，若 Gif 存在，且用户已对该 Gif 点赞，则取消点赞。

=== "响应"

    - 成功取消点赞

        成功取消点赞 Gif，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - Gif 未被点赞过

        > `400 Bad Request`

        ```json
        {
            "code": 5,
            "info": "INVALID_LIKES",
            "data": {}
        }
        ```

    - Gif 不存在

        > `400 Bad Request`

        ```json
        {
            "code": 9,
            "info": "GIFS_NOT_FOUND",
            "data": {}
        }
        ```

### POST /image/comment/[gif_id]

评论 Gif。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文样例：

    ```json
    {
        "content": "这是一条子评论",
        "parent_id": 10
    }
    ```
    ```json
    {
        "content": "这是一条父评论"
    }
    ```

    其中 `parent_id` 为可选字段，表示该评论以 `id` 为 `parent_id` 的评论为二级评论。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。如果 Gif 存在，依据评论等级作出不同行为：
    
    如果评论为二级评论，先检验相应的一级评论是否存在。若存在，则创建一条二级评论。
    
    如果评论为一级评论，直接创建一条一级评论。

=== "响应"

    - 成功评论

        成功评论 Gif，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 2,
                "user": "Alice",
                "content": "你说得对，但是……",
                "pub_time": "2023-04-15T14:41:21.525Z"
            }
        }
        ```

=== "错误"

    - Gif 不存在

        > `400 Bad Request`

        ```json
        {
            "code": 9,
            "info": "GIFS_NOT_FOUND",
            "data": {}
        }
        ```

    - 父评论不存在

        > `400 Bad Request`

        ```json
        {
            "code": 11,
            "info": "COMMENTS_NOT_FOUND",
            "data": {}
        }
        ```

### DELETE /image/comment/delete/[comment_id]

删除 Gif 评论。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。
    
    如果评论存在，且评论发送者为该用户，则删除评论。

=== "响应"

    - 成功删除评论

        成功删除 Gif 评论，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 评论不存在

        > `400 Bad Request`

        ```json
        {
            "code": 11,
            "info": "COMMENTS_NOT_FOUND",
            "data": {}
        }
        ```

### GET /image/comment/[gif_id]

获取 Gif 的所有评论。

=== "请求"

    请求可以在请求头中携带 Authorization 字段，记录 token 值，判断评论是否被当前用户点赞。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，若 Gif 存在，且用户已对该 Gif 点赞，则取消点赞。

=== "响应"

    - 成功获取评论

        成功获取 Gif 评论，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": [
                {
                    "id": 2,
                    "user": "Alice",
                    "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                    "content": "这是一条测试评论",
                    "pub_time": "2023-04-16T07:32:50.906Z",
                    "like": 0,
                    "is_liked": false,
                    "replies": []
                },
                {
                    "id": 1,
                    "user": "Alice",
                    "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                    "content": "这是一条测试评论",
                    "pub_time": "2023-04-16T07:32:49.948Z",
                    "like": 1,
                    "is_liked": true,
                    "replies": [
                        {
                            "id": 3,
                            "user": "Alice",
                            "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                            "content": "这是一条测试评论",
                            "pub_time": "2023-04-16T07:32:55.238Z",
                            "like": 0,
                            "is_liked": false
                        }
                    ]
                }
            ]
        }
        ```

=== "错误"

    - Gif 不存在

        > `400 Bad Request`

        ```json
        {
            "code": 9,
            "info": "GIFS_NOT_FOUND",
            "data": {}
        }
        ```

### POST /image/comment/like/[comment_id]

点赞评论。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，若评论存在，且用户未对该评论点赞，则进行点赞。

=== "响应"

    - 成功点赞

        成功点赞评论，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 评论已被点赞过

        > `400 Bad Request`

        ```json
        {
            "code": 5,
            "info": "INVALID_LIKES",
            "data": {}
        }
        ```

    - 评论不存在

        > `400 Bad Request`

        ```json
        {
            "code": 11,
            "info": "COMMENTS_NOT_FOUND",
            "data": {}
        }
        ```

### POST /image/comment/cancellike/[comment_id]

取消点赞评论。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，若评论存在，且用户已对该评论点赞，则取消点赞。

=== "响应"

    - 成功取消点赞

        成功取消点赞评论，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {}
        }
        ```

=== "错误"

    - 评论未被点赞过

        > `400 Bad Request`

        ```json
        {
            "code": 5,
            "info": "INVALID_LIKES",
            "data": {}
        }
        ```

    - 评论不存在

        > `400 Bad Request`

        ```json
        {
            "code": 11,
            "info": "COMMENTS_NOT_FOUND",
            "data": {}
        }
        ```

## GIF 管理相关

### GET /allnews

返回主页新闻。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求需要携带参数，参数名称为`category`，其中`category`代表新闻类别。

    示例：

    ```shell
    "/allnews?category=home"
    ```

    现在支持的新闻类别暂定为`home、sport、tech、game、health、fashion、ent`等内容，详见`asyNc-web/dics/web-db`

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，如果有效则返回相应类别下的新闻内容，是否按用户进行喜好筛选由后期决定。

    **一次返回新闻不应该超过200条，按时间顺序返回最新的前200条**

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，包含新闻对象的数组。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": [
            {
                "id": 514,
                "title": "Breaking News",
                "url": "https://breaking.news",
                "picture_url": "https://breaking.news/picture.png",
                "media": "Foobar News",
                "pub_time": "2022-10-21T19:02:16.305Z",
                "is_favorite": false,
                "is_readlater": true,
            }
        ]
    }
    ```

    其中`data`是对应类别的新闻组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |:-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|新闻标题|
    |`url`|字符串|是|新闻网址|
    |`picture_url`|字符串|允许为null|新闻图片 URL|
    |`media`|字符串|是|媒体|
    |`pub_time`|字符串|是|新闻发布时间|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|

=== "错误"

    本 API 不应该出错。

### GET/newscount

=== "请求"

    无需特殊请求头

=== "行为"

    返回当前爬虫数据库中新闻总量

=== "响应"

    > 200 OK

    返回一个json格式的正文，包含新闻总量

    ```json
    {
        "code":0,
        "info":'success',
        "data":1145141919810
    }
    ```

    其中`data`即为数据库新闻总量，类型为`number`

=== "错误"

    此 API 不应该返回错误。

## 新闻检索相关

### POST /search

进行搜索。

=== "请求"

    请求附带 JSON 格式的正文。
    样例：

    ```json
    {
        "query": "Hello",
        "page": 2,
        "include": [
            "world",
            "again"
        ],
        "exclude": [
            "goodbye"
        ]
    }
    ```

    正文的 JSON 包含一个字典，字典的各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`query`|字符串|是|查询关键词|
    |`page`|整数|否|页码，正整数，默认为 1|
    |`include`|数组|否|必含词|
    |`exclude`|数组|否|排除词|
    |`sort`|布尔值|否|是否按时间排序，默认为 `false`|

    其中 `include` 与 `exclude` 为字符串数组，包含需要包括在内/排除在外的词。

=== "行为"

    后端接收到请求后，向搜索后端发送查询关键词的搜索请求，并返回指定页码的搜索结果，以及该搜索词的结果共有多少页。
    若 `sort` 为 `true`，则搜索排序逻辑为时间优先，否则为相关性优先，
    搜索结果应包含新闻正文中与搜索词相关的上下文，以及需要标红的位置。

    一页定义为 10 条搜索结果，页码从 1 开始计数。
    若 `page` 不为正整数则应当报错，错误响应在下面定义。
    若 `page` 为正整数则总是正常响应，即使对应的页码并没有搜索结果也是如此。
    此时返回的新闻列表为空。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，包含搜索结果的数组。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": {
            "page_count": 15,
            "news": [
                {
                    "id": 114,
                    "title": "Breaking News",
                    "media": "Foobar News",
                    "url": "https://breaking.news",
                    "pub_time": "2022-10-21T19:02:16.305Z",
                    "content": "BREAKING NEWS!!!",
                    "picture_url": "https://breaking.news/picture.png",
                    "title_keywords": [
                        [1, 3],
                        [7, 9],
                        [10, 15]
                    ],
                    "keywords": [
                        [1, 3],
                        [7, 9],
                        [10, 15]
                    ],
                    "is_favorite": false,
                    "is_readlater": true,
                }
            ]
        }
    }
    ```

    `page_count` 表示这个搜索词的结果一共有多少页。

    `news` 是一个数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`content`|字符串|是|新闻内容与关键词相关的上下文|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`title_keywords`|数组|是|标题中需要标红的关键词位置|
    |`keywords`|数组|是|需要标红的关键词位置|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|

    其中 `title_keywords` 和 `keywords` 是一个数组，每个元素是一个包含两个整数的数组，为一个需要标红的关键词的位置，从 0 开始计数，左闭右开。

=== "错误"

    - 页码不为正整数

        > `400 Bad Request`

        ```json
        {
            "code": 5,
            "info": "INVALID_PAGE",
            "data": {}
        }
        ```

### POST /search/suggest

获取搜索建议。

=== "请求"

    请求附带 JSON 格式的正文。
    样例：

    ```json
    {
        "query": "Hello",
        "include": [
            "world",
            "again"
        ],
        "exclude": [
            "goodbye"
        ]
    }
    ```

    正文的 JSON 包含一个字典，字典的各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`query`|字符串|是|查询关键词|
    |`include`|数组|否|必含词|
    |`exclude`|数组|否|排除词|

    其中 `include` 与 `exclude` 为字符串数组，包含需要包括在内/排除在外的词。

=== "行为"

    后端接收到请求后，向搜索后端发送获取搜索建议的请求，并返回一个搜索建议列表。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，包含搜索建议的数组。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": {
            "suggestions": [
                "Hello world again!",
                "Hello world again and again!"
                "Hello world again, again and again!"
            ]
        }
    }
    ```

=== "错误"

    此 API 不应该返回错误。

## 个性化相关

### POST /personalize

=== "请求"

    请求附带 JSON 格式的正文。
    样例：

    ```json
    {
        "query": "Hello",
    }
    ```

    正文的 JSON 包含一个字典，字典的各字段含义如下：

    | 字段    | 类型   | 必选 | 含义       |
    | ------- | ------ | ---- | ---------- |
    | `query` | 字符串 | 是   | 查询关键词 |

=== "行为"

    后端接收到请求后，向搜索后端发送查询关键词的搜索请求，每次返回的新闻最多不超过200条，并默认按照关匹配程序进行排序。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，包含搜索结果的数组。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": {
            "news": [
                {
                    "id": 114,
                    "title": "Breaking News",
                    "media": "Foobar News",
                    "url": "https://breaking.news",
                    "pub_time": "2022-10-21T19:02:16.305Z",
                    "picture_url": "https://breaking.news/picture.png",
                    "is_favorite": false,
                    "is_readlater": true,
                }
            ]
        }
    }
    ```

    `news` 是一个数组，其中每个对象各字段含义如下：其中`data`是对应类别的新闻组，其中每个对象各字段含义如下：

    | 字段           | 类型   | 必选       | 含义         |
    | :------------- | ------ | ---------- | ------------ |
    | `id`           | 整数   | 是         | 新闻 ID      |
    | `title`        | 字符串 | 是         | 新闻标题     |
    | `url`          | 字符串 | 是         | 新闻网址     |
    | `picture_url`  | 字符串 | 允许为null | 新闻图片 URL |
    | `media`        | 字符串 | 是         | 媒体         |
    | `pub_time`     | 字符串 | 是         | 新闻发布时间 |
    | `is_favorite`  | 布尔   | 是         | 在收藏中     |
    | `is_readlater` | 布尔   | 是         | 在阅读列表中 |

=== "错误"

    此 API 不应该返回错误。

### GET /history

获取用户访问新闻的历史记录。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `page` 代表需要获取的历史记录的页码。

    示例：

    ```
    /history?page=5
    ```

=== "行为"

    后端接收到请求后，返回指定页码的用户历史记录。

    一页定义为 10 条新闻，页码从 1 开始计数。

    若 `page` 不为正整数则应当报错，错误响应在下面定义。

    若 `page` 为正整数则总是正常响应，即使对应的页码并没有历史记录也是如此。

    此时返回的新闻列表为空。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，包含新闻的数组。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": {
            "page_count": 15,
            "news": [
                {
                    "id": 114,
                    "title": "Breaking News",
                    "media": "Foobar News",
                    "url": "https://breaking.news",
                    "pub_time": "2022-10-21T19:02:16.305Z",
                    "picture_url": "https://breaking.news/picture.png",
                    "is_favorite": false,
                    "is_readlater": true,
                    "visit_time": "2022-10-21T19:02:16.305Z"
                }
            ]
        }
    }
    ```

    `page_count` 表示历史记录一共有多少页。

    `news` 是一个数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|
    |`visit_time`|字符串|是|最后点击时间|

=== "错误"

    - 页码不为正整数

        > `400 Bad Request`

        ```json
        {
            "code": 5,
            "info": "INVALID_PAGE",
            "data": {}
        }
        ```

### POST /history

记录用户点击行为。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `id` 代表用户点击的新闻 ID。

    示例：

    ```
    /history?id=191
    ```

=== "行为"

    后端可以根据用户点击的新闻来进行用户标签、搜索历史等功能的更新。

    具体而言，若新闻 ID 存在，则在历史记录中记录此新闻，然后若此 ID 出现在稍后再看列表中，则将其从中删去。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，其中不携带数据。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": {}
    }
    ```

=== "错误"

    - 新闻 ID 不存在

        > `404 Not Found`

        ```json
        {
            "code": 9,
            "info": "NEWS_NOT_FOUND",
            "data": {}
        }
        ```

### DELETE /history

删除某一条历史记录。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `id` 代表需要删除的历史记录的新闻 ID。

    示例：

    ```
    /history?id=981
    ```

=== "行为"

    若新闻 ID 存在，则将其从历史记录中删去。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，其中包含一个新闻数组，为历史记录的首页。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": [
            {
                "id": 114,
                "title": "Breaking News",
                "media": "Foobar News",
                "url": "https://breaking.news",
                "pub_time": "2022-10-21T19:02:16.305Z",
                "picture_url": "https://breaking.news/picture.png",
                "is_favorite": false,
                "is_readlater": true,
                "pub_time": "2022-10-21T19:02:16.305Z",
                "pub_time": "2022-10-21T19:02:16.305Z"
            }
        ]
    }
    ```

    `data` 是一个至多包含 10 条新闻的数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|
    |`visit_time`|字符串|是|最后点击时间|

=== "错误"

    - 新闻 ID 不存在

        > `404 Not Found`

        ```json
        {
            "code": 9,
            "info": "NEWS_NOT_FOUND",
            "data": {}
        }
        ```

### GET /readlater

获取用户的稍后再看列表。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `page` 代表需要获取的稍后再看列表的页码。

    示例：

    ```
    /readlater?page=5
    ```

=== "行为"

    后端接收到请求后，返回指定页码的用户稍后再看列表。

    一页定义为 10 条新闻，页码从 1 开始计数。
    若 `page` 不为正整数则应当报错，错误响应在下面定义。
    若 `page` 为正整数则总是正常响应，即使对应的页码并没有稍后再看列表也是如此。
    此时返回的新闻列表为空。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，包含新闻的数组。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": {
            "page_count": 15,
            "news": [
                {
                    "id": 114,
                    "title": "Breaking News",
                    "media": "Foobar News",
                    "url": "https://breaking.news",
                    "pub_time": "2022-10-21T19:02:16.305Z",
                    "picture_url": "https://breaking.news/picture.png",
                    "summary": "summary",
                    "is_favorite": false,
                    "is_readlater": true,
                    "pub_time": "2022-10-21T19:02:16.305Z"
                }
            ]
        }
    }
    ```

    `page_count` 表示稍后再看列表一共有多少页。

    `news` 是一个数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`summary`|字符串|是|摘要|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|
    |`visit_time`|字符串|是|最后点击时间|

=== "错误"

    - 页码不为正整数

        > `400 Bad Request`

        ```json
        {
            "code": 5,
            "info": "INVALID_PAGE",
            "data": {}
        }
        ```

### POST /readlater

将新闻加入稍后再看列表。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `id` 代表加入稍后再看列表的新闻 ID。

    示例：

    ```
    /readlater?id=191
    ```

=== "行为"

    若新闻 ID 存在，将其加入稍后再看列表。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，其中包含一个新闻数组，为稍后再看列表的首页。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": [
            {
                "id": 114,
                "title": "Breaking News",
                "media": "Foobar News",
                "url": "https://breaking.news",
                "pub_time": "2022-10-21T19:02:16.305Z",
                "picture_url": "https://breaking.news/picture.png",
                "summary": "summary",
                "is_favorite": false,
                "is_readlater": true,
                "pub_time": "2022-10-21T19:02:16.305Z"
            }
        ]
    }
    ```

    `data` 是一个至多包含 10 条新闻的数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`summary`|字符串|是|摘要|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|
    |`visit_time`|字符串|是|最后点击时间|

=== "错误"

    - 新闻 ID 不存在

        > `404 Not Found`

        ```json
        {
            "code": 9,
            "info": "NEWS_NOT_FOUND",
            "data": {}
        }
        ```

### DELETE /readlater

删除某一条稍后再看新闻。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `id` 代表需要删除的稍后再看的新闻 ID。

    示例：

    ```
    /readlater?id=981
    ```

=== "行为"

    若新闻 ID 存在，则将其从稍后再看列表中删去。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，其中包含一个新闻数组，为稍后再看列表的首页。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": [
            {
                "id": 114,
                "title": "Breaking News",
                "media": "Foobar News",
                "url": "https://breaking.news",
                "pub_time": "2022-10-21T19:02:16.305Z",
                "picture_url": "https://breaking.news/picture.png",
                "summary": "summary",
                "is_favorite": false,
                "is_readlater": true,
                "pub_time": "2022-10-21T19:02:16.305Z"
            }
        ]
    }
    ```

    `data` 是一个至多包含 10 条新闻的数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`summary`|字符串|是|摘要|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|
    |`visit_time`|字符串|是|最后点击时间|

=== "错误"

    - 新闻 ID 不存在

        > `404 Not Found`

        ```json
        {
            "code": 9,
            "info": "NEWS_NOT_FOUND",
            "data": {}
        }
        ```

### GET /favorites

获取用户的收藏夹。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `page` 代表需要获取的收藏夹的页码。

    示例：

    ```
    /favorites?page=5
    ```

=== "行为"

    后端接收到请求后，返回指定页码的用户收藏夹。

    一页定义为 10 条新闻，页码从 1 开始计数。
    若 `page` 不为正整数则应当报错，错误响应在下面定义。
    若 `page` 为正整数则总是正常响应，即使对应的页码并没有收藏夹也是如此。
    此时返回的新闻列表为空。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，包含新闻的数组。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": {
            "page_count": 15,
            "news": [
                {
                    "id": 114,
                    "title": "Breaking News",
                    "media": "Foobar News",
                    "url": "https://breaking.news",
                    "pub_time": "2022-10-21T19:02:16.305Z",
                    "picture_url": "https://breaking.news/picture.png",
                    "summary": "summary",
                    "is_favorite": false,
                    "is_readlater": true,
                    "pub_time": "2022-10-21T19:02:16.305Z"
                }
            ]
        }
    }
    ```

    `page_count` 表示收藏夹一共有多少页。

    `news` 是一个数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`summary`|字符串|是|摘要|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|
    |`visit_time`|字符串|是|最后点击时间|

=== "错误"

    - 页码不为正整数

        > `400 Bad Request`

        ```json
        {
            "code": 5,
            "info": "INVALID_PAGE",
            "data": {}
        }
        ```

### POST /favorites

将新闻加入收藏夹。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `id` 代表加入收藏夹的新闻 ID。

    示例：

    ```
    /favorites?id=191
    ```

=== "行为"

    若新闻 ID 存在，将其加入收藏夹。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，其中包含一个新闻数组，为收藏夹的首页。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": [
            {
                "id": 114,
                "title": "Breaking News",
                "media": "Foobar News",
                "url": "https://breaking.news",
                "pub_time": "2022-10-21T19:02:16.305Z",
                "picture_url": "https://breaking.news/picture.png",
                "summary": "summary",
                "is_favorite": false,
                "is_readlater": true,
                "pub_time": "2022-10-21T19:02:16.305Z"
            }
        ]
    }
    ```

    `data` 是一个至多包含 10 条新闻的数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`summary`|字符串|是|摘要|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|
    |`visit_time`|字符串|是|最后点击时间|

=== "错误"

    - 新闻 ID 不存在

        > `404 Not Found`

        ```json
        {
            "code": 9,
            "info": "NEWS_NOT_FOUND",
            "data": {}
        }
        ```

### DELETE /favorites

删除某一条收藏。

=== "请求"

    在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来确定用户身份。

    请求需要携带 query 参数，参数 `id` 代表需要删除的收藏的新闻 ID。

    示例：

    ```
    /favorites?id=981
    ```

=== "行为"

    若新闻 ID 存在，则将其从收藏夹中删去。

=== "响应"

    > `200 OK`

    返回一个 JSON 格式的正文，其中包含一个新闻数组，为收藏夹的首页。

    ```json
    {
        "code": 0,
        "info": "SUCCESS",
        "data": [
            {
                "id": 114,
                "title": "Breaking News",
                "media": "Foobar News",
                "url": "https://breaking.news",
                "pub_time": "2022-10-21T19:02:16.305Z",
                "picture_url": "https://breaking.news/picture.png",
                "summary": "summary",
                "is_favorite": false,
                "is_readlater": true,
                "pub_time": "2022-10-21T19:02:16.305Z"
            }
        ]
    }
    ```

    `data` 是一个至多包含 10 条新闻的数组，其中每个对象各字段含义如下：

    |字段|类型|必选|含义|
    |-|-|-|-|
    |`id`|整数|是|新闻 ID|
    |`title`|字符串|是|标题|
    |`media`|字符串|是|媒体|
    |`url`|字符串|是|新闻 URL|
    |`pub_time`|字符串|是|新闻发布时间|
    |`picture_url`|字符串|否|图片 URL，若有|
    |`summary`|字符串|是|摘要|
    |`is_favorite`|布尔|是|在收藏中|
    |`is_readlater`|布尔|是|在阅读列表中|
    |`visit_time`|字符串|是|最后点击时间|

=== "错误"

    - 新闻 ID 不存在

        > `404 Not Found`

        ```json
        {
            "code": 9,
            "info": "NEWS_NOT_FOUND",
            "data": {}
        }
        ```

