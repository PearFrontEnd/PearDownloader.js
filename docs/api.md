# PearDownloader 文档

## 导入
PearDownloader有两种导入方式：通过script标签导入和npm安装

### script标签导入
首先通过script标签导入pear-downloader.min.js：
```html
<script src="./dist/pear-downloader.min.js"></script>
```
或者使用CDN：
```html
<script src="https://cdn.jsdelivr.net/npm/peardownloader@latest"></script>
```

### npm安装
在项目目录中通过npm安装PearDownloader：
```bash
npm install peardownloader --save
```
然后就可以用require方式引入PearDownloader：
```js
var PearDownloader = require('PearDownloader');
```

# PearDownloader API

## PearDownloader.isWebRTCSupported()

静态方法，检测当前浏览器环境是否支持WebRTC。
```js
if (PearDownloader.isWebRTCSupported()) {
  // WebRTC is supported
  
} else {
  // Use a fallback
  
}
```

## `var downloader = new PearDownloader(url, token, opts)`

创建一个新的PearDownloader实例，url是待下载文件的url，token是登陆pear服务器获取的授权，有效期7个小时。

其中`opts`可以指定PearDownloader的具体配置，相关字段的说明如下：

```js
{
  scheduler: 'WebRTCFirst',                         //节点调度算法，默认IdleFirst，其它内置调度算法有“WebRTCFirst“和”CloudFirst”
  auto: true,                                       //是否全部下载,默认true
  interval: 5000,                                   //滑动窗口的时间间隔,单位毫秒,默认10s
  useDataChannel: true,                             //是否开启data channel,默认true
  dataChannels: 20,                                 //创建data channel的最大数量,默认20
  useTorrent: false,                                //是否开启Browser P2P(基于Webtorrent)，默认true
  magnetURI: magnetURI,                             //可手动传入magnetURI，需先将useTorrent设为true
  trackers:["wss://tracker.openwebtorrent.com"],    //可手动传入tracker服务器，需先将useTorrent设为true
  sources: ['http://example.com/a'],                //指定下载源，增加这个字段后PearDownloader不会再向后台请求节点，建议下载源多于5个以保证流畅播放
  useMonitor: true,                                 //是否开启monitor,会稍微影响性能,默认false
  debug: false,                                      //是否开启debug模式，开启后可以在console中查看log，默认true
  algorithm: 'push'                                 //下载算法，有‘push’和‘pull’两种，默认‘push’，push是无序下载但下载速度相比pull更快
}
```

## `downloader.on('fallback', function () {})`

downloader出现异常时的回调函数，建议在此处作降级处理，如直接调用默认下载器。

## `downloader.on('begin', function (fileLength, chunks) {})`

当PearDownloader完成初始化后会触发该事件，通过回调函数中的fileLength获取文件总大小，chunks获取文件被分成的块数（每块1M）。

## `downloader.on('done', function () {})`

当PearDownloader完成下载会触发该事件。

## `downloader.on('progress', function (downloaded) {})`

通过该事件可以监听PearDownloader的下载进度（下载的字节数除以总的字节数）。(useMonitor需设为true)

## `downloader.on('cloudspeed', function (speed) {})`

通过该事件可以监听cloud（即server节点）的平均下载速度（单位KB/s）。(useMonitor需设为true)

## `downloader.on('fogspeed', function (speed) {})`

通过该事件可以监听fog节点（包括WebRTC和HTTP）的平均下载速度（单位KB/s）。(useMonitor需设为true)

## `downloader.on('fogratio', function (p2pRatio) {})`

通过该事件可以监听fog节点（包括WebRTC和HTTP）总的下载比率（fog下载的字节数除以目前总的下载字节数）。(useMonitor需设为true)

## `downloader.on('sourcemap', function (sourceType, index) {})`

每下载一个buffer都会触发该事件，sourceType是一个string，代表该buffer是从哪个源下载的，有以下几种取值：(useMonitor需设为true)<br/>
* null: 该处的buffer还未下载<br/>
* s: server，从服务器端下载（HTTP协议）<br/>
* n: node，从节点下载（HTTP协议）<br/>
* d: data channel，从节点下载（WebRTC协议）<br/>
* b: browser，从其它浏览器下载（WebRTC协议）<br/>

index是对应的索引。

## `downloader.on('traffic', function (mac, size, type) {})`
通过该事件可以监听每个节点的实时流量，其中mac是节点的mac地址，size是对应节点的瞬时下载流量（字节），type是
节点的类型（http、datachannel等）。(useMonitor需设为true)

请参考[`../examples/downloader-test.html`](/examples/downloader-test.html)来了解API使用方法。
