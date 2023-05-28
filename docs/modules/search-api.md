# Search API

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
    |`type`|字符串|否|`perfect`|搜索模式|
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
