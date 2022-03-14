# 状态模式 State

对象的某些行为受状态的影响，通过对状态的管理来改变对象行为的模式就是状态模式。

## 常见的使用场景

- 编译器词法分析

- 流程审核

- 播放器

- 信号灯

## 代码示例

我们经常会遇到对同一个操作，因为当前所处的状态不一样，产生的行为也不一样。

比如，提交表单的时候：

```html
<button id="btn">提交</button>
<script>
  (function () {
    let isFetching = false;
    document.getElementById("btn").addEventListener(
      "click",
      function () {
        if (isFetching) {
          return false;
        }
        const $this = $(this);
        isFetching = true;
        $.ajax({
          url: "/someurl",
          complete: function () {
            isFetching = false;
            $this.text("提交");
          },
        });
        $this.text("提交中...");
      },
      false
    );
  })();
</script>
```

上面是一个比较常见的示例，提交按钮在点击后发起请求，请求过程中按钮被锁定防止重复提交，这时提交按钮就存在两种状态，锁定和未锁定，并会在两者中进行转换。

现在我们来看一个更常见点的场景示例：

```typescript
// 交通灯，不同的对象通行状态
abstract class TrafficLight {
  abstract carDo(): void;
  abstract humanDo(): void;
}
// 以下红绿灯状态均以行人视角
// 红灯
class RedTrafficLight extends TrafficLight {
  carDo() {
    console.log("车绿灯：汽车可以通过");
  }
  humanDo() {
    console.log("行人红灯：行人停止");
  }
}
// 绿灯
class GreenTrafficLight extends TrafficLight {
  carDo() {
    console.log("车红灯：汽车停止通过");
  }
  humanDo() {
    console.log("行人绿灯：行人可以通过");
  }
}
// 黄灯
class YellowTrafficLight extends TrafficLight {
  carDo() {
    console.log("黄灯：汽车停止通过");
  }
  humanDo() {
    console.log("黄灯：行人停止通过");
  }
}
// 交通运行类，同时也实现了信号灯抽象类
class Traffic implements TrafficLight {
  private light: TrafficLight = null;
  private timer: number;
  // 交通灯各种状态时间
  constructor(
    private greenTime: number,
    private yellowTime: number,
    private redTime: number
  ) {}
  // 车的准备状态
  carDo() {
    this.light.carDo();
  }
  // 行人的准备状态
  humanDo() {
    this.light.humanDo();
  }
  // 改变交通灯的状态
  changeLight(light: TrafficLight) {
    this.light = light;
  }
  // 使得交通
  run() {
    const { greenTime, yellowTime, redTime } = this;
    const green = new GreenTrafficLight();
    const yellow = new YellowTrafficLight();
    const red = new RedTrafficLight();
    const tasks = [
      [
        greenTime,
        () => {
          console.log('信号灯变成绿色');
          this.changeLight(green);
        },
      ],
      [
        yellowTime,
        () => {
          console.log('信号灯变成黄色');
          this.changeLight(yellow);
        },
      ],
      [
        redTime,
        () => {
          console.log('信号灯变成红色');
          this.changeLight(red);
        },
      ],
    ] as const;
    const total = tasks.length;
    const SECOND = 1000;
    let index = 0;
    const loop = () => {
      clearTimeout(this.timer);
      const [time, init] = tasks[index];
      // 初始化信号灯状态
      init();
      // 车通行状态
      this.carDo();
      // 人通行状态
      this.humanDo();
      // 设置定时器
      let leftTime = time;
      this.timer = setInterval(() => {
        leftTime -= 1;
        if (leftTime <= 0) {
          index++;
          if (index === total) index = 0;
          loop();
        } else {
          console.log(`>>>> 倒计时：还剩下${leftTime}秒`);
        }
      }, SECOND);
    };
    loop();
  }
  // 停止交通运行
  stop(){
    clearTimeout(this.timer);
  }
}
// 交通实例
const traffic = new Traffic(10, 3, 15);
// 运行
traffic.run();
// 运行50秒后停止交通运行
setTimeout(() => {
  traffic.stop();
}, 50000);
```
