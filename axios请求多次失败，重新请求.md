# axios拦截器加状态验证，重新请求接口
### 1. 定义常量
```js
// 是否正在重新请求的标记
let OMCServerIsStarting = false;
// 重试队列，每一项将是一个待执行的函数形式
let requests = [];
let configQueue = [];
// 需要被验证的接口路由组
const verifyRoute = ['model', 'simulation', 'file'];
```

### 2. 在http.interceptors.request拦截器中将正在重新请求重启服务连接时的请求接口放进队列中，在服务连接后重新请求
```js
const isVerifyUrl = verifyRoute.includes(config.url.split('/')[1]);
// 判断是否为需要验证的接口及是否正在开始请求服务
if (isVerifyUrl && OMCServerIsStarting) {
  // 将接口存到configQueue中，让其状态一直为pending，等服务重启后重新调用
  return new Promise((resolve) => {
    configQueue.push(() => {
      resolve(config);
    });
  });
}
```

### 3. 在http.interceptors.response中拦截返回值，接口成功时判断状态
```js
// 判断是否需要验证及状态是否为特定值，ResultEnum.REFRESH_STATUS为枚举值
const isVerifyUrl = verifyRoute.includes(response.config.url.split('/')[1]);
if (isVerifyUrl && response.data.status === ResultEnum.REFRESH_STATUS) {
  // 获取当前失败的请求
  const config = response.config;
  if (!OMCServerIsStarting) {
    OMCServerIsStarting = true;
    return startOMCService()
      .then(() => {
        // 已经刷新了用户空间，将所有队列中的请求进行重试
        requests.forEach((cb) => cb());
        configQueue.forEach((cb) => cb());
        // 重试完了别忘了清空这个队列
        requests = [];
        configQueue = [];
        // 重试当前请求并返回promise
        return http(config);
      })
      .catch(() => {
         Message.error('OMC服务连接失败');
       })
       .finally(() => {
         OMCServerIsStarting = false;
       });
  }
  // 正在刷新用户空间，返回一个未执行resolve的promise
  return new Promise((resolve) => {
    // 将resolve放进队列，用一个函数形式来保存，等登录用户空间刷新后直接执行
    requests.push(() => {
      resolve(http(config));
    });
  });
}
```
