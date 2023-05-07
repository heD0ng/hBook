### 前言
在之前秋招的时候, 被面试官问过相关知识, 但是由于当时面试、学校毕业等原因没有系统地学习, 关于PWA的博客也就耽搁到了现在. 

### 什么是PWA?
> PWA（Progressive web apps，渐进式 Web 应用）运用现代的 Web API 以及传统的渐进式增强策略来创建跨平台 Web 应用程序。这些应用无处不在、功能丰富，使其具有与原生应用相同的用户体验优势.

PWA技术其实是存在着兼容性问题, 如果浏览器不支持PWA技术，那么对原有应用不会产生影响，对于支持该项技术的浏览器，他会在原有基础上新增它的特性，让用户得到更好的体验。目前在Vue、React脚手架中已经集成了该项技术，一旦你拥有一个web app项目，那么你的PWA之旅就已经开始了。PWA的优势在于
>1、提升体验(应用可添加桌面、打开应用时像原生应用)
>2、离线缓存
>3、后台通知

对于现在的前端开发或者所有开发而言, 网络是一个必备的条件. 一旦用户缺少网络这一必备条件, 就会让用户无法访问任何应用, 这在一定程度是会丢失用户的. 像淘宝、京东这类电商App, 一旦丢失用户就是Money上的丢失, 所以像这类App目前已经是支持了PWA技术的. PWA的主要优势点还是在于离线可访问, 但很可惜, 我特意去看过一些知名的网站(掘金、知乎等), 好像支持PWA技术的并不多(这也由于国内生态的原因, 如安卓机与IOS带来的一些问题). 但是从前端开发的角度, 学习这一部分的知识是必不可少的, 因为离线访问技术又可能在未来会成为一种趋势, 提前学习和了解才能更好成长.

### 离线缓存
在讲述离线缓存之前, 需要知道一点就是PWA必须运行在https或者localhost的环境下. 离线缓存主要包括2点: 
> 1、数据缓存
> 2、缓存读取

#### Manifest.json
首先需要配置应用缓存的相关信息, 其实也就是个json配置文件manifest.json, 具体配置信息如下:
```javascript
 {
    "name": "hello world",//应用名称
    "short_name": "hello",//短名称，用于在桌面显示
    "start_url": ".",//入口url
    "display": "standalone",//应用的展现模式，一般来说这个模式体验最优
    "background_color": "#fff",//应用的主题颜色，一般会改变你的上方菜单栏背景颜色
    "description": "A simply hello world app.",//应用描述
    "icons": [
	    {//在不同环境下展现的应用图标
	    "src": "images/touch/homescreen48.png",
	    "sizes": "144x144",
	    "type": "image/png"
	    }
    ]
 }
```
使用方式, 在html文件运用link标签, 具体如下
```javascript
<link rel="manifest" href="manifest.json">
```
关于manifest.json的更多配置信息, 可以MDN文档参考:[MDN Web App Manifest](https://developer.mozilla.org/zh-CN/docs/Web/Manifest)

#### 离线缓存
离线缓存包括2部分--service worker、cache storage, 其中service worker主要支持离线访问、cache storage主要支持数据的缓存与读取.

##### service worker
service worker是PWA技术实现中的重要一环, 正是它实现了离线访问. 具体特性详解如下:
>1、service worker允许web应用在网络环境比较差或者是离线的环境下依旧可以使用
>2、service worker可以极大的提升web app的用户体验
>3、service worker是一个独立的 worker 线程，独立于当前网页进程，是一种特殊的web worker
>4、一旦被 install，就永远存在，除非被手动 unregister
>5、用到的时候可以直接唤醒，不用的时候自动睡眠
>6、可编程拦截代理请求和返回，缓存文件，缓存的文件可以被网页进程取到（包括网络离线状态）
>7、离线内容开发者可控

在使用的时候需要先注册service worker, 具体方法如下:
```javascript
window.onload = function() {
// 先判断浏览器是否支持service worker
  if (navigator.serviceWorker) {
  // if('serviceWorker' in navigator)
    navigator.serviceWorker
      .register("./sw.js")
      .then(registration => {
        console.log(registration);
      })
      .catch(err => {
        console.log(err);
      });
  }
};
```
这里的sw.js文件就是我们实现数据的缓存与读取的地方. service worker中也存在生命周期的概念, 具体如下:
>1、install: 缓存数据
>2、activate: 清除旧的缓存
>3、fetch: 拦截页面所有请求，当有拦截到请求就会触发（核心），主要用于操作缓存或者读取网络资源

每个生命周期的具体用法如下:
#####  install阶段:
```javascript
self.addEventListener("install", async e => {
  cacheData(); //调用缓存方法
  await self.skipWaiting(); //跳过等待
  // e.waitUtil(self.skipWaiting()); //另一种跳过等待方式
});

//缓存方法
const CHACH_NAME = "cache_v2";
async function cacheData() {
  const cache = await caches.open(CHACH_NAME); //打开一个数据库
  //缓存清单
  const cacheList = [
    "/",
    "/images/logo.png",
    "/manifest.json",
    "/index.css",
    "/setting.js"
  ]; 
  await cache.addAll(cacheList); //缓存起来
}
// 特别指明就是‘/’, 这个对被解析成‘/index.html’
```
#####  activate阶段:
```javascript
const CHACH_NAME = "cache_v2";//在全局定义了当前数据库名
self.addEventListener("activate", async e => {
  /**查出数据库所有库名，清除旧版本库 */
  const keys = await caches.keys();
  keys.forEach(key => {
    //如果该数据库名不是当前定义的名字就进行删除
    if (key !== CHACH_NAME) {
      caches.delete(key);
    }
  });
  
  /**用于立刻获取页面控制权，确保用户第一次打开浏览器就是立马生效*/
  await self.clients.claim();
});
```

##### fetch阶段
fetch阶段就真正实现离线缓存的中心枢纽，它会拦截所有页面请求，因为这一特性，我们就能在无网络的情况下将用户需要请求的资源从缓存中读取出来返回给用户。根据网络以及缓存, 可以分为以下2种方式--网络优先、缓存优先.
###### 网络优先
```javascript
self.addEventListener("fetch", async e => {
  const req = e.request;//拿到请求头
  await e.respondWith(networkFirst(req));//将用户请求的资源响应给浏览器
});

//网络优先
async function networkFirst(req) {
  /**使用try.catch进行异常捕获*/
  try {
    const res = await fetch(req);
    cache.put(event.request, res.clone());
    return res;
  } catch (error) {
    const cache = await caches.open(CHACH_NAME); //打开一个数据库
    return await cache.match(req);//读取缓存
  }
}
```
需要注意的是, 在网络请求完之后, 应该把本地的缓存替换掉, 缓存最新数据, 如不替换将一直使用的是旧数据, 并且这里需要clone函数, 不然会遇到问题.

###### 缓存优先
```javascript
//缓存优先
async function cacheFirst(req) {
  const cache = await caches.open(CHACH_NAME); //打开一个数据库
  let res = await cache.match(req);//读取缓存
  if (res) {
    return res;
  } else {
    res = await fetch(req);
    return res;
  }
}

```

##### cache storage
上面的service worker中已经使用到了cache storage相关的api, 这里在做一下详细的叙述.

>1、caches.open(cacheName).then(function(cache) {})： 用于打开缓存，返回一个匹配cacheName的cache对象的promise，类似于连接数据库
>2、caches.keys() 返回一个promise对象，包括所有的缓存的key（数据库名）
>3、caches.delete(key) 根据key删除对应的缓存（数据库）
>4、cache.put(req, res) 把请求当成key，并且把对应的响应存储起来
>5、cache.add(url) 根据url发起请求，并且把响应结果存储起来
>6、cache.addAll(urls) 抓取一个url数组，并且把结果都存储起来
>7、cache.match(req) ： 获取req对应的response

一般service worker都是与cache storage搭配使用, 进而实现了离线缓存与访问.

### Notification-通知
在H5中出现了一个新的api, 可以进行通知用户相关信息- Notification Api, 具体使用方式如下:

```javascript
//先获取通知权限
if (Notification.permission == "default") {
  Notification.requestPermission();
}
if (!navigator.onLine) {
  new Notification("提示", { body: "您已断线,现在访问的是缓存内容" });
}
```
需要先获取用户权限,  只有用户点击了允许之后, 才能进行后续操作,具体结果如图;


### 实战
以上这些就是关于PWA的基本知识，但是在实际使用中，大多采用现有依赖或者模块来实现PWA。例如，我们团队所使用的是workbox模块，可以按照以下方式来集成实现PWA功能。

#### 安装

我们需要额外安装依赖：

+ workbox-cli：workbox的功能集成库；
+ register-service-worker：注册service-worker

```
yarn add workbox-cli register-service-worker -S
```



#### 配置PWA

需要分别在vue.config.js、workbox-config.js、main.ts、service-worker.js、public/manifest.json，public/service-worker.js、package.json中进行下面配置；

- vue.config.js中增加pwa配置

```
pwa: {
    name: 'App',
    themeColor: '#4DBA87',
    msTileColor: '#000000',
    appleMobileWebAppCapable: 'yes',
    appleMobileWebAppStatusBarStyle: 'black',

    // configure the workbox plugin
    workboxPluginMode: 'InjectManifest',
    workboxOptions: {
        // swSrc is required in InjectManifest mode.
        swSrc: 'src/sw.js',
        // ...other Workbox options...
    }
},
```



* workbox-config.js：与vue.config.js同级别文件

```
module.exports = {
  "globDirectory": "dist/",
  "globPatterns": [
    "**/*.{css,ico,png,html,js,json}"
  ],
  "swDest": "dist/sw.js",
  "swSrc": "src/sw.js"
};
```



* service-worker.js：与main.js同级别文件

```
importScripts('https://storage.googleapis.com/workbox-cdn/releases/3.6.1/workbox-sw.js');

workbox.loadModule('workbox-strategies');

const cacheVersion = '20230507'
const cdnCacheName = 'cdn' + cacheVersion
const apiCacheName = 'api' + cacheVersion
workbox.core.setCacheNameDetails({
	prefix: 'app',
	suffix: 'v1'
})

self.__precacheManifest = [].concat(self.__precacheManifest || []);

workbox.precaching.suppressWarnings();
workbox.precaching.precacheAndRoute(self.__precacheManifest, {});

// 强制安装新版本的SW
self.addEventListener('install', () => self.skipWaiting());
self.addEventListener('activate', () => self.clients.claim());

// CDN缓存
 workbox.routing.registerRoute(
     new RegExp('^(http|https)://s3plus', i),
     new workbox.strategies.cacheFirst({
     	cacheName: cdnCacheName
     })
 );
 
// api缓存
 workbox.routing.registerRoute(
    ({ url }) => url.origin === 'https://baidu.com',
     new workbox.strategies.NetworkFirst({
     	cacheName: apiCacheName
     })
 );


// 这个是必须有的：缓存打包文件中所有的静态资源，包括CSS、JS文件；
workbox.precaching.precacheAndRoute(self.__WB_MANIFEST);

workbox.precaching.precacheAndRoute([]);
```



* main.js(ts)：vue项目入口文件

```
import { register } from 'register-service-worker';

register(`./service-worker.js`, {
  ready() {},
  cached() {
    console.log('Content has been cached for offline use.');
  },
  updated() {
    console.log('New content is available; please refresh.');
  },
  offline() {
    console.log(
      'No internet connection found. App is running in offline mode.',
    );
  },
  error(error) {
    console.error('Error during service worker registration:', error);
  },
});
```



* public/manifest.json：public文件夹下，新增该json文件，需要注意有对应的icon

```
{
    "name": "App",
    "short_name": "App",
    "icons": [{
            "src": "/img/icons/android-chrome-192x192.png",
            "sizes": "192x192",
            "type": "image/png"
        },
        {
            "src": "/img/icons/android-chrome-512x512.png",
            "sizes": "512x512",
            "type": "image/png"
        }
    ],
    "start_url": "/",
    "display": "standalone",
    "background_color": "#fff",
    "theme_color": "#ffb95c",
    "gcm_sender_id": "my_gcm_sender_id",
    "gcm_user_visible_only": true
}
```



* package.json：在build中，改变打包方式

```
"build": "vue-cli-service build && workbox injectManifest",
```



* public/service-worker.js：public文件夹下，新增该js文件，之前的service-worker.js内容完全一致。



#### 注意

* workbox-config.js：注意打包路径以及service-worker.js在开发环境、生产环境的路径是否准确；
* service-worker.js：workbox存在版本问题，需要注意高版本对应的语法是否正确；
* 自定义缓存数据：
  * CDN缓存：尽量用正则来匹配；
  * API接口数据缓存：
    * 尽量用origin来匹配，尝试过采用正则来匹配，但是并未生效
    * 只能缓存GET请求的数据，对于POST请求的数据，需要额外处理，目前的想法是通过Axios来解决，但还未尝试
* 服务端设置：
  * 这里服务端会读取public中的service-worker.js，要求服务端设置读取JS文件的格式为text/javascript，否则会报MIME类型错误，导致PWA并未生效

### 总结

从自己在PC端的实战来看，PWA最有用的功能还是离线缓存，当无网络时，依旧可以看见网页，但前提是数据被缓存了。
