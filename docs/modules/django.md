# 后端

## 设计原则

1. 数据模型设计：合理设计数据模型，巧妙使用外键关联，支持快速的搜索和过滤操作。

2. 高效的搜索后端：选择 ElasticSearch 作为搜索后端，支持快速的搜索操作、精确的分词操作及自适应的相关度排序。

3. 缓存加速：使用缓存来提高搜索性能。数据库中的数据量与访问速度负相关，可以使用后端内存缓存。为保证缓存命中率，将最近访问的数据存储在内存中，减少索引数据库查询的次数。

4. 异步处理：对于耗时的图片扩展服务，使用 Celery 异步处理方式，将非阻塞请求放入消息队列中进行处理，从而提高系统的吞吐量和响应时间。

5. 灵活性和权限控制：确保搜索、分享功能的灵活性，允许授权用户与未授权用户进行功能使用。

6. 测试和性能优化：进行全面的测试，包括功能测试和性能测试，以确保后端功能的正确性和高效性。根据测试结果优化 api 响应时间。

## 功能实现

### 项目结构

```bash
│  .pycodestyle  # pycodestyle配置文件
│  .pylintrc  # pylint配置文件
│  pytest.ini  # pytest配置文件
│  manage.py  # Django项目管理
│  start.sh  # 运行脚本
│  sonar-project.properties  # sonarqube配置文件
│  requirements.txt  # 依赖列表  
│
├─GifExplorer  # 默认项目文件夹
│      asgi.py
│      settings.py  # 项目设置
│      urls.py  # 项目主路由
│      celery.py  # celery配置
│      wsgi.py
│      __init__.py
│
├─files  # 文件存储
│      gifs  # gif文件
│      tests  # 测试文件
│
├─config  # 后端配置文件
│      config.json  # 后端数据库配置
│
├─utils  # 工具函数
│      utils_request.py  # 返回请求
│      utils_require.py  # 检查请求格式
│      utils_time.py  # 时间戳工具
│
└─main  # 后端主应用
    │  admin.py
    │  apps.py  # apps设置
    │  config.py  # 参数设置
    │  models.py  # 数据模型
    │  search.py  # 搜索模块
    │  tests.py  # 单元测试
    │  helpers.py  # 工具
    │  urls.py  # 应用路由
    │  views.py  # 视图
    │  __init__.py
    │
    └─migrations  # 数据库模型迁移
```

### `views` 模块

1. 根据数据库中信息，响应前端请求
2. 根据前端请求，更新数据库
3. 内容详见 `模块设计-接口设计`

### `token` 校验

#### 创建`token`

实现函数： `main.helpers.create_token(user_name, user_id)`

根据 `user_name`，`user_id`，创建`token`。

#### 解码`token`

实现函数： `main.helpers.decode_token(token)`

根据 `encoded_token`，解码得到`user_name`，`user_id` 并返回。

#### 将`token`加入白名单

实现函数： `main.helpers.add_token_to_white_list(token)`

将 `token` 加入白名单。

#### 检验`token`有效

实现函数： `main.helpers.is_token_valid(token)`

先检查 `token` 是否有效。

#### 将`token`移除

实现函数： `main.helpers.delete_token_from_white_list(token)`

将 `token` 移除白名单。
