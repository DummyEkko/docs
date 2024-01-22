- [performance](#performance)
  - [PerformanceTiming](#performancetiming)
  - [PerformanceNavigationTiming](#performancenavigationtiming)
  - [优化前](#优化前)
  - [优化后](#优化后)
  - [优化点](#优化点)
  - [字体最佳实践](#字体最佳实践)
    - [@font-face](#font-face)
    - [让浏览器更早发现字体](#让浏览器更早发现字体)
      - [内嵌字体声明](#内嵌字体声明)
      - [使用 preload 加载字体](#使用-preload-加载字体)
    - [字体渲染](#字体渲染)

# performance

W3C标准化 引入了 `Navigation Timing API` ，实现了自动、精准的页面性能打点；开发者可以通过 `window.performance` 属性获取。


## PerformanceTiming

   Level 1 规范

   > 已弃用: 不再推荐使用该特性。虽然一些浏览器仍然支持它，但也许已从相关的 web 标准中移除，也许正准备移除或出于兼容性而保留。请尽量不要使用该特性，并更新现有的代码；该特性随时可能无法正常工作。


## PerformanceNavigationTiming

  Level2 而提供了更为精准、更为全面的新的性能统计API-PerformanceNavigationTiming。

  ![img](./assets/ttfb.png)

  ![img](./assets/time.png)


  通过 PerformanceObserver监听资源

  ```JS
  // ttfb
  new PerformanceObserver((entryList) => {
    const [pageNav] = entryList.getEntriesByType('navigation');

    console.log(`TTFB: ${pageNav.responseStart}`);
  }).observe({
    type: 'resource',
    buffered: true
  });
  
  ```

## 优化前

lighthouse 



![lighthouse](./assets/before.png)

![性能](./assets/性能1.png)


## 优化后


![lighthouse](./assets/after.png)

![性能](./assets/WX20240111-112009%402x.png)


## 优化点

1. 优化图片

  - 选择合适尺寸

    例如 100x100 (CSS)像素显示的照片

    | 屏幕分辨率(dpr) | 总像素数 | 未压缩的文件大小（每个像素 4 字节） |
    | :-----| ----: | :----: |
    | 1 倍  | 100 x 100 = 10000 | 40000 字节 |
    | 2 倍 | 100 x 100 x 4 = 40,000	 | 160000 字节 |
    | 3 倍 | 100 x 100 x 9 = 90000	 | 360000 字节 |

    提供自适应图片：

    1. 通过 img srcset 和 sizes
    2. picture source

  - 转换为 WebP 格式

    如果支持webp则：

    WebP 图片比对应的 JPEG 和 PNG 图片小，文件大小通常可缩减 25-35%。这会缩减页面大小并提升性能。

    判断支持的方法：

    1. `<picture> <source type="image/webp" srcset="flower.webp"> ...`
    2. canvas

  - 图片 CDN 优化

  - LCP 图片元素使用 fetchpriority 属性值 "high"，以便浏览器可以尽快开始加载该图片
  
  - 如果图片无法在初始 HTML 中立即发现，请考虑为 LCP 候选图片使用 rel=preload 提示，以便浏览器提前加载该图片。

  - 延迟加载图片 

    - loading lazy
    - Intersection Observer

  - 尽早建立网络连接

2. 尽早建立网络连接

   浏览器必须先建立连接，然后才能从服务器请求资源。建立安全连接包括以下三个步骤：

   1. 查找域名并将其解析为 IP 地址。

   2. 设置与服务器的连接。

   3. 对连接进行加密以确保安全。

   在每一个步骤中，浏览器都会向服务器发送一段数据，然后服务器会发回响应。从出发地到目的地再往回的旅程称为“往返”。

   根据网络状况，单次往返可能会花费大量时间。连接设置过程可能涉及最多三次往返，如果未优化，则更多次。

  - rel=preconnect (DNS、TCP 握手、TLS 协商等)

  - rel=dns-prefetch 

3. css

  - 压缩
  - 推迟非关键的

4. 高效加载第三方 JavaScript

  - 在 `<script>` 标记中使用 async 或 defer 属性
  - 尽早建立与所需来源的连接
  - 延迟加载

## 字体最佳实践

> 如果网页字体尚未加载，浏览器通常会延迟文本呈现

### @font-face

声明将用于引用字体的名称，并指示相应字体文件的位置。

例如：

```css
@font-face {
  font-family: "Open Sans";
  src: url("/fonts/OpenSans-Regular-webfont.woff2") format("woff2");
}


h1 {
  /* 这个时候才会 下载字体 */
  font-family: "Open Sans" 
}
```

`@font-face` 声明本身不会触发字体下载。而是仅在通过网页上的样式引用该字体时才会下载。

### 让浏览器更早发现字体
#### 内嵌字体声明

主文档的 `<head>` 中内嵌字体声明和其他关键样式, 浏览器能更快发现 字体声明 并且下载

#### 使用 preload 加载字体

在使用外部样式表时，预加载最重要的字体可能非常有效，因为浏览器要过一段时间才能发现是否需要字体。


### 字体渲染

默认情况下，如果关联的网页字体尚未加载，基于 Chromium 和 Firefox 的浏览器将会阻止文本呈现，最长可持续 3 秒钟；Safari 将无限期地阻止文本呈现。

可以使用 `font-display` 属性来配置此行为。此选择可能会产生重大影响：font-display 可能会影响 LCP、FCP 和布局稳定性。


font-display 可选值

 | 值    | 屏蔽期 | 交换期） |
 | :-----| ----: | :----: |
 | auto  | 因浏览器而异 | 因浏览器而异 |
 | block | 2-3 秒	 | 无限 |
 | swap | 0 毫秒	 | 无限  |
 | fallback | 100 毫秒	 | 3 秒  |
 | optional | 100 毫秒	 | 无  |


 - 屏蔽期：屏蔽期从浏览器请求网页字体时开始计算。在屏蔽期间，如果网页字体不可用，相应字体将以不可见的后备字体呈现，因此用户将看不到相应文本。如果在屏蔽期结束后该字体不可用，它将以后备字体呈现。
 - 交换期：交换期在屏蔽期之后。如果网页字体在交换期内可用，则会被“交换”。


`font-display` 策略反映了关于性能与美感之间权衡的不同观点。因此，我们很难推荐采用合适的方法，因为推荐方法取决于个人的偏好、网页字体对页面和品牌的重要性，以及较晚的字体在换入新字体时会多么不协调。

- *如果性能是我们的头等大事*: 请使用 font-display: optional。这是最“高效”的方法：文本渲染的延迟不超过 100 毫秒，并且能够保证不会发生与字体交换相关的布局偏移。不过，这样做的缺点是，如果网页字体延迟显示，将无法使用网页字体。
- *如果快速显示文本是首要任务，但您仍想确保使用网页字体*：请使用 font-display: swap，但务必尽早提供字体，以免导致布局偏移。这种方案的缺点是，字体延迟出现会导致偏移。
- *如果确保文本以网页字体显示是首要任务*：请使用 font-display: block，但务必尽早提供字体，以便最大限度地缩短文本延迟。


[参考](https://web.dev/articles/reduce-webfont-size?hl=zh-cn)