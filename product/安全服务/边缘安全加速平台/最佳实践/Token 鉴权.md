
## 功能简介
Token 鉴权为一种访问控制策略，通过配置鉴权规则进行访问校验，过滤不合法的访问请求。可有效防止站点资源被恶意盗刷，保护您的业务内容。

**Token 鉴权如何做到访问控制？**

客户端用户在发起请求时，其访问请求 URL 需按鉴权规则生成鉴权 URL，当且仅当鉴权 URL 中的鉴权信息（例如：时间戳）通过节点校验，即鉴权通过时，该访问请求才会被视为合法请求，节点正常响应。若校验失败，则节点拒绝该访问，直接返回403。

## 操作步骤
1. 登录 [边缘安全加速平台控制台](https://console.cloud.tencent.com/edgeone)，在左侧菜单栏中，单击**规则引擎**。
2. 在规则引擎页面，选择所需站点，单击![](https://qcloudimg.tencent-cloud.cn/raw/fe4d4900f8ad69d506adc49bdb70fa32.png)可按需配置 Token 鉴权规则。
>?目前仅支持匹配条件为全部（任意请求）或 Host 时配置 Token 鉴权操作。
>
参数说明：
<table>
<thead>
<tr>
<th>配置项</th>
<th>说明</th>
</tr>
</thead>
<tbody><tr>
<td>鉴权方式</td>
<td>目前支持4种鉴权签名计算方式，请您根据访问 URL 格式选择合适的方式。详情请参见 <a href="#jqfs">鉴权方式</a></td>
</tr>
<tr>
<td>密钥（主）</td>
<td>鉴权方式对应的主用密码，由6-40位大小写英文字母或数字组成</td>
</tr>
<tr>
<td>密钥（备）</td>
<td>鉴权方式对应的备用密码，由6-40位大小写英文字母或数字组成</td>
</tr>
<tr>
<td>鉴权参数</td>
<td>鉴权参数名称，节点将校验此参数名对应的值。由1 - 100位大小写字母、数字或下划线组成</td>
</tr>
<tr>
<td>有效时长</td>
<td>配置的鉴权 URL 的有效时长，用于判断客户端访问请求是否过期。若当前时间超过”timestamp + 有效时长“时间，则为过期请求，直接返回403；若未过期，则继续校验。<br>单位：秒<br>范围：1 - 630720000</td>
</tr>
</tbody></table>


## 注意事项
1. 鉴权通过后，节点将自动忽略鉴权相关参数后的 URL 作为缓存标识（Cache key），提高缓存命中率，减少回源。
2. 鉴权通过后，若未命中节点缓存，则会继续回源，实际回源 URL 将与 鉴权 URL 格式一致，保留鉴权参数，源站可按需忽略或二次校验。
3. URL 中不能包含中文。

## 鉴权方式[](id:jqfs)

### 方式 A

#### 鉴权 URL 格式
```js.
http://Hostname/Filename?sign=timestamp-rand-uid-md5hash
```

#### 参数说明

| 字段      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| Hostname  | 站点加速域名                                                 |
| Filename  | 实际回源访问的 URL，需以`/`开头                              |
| sign      | 您自定义设置的鉴权参数名称                                   |
| timestamp | 访问 URL 中携带的时间戳<br/>格式：十进制（UNIX 时间戳）      |
| rand      | 随机字符串，由 0 - 100位大小写字母与数字组成                 |
| uid       | 用户 ID，默认为0                                              |
| md5hash   | 通过 MD5 算法计算出的字符串 <br/>算法：MD5（/Filename-timestamp-rand-uid-自定义密钥）<br/><br/>若请求未过期，则节点比较此字符串值与访问请求中携带的 `md5hash` 值：<li> 两值相同，鉴权通过，响应请求</li><li> 两值不同，鉴权失败，返回403</li> |

### 方式 B

#### 鉴权 URL 格式
```js.
http://Hostname/timestamp/md5hash/Filename
```

#### 参数说明

| 字段      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| Hostname  | 站点加速域名                                                 |
| Filename  | 实际回源访问的 URL，需以`/`开头                              |
| timestamp | 访问 URL 中携带的时间戳<br/>格式：YYYYMMDDHHMM               |
| md5hash   | 通过 MD5 算法计算出的字符串<br/>算法：MD5（自定义密钥 + timestamp + /Filename）<br/><br/>若请求未过期，则节点比较此字符串值与访问请求中携带的 `md5hash` 值：<li>两值相同，鉴权通过，响应请求</li><li> 两值不同，鉴权失败，返回403</li> |

### 方式 C

**鉴权 URL 格式**

```js.
http://Hostname/md5hash/timestamp/Filename
```

#### 参数说明

| 字段      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| Hostname  | 站点加速域名                                                 |
| Filename  | 实际回源访问的 URL，需以`/`开头                              |
| timestamp | 访问 URL 中携带的时间戳<br/>格式：十六进制（UNIX 时间戳）    |
| md5hash   | 通过 MD5 算法计算出的字符串<br/>算法：MD5（自定义密钥+ /Filename + timestamp ）<br/><br/>若请求未过期，则节点比较此字符串值与访问请求中携带的 `md5hash` 值：<li>两值相同，鉴权通过，响应请求</li><li> 两值不同，鉴权失败，返回403</li> |


### 方式 D

#### 鉴权 URL 格式

```js.
http://Hostname/Filename?sign=md5hash&t=timestamp
```

#### 参数说明

| 字段      | 说明                                                         |
| :-------- | :----------------------------------------------------------- |
| Hostname  | 站点加速域名                                                 |
| Filename  | 实际回源访问的 URL，需以`/`开头                              |
| sign      | 您自定义设置的鉴权参数名称                                   |
| t         | 您自定义设置的时间戳参数名称                                 |
| timestamp | 访问 URL 中携带的时间戳<br>格式：十进制（UNIX 时间戳）；十六进制（UNIX 时间戳） |
| md5hash   | 通过 MD5 算法计算出的字符串<br/>算法：MD5（自定义密钥 + /Filename + timestamp） <br/><br/>若请求未过期，则节点比较此字符串值与访问请求中携带的 `md5hash` 值：<li> 两值相同，鉴权通过，响应请求</li><li>两值不同，鉴权失败，返回403</li> |
