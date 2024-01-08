# First Input Delay (FID)

## 什么是 FID?

  FID 衡量的是从用户首次与网页互动（即，点击链接、点按按钮或使用由 JavaScript 提供支持的自定义控件）到浏览器能够实际开始处理事件处理脚本以响应该互动的时间。

  应努力将 First Input Delay 控制在 100 毫秒

  一般情况下，之所以发生输入延迟（也称为输入延迟），是因为浏览器的主线程正忙于执行其他操作，因此（尚无法）响应用户。发生这种情况的一种常见原因是，浏览器正忙于解析和执行应用加载的较大 JavaScript 文件。在这样做时，它无法运行任何事件监听器，因为正在加载的 JavaScript 可能会指示它执行其他操作。


## 在 JavaScript 中衡量 FID

  ```js
  new PerformanceObserver((entryList) => {
    for (const entry of entryList.getEntries()) {
      const delay = entry.processingStart - entry.startTime;
      console.log('FID candidate:', delay, entry);
    }
  }).observe({type: 'first-input', buffered: true});

  // 使用 web-vitals JavaScript 库来衡量 FID
  import {onFID} from 'web-vitals';

  // Measure and log FID as soon as it's available.
  onFID(console.log);
  ```

## 优化 First Input Delay

### 拆分长任务

- 将长时间运行的代码拆分为*较小的异步任务*会很有帮助
- 使用 Web Worker

  主线程阻塞是导致输入延迟的主要原因之一。Web 工作器可在后台线程上运行 JavaScript。将非界面操作移至单独的工作器线程可以缩短主线程阻塞时间，从而改善 FID。

- 推迟未使用的 JavaScript

  1. 通过代码将软件包拆分为多个区块
  2. 使用 async 或 defer 推迟所有非关键 JavaScript，包括第三方脚本

- 尽量减少未使用的 polyfill

  1. 如果您使用 Babel 作为转译器，请使用 @babel/preset-env，以便仅包含您计划定位的浏览器所需的 Polyfill。对于 Babel 7.9，请启用 bugfixes 选项，以进一步减少任何不需要的 Polyfill

  2. 使用 module/nomodule 模式提供两个单独的软件包（@babel/preset-env 也通过 target.esmodules 支持此功能）
   
    vite 的 legacy plugin就是采用这种特性
    ```js
    
    <script type="module" src="modern.js"></script>
     <script nomodule src="legacy.js" defer></script>
    ```