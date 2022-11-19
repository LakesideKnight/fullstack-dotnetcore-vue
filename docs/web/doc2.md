

## 安装axios
* 通过以下命令安装axios：
```text
npm install axios;
```
* 在src目录中，新建一个api文件夹用于统一管理后台接口调用，新建api/http.js用于封装axios。

## 封装axios
### 1.引入axios 
```js
//api/http.js
import axios from 'axios';
```
### 2.配置axios 默认请求地址
* 在.env, .env.development中新增字段配置不同环境下后台接口地址：
```text
VUE_APP_APIURL = "http://www.xxx.com"
```
* 配置axios默认地址：
```js
//api/http.js
axios.defaults.baseURL = process.env.VUE_APP_APIURL;
```

### 3.配置请求超时
```js
//api/http.js
axios.defaults.timeout = 30000;
```

### 4.配置请求头格式
```js
//api/http.js
axios.defaults.headers.post['Content-Type'] = 'application/json;charset=UTF-8';
```

### 5.配置请求拦截器
```js
//api/http.js
axios.interceptors.request.use(
    config => {
        // 每次发送请求之前判断是否存在token，如果存在，则统一在http请求的header都加上token，不用每次请求都手动添加了
        // 即使本地存在token，也有可能token是过期的，所以在响应拦截器中要对返回状态进行判断
        //此处也可以加上接口加密相关处理操作
        const token = store.state.token;
        token && (config.headers.Authorization = token);
        return config;
    },
    error => {
        return Promise.error(error);
    });
```

### 6.配置响应拦截器
可以在响应拦截器各状态码下配置相关场景业务代码：
```js
//api/http.js
// 响应拦截器
axios.interceptors.response.use(
    response => {
        if (response.status === 200) {
            return Promise.resolve(response);
        } else {
            return Promise.reject(response);
        }
    },
    // 服务器状态码不是200的情况    
    error => {
        if (error.response.status) {
            switch (error.response.status) {
                // 401: 未登录                               
                case 401:
                    break;
                // 403 token过期                    
                case 403:
                    this.$message.info('登录过期，请重新登录');
                    break;
                // 404请求不存在                
                case 404:
                    this.$message.error('网络请求不存在');
                    break;
                // 500请求不存在
                case 500:
                    this.$message.error('服务端内部错误');
                    break;
                // 其他错误，直接抛出错误提示                
                default:
                    this.$message.error(error.response.statusText);
                    console.log(error);
                    break;
            }
            return Promise.reject(error.response);
        }
    }
);
```

## 封装GET,POST请求
```js
//api/http.js
/** 
 * get方法，对应get请求 
 * @param {String} url [请求的url地址] 
 * @param {Object} params [请求时携带的参数] 
 */
export function get(url, params) {
    return new Promise((resolve, reject) => {
        axios.get(url, { params: params })
            .then(res => {
                resolve(res.data);
            })
            .catch(err => {
                reject(err.data)
            })
    });
}
/** 
 * post方法，对应post请求 
 * @param {String} url [请求的url地址] 
 * @param {Object} params [请求时携带的参数] 
 */
export function post(url, params) {
    return new Promise((resolve, reject) => {
        axios.post(url, params)
            .then(res => {
                resolve(res.data);
            })
            .catch(err => {
                reject(err.data)
            })
    });
}
```

## 接口加密处理
> [!WARNING]
> 待完善


## 后台接口对接
详见[实现模块接口对接](/web/doc3.md#实现模块接口对接)
