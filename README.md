<P align="center">
Aliyun Cdn api Github Action
</P>

## 简介

一款基于阿里云 CDN API 的 Github Action，几乎支持所有的官方 API 调用，具体的细节和使用方式请结合[CDN API 参考](https://help.aliyun.com/document_detail/91856.html)

## 使用步骤

##### 1.明确意图

明确你想要用该 Action 来对你的 CDN 做些什么配置。然后去[官方 API 文档](https://help.aliyun.com/document_detail/106661.html)找到对应的方法，这里以最简单的`DescribeCdnDomainConfigs`举例子

![DescribeCdnDomainConfigs](http://show.newarray.vip/aliyun-cdn-api-action/DescribeCdnDomainConfigs.png)

##### 2.使用 Action

在你的 yml 文件中，使用该 Action 并传入正确的配置项即可即可

```yml
- name: DescribeCdnDomainConfigs
  id: getDescribeCdnDomainConfigsConfig
  # TODO:这里添加最后的插件版本
  uses: ./
  with:
    accessKeyId: ${{secrets.ALI_ACCESS_KEY_ID}}
    accessKeySecret: ${{secrets.ALI_ACCESS_KEY_SECRET}}
    parameters: '{"action": "DescribeCdnDomainConfigs", "domainName": "your domain name", "functionNames": "filetype_based_ttl_set,set_req_host_header" }'
```

##### 3.获取 API 返回结果

该 Action 只有一个输出项`responseBody`，应该也够用了。主要根据该配置项能够获取两个有效信息

- 1.调用 cdn api 是否成功
- 2.能够根据`responseBody`获取一些有用的信息，比如获取配置项的的`configId`，通过 configId 可以在接下来的步骤中，实现对指定配置的更新和删除

## 参数说明

##### 入参

- accessKeyId
- accessKeySecret
- parameters

对于前两个参数，直接在阿里云控制台`AccessKey管理`中获取即可。而`parameters`参数比较重要，它的类型是字符串对象。形如：

下面的例子是以`DescribeCdnDomainConfigs`方法做比喻，具体的配置项还需要结合你的使用场景做变更

```javascript
'{"action": "DescribeCdnDomainConfigs", "domainName": "your domain name", "functionNames": "filetype_based_ttl_set,set_req_host_header" }'
```

一切配置的信息都在该参数重配置，常见的有

- action
  - api 方法名，比如上述配置中的`DescribeCdnDomainConfigs`
- domainName
  - cdn 域名
- functionNames
  - 功能函数名称

##### 出参

- responseBody
  - 调用该 api 的返回结果作为`输出项`

```yml
- name: DescribeCdnDomainConfigs
  # 该步骤的id，后续的步骤可以通过steps.yourid.outputs.responseBody拿到域名配置信息
  id: getDescribeCdnDomainConfigsConfig
  # TODO:这里添加最后的插件版本
  uses: ./
  with:
    accessKeyId: ${{secrets.ALI_ACCESS_KEY_ID}}
    accessKeySecret: ${{secrets.ALI_ACCESS_KEY_SECRET}}
    parameters: '{"action": "DescribeCdnDomainConfigs", "domainName": "your domain name", "functionNames": "filetype_based_ttl_set,set_req_host_header" }'

- name: View My Cdn Domain config
  run: |
    echo ${{steps.getDescribeCdnDomainConfigsConfig.outputs.responseBody}}
```

## 对于 parameters 格式的阐述

一般开发者**不需要**过多关注下面的内容，除非你有一些**高级配置**需要配置，具体的配置项请参考[aliyun/tea-util](https://github.com/aliyun/tea-util/blob/10d152a3838594532b734956a3d72c81c94b4241/ts/src/client.ts#L8)

基本的参数，只用放到字符串对象顶层即可，比如`action` `domainName` `functionNames`等，但是对于一些高级的配置项，需要放置于`runtimeOptions`字段中，且`runtimeOptions`字段名是不可修改的

```javascript
'{
  "action": "DescribeCdnDomainConfigs",
  "domainName": "your domain name",
  "functionNames": "filetype_based_ttl_set,set_req_host_header",
  "runtimeOptions": {
    autoretry?: boolean;
    ignoreSSL?: boolean;
    maxAttempts?: number;
    backoffPolicy?: string;
    backoffPeriod?: number;
    readTimeout?: number;
    connectTimeout?: number;
    httpProxy?: string;
    httpsProxy?: string;
    noProxy?: string;
    ...
  }
}'

```

另外除了`runtimeOptions`字段，其他项的 value，在我的程序中都会进行判断，如果`typeof value === 'object'`，我都会调用`JSON.stringify`进行干涉

```javascript
let {action, runtimeOptions, ...requestOptions} = JSON.parse(parameters)

for (let [optionsKey, optionValue] of Object.entries(requestOptions)) {
  if (typeof optionValue === 'object') {
    requestOptions[optionsKey] = Util.toJSONString(requestOptions.functions)
  }
}
```

这么做的目的是因为据我了解，大部分的配置项类型都是字符串，如果不做任何处理的话，可能在经过程序中的 `JSON.parse(parameters)`解析参数过后，会导致类型的问题致使运行程序失败

对于我这么做的方式，我也不确定它到底对不对，但是至少看起来能够保证程序正常的运行。

## Contributing

Feel free to submit [issues](https://github.com/weirui88888/aliyun-cdn-api/issues) or [prs](https://github.com/weirui88888/aliyun-cdn-api/pulls) to me.
