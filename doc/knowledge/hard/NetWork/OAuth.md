<!--
 * @Author: yyt
 * @Date: 2020-05-23 10:45:43
 * @LastEditTime: 2020-05-23 11:18:01
 * @LastEditors: yyt
 * @FilePath: /taoNote/doc/knowledge/hard/NetWork/OAuth.md
-->

# OAuth

## 原理

> OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

## 令牌与密码差异

- 令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效，用户不修改，就不会发生变化。
- 令牌可以被数据所有者撤销，会立即失效。密码一般不允许被他人撤销。
- 令牌有权限范围（scope），比如只能进小区的二号门。对于网络服务来说，只读令牌就比读写令牌更安全。密码一般是完整权限。

## 四种方式

### 授权码式

> 这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

- A 前端跳转 B，同时携带`clientId`,`A后端的callBack`
- B 要求用户登录，之后询问是否同意授权,如果同意，则会打到 A 后端的 callBack 请求，并携带`授权码code`
- A 后端拿到`授权码code`，并使用`client_secret`去请求 B 的`令牌token`，因为是后端请求，所以很安全
- A 后端拿到`令牌token`，去请求 B 的 API

### 隐藏式

> 有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端,是很不安全的。

- A 前端跳转 B，同时携带`clientId`,`A前端地址的callBack`
- B 直接将`令牌token`返回到 A 前端地址的锚点上，比如`https://a.com/callback#token=ACCESS_TOKEN`

### 密码式

> 直接使用密码去请求凭证，需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。

- A 前端请求 B 接口，同时携带`clientId`,`username`,`password`
- B 直接在返回里给出`令牌token`

### 凭证式

> 适用于没有前端的命令行应用，即在命令行下请求令牌。

- A 在命令行向 B 发出请求,携带`clientId`,`client_secret`
- B 验证后，直接返回`令牌token`

## 令牌使用,更新

- 网站拿到令牌以后，就可以向 B 网站的 API 请求数据了。每个发到 API 的请求，都必须带有令牌。可以统一携带在 header 中

```js
const result = await axios({
  method: "get",
  url: `https://api.github.com/user`,
  headers: {
    accept: "application/json",
    Authorization: `token ${accessToken}`,
  },
});
```

- 令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0 允许用户自动更新令牌。

## 例子

### Github

- A 网站让用户跳转到 GitHub。
- GitHub 要求用户登录，然后询问"A 网站要求获得 xx 权限，你是否同意？"
- 用户同意，GitHub 就会重定向回 A 网站，同时发回一个授权码。
- A 网站使用授权码，向 GitHub 请求令牌。
- GitHub 返回令牌.
- A 网站使用令牌，向 GitHub 请求用户数据。
