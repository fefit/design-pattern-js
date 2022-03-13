# 访问者模式 Visitor

## 代码示例

设想一个炒菜的厨师，需要为不同类型的顾客提供不同口味的食物，如果这个工作全部由厨师来做的话，他就得需要知道不同顾客都需要哪些口味，而这些显然不是厨师的主要工作。显然，饭店里也是这么做的，服务员会根据顾客的口味，将要求写在菜品订单上。这样，厨师就只用专注于炒菜时做哪种口味就好了。服务员充当了顾客到厨师的传递层，将两者隔离开来。

```typescript
// 厨师类
class Cook {
  // 接收服务员的任务
  receive(w: Waiter) {
    w.printMenu();
    this.cook();
  }
  // 炒菜
  cook() {
    console.log("按照单子炒菜");
  }
}
// 服务员
// 充当访问者
class Waiter {
  // 所有接待的顾客列表
  customers: Customer[] = [];
  // 知晓顾客是湖南人
  knowUrHunanren(c: HunanCustomer) {
    console.log(`${c.from}人：加辣`);
  }
  // 知晓顾客是四川人
  knowUrSichuanren(c: SichuanCustomer) {
    console.log(`${c.from}人：麻辣`);
  }
  // 知晓顾客是广东人
  knowUrGuangdongren(c: GuangdongCustomer) {
    console.log(`${c.from}人：清淡`);
  }
  // 接收顾客
  receive(...customers: Customer[]) {
    this.customers.push(...customers);
  }
  // 打印菜单
  printMenu() {
    this.customers.map((customer) => {
      customer.require(this);
    });
    this.customers = [];
  }
}
// 顾客，抽象类
// 都有从哪来的信息及与服务员说出自己需求的功能
abstract class Customer {
  abstract readonly from: string;
  abstract require(w: Waiter): void;
}

// 湖南顾客
class HunanCustomer implements Customer {
  readonly from = "湖南";
  require(w: Waiter) {
    w.knowUrHunanren(this);
  }
}

// 四川顾客
class SichuanCustomer implements Customer {
  readonly from = "四川";
  require(w: Waiter) {
    w.knowUrSichuanren(this);
  }
}

// 广东顾客
class GuangdongCustomer implements Customer {
  readonly from = "广东";
  require(w: Waiter) {
    w.knowUrGuangdongren(this);
  }
}
// 增加服务员
const waiter = new Waiter();
// 增加厨师
const cook = new Cook();
// 接待顾客
waiter.receive(new GuangdongCustomer());
waiter.receive(new HunanCustomer());
waiter.receive(new SichuanCustomer());
// 厨师从服务员那拿订单
cook.receive(waiter);
```
