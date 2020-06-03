<!--
 * @Description: 
 * @Author: yyt
 * @Date: 2020-06-03 09:32:00
 * @LastEditors: yyt
 * @LastEditTime: 2020-06-03 09:37:41
-->

# 使用 sentry

> 通过之前的错误收集，我们已经可以自己搭建一个异常监控平台了，当然你也可以选择其他成熟的监控平台，Sentry 就是这样一个平台。Sentry 是一个开源的实时错误追踪系统，可以帮助开发者实时监控并修复异常问题。提供了对多种主流语言和框架的支持，包括 React、Angular、Node、Django、RoR、PHP、Laravel、Android、.NET、JAVA 等。我们可以直接使用它家提供的在线服务，也可以本地自行搭建；

- 因为 onerror 信息比监听 error 更全，所以 sentry 是用的 onerror，和监听 unhandledrejection,这样我们只需要补全`资源错误`，`vue/react项目错误`,`接口报错`就可以了
