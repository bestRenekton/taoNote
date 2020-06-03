<!--
 * @Description:
 * @Author: yyt
 * @Date: 2020-06-03 09:32:00
 * @LastEditors: yyt
 * @LastEditTime: 2020-06-03 10:36:58
-->

# sentry

> Sentry 是一个开源的实时错误追踪系统，可以帮助开发者实时监控并修复异常问题。提供了对多种主流语言和框架的支持，包括 React、Angular、Node、Django、RoR、PHP、Laravel、Android、.NET、JAVA 等。我们可以直接使用它家提供的在线服务，也可以本地自行搭建；

## 使用在线服务

- 没什么好说的，官网直接用

## 私有化部署

- 拥有自己的异常监控平台，就很香

  ![sentry异常列表页](https://yangyuetao.cn/demos/static/img/taoNote/5.png)

### 拉取镜像

```
docker pull redis
docker pull postgres
docker pull sentry
```

### 启动 redis 和 sentry

- docker run -d --name sentry-redis redis

- docker run -d --name sentry-postgres -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=sentry postgres

### 获取密钥

- docker run --rm sentry config generate-secret-key
- 这里会获得密钥，自己保存好，后面会用

### 启动服务

- docker run -it --rm -e SENTRY_SECRET_KEY='你的密钥' --link sentry-postgres:postgres --link sentry-redis:redis sentry upgrade `（这一步会提示使用邮箱和密码创建用户，之后可以进入my-sentry,使用sentry createuser再次建立）`

- docker run -d -p 9000:9000 --name my-sentry -e SENTRY_SECRET_KEY='你的密钥' --link sentry-redis:redis --link sentry-postgres:postgres sentry

- docker run -d --name sentry-cron -e SENTRY_SECRET_KEY='你的密钥' --link sentry-postgres:postgres --link sentry-redis:redis sentry run cron

- docker run -d --name sentry-worker-1 -e SENTRY_SECRET_KEY='你的密钥' --link sentry-postgres:postgres --link sentry-redis:redis sentry run worker

## 查看容器

- docker ps -a 查看容器，其中
  - my-sentry 是主程序
  - sentry-worker 是 sentry 的异步队列
  - sentry-corn 是 sentry 的定时任务
  - sentry-postgres 是数据库
  - sentry-redis 是缓存

## 配置邮箱

- 像测试邮件是使用 sentry 的主程序测试的，而平常收到的异常日志和邀请邮件是 sentry-worker 异步执行的，如果只改了主程序，那么就会出现测试邮件的时候正常，却收不到异常日志和邀请邮件的情况。
- 进入主容器 my-sentry
- cd /etc/sentry/
- vim config.yml
- 如果没有安装 vim
  - apt-get update
  - apt-get install vim -y
- 配置 config.yml 邮箱设置

  ```bash
  ###############
  # Mail Server #
  ###############
  # 修改这里，取消原本的注释
  mail.backend: 'smtp'
  mail.host: 'smtp.qq.com' #邮箱对应的smtp域名
  mail.port: 587 #邮箱对应端口
  mail.username: 'xx@xx.com'  #你的邮箱
  mail.password: 'pwd'  #你设置的密码，注意不是邮箱登录密码
  mail.use-tls: true #是否使用tls连接
  #The email address to send on behalf of
  mail.from: 'xx@xx.com' #发送者，填的和user一样就行
  # If you'd like to configure email replies, enable this.
  ```

- exit 退出容器
- docker restart xxxxx 重启容器
- 然后将 sentry-worker 容器也一样改掉就行。

## 项目使用

- 因为 onerror 信息比监听 error 更全，所以 sentry 是用的 onerror，和监听 unhandledrejection,这样我们只需要补全`资源错误`，`vue/react项目错误`,`接口报错`就可以了

### 整一个 Sentry 类出来，实现监听资源错误上报和暴露上报方法

```ts
// Report.ts
import * as Sentry from "@sentry/browser";
import * as Integrations from "@sentry/integrations";

type Environment = "production" | "development";
interface ReportOptions {
  enabled?: boolean;
  dsn: string;
  release: string;
  environment: Environment;
}
class Report {
  public Vue: any;
  public options: ReportOptions;
  private static instance: Report;

  constructor(Vue: any, options: ReportOptions) {
    this.Vue = Vue;
    this.options = options;
  }

  public static getInstance(Vue: any, options: ReportOptions) {
    if (!this.instance) {
      this.instance = new Report(Vue, options);
      this.instance.init();
      this.instance.loadListener();
    }
    return this.instance;
  }

  // 初始化
  public init() {
    Sentry.init({
      enabled: this.options.enabled,
      dsn: this.options.dsn,
      integrations: [
        new Integrations.Vue({ Vue: this.Vue, attachProps: true }),
      ],
      release: this.options.release,
      environment: this.options.environment,
    });
  }
  // 主动上报
  public log(info: any) {
    Sentry.withScope((scope) => {
      Object.keys(info).forEach((key) => {
        if (key !== "error") {
          scope.setExtra(key, info[key]);
        }
      });
      Sentry.captureException(info.error || new Error("未知错误"));
    });
  }
  // 全局监控资源加载错误
  public loadListener() {
    window.addEventListener(
      "error",
      (event) => {
        // 过滤 js error
        const target = event.target || event.srcElement;
        const isElementTarget =
          target instanceof HTMLScriptElement ||
          target instanceof HTMLLinkElement ||
          target instanceof HTMLImageElement;
        if (!isElementTarget) {
          return false;
        }
        // 上报资源地址
        const url =
          (target as HTMLScriptElement | HTMLImageElement).src ||
          (target as HTMLLinkElement).href;

        this.log({
          error: new Error(`ResourceLoadError: ${url}`),
          type: "resource load",
        });
      },
      true
    );
  }
}
export default Report;
```

### 添加 vue 全局错误上报

```ts
//main.ts
//全局监控
const sentry = Report.getInstance(Vue, {
  enabled: process.env.NODE_ENV === "production",
  dsn: SENTRYDSN,
  release: version,
  environment: process.env.NODE_ENV,
});
Vue.prototype.$sentry = sentry;
Vue.config.errorHandler = (error, vm, info) => {
  console.log("errorHandler", error, vm, info);
  vm.$sentry.log({
    error,
    type: "vue errorHandler",
    vm,
    info,
  });
};
let VueApp = new Vue({
  router,
  store,
  render: (h) => h(App),
}).$mount("#app");

export default VueApp;
```

### 添加 axios 接口错误上报

```ts
//fetch.ts
import VueApp from "@/main";
axiosInstance.interceptors.response.use(
  (res) => {
    ...
  },
  err => {
    VueApp.$sentry.log({
      error: err,
      type: "callNative",
      config: err.config
    });
    ...
  }
);
```
