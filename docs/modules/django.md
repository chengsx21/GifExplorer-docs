# 后端

## 设计原则

1. 数据模型设计：合理设计数据模型，巧妙使用外键关联，支持快速的搜索和过滤操作。

2. 高效的搜索后端：选择 Elastic Search 作为搜索后端，支持快速的搜索操作、精确的分词操作及自适应的相关度排序。

3. 缓存加速：使用缓存来提高搜索性能。数据库中的数据量与访问速度负相关，可以使用后端内存缓存。为保证缓存命中率，将最近访问的数据存储在内存中，减少索引数据库查询的次数。

4. 异步处理：对于耗时的图片扩展服务，使用 Celery 异步处理方式，将非阻塞请求放入消息队列中进行处理，从而提高系统的吞吐量和响应时间。

5. 灵活性和权限控制：确保搜索、分享功能的灵活性，允许授权用户与未授权用户进行功能使用。

6. 测试和性能优化：进行全面的测试，包括功能测试和性能测试，以确保后端功能的正确性和高效性。根据测试结果优化 api 响应时间。

## 功能设计

### 新闻缓存

1. 定时从新闻数据库获取最新新闻并缓存至内存
2. 缓存搜索内容至内存
3. 维护新闻首页展示内容
4. 向其它内部模块快速提供新闻内容
5. 启动（退出）时加载（保存）本地缓存
6. 维护缓存大小

### 用户新闻管理器

1. 记录与用户交互的新闻及交互信息
2. 缓存加速
3. 维护AI摘要内容

### `token`校验

1. 用户登陆时生成`token`（`7`天时效）
2. 用户退出登录时注销`token`
3. 维护`token`白名单
4. 校验请求中`token`是否有效

### 用户管理

1. 创建用户
2. 用户登录（登出）
3. 更新（获取）用户标签
4. 修改（获取）用户信息
5. 修改（获取）用户头像

### 视图模块

1. 根据数据库中信息，响应前端请求
2. 根据前端请求，更新数据库

## 实现

### 项目结构

```shell
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

### 新闻获取函数： `get_data_from_db`
实现函数：`main.tools.get_data_from_db(news_id)`

按优先顺序根据`news_id`从`新闻缓存管理器`、`用户新闻管理器`、`新闻数据库`中获取新闻并返回。

### 新闻缓存定时更新器
实现类：`main.managers.DBScanner.DBScanner(db_connection, news_cache, get_data_from_db, testing_mode=False, front_page_news_num=10, db_check_interval=8, db_news_look_back=65536, db_update_minimum_interval=8)`

| 参数                  | 含义              |
| --------------------- | ----------------- |
| `db_connection`       | `新闻数据库连接`  |
| `news_cache`          | `新闻缓存管理器`  |
| `get_data_from_db`    | `新闻获取函数`    |
| `testing_mode`        | `测试模式`        |
| `front_page_news_num` | `首页新闻数`      |
| `db_check_interval`   | `数据库扫描间隔`  |
| `db_news_look_back`   | `数据库扫描窗口`  |
| `db_update_minimum_interval`           | `缓存更新间隔下限`                |

#### 启动： `self.run()`
在子线程中启动。启动时根据`首页新闻数`调用`新闻获取函数`从`新闻数据库`中快速获取满足新闻首页展示数量的最新新闻。启动后`扫描`数据库变更。

#### 扫描： `self.check_db_update()`
根据`数据库扫描间隔`定期`获取数据库新闻数`。若`新闻数据库`中新闻数目更新，则根据`缓存更新间隔下限`判断是否更新缓存并执行。

#### 获取数据库新闻数： `self.get_db_news_num()`
从`新闻数据库`中获取数据库新闻总数。

#### 更新缓存： `self.update_cache()`
调用`新闻缓存管理器`的更新缓存方法。

#### 测试模式： `self.testing_mode=True`
根据`数据库扫描窗口`从`新闻数据库`获取新加入的新闻，并添加至`新闻缓存管理器`。

### 新闻缓存管理器
实现类：`main.managers.NewsCache.NewsCache()`

#### 添加新闻至缓存： `self.update_cache(news_list)`
将`news_list`中的新闻加入缓存，并维护缓存大小（删除旧新闻）。

#### 获取首页分类新闻： `self.get_cache(category)`
获取缓存中类别为`category`的新闻。

### 用户新闻管理器
实现类：`main.managers.LocalNewsManager.LocalNewsManager()`

管理与用户产生过交互的新闻。

#### 添加新闻至缓存： `self.add_to_cache(news)`
将`news`新闻加入缓存，并维护缓存大小（删除旧新闻）。

#### 存贮至用户新闻数据库： `self.save_one_local_news(news)`
将`news`新闻加入`用户新闻数据库`，并`添加新闻至缓存`：。

#### 从用户新闻数据库获取： `self.get_one_local_news(news_id)`
按优先顺序根据`news_id`从`缓存`和`用户新闻数据库`中获取新闻。

### `token`校验

#### `白名单`
样式：
```
TOKEN_WHITE_LIST={
    "user_id 1": ["encoded_token 1","encoded_token 2",...],
    "user_id 2": ["encoded_token 1","encoded_token 2",...],
    ...
}
```

#### 创建`token`
实现函数： `main.tools.create_token(user_name, user_id=0)`

根据`user_name`，`user_id`，`当前时间`创建`token`。

#### 解码`token`
实现函数： `main.tools.decode_token(encoded_token)`

根据`encoded_token`，解码得到`user_name`，`user_id`并返回。

#### 检验`token`过期
实现函数： `main.tools.token_expired(token)`

检查`token`是否过期。

#### 将`token`加入白名单
实现函数： `main.tools.add_token_to_white_list(encoded_token)`

将`encoded_token`加入`白名单`。

#### 检验`token`有效
实现函数： `main.tools.check_token_in_white_list(encoded_token)`

先检查`encoded_token`是否在`白名单`，之后`检验token过期`。

#### 将`token`移除
实现函数： `main.tools.del_token_from_white_list(encoded_token)`

将`encoded_token`移除`白名单`。

