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

## **用户管理相关 /user**

### POST /register

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

        成功发送用户注册校验邮件，则返回如下响应：

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

### POST /salt

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

        成功返回 salt，则返回如下响应：

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

### GET /verify/[token]

校验一个注册的用户。

=== "请求"

    无需请求正文。

=== "行为"

    后端检验 token 对应的待验证用户是否存在、未验证且处在待验证时间内。

    满足这三个条件，则注册用户、将密码进行哈希后存储并返回登录 Token，否则返回错误信息。

=== "响应"

    - 成功校验用户

        成功校验用户，返回如下响应并转入已登录状态：

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

### POST /login

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

### POST /modifypassword

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

        成功修改密码，返回如下响应：
        
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

### POST /avatar

修改用户头像。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求正文是包含头像文件的 form-data 表单：

    ```form-data
    {
        "file": avatar_file
    }
    ```

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。
    
    接下来判断上传头像文件是否存在，文件存在且格式正确则进行修改，并返回头像的 base64 编码。

=== "响应"

    - 修改成功

        成功修改头像，返回如下响应：
        
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

### GET /avatar

获取用户头像。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，并返回相应用户头像的 base64 编码。

=== "响应"

    - 获取成功

        成功获取，返回如下响应：
        
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

### POST /signature

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

        成功修改签名，返回如下响应：

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

### POST /logout

登出用户。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 `token` 是否有效。如有效，将该用户登出。

=== "响应"

    - 成功登出

        成功登出，返回如下响应：

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

### POST /checklogin

检查用户登录状态。

=== "请求"

    请求需要在请求头中携带 Authorization 字段，记录 token 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，检验 `token` 是否有效，并返回登陆状态。

=== "响应"

    - 登录状态有效

        登录状态有效，返回如下响应：

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

### GET /profile/[user_id]

返回用户主页信息。

=== "请求"

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验用户 id 是否有效，并返回用户相关信息。

=== "响应"

    - 用户存在

        用户信息存在，返回如下响应：

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

### GET /info/[user_id]

返回简短的用户信息。

=== "请求"

    请求可以在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 和 id 是否有效，如果有效则返回简短的用户信息。

=== "响应"

    - 用户存在

        用户信息存在，返回如下响应：

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

## **泛社交相关 /user**

### POST /follow/[user_id]

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

### POST /unfollow/[user_id]

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

### GET /followers/[user_id]

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

### GET /followings/[user_id]

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

### POST /message/post

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

### GET /message/list

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

### GET /message/read/[user_id]

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

### GET /readhistory

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

        用户历史记录获取成功，返回如下响应：
        
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

### POST /readhistory

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

        用户历史记录添加成功，返回如下响应：
        
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

### GET /personalize

返回用户偏好的 Gif 图组.

=== "请求"

    在请求头中携带 Authorization 字段来记录 token，可通过 token 来确定用户身份。无需附带请求正文。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，并向搜索后端发送用户 tags 作为查询关键词的搜索请求。

    每次返回的 Gif 最多不超过 10 个，并默认按照关匹配程序进行排序。

=== "响应"

    - 获取推荐成功

        用户个性推荐获取成功，返回如下响应：
        
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

## **泛社交相关 /image**

### POST /like/[gif_id]

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

### POST /cancellike/[gif_id]

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

### POST /comment/[gif_id]

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

### DELETE /comment/delete/[comment_id]

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

### GET /comment/[gif_id]

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

### POST /comment/like/[comment_id]

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

### POST /comment/cancellike/[comment_id]

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

## **GIF 管理相关 /image**

### POST /upload

上传 Gif。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求正文是包含 Gif 文件的 form-data 表单：

    ```form-data
    {
        "file": gif_file
        "title": "Pretty Cat"
        "category": "animals"
        "tags": ["funny", "cat"]
    }
    ```

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。
    
    若经过 Hash 检测得该 Gif 已经被上传过, 直接返回相应 Gif 的 id，否则返回 Gif 内容。

=== "响应"

    - Gif 已经被上传过

        Gif 已经被上传过，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 514,
                "duplication": true
            }
        }
        ```

    - Gif 未被上传过

        Gif 未被上传过，返回如下响应：

        > `200 OK`
        
        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 514,
                "duplication": false
            }
        }
        ```

=== "错误"

    - Gif 不存在或格式非法

        > `400 Bad Request`

        ```json
        {
            "code": 10,
            "info": "INVALID_GIF",
            "data": {}
        }
        ```

### POST /update/[gif_id]

更改 Gif 类别标签信息。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求正文样例：

    ```json
    {
        "category": "animals",
        "tags": ["funny", "cat"]
    }
    ```

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。如果有效则更新 Gif 内容。

=== "响应"

    - Gif 信息更改成功

        Gif 信息更改成功，返回如下响应：
        
        > `200 OK`
        
        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "id": 514
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

### POST /resize

压缩并上传 Gif。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求正文是包含 Gif 文件的 form-data 表单：

    ```form-data
    {
        "file": gif_file
        "title": "Pretty Cat"
        "category": "animals"
        "tags": ["funny", "cat"]
        "ratio": "0.8"
    }
    ```

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。如果有效则创建 Gif 压缩任务。

=== "响应"

    - 任务创建成功

        任务创建成功，返回如下响应：
        
        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "task_id": "ec1e476d-4f95-4d5e-94f3-b094de82c500",
                "task_status": "PENDING"
            }
        }
        ```

=== "错误"

    - Gif 不存在或格式非法

        > `400 Bad Request`

        ```json
        {
            "code": 10,
            "info": "INVALID_GIF",
            "data": {}
        }
        ```

    - 压缩比例不为 0 ~ 1 浮点数

        > `400 Bad Request`

        ```json
        {
            "code": 21,
            "info": "INVALID_RATIO",
            "data": {}
        }
        ```

### POST /video

上传视频转 Gif。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    请求正文是包含视频文件的 form-data 表单：

    ```form-data
    {
        "file": video_file,
        "title": "Pretty Cat",
        "category": "animals"
        "tags": ["funny", "cat"]
    }
    ```

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。如果有效则创建视频转换任务。

=== "响应"

    - 任务创建成功

        任务创建成功，返回如下响应：
        
        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "task_id": "ec1e476d-4f95-4d5e-94f3-b094de82c500",
                "task_status": "PENDING"
            }
        }
        ```

=== "错误"

    - 视频不存在或格式非法

        > `400 Bad Request`

        ```json
        {
            "code": 19,
            "info": "INVALID_VIDEO",
            "data": {}
        }
        ```

### POST /watermark/[gif_id]

Gif 添加水印。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    无需附带请求正文。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。如果用户为 Gif 上传者则创建水印添加任务。

=== "响应"

    - 任务创建成功

        任务创建成功，返回如下响应：
        
        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "task_id": "ec1e476d-4f95-4d5e-94f3-b094de82c500",
                "task_status": "PENDING"
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

### GET /taskcheck

检查用户非阻塞任务状态。

=== "请求"

    请求需要在请求头中携带 `Authorization` 字段，记录 `token` 值。

    无需附带请求正文。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效。
    
    返回当前用户所有任务的状态，如果成功同时返回任务结果。

=== "响应"

    - 任务状态获取成功

        任务状态获取成功，返回如下响应：
        
        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "task_count": 5,
                "task_data": [
                    {
                        "task_id": "ec1e476d-4f95-4d5e-94f3-b094de82c500",
                        "task_type": "resize",
                        "task_status": "STARTED",
                        "task_time": "2023-04-15T13:56:38.484Z"
                    },
                    {
                        "task_id": "ec1e476d-4f95-4d5e-94f3-b094de82c500",
                        "task_type": "watermark",
                        "task_status": "STARTED",
                        "task_time": "2023-04-15T13:57:38.484Z"
                    },
                    {
                        "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                        "task_type": "video",
                        "task_status": "SUCCESS",
                        "task_time": "2023-04-15T13:58:38.484Z",
                        "task_result": {
                            "id": 6
                        }
                    }
                ]
            }
        }
        ```

        `task_status` 有 `SATRTED`，`SUCCESS`，`FAILURE` 状态。

        `task_type` 有三种格式：`resize`，`watermark`，`video`。

        - Gif resize

            === "成功压缩上传"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "resize",
                    "task_status": "SUCCESS",
                    "task_time": "2023-04-15T13:58:38.484Z",
                    "task_result": {
                        "id": 234,
                        "duplication": false
                    }
                }
                ```

            === "重复压缩上传"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "resize"
                    "task_status": "SUCCESS",
                    "task_time": "2023-04-15T13:58:38.484Z",
                    "task_result": {
                        "id": 234,
                        "duplication": true
                    }
                }
                ```

            === "压缩失败"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "resize",
                    "task_status": "FAILURE",
                    "task_time": "2023-04-15T13:58:38.484Z"
                }
                ```

            === "压缩超时"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "resize",
                    "task_status": "FAILURE",
                    "task_time": "2023-04-15T13:58:38.484Z",
                    "task_result": {
                        "code": 23,
                        "info": "TOO_LONG_TIME"
                    }
                }
                ```

        - Gif watermark

            === "成功添加水印"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "watermark",
                    "task_status": "SUCCESS",
                    "task_time": "2023-04-15T13:58:38.484Z",
                    "task_result": {
                        "id": 234
                    }
                }
                ```

            === "Gif 过小"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "watermark",
                    "task_status": "FAILURE",
                    "task_time": "2023-04-15T13:58:38.484Z",
                    "task_result": {
                        "id": 234,
                        "code": 20,
                        "info": "GIF_TOO_SMALL"
                    }
                }
                ```

            === "添加水印失败"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "watermark",
                    "task_status": "FAILURE",
                    "task_time": "2023-04-15T13:58:38.484Z"
                }
                ```

            === "添加水印超时"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "watermark",
                    "task_status": "FAILURE",
                    "task_time": "2023-04-15T13:58:38.484Z",
                    "task_result": {
                        "code": 23,
                        "info": "TOO_LONG_TIME"
                    }
                }
                ```

        - Gif video

            === "成功转换视频"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "video",
                    "task_status": "SUCCESS",
                    "task_time": "2023-04-15T13:58:38.484Z",
                    "task_result": {
                        "id": 234,
                        "duplication": false
                    }
                }
                ```

            === "视频转换失败"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "video",
                    "task_status": "FAILURE",
                    "task_time": "2023-04-15T13:58:38.484Z"
                }
                ```

            === "视频转换超时"

                ```json
                {
                    "task_id": "efe4811e-bdbf-49e0-80ad-a93749159cd0",
                    "task_type": "video",
                    "task_status": "FAILURE",
                    "task_time": "2023-04-15T13:58:38.484Z",
                    "task_result": {
                        "code": 23,
                        "info": "TOO_LONG_TIME"
                    }
                }
                ```

=== "错误"

    本 API 不应返回错误。

### GET /detail/[gif_id]

返回 Gif 信息。

=== "请求"

    可以在请求头中携带 `Authorization` 字段来记录 token，可通过 token 来判断 Gif 是否被当前用户点赞。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 Gif id 是否有效，并返回 Gif 相关信息。

=== "响应"

    - Gif 存在

        Gif 存在，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "Succeed",
            "data": {
                "gif_data": {
                    "id": 344,
                    "title": "gif with tags: cute",
                    "uploader": "spider",
                    "width": 312,
                    "height": 300,
                    "category": "cute",
                    "tags": [
                        "cute", "cat"
                    ],
                    "duration": 1.92,
                    "pub_time": "2023-05-10T10:52:35.620Z",
                    "like": 0,
                    "is_liked": false
                },
                "user_data": {
                    "id": 38,
                    "user_name": "spider",
                    "signature": "",
                    "mail": "gifexplorer_spider@126.com",
                    "avatar": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAIACAIAAAB7GkOtAAABBmlDQ1BJQ0MgUHJvZmlsZQAAeJxjYGCSYAACFgMGhty8kqIgdyeFiMgoBQYkkJhcXMCAGzAyMHy7BiIZGC7r4lGHC3CmpBYnA+kPQFxSBLQcaGQKkC2SDmFXgNhJEHYPiF0UEuQMZC8AsjXSkdhJSOzykoISIPsESH1yQRGIfQfItsnNKU1GuJuBJzUvNBhIRwCxDEMxQxCDO4MTGX7ACxDhmb+IgcHiKwMD8wSEWNJMBobtrQwMErcQYipAP/C3MDBsO1+QWJQIFmIBYqa0NAaGT8sZGHgjGRiELzAwcEVj2oGICxx+VQD71Z0hHwjTGXIYUoEingx5DMkMekCWEYMBgyGDGQCSpUCz8yM2qAABAABJREFUeJzk/Vmzbdl1Hoh93xhzrrV2c7rbZt5MZIMu0RAgKJIig0VSpa4kMUoul0vh0IPDoQi7XuwIh3+M3yocDkf4QU1VOaSyJFOiRLJEASTRkgAJIIHsM+/N299zzm7WWnOOMfyw9jn3ZuZNIBNIUCVrBCLz5ME+e88912zG+MY3vsH4HwIVGEfzoajXhcY83evGu10P+qo/XW22Q81mXbXcz/==",
                    "followers": 5,
                    "following": 3,
                    "register_time": "2023-05-10T10:34:42.789Z",
                    "is_followed": true
                }
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

### DELETE /detail/[gif_id]

删除 Gif。

=== "请求"

    需要在请求头中携带 `Authorization` 字段来记录 token。

    请求正文无需附带内容。

=== "行为"

    后端接受到请求之后，先检验 token 是否有效，并删除相应 Gif。

=== "响应"

    - 删除 Gif 成功

        删除 Gif 成功，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "Succeed",
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

### GET /previewlow/[gif_id]

低分辨率预览 Gif。

=== "请求"

    无需附带请求正文。

=== "行为"

    返回低分辨率 Gif 内容进行预览。

=== "响应"

    - Gif 预览成功
        
        Gif 预览成功，返回一个 HttpResponse 格式的 FileWrapper。

        > 200 OK

        ```python
        ...
        file_wrapper = FileWrapper(gif_file)
        response = HttpResponse(file_wrapper, content_type='image/gif')
        response['Content-Disposition'] = f'inline; filename="{gif.name}"'
        return response
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

### GET /image/preview/[gif_id]

预览 Gif。

=== "请求"

    无需附带请求正文。

=== "行为"

    返回 Gif 内容进行预览。

=== "响应"

    - Gif 预览成功
        
        Gif 预览成功，返回如下响应，返回一个 HttpResponse 格式的 FileWrapper。

        > 200 OK

        ```python
        ...
        file_wrapper = FileWrapper(gif_file)
        response = HttpResponse(file_wrapper, content_type='image/gif')
        response['Content-Disposition'] = f'inline; filename="{gif.name}"'
        return response
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

### GET /download/[gif_id]

下载 Gif。

=== "请求"

    无需附带请求正文。

=== "行为"

    返回 Gif 内容进行下载。

=== "响应"

    - Gif 下载成功
        
        Gif 下载成功，返回一个 HttpResponse 格式的 FileWrapper。

        > 200 OK

        ```python
        ...
        file_wrapper = FileWrapper(gif_file)
        response = HttpResponse(file_wrapper, content_type='application/octet-stream')
        response['Content-Disposition'] = f'attachment; filename="{gif.title}.gif"'
        return response
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

### POST /downloadzip

批量下载 Gif。

=== "请求"

    请求正文样例：

    ```json
    {
        "gif_ids": [1, 2, 3, 4]
    }
    ```

=== "行为"

    返回相应 id 的 Gif 进行下载。

=== "响应"

    - Gif 批量下载成功
        
        Gif 批量下载成功，返回一个 HttpResponse 格式的 ZipFile

        > 200 OK

        ```python
        ...
        with zipfile.ZipFile(zip_buffer, mode='w') as zip_file:
        for gif in gifs:
            gif_file = open(gif.giffile.file.path, 'rb')
            zip_file.writestr(f"{gif.title}.gif", gif_file.read())
            gif_file.close()
        response = HttpResponse(zip_buffer.getvalue(), content_type='application/zip')
        time = datetime.datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
        response['Content-Disposition'] = f'attachment; filename="{time}.zip"'
        return response
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

### GET /gifscount

批量下载 Gif。

=== "请求"

    无需附带请求正文。

=== "行为"

    返回数据库中 Gif 总量。

=== "响应"

    - Gif 总量获取成功
        
        Gif 总量获取成功，返回如下响应：

        > 200 OK

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": 79473
        }
        ```

=== "错误"

    此 API 不应该返回错误。

## **GIF 检索相关 /image**

### POST /search

搜索 Gif。

=== "请求"

    请求正文样例：

    ```json
    {
        "target": "title",
        "keyword": "cat picture",
        "filter": [
            {"range": {"width": {"gte": 0, "lte": 2000}}},
            {"range": {"height": {"gte": 0, "lte": 2000}}},
            {"range": {"duration": {"gte": 0, "lte": 100}}}
        ],
        "category": "animal",
        "tags": ["animal", "cat"],
        "type": "perfect",
        "page": 2
    }
    ```

    各字段含义如下：

    |字段|类型|必选|默认值|含义|
    |-|-|-|-|-|
    |`target`|字符串|否|`title`|搜索使用的指标|
    |`keyword`|字符串|否|` `|搜索关键词|
    |`filter`|数组|否|`[]`|Gif 尺寸、时长|
    |`category`|字符串|否|` `|Gif 类别|
    |`tags`|数组|否|`[]`|Gif 标签|
    |`type`|字符串|否|`partial`|搜索模式|
    |`page`|整数|否|1|页码|

    - `target` 和 `keyword` 必须都非空串，或者都为空串才合法。

    - `target` 必须为 ` `，`uploader`，`title` 三者之一。
  
    - `type` 必须为 `perfect`，`partial`，`fuzzy`，`regex`，`related` 之一。
  
=== "行为"

    后端接收到请求后，向搜索后端发送查询关键词的搜索请求，返回指定页码的搜索结果，以及该搜索词的结果页数。

    一页定义为 10 条搜索结果，页码从 1 开始计数。

    若 `page` 不为正整数则应当报错，错误响应在下面定义。

    若 `page` 为正整数则总是正常响应，即使对应的页码并没有搜索结果也是如此。此时返回的 Gif 列表为空。

    - `target` 和 `keyword`

        - 如果 `type` 不为 `regex`：

            - 若 `target` 和 `keyword` 同时为空串，表示没有本项限制。
  
            - 恰有一者为空，返回格式错误。
  
        - 如果 `type` 为 `regex`：

            - `target` 必须为 `uploader`，`title` 之一，缺省值为 `title`。

            - `keyword` 若为空串，表示没有本项限制。

    - 若 `filter` 为空数组，表示没有本项限制。
  
    - 若 `category` 为空串，表示没有本项限制。

    - `tags`

        - 若 `tags` 为空数组，表示没有本项限制。

        - 请求的每个 `tag` 在返回的图片中都包含。换言之，请求的 `tags` 是返回的每个图片的 `tags` 的子集。

    - `type`
        - 若 `type` 为 `perfect`，则进行精确匹配搜索（默认选项）；

        - 若 `type` 为 `partial`，则进行部分匹配搜索；

        - 若 `type` 为 `fuzzy`，则进行模糊搜索（字符意义上的模糊）；

        - 若 `type` 为 `related`，则进行关联搜索（语义关联）；

        - 若 `type` 为 `regex`，则通过正则表达式搜索。

            - 用户输入的正则表达式语法采用 `python` 正则表达式语法，并且需要转义。例如，要搜索以 `.com` 结尾的内容，需要输入 `\.com$`；要搜索包含 `apple.` 的内容，需要输入 `apple\.`。
            
            - 特别地，如果使用空串，表示搜索任意内容。

    - 搜索排序逻辑为相关性优先。
  
=== "响应"

    - 获取搜索结果成功
        
        获取搜索结果成功，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "page_count": 15,
                "page_data": [
                    {
                        "id": 117,
                        "name": "pretty.gif",
                        "title": "Pretty Gif",
                        "width": 400,
                        "height": 250,
                        "duration": 5.2,
                        "uploader": "Bob",
                        "uploader_id": 4,
                        "category": "beauty",
                        "tags": ["beauty", "fun"],
                        "like": 412,
                        "pub_time": "2023-04-25T17:13:55.648217Z"
                    }
                ]
            }
        }
        ```

=== "错误"

    - 页码不为正整数

        > `400 Bad Request`

        ```json
        {
            "code": 6,
            "info": "INVALID_PAGES",
            "data": {}
        }
        ```

### POST /search/suggest

获取搜索建议。

=== "请求"

    请求正文样例：

    ```json
    {
        "query": "Hello",
        "target": "title",
        "correct": true
    }
    ```

    正文的 JSON 包含一个字典，字典的各字段含义如下：

    |字段|类型|必选|默认值|含义|
    |-|-|-|-|-|
    |`query`|字符串|否|` `|当前搜索框已输入内容|
    |`target`|字符串|否|`title`|当前搜索对象|
    |`correct`|布尔值|否|`true`|是否纠错|

=== "行为"

    后端接收到请求后，向搜索后端发送请求，并返回一个补全建议和纠错建议列表。

    - 如果 `body["target"] == "title"`：

        - 先获取补全建议。

        - 如果补全建议结果较少且需要纠错，那么再获取纠错建议。

        - 返回结果中，补全建议在前，纠错建议在后。

    - 如果 `body["target"] == "uploader"`：

        - 直接获取纠错建议。

    - 如果 `body["target"`]` 不存在，或存在但其他值：

        - 等效于 `body["target"] == "title"` 的情况。

=== "响应"

    - 获取搜索建议成功
        
        获取搜索建议成功，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "suggestions": [
                    "Hello world again!",
                    "Hello world again and again!",
                    "Hello world again, again and again!"
                ]
            }
        }
        ```

=== "错误"

    此 API 不应返回错误。

### POST /search/hotwords

获取搜索热词。

=== "请求"

    无需附带请求正文。

=== "行为"

    后端接收到请求后，向搜索后端发送请求，并返回高频搜索词列表。

=== "响应"

    - 获取搜索热词成功
        
        获取搜索热词成功，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": ["spider", "dog", "hello", "large", "still", "word"]
        }
        ```

=== "错误"

    此 API 不应返回错误。

### GET /createlink/[gif_id]

    生成 Gif 预览和下载的分享链接，有效期为 1 天。

=== "请求"

    无需附带请求正文。

=== "行为"

    返回 Gif 的分享链接。

=== "响应"

    - 生成分享链接成功
        
        生成分享链接成功，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "preview_link": "https://gifexplorer-backend-nullptr.app.secoder.net/image/preview/klzh7dUapmke",
                "download_link": "https://gifexplorer-backend-nullptr.app.secoder.net/image/download/klzh7dUapmke"
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

### POST /image/createziplink

    生成 Gif 批量下载的分享链接，有效期为 1 天。

=== "请求"

    请求正文样例：

    ```json
    {
        "gif_ids": [1, 2, 3]
    }
    ```

=== "行为"

    返回 Gif 的批量下载链接。

=== "响应"

    - 生成分享链接成功
        
        生成分享链接成功，返回如下响应：

        > `200 OK`

        ```json
        {
            "code": 0,
            "info": "SUCCESS",
            "data": {
                "downloadzip_link": "https://gifexplorer-backend-nullptr.app.secoder.net/image/downloadzip?token=klzh7dUapmke"
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
