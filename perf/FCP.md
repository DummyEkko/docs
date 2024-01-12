# First Contentful Paint (FCP)

## 什么是 FCP？

  首次内容绘制 (FCP) 指标用于测量从用户首次导航到页面的那部分时间，到页面内容的任何部分呈现在屏幕上的时间。
  对于此指标，“内容”是指文本、图片（包括背景图片）、`<svg>` 元素或非白色 `<canvas>` 元素。

  *要点*：请务必注意，FCP 包含上一个页面的所有卸载时间、连接设置时间、重定向时间以及首字节时间 (TTFB)

  ![img](https://web.dev/static/articles/fcp/image/fcp-timeline-googlecom_856.png?hl=zh-cn)

  FCP 发生在第二帧中，因为此时第一个文本和图片元素渲染到屏幕上。

  为了提供良好的用户体验，网站应努力将 First Contentful Paint 控制在 1.8 秒以内

## 在js中 测量 FCP

  ```JS
    new PerformanceObserver((entryList) => {
    for (const entry of entryList.getEntriesByName('first-contentful-paint')) {
        console.log('FCP candidate:', entry.startTime, entry);
    }
    }).observe({type: 'paint', buffered: true});


    import {onFCP} from 'web-vitals';

    // Measure and log FCP as soon as it's available.
    onFCP(console.log);
  ```


## 如何改进 FCP

- 消除阻塞渲染的资源
- 缩减 CSS 大小
  - 移除未使用的 CSS
- 移除未使用的 JavaScript
- 预先连接到所需的源站 
  -  rel=preconnect
  -  rel=dns-prefetch
- 缩短服务器响应时间 (TTFB)
  - 避免多次网页重定向
  - 采用高效的缓存政策提供静态资源
- 避免网络负载庞大
- 避免 DOM 规模过大
- 最大限度地降低关键请求深度
- 确保文本在网页字体加载期间保持可见
- 尽量减少请求数和传输大小