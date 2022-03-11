# 观察者模式 Observer

如果对象在触发某些事件行为的时候，通过发布-订阅机制，使得多个其它对象能“观察”到该事件行为的模式，被成为观察者模式。

## 常见的使用场景

- RSS 订阅/邮件订阅

- MutationObserver: 监听 DOM 树更改

- DOM 事件

- 异步回调

## 代码示例

我们可能碰到过这样的场景，一个插件

```typescript
class Product{

}

class Market {
  private onready: boolean = false;
}
```
