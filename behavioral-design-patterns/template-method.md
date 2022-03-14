# 模板方法模式 Template Method

模板方法模式是这样一种模式，当子类中有重复的逻辑时，我们把相同逻辑的部分提取成通用方法在父类中实现（也即模板方法），而在子类中实现差异的逻辑部分。

## 代码示例

```typescript
type ResponseData<T> = {
  errno: number;
  errmsg?: string;
  data?: T;
};
// 定义一个通用模板类
abstract class Template<
  T = unkown,
  U extends ResponseData<T> = ResponseData<T>
> {
  constructor(public readonly ele: Element) {}
  // 抽象方法，所有子类必须实现
  abstract fetchData(): Promise<U>;
  abstract renderHtml(data: T | undefined): string;
  // 通用方法，设置html
  setHtml(html: string) {
    // this.ele.innerHTML = html;
    console.log(html);
  }
  // 设置加载状态
  loading() {
    this.setHtml("数据请求中...");
  }
  // 显示错误信息
  error(e: Error) {
    this.setHtml(`渲染失败，错误：${e.message}`);
  }
  // render 模板方法
  async render() {
    try {
      this.loading();
      const res = await this.fetchData();
      if (res?.errno === 0) {
        const html = this.renderHtml(res?.data);
        this.setHtml(html);
      } else {
        this.error(new Error(res?.errmsg));
      }
    } catch (e) {
      this.error(e as unkown as Error);
    }
  }
}
// 数据实现逻辑
const sleepWith = (time: number, rejectReason: string = ""): Promise<void> => {
  return new Promise((resolve, reject) => {
    setTimeout(
      () => (rejectReason ? reject(new Error(rejectReason)) : resolve()),
      time
    );
  });
};
// 子类: 菜单区域模板
type MenuData = Array<{ name: string; title: string }>;

class MenuTemplate extends Template<MenuData, ResponseData<MenuData>> {
  // 实现数据获取逻辑
  fetchData(): Promise<ResponseData<MenuData>> {
    return sleepWith(1000).then(() => {
      return {
        errno: 0,
        data: [
          {
            name: "",
            title: "",
          },
        ],
      };
    });
  }
  // 实现生成html逻辑
  renderHtml(data: MenuData): string {
    return data.reduce((html, item) => {
      html += `<span>${item.name}</span><span>${item.title}</span>`;
      return html;
    }, "");
  }
}

// 子类：主区域模板
type BodyData = Array<{
  block: string;
  title: string;
  content: string;
}>;

class BodyTemplate extends Template<BodyData> {
  // 实现数据获取逻辑
  fetchData(): Promise<ResponseData<BodyData>> {
    return sleepWith(1000).then(() => {
      return {
        errno: 1,
        errmsg: "图片列表请求失败",
      };
    });
  }
  // 实现生成html逻辑
  renderHtml(data: BodyData): string {
    return data.reduce((html, item) => {
      html += `<div class="${item.block}"><h3>${item.title}</h3><div class="content">${item.content}</div></div>`;
      return html;
    }, "");
  }
}

const menu = new MenuTemplate(document.getElementById("menu"));
const body = new BodyTemplate(document.getElementById("body"));
(async () => {
  await Promise.all([menu.render(), body.render()]);
})();
```
