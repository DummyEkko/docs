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
  new PerformanceObserver((entryList) => {
    const [pageNav] = entryList.getEntriesByType('navigation');

    console.log(`TTFB: ${pageNav.responseStart}`);
  }).observe({
    type: 'resource',
    buffered: true
  });
  
  ```