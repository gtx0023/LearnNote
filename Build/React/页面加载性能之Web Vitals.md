# 页面加载性能之Web Vitals

新建React初始化项目的时候发现的代码
```javascript
//src/index.js
import reportWebVitals from './reportWebVitals';

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

//reportWebVitals
const reportWebVitals = onPerfEntry => {
  if (onPerfEntry && onPerfEntry instanceof Function) {
    import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS(onPerfEntry);
      getFID(onPerfEntry);
      getFCP(onPerfEntry);
      getLCP(onPerfEntry);
      getTTFB(onPerfEntry);
    });
  }
};

export default reportWebVitals;
```
## 百度之后发现

Web Vitals是Google的一项重大举措，旨在为质量信号提供统一的指导，这对于在Web上提供出色的用户体验来说很重要。

网站的开发者需要了解自己的网站给用户带来的体验，但不一定要成为性能优化的专家。Web Vitals旨在简化流程，并帮助网站开发者聚焦在核心性能指标上，也称为Core Web Vitals。

## Core Web Vitals
Core Web Vitals是Web Vitals的一个子集，适用于所有网页，应该被所有开发者去进行测量，也将在所有Google提供的工具中浮现。每一个Core Web Vitals都代表了用户体验独特的一面，可以用现场数据测试，能反映出以用户为核心的关键结果的真实体验。

构成Core Web Vitals的核心指标，将随着时间的推移而发展。当下2020年我们仅仅关注三个方面: 加载、可交互性和视觉稳定性。包含以下指标（以及各自的阈值）:

* Largest Contentful Paint (LCP): 衡量加载性能。为了提供一个好的用户体验，LCP应该在2.5秒内。
* First Input Delay (FID): 衡量可交互性。为了提供一个好的用户体验，FID应该在100毫秒内。
* Cumulative Layout Shift (CLS): 衡量视觉稳定性。为了提供一个好的用户体验，CLS应该小于0.1。
对上面每一个指标而言，为了保证覆盖到大部分用户，一般阈值设置在75%的页面加载达标即可，包括手机和pc站点。

JavaScript中测试Core Web Vitals
每一个Core Web Vitals都可用JS提供的Web API来测试。

最简单的方式，就是集成 Web Vitals 的js库，这是Google提供的一个小型可以生产环境使用的统计性能的库，涵盖了基本所有指标。

```javascript
import {getCLS, getFID, getLCP} from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify(metric);
  // Use `navigator.sendBeacon()` if available, falling back to `fetch()`.
  (navigator.sendBeacon && navigator.sendBeacon('/analytics', body)) ||
      fetch('/analytics', {body, method: 'POST', keepalive: true});
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getLCP(sendToAnalytics);
```

如果不需要发送统计，可以直接使用 Web Vitals扩展 ，这个扩展其实就是集成了上面提到的js库，实时的展现每个页面的指标值。

## Core Web Vitals实验数据测试工具
一般可以用开发者工具和Lighthouse，这两个都能测试FCP和CLS，但FID无法测试，可以用TBT替代。

## 其他Web Vitals
除了核心之外，还有其他类型的Web Vitals，当然这些一般都是核心的补充，为一些特定的场景提供服务。

例如，Time to First Byte (TTFB) 和 First Contentful Paint (FCP) 都是关于加载性能的，两者都有助于诊断LCP (缓慢的服务端响应，或者渲染阻塞的资源)。

同上，Total Blocking Time (TBT) 和 Time to Interactive (TTI) 则是影响FID的实验性指标，他们不属于核心，因为不能测试现场数据，不能反映用户为核心的关键结果。

不断发展的Web Vitals
Web Vitals 和 Core Web Vitals代表了当今开发人员用来衡量我们整个Web体验质量的最佳可用信号，但这些信号还不完善，未来会有更多改善和提升的点。

总结
Core Web Vitals是与用户为中心的理念密切相关的指标，一般不会怎么变化，相对稳定，一旦发生改变，或者阈值有了调整，影响很大，开发者需要对这种更新有预知或者年度的预测。

而其他的Web Vitals经常是辅助Core Web Vitals的，可能比其更具有实验性。因此，它们的定义和阈值的改动可能会很频繁

对所有Web Vitals来说，下面这个公开文档会实时更新:

http://bit.ly/chrome-speed-metrics-changelog

参考
https://web.dev/vitals/
