Design RESTful interface
==================

需要表达的信息
------------
- Resource资源
- Operation针对资源的操作
  - Select one, Query, Insert, Update, Delete
- Parameter参数
- API Version版本

表达的途径
---------
- 输入
  - URL
  - HTTP Method
  - HTTP Head
  - HTTP Body
- 输出
  - HTTP Response code
  - HTTP Head
  - HTTP body
* MIME type

什么途径表达什么信息
------------------
TODO:

Request
--------

- 路径 Endpoint format
  - /version/resource_name/parameters

- HTTP Method
  - GET（SELECT）：从服务器取出资源（一项或多项）。
  - POST（CREATE）：在服务器新建一个资源。
  - PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
  - PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
  - DELETE（DELETE）：从服务器删除资源。

- 过滤条件 Filtering
  - ?limit=10：指定返回记录的数量
  - ?offset=10：指定返回记录的开始位置。
  - ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
  - ?animal_type_id=1：指定筛选条件

- Example
  - http://hostname/v1/preference/?limit=10

Response
--------
- Status code or HTTP response code
  - 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
  - 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
  - 204 NO CONTENT - [DELETE]：用户删除数据成功。
  - 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。。
  - 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
  - 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
- Error message
  - 客户端要转换Json字符串到对象，所以必须明确什么时候返回业务数据，什么时候返回Error message。

Questions
=========

How to version REST URLs
------------------------
TODO:Store in URL or HTTP header


Ref
--
http://www.slideshare.net/samejack/rest-to-restful-web-service
http://www.ruanyifeng.com/blog/2014/05/restful_api.html
