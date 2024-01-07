# Cumulative Layout Shift (CLS)


## CLS 是什么？

  衡量的是视觉稳定性。为了提供良好的用户体验，页面应将 CLS 保持在 0.1. 或更低。
  CLS 是测量整个页面生命周期（页面可见性变成隐藏）内发生的所有 意外布局偏移 中最大一的 布局偏移分数。；每当一个已渲染的可见元素的位置从一个可见位置变更到下一个可见位置时，就发生了 布局偏移 。


  <!-- <video src="https://web.dev/articles/cls/video/web-dev-assets/layout-instability-api/layout-instability-poster.png?hl=zh-cn"> -->

  <video autoplay="" controls="" height="510" loop="" muted="" poster="https://web.dev/articles/cls/video/web-dev-assets/layout-instability-api/layout-instability-poster.png?hl=zh-cn" width="658">
    <source src="https://web.dev/static/articles/cls/video/web-dev-assets/layout-instability-api/layout-instability2.webm?hl=zh-cn" type="video/webm; codecs=vp8">
    <source src="https://web.dev/static/articles/cls/video/web-dev-assets/layout-instability-api/layout-instability2.mp4?hl=zh-cn" type="video/mp4; codecs=h264">
  </video>


## 在js中 测量 CLS

  ```js
  
  new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('Layout shift:', entry);
  }
  }).observe({type: 'layout-shift', buffered: true});


// 使用 web-vitals JavaScript 库来衡量 CLS
  import {onCLS} from 'web-vitals';

    // Measure and log CLS in all situations
    // where it needs to be reported.
    onCLS(console.log);
  ```


## 如何改善 CLS

- 借助 CSS transform 属性，您可以为元素添加动画效果，而不触发布局偏移：
  请使用 transform: scale()，而不是更改 height 和 width 属性。
  如需移动元素，请避免更改 top、right、bottom 或 left 属性，并改用 transform: translate()。

- 确定图片的尺寸，iframe尺寸，以及动态插入内容的尺寸

- 使用 `<link rel=preload>` 尽早加载关键网页字体。预加载字体有更高的机会实现首次渲染，在这种情况下，不会发生布局偏移