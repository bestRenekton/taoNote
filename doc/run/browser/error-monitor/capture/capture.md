<!--
 * @Description:
 * @Author: yyt
 * @Date: 2020-06-03 09:31:53
 * @LastEditors: yyt
 * @LastEditTime: 2020-06-03 09:49:02
-->

# 错误分类

- js 错误
- 资源请求错误
- 接口报错

# 捕获方法分类

- try-catch
- window.onerror=function(){...}
- window.addEventListener('error',function(){...})
- window.addEventListener("unhandledrejection", (e) => {...});
- vue 项目 Vue.config.errorHandler = (error, vm, info) => {}
- react 项目 componentDidCatch(err,info){}

## try-catch

- js 报错机制，如果一个任务执行了一个错误的方法，后续程序执行停止了。但是另外一个任务并不会影响。

```js
setTimeout(() => {
  console.log("start-1");
  console.log(a);
  console.log("end-1");
}, 0);
setTimeout(() => {
  console.log("start-2");
  console.log("end-2");
}, 0);
```

- try catch 捕捉错误

```js
setTimeout(() => {
  try {
    console.log("start-1");
    console.log(a);
    console.log("end-1");
  } catch (err) {
    console.log("catch", err);
  }
}, 0);
setTimeout(() => {
  console.log("start-2");
  console.log("end-2");
}, 0);
```

- 你可能想在底层 try-catch，但是 try-catch 在异步中会失效

```js
try {
  setTimeout(() => {
    console.log("start-1");
    console.log(a);
    console.log("end-1");
  }, 0);
  setTimeout(() => {
    console.log("start-2");
    console.log("end-2");
  }, 0);
} catch (err) {
  console.log("catch", err);
}
```

## window.onerror

- window.onerror 最大的好处就是可以同步任务还是异步任务都可捕获。

```js
setTimeout(() => {
  console.log("start-1");
  console.log(a);
  console.log("end-1");
}, 0);
setTimeout(() => {
  console.log("start-2");
  console.log("end-2");
}, 0);
window.onerror = (...args) => {
  console.log("onerror", args);
  return true; //如果不返回true,报错依然会上抛
};
```

- 但是无法获取资源错误，比如`<img src="./xxxxx.png" />`

## 监听 error 事件

- 可以使用 addEventListener error 来捕获资源的错误

```js
window.addEventListener(
  "error",
  (args) => {
    console.log("linstener err", args);
    return true;
  },
  {
    capture: true, //必须是捕获阶段，才可以捕获到资源错误
  }
);
```

- window.onerror 和监听 error 事件，都捕获不到 promise，可以监听 unhandledrejection 来解决

## 监听 unhandledrejection 事件

```js
window.addEventListener("unhandledrejection", (e) => {
  console.log("unhandledrejection", e.reason);
  throw e.reason; //这里可以抛出错误事件，之后交由window.onerror统一处理
});
new Promise((res, rej) => {
  error;
});
//await也可以监听到
(async () => {
  await e;
})();
```

## 总结

| 异常类型                | 同步 | 异步 | 资源加载 | promise |
| ----------------------- | ---- | ---- | -------- | ------- |
| try-catch               | 支持 |      |          |         |
| onerror                 | 支持 | 支持 |          |         |
| 监听 error              | 支持 | 支持 | 支持     |         |
| 监听 unhandledrejection |      |      |          | 支持    |

# 终极方案

## 思路

- 针对 js 错误
  - 使用`onerror`,因为 onerror 比监听 error 的信息更全
  - 使用`监听unhandledrejection`
- 资源请求错误
  - 使用`监听error`
- 接口报错
  - axios 中统一封装
- 框架错误
  - vue 项目 `Vue.config.errorHandler` = (error, vm, info) => {}
  - react 项目 `componentDidCatch`(err,info){}

## 效果

![实践](https://yangyuetao.cn/demos/static/img/taoNote/2.png)

## 实践

- 全局异常汇总

```js
function reportError(type, err, ...other) {
  console.group(`捕获到错误：${err}`);
  console.log(`类型:${type}`);
  console.log(err);
  console.log(`其他：${other}`);
  console.groupEnd();
  //这里可以做上传错误之类的...
}
```

- js 错误

```js
window.onerror = function (...args) {
  reportError("JsError", args);
};
```

- 加载资源错误

```js
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
    reportError("ResourceLoadError", url);
  },
  true
);
```

- promise 类错误

```js
window.addEventListener("unhandledrejection", (e) => {
  throw e.reason; //抛出错误事件统一处理
});
```

- vue 全局错误

```js
Vue.config.errorHandler = (error, vm, info) => {
  reportError(info, error);
};
```

- axios 统一接口错误

```js
    ...
    .catch((error) => {
      reportError(
        "NetworkError",
        error,
        `${opt.method ? opt.method : "get"}===${opt.url}===${JSON.stringify(
          opt.data
        )}`
      );
    }
```

# 使用 sentry

> 通过之前的错误收集，我们已经可以自己搭建一个异常监控平台了，当然你也可以选择其他成熟的监控平台，Sentry 就是这样一个平台。Sentry 是一个开源的实时错误追踪系统，可以帮助开发者实时监控并修复异常问题。提供了对多种主流语言和框架的支持，包括 React、Angular、Node、Django、RoR、PHP、Laravel、Android、.NET、JAVA 等。我们可以直接使用它家提供的在线服务，也可以本地自行搭建；

[使用 Sentry](../sentry/sentry.md)
