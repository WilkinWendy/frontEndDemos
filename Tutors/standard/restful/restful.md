# restful 接口设计指南

## 关于本文档

### 编写目的

本文希望通过简要的介绍，能让读者对**restful接口设计**有初步的认识，并能按照实践指导设计符合rest标准的接口，对于不符合设计标准的接口，拥有一定的鉴别力和修改思路。

### 期望读者

任何对restful 接口设计感兴趣的同学，拥有接口设计经验者更佳，了解http协议者更秀。

## restful是什么？
**REST**全称是**Representational State Transfer**，中文意思是表述性状态转移。
这是一个谓宾短语。
我认为更易理解的表述应该是 **资源状态转移** 。

如何理解呢？解释如下：

restful风格认为:
1. 服务端定义的都是资源（面向对象思想）
2. 接口都是在进行资源的增删改查操作

下面详细介绍一下restful接口的设计

### 从http讲起
![restfulflow](./restfulflow.png)

[点我放大图片](./restfulflow.png)

### restful的一些设计准则
其中带 * 项在现今场景一般并未实现
####   保证URI的可描述性
URI的设计应该遵循可寻址性原则，具有自描述性，需要在形式上给人以直觉上的关联。
即，能通过URI部分，推理出是在操作何种资源。
1. 资源原子化。（也不全是，后面会有提到）
1. 使用_或-来让URI可读性更好
2. 使用/来表示资源的层级关系
3. 使用?用来过滤资源
4. ,或;可以用来表示同级资源的关系

####  统一语义
约定资源操作的**动词语义**与返回**状态码语义**。

一般来说，常用的是前四个动词
1. post 创建资源
2. put 修改资源
3. get 获取资源
4. delete 删除资源
5. *option* 获取服务器的功能性信息,例如CORS跨域解决方案中作为预检请求
6. *head* 获取get请求的元信息，返回结果包含在响应头，例如嗅探下载接口的返回请求头
   
同时约定了不同动词在不同响应结果发生时，应当返回的http状态码。

以下只列举了部分。

1. 200（OK）- 如果现有资源已被更改
2. 201（created）- 如果新资源被创建
3. 202（accepted）- 已接受处理请求但尚未完成（异步处理）
4. 301（Moved Permanently）- 资源的URI被更新
5. 303（See Other）- 其他，通过其他URI表达响应结果
6. 400（bad request）- 指代坏请求
7. 404 （not found）- 资源不存在
8. 500 （internal server error）- 通用错误响应
   
####  确定资源的表述
通过请求头来控制，所需的资源的表达返回方式

> 例如：
> 
> Accept:Application/xml; version=1.0,2.0 
> 
> 客户端期望响应体返回值是xml格式，并且期望的服务端版本是1.0或者2.0版本

####  返回资源的链接*

在返回结果里边加入链接来引导客户端。
在ajax盛行的年代，这一条经常并未实现

####  必须是状态的转移
客户端调用**非安全接口** 后，产生的结果是资源状态的变更（转移）

> 安全接口，幂等接口 概念及意义介绍

## 为什么要使用restful规则来设计接口



## 如何设计restful接口
正常来说设计restful接口应当设计 请求头（其中包含http动词）、URI、响应头。
为了简单起见，目前只基于 URI、http动词来进行设计。

    我觉得有必要介绍一下URI,完整URI如下，而RESTFUL 接口相关的HTTP URI 去除了用户名、密码
    [协议名]://[用户名]:[密码]@[服务器地址]:[服务器端口号]/[路径]?[查询字符串]#[片段ID]'
    [protocal]://username:password@address:port/pathname/?query#hash

以下为我个人总结的一般场景接口设计的步骤和技巧（特殊情况后面会单独再讨论）。

### 1. 描述接口业务(准备工作)
我们需要分析业务中的对象模型
然后以合适的表达来描述接口所做的事情。

一般都可以概括为:

谓语+定语+宾语（+结果）

> 获取指定某项目下的数据源集合
> 获取指定某项目的某数据源（的信息）
> 计算指定两个商品（的差价）

### 2. 识别资源（确定pathname）

一般来说，上述的 **定语+宾语** 部分 就是资源

识别资源后我们就可以确定URI中的pathname部分。

规则我大致总结为下面几种情况及其延伸

- **/资源关键字/id**  资源是单数
- **/资源关键字**  资源是复数
- **/父资源关键字/id/资源关键字/id**  父资源是单数，子资源是单数
- **/父资源关键字/id/资源关键字**  父资源是单数，子资源是复数




### 3. 识别动作（确定动词）

上述的 **谓语（+结果）** 部分 就是动作。
这里我们分两种场景予以讨论。

#### 常规资源增删改查 

使用 post  delete  put get 分别对应增删改查

#### 操作类 
注意此场景下操作会被视为一种操作资源影响URI。

> /父资源关键字/id/资源关键字/id/操作资源关键字
  
##### 安全接口 
使用 get 动词 
##### 非安全接口
使用 post 动词 

### 4. 识别参数（确定query部分）

根据业务需求确定，本次请求需要传递的信息，信息的决定了有哪些参数。

确定了有哪些参数，又如何确定参数应该放是在path、query、body 中的哪一部分呢？

已经出现在path中的参数，已经传递不用再考虑。

剩余参数我总结如下规则：

- post、put
  - 操作类 参数尽可能放在query，不满足业务需求，可放在body
  - 非操作类 body
- get、delete 参数只能放在query部分

### 5. 输出设计结果（完成）


## 特殊场景案例

### 把操作也视为一种资源

### 异步长时操作


### 使用put来创建资源的场景  

### 如何权衡接口的可描述性
复合资源场景


## 总结

总的来说，restful 接口设计最重要的思想是 **接口资源化**，实际执行并没有绝对的标准，我在这里给出了一套实践方案，希望能对大家提供一定的指导。

实际设计中根据业务情况选择最佳实践为宜。


## 参考内容
[restful cook book](./RESTful+Web+Services+Cookbook++中文版_12879413.pdf)

