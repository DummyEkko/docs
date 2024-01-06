# Largest Contentful Paint (LCP)

## 什么是 LCP

  > 视口内可见的最大图片或文本块的呈现时间（相对于用户首次导航到页面的时间）。

  > 良好的用户体验 `lcp` 应该控制在 *2.5* 秒以内


  LCP 会考虑的元素包括：

  - `<img>` 元素
  - `<svg>` 元素内的 `<image>` 元素
  - 包含海报图片的 `<video>` 元素（使用海报图片加载时间）
  - 使用通过 url() 函数（而不是 CSS 渐变）加载背景图片的元素
  - 块级元素，包含文本节点或其他内嵌级文本元素子项。
  - 为自动播放 `<video>` 元素而绘制的第一帧
  - 动画图片格式（例如 GIF 动画）的第一帧

## 与FCP 的区别

    FCP 用于衡量何时将任何内容绘制到屏幕上；当绘制主要内容时，LCP 会测量，以便 LCP 具有更强的选择性。


## 在js中 测量 LCP

    ```js
    new PerformanceObserver((entryList) => {
        for (const entry of entryList.getEntries()) {
            console.log('LCP candidate:', entry.startTime, entry);
        }
    }).observe({type: 'largest-contentful-paint', buffered: true});
    ```

    或者使用： web-vitals 库

    ```JS
        import {onLCP} from 'web-vitals';

        // Measure and log LCP as soon as it's available.
        onLCP(console.log);
    ```

## 如何提高LCP
