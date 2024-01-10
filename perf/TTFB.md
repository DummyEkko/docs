# TTFB（Time to frist byte

加载第一个字节所需时间

## 什么是 TTFB？

  TTFB 是一个指标，用于测量资源请求与响应的第一个字节开始到达之间的时间。

  ![网络请求阶段及其关联时间的图表。TTFB 测量 startTime 和 responseStart 之间的间隔时间。](./assets/ttfb.png)

  TTFB 测量 startTime 和 responseStart 之间的间隔时间。


  TTFB 是以下请求阶段的总和：

  - 重定向时间
  - Service Worker 启动时间（如果适用）
  - DNS 查找
  - 连接和 TLS 协商
  - 请求，直到响应的第一个字节已到达

  缩短连接设置时间和后端的延迟时间有助于降低 TTFB。


  TTFB控制在 0.8 秒以内。


## js 获取

  ```JS
  import {onTTFB} from 'web-vitals';

  // Measure and log TTFB as soon as it's available.
  onTTFB(console.log);


  // 或者 
  new PerformanceObserver((entryList) => {
    const [pageNav] = entryList.getEntriesByType('navigation');

    console.log(`TTFB: ${pageNav.responseStart}`);
  }).observe({
    type: 'resource',
    buffered: true
  });
  ```

## 优化TTFB

- 使用内容分发网络 (CDN)

- 避免多次网页重定向

- 尽可能使用缓存的内容