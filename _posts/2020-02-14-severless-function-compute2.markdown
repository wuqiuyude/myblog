---
layout: post
title: '前端severless实践-阿里云函数计算'
subtitle: 'severless'
February 14, 2020'
date: 2020-02-14 19:00:00
author: 'wuqiuyu'
header-img: 'img/in-post/timg.jpeg'
header-mask: 0.3
catalog: true
tags:
  - 函数计算
  - serverless
---

> 工作过程中需要完成一个年终总结的项目，由于该项目涉及一些和后端的交互，但是交互比较简单，刚好最近severless大热，所以尝试用severless解决。这里主要用到的是阿里云的函数计算<br/>

<div class="lake-engine-view lake-typography-traditional" tabindex="0"><h1 id="VaMeA"><a class="lake-anchor" style="top: 13px;"><span class="lake-anchor-button lake-icon lake-icon-h1"></span></a>什么是函数计算</h1><p style="text-indent: 2em;">函数计算是事件驱动的全托管计算服务。使用函数计算，您无需采购与管理服务器等基础设施，只需编写并上传代码。函数计算为您准备好计算资源，弹性地可靠地运行任务，并提供日志查询、性能监控和报警等功能。</p><p>借助函数计算，您可以快速构建任何类型的应用和服务，并且只需为任务实际消耗的资源付费。（来自阿里云）</p><p style="text-indent: 2em;">简单的讲就是你可以只关心业务代码的实现，不需要关系其他任何服务器配置，代理配置，监控等等运维相关的事情，只有把实现的代码托管到函数计算就行。</p><p style="text-indent: 2em;"><span data-card-type="inline" data-lake-card="image" contenteditable="false" data-card-value="data:%7B%22src%22%3A%22https%3A%2F%2Fcdn.nlark.com%2Fyuque%2F0%2F2020%2Fpng%2F101638%2F1578994519614-8b17a758-1770-46d2-9122-a743e779ca16.png%22%2C%22originWidth%22%3A884%2C%22originHeight%22%3A273%2C%22name%22%3A%22image.png%22%2C%22size%22%3A24647%2C%22display%22%3A%22inline%22%2C%22align%22%3A%22left%22%2C%22linkTarget%22%3A%22_blank%22%2C%22status%22%3A%22done%22%2C%22ocrLocations%22%3A%5B%7B%22x%22%3A371.09583%2C%22y%22%3A22.099998%2C%22width%22%3A70.90414000000004%2C%22height%22%3A19.337498000000004%2C%22text%22%3A%22%E5%88%9B%E5%BB%BA%E5%87%BD%E6%95%B0%22%7D%2C%7B%22x%22%3A233.89165%2C%22y%22%3A39.595833%2C%22width%22%3A68.14167999999998%2C%22height%22%3A19.337497%2C%22text%22%3A%22%E5%88%9B%E5%BB%BA%E6%9C%8D%E5%8A%A1%22%7D%2C%7B%22x%22%3A502.77496%2C%22y%22%3A39.595833%2C%22width%22%3A124.31249999999994%2C%22height%22%3A19.337497%2C%22text%22%3A%22%E6%98%AF%E5%90%A6%E4%BD%BF%E7%94%A8%E8%A7%A6%E5%8F%91%E5%99%A8%22%7D%2C%7B%22x%22%3A741.2708%2C%22y%22%3A121.549995%2C%22width%22%3A88.39999999999998%2C%22height%22%3A20.258325000000013%2C%22text%22%3A%22%E5%88%9B%E5%BB%BA%E8%A7%A6%E5%8F%91%E5%99%A8%22%7D%2C%7B%22x%22%3A86.55833%2C%22y%22%3A122.470825%2C%22width%22%3A72.74582%2C%22height%22%3A21.179175%2C%22text%22%3A%22%E5%87%BD%E6%95%B0%E8%AE%A1%E7%AE%97%22%7D%2C%7B%22x%22%3A199.82082%2C%22y%22%3A208.10832%2C%22width%22%3A70.90416000000002%2C%22height%22%3A19.337500000000006%2C%22text%22%3A%22%E4%B8%9A%E5%8A%A1%E5%A2%9E%E7%9B%8A%22%7D%2C%7B%22x%22%3A512.9041%2C%22y%22%3A209.02916%2C%22width%22%3A71.82500000000005%2C%22height%22%3A17.495840000000015%2C%22text%22%3A%22%E6%89%A7%E8%A1%8C%E5%87%BD%E6%95%B0%22%7D%2C%7B%22x%22%3A375.69998%2C%22y%22%3A226.525%2C%22width%22%3A70.90418%2C%22height%22%3A18.416650000000004%2C%22text%22%3A%22%E6%9F%A5%E7%9C%8B%E6%97%A5%E5%BF%97%22%7D%5D%2C%22style%22%3A%22none%22%2C%22search%22%3A%22%E5%88%9B%E5%BB%BA%E5%87%BD%E6%95%B0%20%E5%88%9B%E5%BB%BA%E6%9C%8D%E5%8A%A1%20%E6%98%AF%E5%90%A6%E4%BD%BF%E7%94%A8%E8%A7%A6%E5%8F%91%E5%99%A8%20%E5%88%9B%E5%BB%BA%E8%A7%A6%E5%8F%91%E5%99%A8%20%E5%87%BD%E6%95%B0%E8%AE%A1%E7%AE%97%20%E4%B8%9A%E5%8A%A1%E5%A2%9E%E7%9B%8A%20%E6%89%A7%E8%A1%8C%E5%87%BD%E6%95%B0%20%E6%9F%A5%E7%9C%8B%E6%97%A5%E5%BF%97%22%2C%22width%22%3A884%2C%22height%22%3A273%7D"><span data-card-element="body"><span data-card-element="center"><span class="lake-image">
    <span class="lake-image-content lake-image-content-isvalid">
      <span data-role="detail" class="lake-image-detail">
        <span class="lake-image-meta" style="">
          <span class="lake-image-warning" style="display: none;">
            <i class="anticon anticon-exclamation-circle">
              <svg viewBox="64 64 896 896" class="" data-icon="exclamation-circle" width="1em" height="1em" fill="currentColor" aria-hidden="true">
                <path d="M512 64C264.6 64 64 264.6 64 512s200.6 448 448 448 448-200.6 448-448S759.4 64 512 64zm0 820c-205.4 0-372-166.6-372-372s166.6-372 372-372 372 166.6 372 372-166.6 372-372 372z"></path><path d="M464 688a48 48 0 1 0 96 0 48 48 0 1 0-96 0zM488 576h48c4.4 0 8-3.6 8-8V296c0-4.4-3.6-8-8-8h-48c-4.4 0-8 3.6-8 8v272c0 4.4 3.6 8 8 8z"></path>
              </svg>
            </i>
          </span>
          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1578994519614-8b17a758-1770-46d2-9122-a743e779ca16.png?x-oss-process=image/resize,w_746" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1578994519614-8b17a758-1770-46d2-9122-a743e779ca16.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 148px;">
​          

函数计算支持多种语言：

nodejs

python

php

java

c#

Custom runtime

# 怎么开始一个函数计算

函数计算可以通过四种方式创建：

- - [使用控制台编写函数（**推荐**）](https://help.aliyun.com/document_detail/51783.html?spm=a2c4g.11186623.6.554.1e1b10f3PhAs0U#using-console)
  - [使用 Fun 创建函数（**推荐**）](https://help.aliyun.com/document_detail/51783.html?spm=a2c4g.11186623.6.554.1e1b10f3PhAs0U#using-fun)
  - [使用 fcli 创建函数](https://help.aliyun.com/document_detail/51783.html?spm=a2c4g.11186623.6.554.1e1b10f3PhAs0U#using-fcli)
  - [使用 SDK 创建函数](https://help.aliyun.com/document_detail/51783.html?spm=a2c4g.11186623.6.554.1e1b10f3PhAs0U#using-sdk)

  （官方文档都有就不详细介绍）

## 创建服务

什么是服务？在函数计算中，服务就是管理函数计算的基本资源单位。你可以在服务级别上授权、配置日志和创建函数等。

### 服务的属性

- serviceName（必选）：服务的名字
- description（可选）：服务的描述。
- role（可选）：授予函数计算执行函数所需权限, 使用场景包括：

- - 授权函数计算服务使用用户的日志服务资源存储和分析函数运行日志。
  - 授权函数计算服务运行需要访问其他云资源的函数。
  - 关于role的使用细节，请参考[函数计算权限管理](https://help.aliyun.com/document_detail/52885.html)。

- logConfig（可选）：设置日志服务的项目和日志库，存储和分析函数运行日志。

- - 如果您未配置该项，则无法查看函数运行日志。强烈建议您开启日志服务，并配置该属性.

- vpcConfig (可选) : 配置VPC选项可让函数访问指定VPC。
- internetAccess (可选) : 设为true可让函数访问公网。



在通过代码创建的项目中，服务配置在一个叫做template.yml的文件中



​          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1578996221526-46436f15-bf9f-4e1f-886d-9c8c1d9af4ae.png" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1578996221526-46436f15-bf9f-4e1f-886d-9c8c1d9af4ae.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 205px;">
​          

## 创建函数

函数是系统调度和运行的单位。函数必须从属于服务，一个服务下的所有函数都共享该服务的属性，例如授权，日志设置。

### 函数属性

- - **functionName（必选）：**函数的名字。在当前服务内唯一。
  - **runtime（必选）：** 函数运行时类型。
  - **code（必选）：** 代码包。Java 语言需要上传 jar 包，其他语言上传 zip 包，可以存放在 OSS 上，或者直接上传代码包。
  - **handler（必选）：** 处理函数，它是函数计算系统运行用户函数的调用入口。
  - **description（可选）：** 函数的描述。函数计算系统并不会使用该属性值，但建议您为服务设置一个简洁、清晰的描述。
  - **timeout（可选）：** 函数的最大运行时间，单位为秒。
  - **MemorySize（可选）：** 函数运行所需的内存资源，单位为 MB。取值范围为 [128, 3072]，以 64 MB 为步长递进。
  - **Initializer（可选）：** [initializer](https://help.aliyun.com/document_detail/94670.html) 入口，它是函数计算系统运行用户 initializer 函数的调用入口。
  - **InitializationTimeout（可选）：** initializer 最大运行时间，单位为秒。



函数属性也配置在template.yml中

​          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1578998549947-1099926b-0863-4996-9025-703d61dec716.png" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1578998549947-1099926b-0863-4996-9025-703d61dec716.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 400px;">
​       

# 触发器

触发器是触发函数执行的方式。在事件驱动的计算模型中，事件源是事件的生产者，函数是事件的处理者，而触发器提供了一种集中的和统一的方式来管理不同的事件源。在事件源中，当事件发生时，如果满足触发器定义的规则，事件源则调用触发器所对应的函数。

## 触发器的基本信息

- `triggerName`：触发器名称。
- `triggerType`：触发器类型，例如（oss、timer、http）。
- `sourceArn`：触发函数执行的资源描述符，有些触发器需要设置（涉及到阿里云其他服务触发函数计算执行的，例如 OSS 触发器，SLS 触发器），有些触发器不需要设置（不涉及到阿里云其他服务触发函数计算执行的，例如定时触发器，HTTP 触发器）。例如 OSS 触发器的 sourceArn 格式为 `acs:oss:region:accountId:bucketName`。
- `invocationRole`：触发角色，事件源需要扮演一个角色来触发函数的执行，要求这个角色有触发函数执行的权限

- - 关于事件源角色的详细信息请参考文章 [权限简介](https://help.aliyun.com/document_detail/52885.html)。

- `qualifier`：触发的服务版本或别名。更多版本和别名的使用请参考[版本管理](https://help.aliyun.com/document_detail/96464.html)。
- `triggerConfig`：触发器的配置信息，各触发器的配置信息请参考该触发器对应的文档。



### 双向集成触发器

| 触发器名称        | 文档链接                                                     | 示例链接                                                     |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| OSS 事件触发器    | [OSS 事件触发器文档](https://help.aliyun.com/document_detail/62922.html) | [使用 OSS 触发函数计算示例](https://help.aliyun.com/document_detail/74761.html) |
| HTTP 触发器       | [HTTP 触发器](https://help.aliyun.com/document_detail/71229.html) | [基于 HTTP 触发器搭建 Web Server](https://help.aliyun.com/document_detail/74768.html) |
| 定时触发器        | [定时触发器](https://help.aliyun.com/document_detail/68172.html) | [使用定时触发器触发函数计算示例](https://help.aliyun.com/document_detail/74676.html) |
| MNS 主题触发器    | [MNS 主题触发器](https://help.aliyun.com/document_detail/97032.html) | [使用 MNS 主题触发器触发函数计算示例](https://help.aliyun.com/document_detail/97009.html) |
| Table Store触发器 | [Table Store触发器](https://help.aliyun.com/document_detail/100092.html) | [使用函数计算做数据清洗](https://help.aliyun.com/document_detail/71884.html) |
| CDN 事件触发器    | [CDN 事件触发器](https://help.aliyun.com/document_detail/73333.html) | [CDN 事件触发器示例](https://help.aliyun.com/document_detail/75119.html) |
| SLS 触发器        | [SLS 触发器](https://help.aliyun.com/document_detail/84386.html) | [日志服务触发器示例](https://help.aliyun.com/document_detail/84090.html) |

### 单向集成触发器

| 触发器名称     | 示例链接                                                     |
| -------------- | ------------------------------------------------------------ |
| API 网关触发器 | [API 网关触发器](https://help.aliyun.com/document_detail/54788.html) |
| Datahub 触发器 | [Datahub 触发器](https://help.aliyun.com/document_detail/60325.html) |
| IoT 触发器     | [IoT 触发器](https://help.aliyun.com/document_detail/64234.html) |



在template.yml中配置触发器

​          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1578999032921-ada8d3db-5199-4328-a029-e96aaf712f6d.png" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1578999032921-ada8d3db-5199-4328-a029-e96aaf712f6d.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 400px;">
​          如何实现一个简单的函数计算



## 创建函数入口

1. `exports.handler = ***\*function\****(event, context, callback) {`
2. `    callback(**null**, 'hello world');`
3. `};`

#### 函数名

- `exports.handler` 需要与创建函数时的 “handler” 字段相对应：例如创建函数时指定的 handler 为`index.handler`，那么函数计算会去加载 `index.js` 文件中定义的 `handler` 函数。

#### event 参数

- event 参数是您调用函数时传入的数据，其类型是 `Buffer`，是函数的输入参数。
- 函数不对它的内容进行任何解释，传递给函数的 event 是 `Buffer` 类型。您在函数中可以根据实际情况对 event 进行转换：例如输入数据是一个 JSON string ，您可以把它转换成一个 `object` :例如：传入的 event :

1. 1. `{`
   2. `"key": "value"`
   3. `}`

- 函数代码：

1. 1. `exports.handler = ***\*function\****(event, context, callback) {`
   2. `***\*var\**** eventObj = **JSON**.parse(event.toString());`
   3. `callback(**null**, eventObj['key']);`
   4. `};`

#### context 参数

- context 参数中包含一些函数的运行时信息（例如 request id / 临时 AK 等）。您在代码中可以使用这些信息。其类型是 `object`

## 创建初始化函数initializer

Initializer 是函数的初始化逻辑入口，不同于请求处理逻辑入口的 handler。在有函数初始化的需求场景中，设置了 Initializer 后，函数计算首先调用 initializer 完成函数的初始化，成功后再调用 handler 处理请求；没有函数初始化的需求则可以跳过 initializer，直接调用 handler 处理请求。



1. `exports.my_initializer = **function**(context, callback) {`
2. `    console.log('hello world');`
3. `    callback(**null**, "");`
4. `};`



- 执行时机
  运行函数逻辑的进程称之为函数实例，运行在容器内。系统会根据用户负载伸缩函数实例。每当有新函数实例创建时，系统会首先调用 initializer。系统保证一定 initializer 执行成功后才会执行 handler 逻辑。
- 最多成功执行一次
  系统保证每个函数实例启动后只会成功执行一次 initializer 。如果执行失败，那么该函数实例在收到 Invoke 请求之后都会先执行 initializer。一旦执行成功，那么该实例的生命周期内不会再执行 initializer ，收到 Invoke 请求之后只执行请求处理函数。



在nodejs中必须调用callback，否则会超时。



# 如何实现一个http请求



使用HTTP 触发器实现一个http请求。HTTP 触发器是众多函数计算触发器中的一种，通过发送 HTTP 请求触发函数执行。主要适用于快速构建 Web 服务等场景。HTTP 触发器支持 HEAD、POST、PUT、GET 和 DELETE 方式触发函数。

使用限制：

- 函数一旦设置 HTTP 触发器后不能设置其他类型触发器。
- 每个函数只能创建一个 HTTP 触发器。



## 使用原生的函数计算实现

函数计算提供了直接实现http请求的方法

### 创建入口函数

1. `**var** getRawBody = **require**('raw-body')`
2. `**module**.exports.handler = **function** (request, response, context) {`
3. `    // get requset header`
4. `    **var** reqHeader = request.headers`
5. `    **var** headerStr = ' '`
6. `    **for** (**var** key **in** reqHeader) {`
7. `        headerStr += key + ':' + reqHeader[key] + '  '`
8. `    };`
9. 
10. `    // get request info`
11. `    **var** url = request.url`
12. `    **var** path = request.path`
13. `    **var** queries = request.queries`
14. `    **var** queryStr = ''`
15. `    **for** (**var** param **in** queries) {`
16. `        queryStr += param + "=" + queries[param] + '  '`
17. `    };`
18. `    **var** method = request.method`
19. `    **var** clientIP = request.clientIP`
20. 
21. `    // get request body`
22. `    getRawBody(request, **function** (err, data) {`
23. `        **var** body = data`
24. `        // you can deal with your own logic here`
25. 
26. `        // set response`
27. `        // var respBody = new Buffer('requestURI' + requestURI + ' path' + path +  ' method' + method + ' clientIP' + clientIP)`
28. `        **var** respBody = **new** Buffer('requestHeader:' + headerStr + '\n' + 'url: ' + url + '\n' + 'path: ' + path + '\n' + 'queries: ' + queryStr + '\n' + 'method: ' + method + '\n' + 'clientIP: ' + clientIP + '\n' + 'body: ' + body + '\n')`
29. `        // var respBody = new Buffer( )`
30. `        response.setStatusCode(200)`
31. `        response.setHeader('content-type', 'application/json')`
32. 
33. `        response.send(respBody)`
34. 
35. `    })`
36. `};`

## 使用express实现



具体细节，可以看这里，[搭建基于 express 的 Serverless Web 应用](https://help.aliyun.com/document_detail/147099.html?spm=a2c4g.11186623.6.715.57c029e9Zhiypz)

这里主要提的一点是在express中无法使用body-parse的中间件，所以解析body的代码得自己实现          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579059013968-e816de9f-5182-441f-bce9-b5cea12db6b7.png?x-oss-process=image/resize,w_746" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579059013968-e816de9f-5182-441f-bce9-b5cea12db6b7.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 344px;">
          


​          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579059034415-48f84650-adf0-4ef3-bea9-acdb8c4eee79.png?x-oss-process=image/resize,w_746" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579059034415-48f84650-adf0-4ef3-bea9-acdb8c4eee79.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 520px;">
​          

## 配置访问域名

实现http触发器之后，可以直接使用阿里云提供的域名访问，也可以自己配置域名。

### 使用流程

- 使用 HTTP 触发器搭建 Web Server ;

- - 搭建过程请参考 [HTTP 触发器示例](https://help.aliyun.com/document_detail/74768.html)。

- 绑定自定义域名

- - 步骤一： 域名需要在阿里云备案或接入阿里云备案（海外集群不需要备案）；

- - - 备案请参考文章 [阿里云备案](https://help.aliyun.com/document_detail/61819.html) 。

- - 步骤二：域名需要解析到您的[endpoint](https://help.aliyun.com/document_detail/52984.html)上，即需要设置域名的 CNAME 到您对应区域的 endpoint，先设置 CNAME 再到函数计算进行绑定；

- - - 域名解析请参考文章 [设置域名解析](https://help.aliyun.com/document_detail/29716.html)；
    - 例如：您的域名为 `app.com`，您的 accountID 为 12345，区域为上海，需要设置 `app.com` 的 CNAME 为 `12345.cn-shanghai.fc.aliyuncs.com`。

- - 步骤三：在函数计算绑定自定义域名，并设置不同的路径到不同函数。

- - - **同一域名绑定的函数必须在同一区域，可以属于不同服务**；
    - 例如，您可以设置路径 `“/a”` 的请求到 `service1` 的 `function1` 执行，设置路径 `“/b”` 的请求到 `service2` 的 `function2` 执行；
    - 只有设置了 [HTTP 触发器](https://help.aliyun.com/document_detail/71229.html) 的函数才可以通过自定义域名的请求触发执行。

- 设置路由规则


​          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579059940539-0fe53ce7-ec7e-45e0-b8b0-270955feeed1.png?x-oss-process=image/resize,w_746" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579059940539-0fe53ce7-ec7e-45e0-b8b0-270955feeed1.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 229px;">
​          



# 如何访问mysql

访问mysql主要是需要安装第三方的依赖性，在fun的模式下会比较方便。直接在项目下通过npm 安装，然后fun deploy就行。

​          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579057433171-1d292163-9694-4ec6-89f0-726f3d531d67.png?x-oss-process=image/resize,w_746" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579057433171-1d292163-9694-4ec6-89f0-726f3d531d67.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 367px;">
​          



 需要注意的是，直接访问阿里云的云数据库是不行的，因为阿里云的数据库访问需要通过白名单，直接访问回被拒绝。但是由于运行函数计算的时候，IP地址是不固定的，所以没有办法直接只用IP地址设置白名单。因为需要使用持专有网络VPC。
          <img data-role="image" src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579060213750-2784874f-d819-4cc1-8eb3-5cdf0768e2c8.png?x-oss-process=image/resize,w_746" data-raw-src="https://cdn.nlark.com/yuque/0/2020/png/101638/1579060213750-2784874f-d819-4cc1-8eb3-5cdf0768e2c8.png" class="image lake-drag-image" alt="image.png" title="image.png" style="border: none; box-shadow: none; visibility: visible; width: 479px; height: 288px;">
        

当前文档的最新内容尚未更新，可进入编辑页面继续编辑或进行更新，更新后内容在阅读页可见

[吴秋雨](https://lukou.yuque.com/wuqiuyu) 最后更改于 03-18 18:03

# 阿里云函数计算在前端的实践

# 什么是函数计算

函数计算是事件驱动的全托管计算服务。使用函数计算，您无需采购与管理服务器等基础设施，只需编写并上传代码。函数计算为您准备好计算资源，弹性地可靠地运行任务，并提供日志查询、性能监控和报警等功能。

借助函数计算，您可以快速构建任何类型的应用和服务，并且只需为任务实际消耗的资源付费。（来自阿里云）

简单的讲就是你可以只关心业务代码的实现，不需要关系其他任何服务器配置，代理配置，监控等等运维相关的事情，只有把实现的代码托管到函数计算就行。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1578994519614-8b17a758-1770-46d2-9122-a743e779ca16.png)

函数计算支持多种语言：

nodejs

python

php

java

c#

Custom runtime



# 怎么开始一个函数计算

函数计算可以通过四种方式创建：

- - [使用控制台编写函数（**推荐**）](https://help.aliyun.com/document_detail/51783.html?spm=a2c4g.11186623.6.554.1e1b10f3PhAs0U#using-console)
  - [使用 Fun 创建函数（**推荐**）](https://help.aliyun.com/document_detail/51783.html?spm=a2c4g.11186623.6.554.1e1b10f3PhAs0U#using-fun)
  - [使用 fcli 创建函数](https://help.aliyun.com/document_detail/51783.html?spm=a2c4g.11186623.6.554.1e1b10f3PhAs0U#using-fcli)
  - [使用 SDK 创建函数](https://help.aliyun.com/document_detail/51783.html?spm=a2c4g.11186623.6.554.1e1b10f3PhAs0U#using-sdk)

  （官方文档都有就不详细介绍）



## 创建服务

什么是服务？在函数计算中，服务就是管理函数计算的基本资源单位。你可以在服务级别上授权、配置日志和创建函数等。

### 服务的属性

- serviceName（必选）：服务的名字
- description（可选）：服务的描述。
- role（可选）：授予函数计算执行函数所需权限, 使用场景包括：

- - 授权函数计算服务使用用户的日志服务资源存储和分析函数运行日志。
  - 授权函数计算服务运行需要访问其他云资源的函数。
  - 关于role的使用细节，请参考[函数计算权限管理](https://help.aliyun.com/document_detail/52885.html)。

- logConfig（可选）：设置日志服务的项目和日志库，存储和分析函数运行日志。

- - 如果您未配置该项，则无法查看函数运行日志。强烈建议您开启日志服务，并配置该属性.

- vpcConfig (可选) : 配置VPC选项可让函数访问指定VPC。
- internetAccess (可选) : 设为true可让函数访问公网。



在通过代码创建的项目中，服务配置在一个叫做template.yml的文件中



![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1578996221526-46436f15-bf9f-4e1f-886d-9c8c1d9af4ae.png)



## 创建函数

函数是系统调度和运行的单位。函数必须从属于服务，一个服务下的所有函数都共享该服务的属性，例如授权，日志设置。

### 函数属性

- - **functionName（必选）：**函数的名字。在当前服务内唯一。
  - **runtime（必选）：** 函数运行时类型。
  - **code（必选）：** 代码包。Java 语言需要上传 jar 包，其他语言上传 zip 包，可以存放在 OSS 上，或者直接上传代码包。
  - **handler（必选）：** 处理函数，它是函数计算系统运行用户函数的调用入口。
  - **description（可选）：** 函数的描述。函数计算系统并不会使用该属性值，但建议您为服务设置一个简洁、清晰的描述。
  - **timeout（可选）：** 函数的最大运行时间，单位为秒。
  - **MemorySize（可选）：** 函数运行所需的内存资源，单位为 MB。取值范围为 [128, 3072]，以 64 MB 为步长递进。
  - **Initializer（可选）：** [initializer](https://help.aliyun.com/document_detail/94670.html) 入口，它是函数计算系统运行用户 initializer 函数的调用入口。
  - **InitializationTimeout（可选）：** initializer 最大运行时间，单位为秒。



函数属性也配置在template.yml中



![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1578998549947-1099926b-0863-4996-9025-703d61dec716.png)



# 触发器

触发器是触发函数执行的方式。在事件驱动的计算模型中，事件源是事件的生产者，函数是事件的处理者，而触发器提供了一种集中的和统一的方式来管理不同的事件源。在事件源中，当事件发生时，如果满足触发器定义的规则，事件源则调用触发器所对应的函数。

## 触发器的基本信息

- `triggerName`：触发器名称。
- `triggerType`：触发器类型，例如（oss、timer、http）。
- `sourceArn`：触发函数执行的资源描述符，有些触发器需要设置（涉及到阿里云其他服务触发函数计算执行的，例如 OSS 触发器，SLS 触发器），有些触发器不需要设置（不涉及到阿里云其他服务触发函数计算执行的，例如定时触发器，HTTP 触发器）。例如 OSS 触发器的 sourceArn 格式为 `acs:oss:region:accountId:bucketName`。
- `invocationRole`：触发角色，事件源需要扮演一个角色来触发函数的执行，要求这个角色有触发函数执行的权限

- - 关于事件源角色的详细信息请参考文章 [权限简介](https://help.aliyun.com/document_detail/52885.html)。

- `qualifier`：触发的服务版本或别名。更多版本和别名的使用请参考[版本管理](https://help.aliyun.com/document_detail/96464.html)。
- `triggerConfig`：触发器的配置信息，各触发器的配置信息请参考该触发器对应的文档。



### 双向集成触发器

| 触发器名称        | 文档链接                                                     | 示例链接                                                     |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| OSS 事件触发器    | [OSS 事件触发器文档](https://help.aliyun.com/document_detail/62922.html) | [使用 OSS 触发函数计算示例](https://help.aliyun.com/document_detail/74761.html) |
| HTTP 触发器       | [HTTP 触发器](https://help.aliyun.com/document_detail/71229.html) | [基于 HTTP 触发器搭建 Web Server](https://help.aliyun.com/document_detail/74768.html) |
| 定时触发器        | [定时触发器](https://help.aliyun.com/document_detail/68172.html) | [使用定时触发器触发函数计算示例](https://help.aliyun.com/document_detail/74676.html) |
| MNS 主题触发器    | [MNS 主题触发器](https://help.aliyun.com/document_detail/97032.html) | [使用 MNS 主题触发器触发函数计算示例](https://help.aliyun.com/document_detail/97009.html) |
| Table Store触发器 | [Table Store触发器](https://help.aliyun.com/document_detail/100092.html) | [使用函数计算做数据清洗](https://help.aliyun.com/document_detail/71884.html) |
| CDN 事件触发器    | [CDN 事件触发器](https://help.aliyun.com/document_detail/73333.html) | [CDN 事件触发器示例](https://help.aliyun.com/document_detail/75119.html) |
| SLS 触发器        | [SLS 触发器](https://help.aliyun.com/document_detail/84386.html) | [日志服务触发器示例](https://help.aliyun.com/document_detail/84090.html) |

### 单向集成触发器

| 触发器名称     | 示例链接                                                     |
| -------------- | ------------------------------------------------------------ |
| API 网关触发器 | [API 网关触发器](https://help.aliyun.com/document_detail/54788.html) |
| Datahub 触发器 | [Datahub 触发器](https://help.aliyun.com/document_detail/60325.html) |
| IoT 触发器     | [IoT 触发器](https://help.aliyun.com/document_detail/64234.html) |



在template.yml中配置触发器

![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1578999032921-ada8d3db-5199-4328-a029-e96aaf712f6d.png)





#  

# 如何实现一个简单的函数计算



## 创建函数入口

1. `exports.handler = ***\*function\****(event, context, callback) {`
2. `    callback(**null**, 'hello world');`
3. `};`

#### 函数名

- `exports.handler` 需要与创建函数时的 “handler” 字段相对应：例如创建函数时指定的 handler 为`index.handler`，那么函数计算会去加载 `index.js` 文件中定义的 `handler` 函数。

#### event 参数

- event 参数是您调用函数时传入的数据，其类型是 `Buffer`，是函数的输入参数。
- 函数不对它的内容进行任何解释，传递给函数的 event 是 `Buffer` 类型。您在函数中可以根据实际情况对 event 进行转换：例如输入数据是一个 JSON string ，您可以把它转换成一个 `object` :例如：传入的 event :

1. 1. `{`
   2. `"key": "value"`
   3. `}`

- 函数代码：

1. 1. `exports.handler = ***\*function\****(event, context, callback) {`
   2. `***\*var\**** eventObj = **JSON**.parse(event.toString());`
   3. `callback(**null**, eventObj['key']);`
   4. `};`

#### context 参数

- context 参数中包含一些函数的运行时信息（例如 request id / 临时 AK 等）。您在代码中可以使用这些信息。其类型是 `object`

## 创建初始化函数initializer

Initializer 是函数的初始化逻辑入口，不同于请求处理逻辑入口的 handler。在有函数初始化的需求场景中，设置了 Initializer 后，函数计算首先调用 initializer 完成函数的初始化，成功后再调用 handler 处理请求；没有函数初始化的需求则可以跳过 initializer，直接调用 handler 处理请求。



1. `exports.my_initializer = **function**(context, callback) {`
2. `    console.log('hello world');`
3. `    callback(**null**, "");`
4. `};`



- 执行时机
  运行函数逻辑的进程称之为函数实例，运行在容器内。系统会根据用户负载伸缩函数实例。每当有新函数实例创建时，系统会首先调用 initializer。系统保证一定 initializer 执行成功后才会执行 handler 逻辑。
- 最多成功执行一次
  系统保证每个函数实例启动后只会成功执行一次 initializer 。如果执行失败，那么该函数实例在收到 Invoke 请求之后都会先执行 initializer。一旦执行成功，那么该实例的生命周期内不会再执行 initializer ，收到 Invoke 请求之后只执行请求处理函数。



在nodejs中必须调用callback，否则会超时。



# 如何实现一个http请求



使用HTTP 触发器实现一个http请求。HTTP 触发器是众多函数计算触发器中的一种，通过发送 HTTP 请求触发函数执行。主要适用于快速构建 Web 服务等场景。HTTP 触发器支持 HEAD、POST、PUT、GET 和 DELETE 方式触发函数。

使用限制：

- 函数一旦设置 HTTP 触发器后不能设置其他类型触发器。
- 每个函数只能创建一个 HTTP 触发器。



## 使用原生的函数计算实现

函数计算提供了直接实现http请求的方法

### 创建入口函数

1. `**var** getRawBody = **require**('raw-body')`
2. `**module**.exports.handler = **function** (request, response, context) {`
3. `    // get requset header`
4. `    **var** reqHeader = request.headers`
5. `    **var** headerStr = ' '`
6. `    **for** (**var** key **in** reqHeader) {`
7. `        headerStr += key + ':' + reqHeader[key] + '  '`
8. `    };`
9. 
10. `    // get request info`
11. `    **var** url = request.url`
12. `    **var** path = request.path`
13. `    **var** queries = request.queries`
14. `    **var** queryStr = ''`
15. `    **for** (**var** param **in** queries) {`
16. `        queryStr += param + "=" + queries[param] + '  '`
17. `    };`
18. `    **var** method = request.method`
19. `    **var** clientIP = request.clientIP`
20. 
21. `    // get request body`
22. `    getRawBody(request, **function** (err, data) {`
23. `        **var** body = data`
24. `        // you can deal with your own logic here`
25. 
26. `        // set response`
27. `        // var respBody = new Buffer('requestURI' + requestURI + ' path' + path +  ' method' + method + ' clientIP' + clientIP)`
28. `        **var** respBody = **new** Buffer('requestHeader:' + headerStr + '\n' + 'url: ' + url + '\n' + 'path: ' + path + '\n' + 'queries: ' + queryStr + '\n' + 'method: ' + method + '\n' + 'clientIP: ' + clientIP + '\n' + 'body: ' + body + '\n')`
29. `        // var respBody = new Buffer( )`
30. `        response.setStatusCode(200)`
31. `        response.setHeader('content-type', 'application/json')`
32. 
33. `        response.send(respBody)`
34. 
35. `    })`
36. `};`

## 使用express实现



具体细节，可以看这里，[搭建基于 express 的 Serverless Web 应用](https://help.aliyun.com/document_detail/147099.html?spm=a2c4g.11186623.6.715.57c029e9Zhiypz)

这里主要提的一点是在express中无法使用body-parse的中间件，所以解析body的代码得自己实现



![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1579059013968-e816de9f-5182-441f-bce9-b5cea12db6b7.png)          

![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1579059034415-48f84650-adf0-4ef3-bea9-acdb8c4eee79.png)

#  

## 配置访问域名

实现http触发器之后，可以直接使用阿里云提供的域名访问，也可以自己配置域名。

### 使用流程

- 使用 HTTP 触发器搭建 Web Server ;

- - 搭建过程请参考 [HTTP 触发器示例](https://help.aliyun.com/document_detail/74768.html)。

- 绑定自定义域名

- - 步骤一： 域名需要在阿里云备案或接入阿里云备案（海外集群不需要备案）；

- - - 备案请参考文章 [阿里云备案](https://help.aliyun.com/document_detail/61819.html) 。

- - 步骤二：域名需要解析到您的[endpoint](https://help.aliyun.com/document_detail/52984.html)上，即需要设置域名的 CNAME 到您对应区域的 endpoint，先设置 CNAME 再到函数计算进行绑定；

- - - 域名解析请参考文章 [设置域名解析](https://help.aliyun.com/document_detail/29716.html)；
    - 例如：您的域名为 `app.com`，您的 accountID 为 12345，区域为上海，需要设置 `app.com` 的 CNAME 为 `12345.cn-shanghai.fc.aliyuncs.com`。

- - 步骤三：在函数计算绑定自定义域名，并设置不同的路径到不同函数。

- - - **同一域名绑定的函数必须在同一区域，可以属于不同服务**；
    - 例如，您可以设置路径 `“/a”` 的请求到 `service1` 的 `function1` 执行，设置路径 `“/b”` 的请求到 `service2` 的 `function2` 执行；
    - 只有设置了 [HTTP 触发器](https://help.aliyun.com/document_detail/71229.html) 的函数才可以通过自定义域名的请求触发执行。

- 设置路由规则

![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1579059940539-0fe53ce7-ec7e-45e0-b8b0-270955feeed1.png)



# 如何访问mysql

访问mysql主要是需要安装第三方的依赖性，在fun的模式下会比较方便。直接在项目下通过npm 安装，然后fun deploy就行。

## ![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1579057433171-1d292163-9694-4ec6-89f0-726f3d531d67.png)



需要注意的是，直接访问阿里云的云数据库是不行的，因为阿里云的数据库访问需要通过白名单，直接访问回被拒绝。但是由于运行函数计算的时候，IP地址是不固定的，所以没有办法直接只用IP地址设置白名单。因为需要使用持专有网络VPC。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/101638/1579060213750-2784874f-d819-4cc1-8eb3-5cdf0768e2c8.png)

具体细节可以看这篇文章

[Serverless 解惑——函数计算如何访问 MySQL 数据库](https://blog.csdn.net/yunqiinsight/article/details/103887330)