restful是用来规范服务器提供数据的api的一种约束，代表着分布式服务的架构风格。
# 缩写
  * REpresentational State Transfer

# 设计原则
* 客户端-服务器
 通过将用户UI与数据存储分开，我们可以简化服务器组件来提高跨多个平台的用户界面的可移植性并提高可伸缩性。
* 无状态
  从客户端到服务器的每个请求都必须包含理解请求所需的所有信息，并且不能利用服务器上任何存储的上下文。 这表示你应该尽可能的避免使用session，由客户端自己标识会话状态。（token）
  使REST API无状态有一些非常显着的优点：
  无状态通过将API部署到多个服务器，有助于将API扩展到数百万并发用户。任何服务器都可以处理任何请求，因为没有与会话相关的依赖。（集群）
  无状态使得REST API不那么复杂 - 可以删除所有服务器端状态同步逻辑。（删除session，清理多余空间）
  无状态API也很容易缓存。特定软件可以通过查看该一个请求来决定是否缓存HTTP请求的结果。从先前的请求中获得的状态可能会影响这个请求的可缓存性，这并不存在任何不确定性。它提高了应用程序的性能。
  服务器永远不会忘记每个客户端身份，因为客户端会在每个请求中发送所有必要的信息。（携带token）
  那么无状态又要怎么实现呢？前面我们已经说了，服务端不应该再保存session会话，这个工作全部交由http请求去标识，而最常见的做法则是使用token。生成token可以考虑使用jwt，oauth。
  * token
    * jwt
    * oauth

* 规范接口
  REST接口约束定义：资源识别; 请求动作; 响应信息; 它表示通过uri标出你要操作的资源，通过请求动作（http method）标识要执行的操作，通过返回的状态码来表示这次请求的执行结果。uri资源的描述构成了uri，它一般有以下约束：
  * url设计规则：动词+名词
    客户端发出的数据操作指令都是"动词 + 宾语"的结构。比如，GET /articles这个命令，GET是动词，/articles是宾语。
  * 宾语必须是名词，
    * 建议都使用复数 URL
    * 避免多级 URL
  如 user, student, class
  http://api.example.com/class-management/students
  http://api.example.com/device-management/managed-devices/{device-id}
  http://api.example.com/user-management/users/
  http://api.example.com/user-management/users/{id}

  * http method对应不同的请求动作（数据库或者业务逻辑）
    * GET：查询：（read）
      HTTP GET /devices?startIndex=0&size=20
      HTTP GET /configurations?startIndex=0&size=20
      HTTP GET /devices/{id}/configurations
      HTTP GET /devices/{id}
    * POST：新增：（Create）
      HTTP POST /device
    * PUT 更新操作（代表更新一个实体的所有属性）（Update）
      HTTP PUT /devices/{id}
    * PATCH 部分更新（代表更新一个尸体的部分属性）（Update）
      由于有的浏览器兼容性问题，一般推荐使用put
      HTTP PATCH /devices/{id}
    * DELETE 删除
      HTTP DELETE /devices/{id}

    * 动词覆盖：
    有些客户端只能使用GET和POST这两种方法。服务器必须接受POST模拟其他三个方法（PUT、PATCH、DELETE）。

    这时，客户端发出的 HTTP 请求，要加上X-HTTP-Method-Override属性，告诉服务器应该使用哪一个动词，覆盖POST方法。

    ···html
    POST /api/Person/4 HTTP/1.1
    X-HTTP-Method-Override: PUT
    ···
    上面代码中，X-HTTP-Method-Override指定本次请求的方法是PUT，而不是POST。

  * 使用连字符（ - ）而不是（_）来提高URI的可读性
    http://api.example.com/inventory-management/managed-entities/{id}/install-script-location //更易读
    http://api.example.com/inventory_management/managed_entities/{id}/install_script_location //更容易出错
  * 在URI中使用小写字母
    http://api.example.org/my-folder/my-doc
  * 不要使用文件扩展名
    文件扩展名看起来很糟糕，不会增加任何优势。删除它们也会减少URI的长度。没理由保留它们。
    http://api.example.com/device-management/managed-devices.xml / 不要使用它 /
    http://api.example.com/device-management/managed-devices / *这是正确的URI * /

  * 使用查询组件过滤URI集合
    很多时候，我们会遇到需要根据某些特定资源属性对需要排序，过滤或限制的资源集合的要求。为此，请不要创建新的API - 而是在资源集合API中启用排序，过滤和分页功能，并将输入参数作为查询参数传递。例如
    http://api.example.com/device-management/managed-devices
    http://api.example.com/device-management/managed-devices?region=USA
    http://api.example.com/device-management/managed-devices?region=USA&brand=XYZ
    http://api.example.com/device-management/managed-devices?region=USA&brand=XYZ&sort=installation-date

  * 不要在末尾使用/
    作为URI路径中的最后一个字符，正斜杠（/）不会添加语义值，并可能导致混淆。最好完全放弃它们。

  * 使用http状态码定义api执行结果
    * 1xx：信息
      通信传输协议级信息。
    * 2xx：成功
      表示客户端的请求已成功接受。
      进一步精确：
      * POST返回201状态码，表示生成了新的资源
      * DELETE返回204状态码，表示资源已经不存在。
      * 202 Accepted状态码表示服务器已经收到请求，但还未进行处理，会在未来再处理，通常用于异步操作。

    * 3xx：重定向
      表示客户端必须执行一些其他操作才能完成其请求。
      API 用不到301状态码（永久重定向）和302状态码（暂时重定向，307也是这个含义），因为它们可以由应用级别返回，浏览器会直接跳转，API 级别可以不考虑这两种情况。
      进一步精确：
      * 303 See Other（303用于POST、PUT和DELETE请求。）
        收到303以后，浏览器不会自动跳转，而会让用户自己决定下一步怎么办

    * 4xx：客户端错误
      此类错误状态代码指向客户端。
      * 400 Bad Request：服务器不理解客户端的请求，未做任何处理。
      * 401 Unauthorized：用户未提供身份验证凭据，或者没有通过身份验证。
      * 403 Forbidden：用户通过了身份验证，但是不具有访问资源所需的权限。
      * 404 Not Found：所请求的资源不存在，或不可用。
      * 405 Method Not Allowed：用户已经通过身份验证，但是所用的 HTTP 方法不在他的权限之内。
      * 410 Gone：所请求的资源已从这个地址转移，不再可用。
      * 415 Unsupported Media Type：客户端要求的返回格式不支持。比如，API 只能返回 JSON 格式，但是客户端要求返回 XML 格式。
      * 422 Unprocessable Entity ：客户端上传的附件无法处理，导致请求失败。
      * 429 Too Many Requests：客户端的请求次数超过限额。
    * 5xx：服务器错误
      服务器负责这些错误状态代码。
      * 500 Internal Server Error：客户端请求有效，服务器处理时发生了意外。
      * 503 Service Unavailable：服务器无法处理请求，一般用于网站维护状态。

      另外完整的更为详细的状态码此处不做赘述。一般简化版本的api会使用200，400，500，其中400代表客户端调用有误，将错误信息放入response：
      {
        "error": "username.or.password.error"
      }
      eg，另有用户说：400 用于表示客户端传参错误(或者不完全),, 200 有可能也是不正常的响应(当然不能算是错误), 比如用户名或者密码不匹配.
  * api版本定义
    当我们需要对现有的api接口升级的时候，因为该api接口已经投入使用，所以新添加的业务可能无法保证兼容原来的逻辑，这个时候就需要新的接口，而这个接口一般表示对原来的接口的升级（不同版本），那版本怎么定义呢？
    * URI版本控制（推荐）
      http://api.example.com/v1
      http://apiv1.example.com
    * 使用自定义请求标头进行版本控制
      Accept-version：v1
      Accept-version：v2
    * 使用Accept header 进行版本控制
      Accept:application / vnd.example.v1 + json
      Accept:application / vnd.example + json; version = 1.0
* 可缓存
  缓存约束要求将对请求的响应中的数据隐式或显式标记为可缓存或不可缓存。如果响应是可缓存的，则客户端缓存有权重用该响应数据以用于以后的等效请求。 它表示get请求响应头中应该表示有是否可缓存的头（Cache-Control)
# 服务器回应：
* 不要返回纯本文
  *  服务器的response：HTTP 头的Content-Type属性要设为application/json。
  *  客户端请求的 HTTP 头的ACCEPT属性也要设成application/json。
* 发生错误时，不要返回 200 状态码
* 提供链接
  * HATEOAS，返回都有link，告诉用户能有什么操作；
    回应中，给出相关链接，便于下一步操作。这样的话，用户只要记住一个 URL，就可以发现其他的 URL。
  *
