# 策略模式 Strategy

策略模式是这样一种行为模式，针对同一个问题，有多种可行方案，我们将各种方案的实现策略封装在自己的类里，同时暴露出相同的接口方法给外部使用，这就是策略模式。

## 常见使用场景

- 线路规划

- 

## 示例代码

```typescript
// 策略抽象类
abstract class Strategy {
  static type: string;
  abstract take(distance: number): number;
}
// 步行策略
class WalkStrategy extends Strategy {
  static type = "walk";
  take(distance: number): number {
    return distance / 1.2 / 60;
  }
}
// 驾车策略
class CarStrategy extends Strategy {
  static type = "car";
  take(distance: number): number {
    return distance / 20 / 60;
  }
}
// 骑行策略
class BikeStrategy extends Strategy {
  static type = "bike";
  take(distance: number): number {
    return distance / 4 / 60;
  }
}
// 上下文，策略的执行者
class Context {
  private strategy: Strategy = null;
  setStrategy(strategy: Strategy) {
    this.strategy = strategy;
  }
  executeStrategy(distance: number): number {
    console.log(`${this.strategy.constructor.name}: ${distance}米`);
    return this.strategy.take(distance);
  }
}

type CombineStrategy = Array<[number, { new (): Strategy } & typeof Strategy]>;
// 路线
class Route {
  private strategyInstances: { [index: string]: Strategy } = {};
  private strategies: Array<CombineStrategy> = [];
  private context = new Context();
  constructor(private readonly from: string, private readonly to: string) {}
  setCombineStrategy(...args: CombineStrategy) {
    this.strategies.push(args);
  }
  calcTakeTime() {
    const { context, strategyInstances } = this;
    let loop = 0;
    for (const combine of this.strategies) {
      let time = 0;
      for (const [distance, strategy] of combine) {
        const { type } = strategy;
        const instance =
          strategyInstances[type] || (strategyInstances[type] = new strategy());
        context.setStrategy(instance);
        time += context.executeStrategy(distance);
      }
      console.log(`方案${++loop}：共计耗时${time / 60}分钟`);
    }
  }
}

const route = new Route("酒仙桥", "草场地");
route.setCombineStrategy([5000, CarStrategy], [300, BikeStrategy]);
route.setCombineStrategy([4000, BikeStrategy]);
route.setCombineStrategy([3500, BikeStrategy], [500, WalkStrategy]);
route.calcTakeTime();
```
