<a name="head"></a>
## RESTful API 接口规范/协议

### > 参考

+ [阮一峰 —— RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

+ [菜鸟教程 —— RESTful 架构详解](https://www.runoob.com/w3cnote/restful-architecture.html)

+ [GitHub API](https://developer.github.com/v3/)

1. [域名](#域名)

1. [版本（Versioning）](#版本（Versioning）)

1. [路径（Endpoint）](#路径（Endpoint）)

1. [HTTP 动词](#HTTP-动词)

1. [过滤信息（Filtering）](#过滤信息（Filtering）)

1. [状态码（Status Codes）](#状态码（Status-Codes）)

1. [错误处理（Error handing）](#错误处理（Error-handing）)

1. [返回结果](#返回结果)

1. [超媒体（Hypermedia API）](#超媒体（Hypermedia-API）)

1. [其他](#其他)

### 域名

+ 应该尽量将 API 部署在专用域名之下。

    ```markdown
    https://api.example.com
    ```

+ 如果确定 API 很简单，不会有进一步扩展，可以考虑放在主域名下。

    ```markdown
    https://example.org/api/
    ```

### 版本（Versioning）

+ 应该将 API 的版本号放入 URL。

    ```markdown
    https://www.example.com/app/1.0/foo

    https://www.example.com/app/1.1/foo

    https://www.example.com/app/2.0/foo
    ```

+ 另一种做法是，将版本号放在 HTTP 头信息中，但不放入 URL 方便和直观，GitHub 就采用了这种做法。

    > 因为不同的版本，可以理解成同一种资源的不同表现形式，所以应该采用同一个 URL。版本号可以在 HTTP 请求头信息的 Accept 字段中进行区分

    ```markdown
    Accept: vnd.example-com.foo+json; version=1.0

    Accept: vnd.example-com.foo+json; version=1.1

    Accept: vnd.example-com.foo+json; version=2.0
    ```

### 路径（Endpoint）

&emsp;&emsp;路径又称“终点”（endpoint），表示 API 的具体网址，每个网址代表一种资源（resource）

1. **资源作为网址，只能有名词，不能有动词，而且所用的名词往往与数据库的表名对应**

    > 举例来说，以下是不好的例子

    ```markdown
    /getProducts

    /listOrders

    /retreiveClientByOrder?orderId=1
    ```

    > 对于一个简洁结构，你应该始终用名词，此外，利用的 HTTP 方法可以分离网址中的资源名称的操作

    ```markown
    GET /products   : 将返回所有产品清单
    POST /products  : 将产品新建到集合
    GET /products/4 : 将获取产品 4
    PATCH（或）PUT /products/4  : 将更新产品 4
    ```

1. **API 中的名词应该使用复数，无论子资源或者所有资源**

    > 举例来说，获取产品的 API 可以这样定义

    ```markdown
    获取单个产品 : http://127.0.0.1:8080/AppName/rest/products/1

    获取所有产品 : http://127.0.0.1:8080/AppName/rest/products
    ```

### HTTP 动词

&emsp;&emsp;对于资源的具体操作类型，由 HTTP 动词表示。常用的 HTTP 动词有下面四个（括号里是对应的 SQL 命令）：

+ GET（SELECT）：从服务器取出资源（一项或多项）

+ POST（CREATE）：在服务器新建一个资源

+ PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）

+ DELETE（DELETE）：从服务器删除资源

> CURD -> Create、Update、Read、Delete 增删查改，这四个数据库的常用操纵

&emsp;&emsp;还有三个不常用的 HTTP 动词：

+ PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）

+ HEAD：获取资源的元数据

+ OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的

&emsp;&emsp;下面是一些例子：

```http
GET /zoos                  : 列出所有动物园
POST /zoos                 : 新建一个动物园（上传文件）
GET /zoos/ID               : 获取某个指定动物园的信息
PUT /zoos/ID               : 更新某个指定动物园的信息（提供该动物园的全部信息）
PATCH /zoos/ID             : 更新某个指定动物园的信息（提供该动物园的部分信息）
DELETE /zoos/ID            : 删除某个动物园
GET /zoos/ID/animals       : 列出某个指定动物园的所有动物
DELETE /zoos/ID/animals/ID : 删除某个指定动物园的指定动物
```

### 过滤信息（Filtering）

&emsp;&emsp;如果记录数量很多，服务器不可能都将它们返回给用户，API 应该提供参数，过滤返回结果。下面是一些常见的参数，`query_string` 查询字符串，地址栏后面问号后面的数据，格式：`name=xx&sss=xxx`

```http
完整的 URL 地址格式：
协议://域名(IP):端口号/路径?查询字符串#锚点
```

```http
?limit=10              : 指定返回记录的数量
?offset=10             : 指定返回记录的开始位置
?page=2&per_page=100   : 指定第几页，以及每页的记录数
?sortby=name&order=asc : 指定返回结果按照哪个属性排序，以及排序顺序
?animal_type_id=1      : 指定筛选条件
```

> 参数的设计允许存在冗余，即允许 API 路径和 URL 参数偶尔有重复。比如：GET /zoos/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的

### 状态码（Status Codes）

```http
1xx 表示当前本次请求还是持续，没结束
2xx 表示当前本次请求成功
3xx 表示当前本次请求成功，但是服务器进行代理操作/重定向
4xx 表示当前本次请求失败，主要是客户端发生了错误
5xx 表示当前本次氢气失败，主要是服务器发生了错误
```

+ 服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的 HTTP 动词）：

    |状态码|HTTP 动词|描述|
    |:-----|:-------:|:--|
    |200 OK|\[GET\]|服务器成功返回用户请求的数据|
    |201 CREATED|\[POST/PUT/PATCH\]|用户新建或修改数据成功|
    |202 Accepted|\[\*\]|表示一个请求已经进入后台排队（异步任务）|
    |204 NO CONTENT|\[DELETE\]|用户删除数据成功|
    |400 INVALID REQUEST|\[POST/PUT/PATCH\]|用户发出的请求有错误，服务器没有进行新建或修改数据的操作|
    |401 Unauthorized|\[\*\]|表示用户没有权限（令牌、用户名、密码错误）|
    |403 Forbidden|\[\*\]|表示用户得到授权（与 401 错误相对），但是访问是被禁止的|
    |404 NOT FOUND|\[\*\]|用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的|
    |406 Not Acceptable|\[GET\]|用户请求的格式不可得（比如用户请求 JSON 格式，但是只有 XML 格式）|
    |410 Gone|\[GET\]|用户请求的资源被永久删除，且不会再得到的|
    |422 Unprocesable entity|\[POST/PUT/PATCH\]|当创建一个对象时，发送一个验证错误|
    |500 INTERNAL SERVER ERROR|\[\*\]|服务器发生错误，用户将无法判断发出的请求是否成功|
    |507||数据存储出错，往往数据库操作错误出错|

    > 状态码的完全列表参见 [这里](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。

### 错误处理（Error handing）

&emsp;&emsp;如果状态码是 4xx，服务器就应该向用户返回出错信息。一般来说，返回的信息中将 erro 作为键名，出错信息作为键值即可：

```json
{
    error: "Invalid API key"
}
```

### 返回结果

&emsp;&emsp;针对不同的操作，服务器向用户返回的结果应该符合以下规范：

+ GET /collection       : 返回资源对象的列表（数组）

+ GET /collection/ID    : 返回单个资源对象（json）

+ POST /collection      : 返回新生成的资源对象（json）

+ PUT /collection/ID    : 返回完整的资源对象（json）

+ PATCH /collection/ID  : 返回完整的资源对象（json）

+ DELETE /collection/ID : 返回一个空文档（空字符串）

### 超媒体（Hypermedia API）

&emsp;&emsp;RESTful API 最好做到 Hypermedia（即返回结果中提供链接，连向其他 API 方法），使得用户不查文档，也知道下一步应该做什么。

+ 比如，GitHub 的 API 就是这种设计，访问 [api.github.com](https://api.github.com/) 会得到一个所有可用 API 的网址列表：

    ```json
    {
        "current_user_url": "https://api.github.com/user",
        "authorizations_url": "https://api.github.com/authorizations",
        // ...
    }
    ```

+ 从上面可以看到，如果想获取当前用户的信息，应该访问 [api.github.com/user](https://api.github.com/user)，然后就得到了下面结果：

    ```json
    {
        "message": "Requires authentication",
        "documentation_url": "https://developer.github.com/v3/users/#get-the-authenticated-user"
    }
    ```

+ 上面代码表示，服务器给出了提示信息，以及文档的网址。

### 其他

&emsp;&emsp;服务器返回的数据格式，应该尽量使用 JSON，避免使用 XML。
