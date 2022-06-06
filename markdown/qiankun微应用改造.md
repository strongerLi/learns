## qiankun微应用改造

主应用配置：

```javascript
registerMicroApps([
    {
        name: 'sdsecApp',
        entry: '//localhost:8082',
        container: '#container',
        activeRule: '/sdsecfen',
    },
    ]);
    // 启动 qiankun
    start();
```

子应用路由配置：

```javascript
const routerInstance = new Router({
    base: '/sdsecfen',
    mode: 'history',
    routes: ROUTES
  })
```

当子应用vue.config.js配置中不设置publicPath时

通过主应用能正常访问子应用的路由

当增加publicPath时：

```
module.exports = {
  publicPath: '/sdsecfen/',
  outputDir: 'dist/sdsecfen',
  //...
}
```

主应用访问同样的路由报错

![1647912523589](C:\Users\win 10\AppData\Roaming\Typora\typora-user-images\1647912523589.png)