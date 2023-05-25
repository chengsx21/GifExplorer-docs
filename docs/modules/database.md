# 数据库设计

使用 PostgreSQL，设计了如下表结构：

## **`User 数据表`**

### `UserInfo`

存储用户个人信息。

| Name                 | Type            | Attribution                              |
| :------------------: | :-------------: | :--------------------------------------: |
| `id`                 | `BigAutoField`  | `primary_key=true`                       |
| `user_name`          | `CharField`     | `max_length=12, unique=True`             |
| `password`           | `CharField`     | `max_length=20`                          |
| `salt`               | `CharField`     | `max_length=40`                          |
| `signature`          | `CharField`     | `max_length=200, blank=True`             |
| `mail`               | `CharField`     | `max_length=100, blank=True`             |
| `register_time`      | `DateTimeField` | `auto_now_add=True`                      |
| `avatar`             | `TextField`     | `blank=True`                             |
| `followings`         | `JSONField`     | `null=True, blank=True, default=dict`    |
| `followers`          | `JSONField`     | `null=True, blank=True, default=dict`    |
| `favorites`          | `JSONField`     | `null=True, blank=True, default=dict`    |
| `tags`               | `JSONField`     | `null=True, blank=True, default=dict`    |
| `comment_favorites`  | `JSONField`     | `null=True, blank=True, default=list`    |
| `read_history`       | `JSONField`     | `null=True, blank=True, default=dict`    |
| `search_history`     | `JSONField`     | `null=True, blank=True, default=dict`    |
| `task_history`       | `JSONField`     | `null=True, blank=True, default=dict`    |

### `UserVerification`

用户注册时用于邮件验证。

| Name                 | Type            | Attribution                              |
| :------------------: | :-------------: | :--------------------------------------: |
| `user_name`          | `CharField`     | `max_length=12, unique=True`             |
| `token`              | `CharField`     | `null=True, blank=True, max_length=60`   |
| `mail`               | `CharField`     | `null=True, blank=True, max_length=100`  |
| `is_verified`        | `BooleanField`  | `default=False`                          |
| `password`           | `CharField`     | `null=True, blank=True, max_length=20`   |
| `salt`               | `CharField`     | `null=True, blank=True, max_length=40`   |
| `created_at`         | `DateTimeField` | `default=None, null=True, blank=True`    |

### `UserToken`

存储用户登录后的 `token` 状态。

| Name                 | Type                    | Attribution                              |
| :------------------: | :---------------------: | :--------------------------------------: |
| `user_id`            | `PositiveIntegerField`  | `default=0`                              |
| `token`              | `CharField`             | `max_length=200`                         |

## **`Gif 数据表`**

### `GifMetadata`

存储 Gif 的元数据，与 Gif 文件是分开存储的。

| Name                 | Type                    | Attribution                              |
| :------------------: | :---------------------: | :--------------------------------------: |
|`id`                  | `AutoField`             | `primary_key=True`                       |
|`name`                | `CharField`             | `null=True, blank=True, max_length=200`  |
|`title`               | `CharField`             | `max_length=200`                         |
|`width`               | `PositiveIntegerField`  | `default=0`                              |
|`height`              | `PositiveIntegerField`  | `default=0`                              |
|`duration`            | `FloatField`            | `default=0.0`                            |
|`uploader`            | `PositiveIntegerField`  | `default=1`                              |
|`category`            | `CharField`             | `null=True, blank=True, max_length=20`   |
|`tags`                | `JSONField`             | `null=True, blank=True, default=list`    |
|`likes`               | `PositiveIntegerField`  | `default=0`                              |
|`pub_time`            | `DateTimeField`         | `auto_now_add=True`                      |

### `GifFile`

存储 Gif 文件，使用外键与 `GifMetadata` 关联。

| Name                 | Type                    | Attribution                              |
| :------------------: | :---------------------: | :--------------------------------------: |
|`file`                | `ImageField`            | `upload_to='gifs/'`                      |
|`metadata`            | `OneToOneField`         | `GifMetadata, on_delete=models.CASCADE`  |

### `GifComment`

存储 Gif 的评论，为实现二级评论，使用外键将 `parent` 字段关联到自身。

| Name     | Type                  | Attribution                                                                      |
| :------: | :-------------------: | :------------------------------------------------------------------------------: |
|`id`      | `AutoField`           | `primary_key=True`                                                               |
|`metadata`| `ForeignKey`          | `GifMetadata, on_delete=models.CASCADE, related_name='comments'`                 |
|`parent`  | `ForeignKey`          | `'self', null=True, blank=True, on_delete=models.CASCADE, related_name='replies'`|
|`user`    | `ForeignKey`          | `UserInfo, on_delete=models.CASCADE`                                             |
|`content` | `TextField`           | `max_length=200`                                                                 |
|`likes`   | `PositiveIntegerField`| `default=0`                                                                      |
|`pub_time`| `DateTimeField`       | `auto_now_add=True`                                                              |

### `GifFingerprint`

上传 Gif 时，使用 `fingerprint` 字段存储 Gif 的哈希值，判断是否重复。

| Name                 | Type                    | Attribution                              |
| :------------------: | :---------------------: | :--------------------------------------: |
|`gif_id`              | `PositiveIntegerField`  | `default=0`                              |
|`fingerprint`         | `CharField`             | `max_length=64, unique=True`             |

### `GifShare`

存储 Gif 分享链接的有效期信息。

| Name                 | Type                    | Attribution                              |
| :------------------: | :---------------------: | :--------------------------------------: |
|`token`               | `TextField`             | `max_length=200`                         |
|`gif_ids`             | `JSONField`             | `default=list`                           |
|`pub_time`            | `DateTimeField`         | `auto_now_add=True`                      |

## **`Message 数据表`**

存储用户间的私信信息。

### `Message`

| Name         | Type                    | Attribution                                                            |
| :----------: | :---------------------: | :--------------------------------------------------------------------: |
|`sender`      | `ForeignKey`            | `UserInfo, on_delete=models.CASCADE, related_name='sent_messages'`     |
|`receiver`    | `ForeignKey`            | `UserInfo, on_delete=models.CASCADE, related_name='received_messages'` |
|`message`     | `TextField`             | `max_length=200`                                                       |
|`is_read`     | `BooleanField`          | `default=False`                                                        |
|`task_time`   | `DateTimeField`         | `auto_now_add=True`                                                    |

## **`Task 数据表`**

存储用户非阻塞任务的状态信息。

### `TaskInfo`

| Name                 | Type                    | Attribution                              |
| :------------------: | :---------------------: | :--------------------------------------: |
|`id`                  | `BigAutoField`          | `primary_key=True`                       |
|`task_id`             | `CharField`             | `null=True, blank=True, max_length=200`  |
|`task_type`           | `CharField`             | `max_length=200`                         |
|`task_status`         | `CharField`             | `max_length=200`                         |
|`task_time`           | `DateTimeField`         | `auto_now_add=True`                      |
|`task_result`         | `JSONField`             | `null=True, blank=True, default=dict`    |
