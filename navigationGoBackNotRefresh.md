关于导航(Navigation)回退后不刷新页面
==========================

上下文：
```json
{
  "react": "16.0.0-alpha.12",
  "react-native": "0.48.3",
  "react-navigation": "^1.0.0-beta.11"
}
```

问题：

```
  ViewA -> ViewB  页面A跳转到页面B
  ViewB -> ViewA  页面B返回(goBack)页面A
  发现页面A没有刷新

```

需求：需要刷新

分析：目前尚无最优的解决方案，官方也正在更新中，有几种方案。
- 1.使用redux，只要改变store就会触发state更新。
- 2.路由跳着传参数，间接出发state更新（我未尝试）。
- 3.写个事件订阅器，我比较建议的方式。

解决：自己写一个订阅器，就是实现  `Pub/Sub 模式`，在`ViewA`订阅事件， 在`ViewB`生命周期的`componentWillUnmount`中发布事件。

PubSub.js
```js
var events = {};

export default {
  subscribe(event, listener) {
    if (!events.hasOwnProperty(event)) {
      events[event] = [];
    }
    events[event].push(listener);
    return this;
  },
  remove(event, listener) {
    if (!events.hasOwnProperty(event)) {
      return;
    }
    let index = events[event].findIndex(_ => _ === listener);
    if (index >= 0) {
      events[event].splice(index, 1);
    }
    return this;
  },
  publish(event, args) {
    if (!events.hasOwnProperty(event)) {
      return;
    }
    events[event].forEach( _ => {
      _(args);
    })
  } 
};
```

ViewA.js
```js
  import Pubsub from './Pubsub.js';
  class ... {
    componentWillUnmount() {
      Pubsub.remove('refresh', this._refresh);
    }
    
    componentDidMount() {
      this._refresh();
      Pubsub.subscribe('refresh', this._refresh);
    }
  }
```

ViewB.js 
```js
  import Pubsub from './Pubsub.js';
  class ... {
     componentWillUnmount() {
      Pubsub.publish('refresh')
    }
  }

```
