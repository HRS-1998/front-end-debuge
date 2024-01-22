1. vue 源码调试事项

   - 在 vue-cli 创建的 vue 项目中 vue.config.js

   ```js
   configureWebpack(config) {
    config.devtool = "source-map";
   },
   ```

   - 下载 vue3 源码，执行 npm run build --sourcemap 将生成的 packages 下的 runtime-core 的 dist 复制到正常 vue3 项目中 nodemodules/@vue/runtime-core 中替换

   - 以上可以基本实现源码调试，但是路径可能不对，在 vue3 的源码的 rollup.config.js 中添加

   ```js
   output.sourcemapPathTransform = (relativeSourcePath, sourcemapPath) => {
     const newSourcePath = path.join(
       path.dirname(sourcemapPath),
       relativeSourcePath
     );
     return newSourcePath;
   };
   ```

   之后重新打包替换 runtime-core

2. 网页性能指标 web Vitals （这里可以看 studyRecord 中的 js/ObserveApi/performanceObserveApi）

==**『TTFB』**==  
Time To First Byte 首字节到达，从开始加载网页到接收到第一个字网页内容之间的耗时，用来衡量网页加载体验。

```js
//可以通过以下两种方式计算TTFB
//1.js
const { responseStart, requestStart } = performance.timing;
const TTFB = responseStart - requestStart;

//2.js
new PerformanceObserver((entryList) => {
  const entries = entryList.getEntries();
  for (const entry of entries) {
    if (entry.resposeStart > 0) {
      console.log(`TTFB:${entry.responseStar},${entry.name}`);
    }
  }
}).observe({
  type: "resource",
  buffered: true,
});
```

==**『TP』**==  
First Paint，首次绘制，第一个像素绘制到页面上的时间

```js
const paint = performance.getEntriesByType("paint");
const FP = paint[0].startTime;
```

==**『FCP』**==  
First Contentful Paint，首次内容绘制，从开始加载网页到第一个文本、图像、svg、非白色的 canvas 渲染完成之间的耗时。

```js
//1.js
const paint = performance.getEntriesByType("paint");
const FCP = paint[1].startTime;

//2.js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntriesByName("first-contentful-paint")) {
    console.log("FCP candidate:", entry.startTime, entry);
  }
}).observe({ type: "paint", buffered: true });
```

==**『LCP』**==  
Largest Contentful Paint，最大内容绘制,最大的内容（文字/图片）渲染的时间。

计算方式是从网页开始渲染到渲染完成，每次渲染内容都对比下大小，如果是更大的内容，就更新下 LCP 的值：

```js
let LCP = 0;

const performanceObserver = new PerformanceObserver((entryList, observer) => {
  const entries = entryList.getEntries();
  observer.disconnect();

  LCP = entries[entries.length - 1].startTime;
});

performanceObserver.observe({ entryTypes: ["largest-contentful-paint"] });
```

**==『FMP』==**
First Meaningful Paint，首次有意义的绘制。前面的 FCP、LCP 记录的是内容、最大内容的渲染，但这些内容并不一定关键，比如视频网站，渲染视频最关键，别的内容的渲染不是最重要的。FMP 就是记录关键内容渲染的时间。

```js
// 计算它的话我们可以手动给html元素加一个标识：
<video elementtiming="meaningful" />;

//js文件
let FMP = 0;
const performanceObserver = new PerformanceObserver((entryList, observer) => {
  const entries = entryList.getEntries();
  observer.disconnect();

  FMP = entries[0].startTime;
});
performanceObserver.observe({ entryTypes: ["element"] });
```

**==『DCL』==**
DomContentloaded，html 文档被完全加载和解析完之后，DOMContentLoaded 事件被触发，无需等待 stylesheet、img 和 iframe 的加载完成。

```js
const { domContentLoadedEventEnd, fetchStart } = performance.timing;
const DCL = domContentLoadedEventEnd - fetchStart;
```

**==『L』==**
Load， html 加载和解析完，它依赖的资源（iframe、img、stylesheet）也加载完触发

```js
const { loadEventEnd, fetchStart } = performance.timing;
const L = loadEventEnd - fetchStart;
```

**==『TTI』==**
Time to Interactive，可交互时间。
计算方式为：
从 FCP 后开始计算;

持续 5 秒内无长任务（大于 50ms） 且无两个以上正在进行中的 GET 请求;

往前回溯至 5 秒前的最后一个长任务结束的时间，没有长任务的话就是 FCP 的时间。

```js
const { domInteractive, fetchStart } = performance.timing;
const TTI = domInteractive - fetchStart;
```

**==『FID』==**
First Input Delay，用户第一次与网页交互（点击按钮、点击链接、输入文字等）到网页响应事件的时间。

```js
let FID = 0;

const performanceObserver = new PerformanceObserver((entryList, observer) => {
  const entries = entryList.getEntries();
  observer.disconnect();

  FID = entries[0].processingStart - entries[0].startTime;
});

performanceObserver.observe({ type: ["first-input"], buffered: true });
```

**==『TBT』==**
Total Blocking Time，记录在首次内容渲染（FCP）到可以处理交互（TTI）之间所有长任务（超过 50ms 的 longtask）的阻塞时间总和

**==『CLS』==**
Cumulative Layout Shift，累计布局偏移，记录了页面上非预期的位移波动。计算方式为：位移影响的面积 \* 位移距离

**==『SL』==**
Speed Index，速度指数，页面可见部分的显示速度, 单位是时间，这个指标也有快、适度、慢的区间，用性能测试工具测量时会转为相应的分数

基于以上，谷歌选出了 3 个核心指标，分别是 ==LCP、FID、CLS==
这三个核心指标分别代表了加载性能、交互性能、视觉稳定性。


3. 内存泄漏
**==『console.log』==**

**==『定时器』==**

**==『dom移除』==**

**==『闭包引用变量』==**

**==『全局变量』==**

