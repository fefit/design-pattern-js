# 观察者模式 Observer

如果对象在触发某些事件行为的时候，通过发布-订阅机制，使得多个其它对象能“观察”到该事件行为的模式，被成为观察者模式。

## 常见的使用场景

- RSS 订阅/邮件订阅

- MutationObserver: 监听 DOM 树更改

- DOM 事件

- 异步回调

## 代码示例

```typescript
// 发布订阅行为
type Action = "add" | "modify" | "delete";
// 订阅者接收到的数据格式
type Data = Array<{
  url: string;
  title: string;
}>;
// 综合订阅者行为和数据的变化数据
interface Change {
  action: Action;
  data: Data;
}
// 观察者抽象类
// 所有观察者都有通用方法`receive`用来接收消息
abstract class User {
  static buildInfo(change: Change): string {
    return change.action + '文章：' + change.data.reduce((ret, item) => {
      ret += `<p><a href="${item.url}">${item.title}</a></p>`;
      return ret;
    }, "");
  }
  abstract readonly id: number;
  abstract receive(change: Change): void;
}
// 带邮箱信息的观察者
class EmailUser implements User {
  constructor(public readonly id: number, private readonly email: string) {}
  private sendMail(content: string) {
    console.log(`发送邮件到${this.email}:${content}`);
  }
  receive(change: Change) {
    this.sendMail(User.buildInfo(change));
  }
}
// 仅带站点ID的观察者
class SiteUser implements User {
  constructor(public readonly id: number) {}
  private sendSiteInfo(content: string) {
    console.log(`发送站内信到${this.id}:${content}`);
  }
  receive(change: Change) {
    this.sendSiteInfo(User.buildInfo(change));
  }
}
// 类似工厂模式，针对有email的用户创建两个用户对象
class UserFactory {
  private users: User[] = [];
  constructor(public readonly id: number, public readonly email?: string) {
    if (email) {
      this.users.push(new EmailUser(id, email));
    }
    this.users.push(new SiteUser(id));
  }
  listen(manager: ActionManager, action: Action) {
    for (const user of this.users) {
      manager.subscribe(action, user);
    }
  }
  unlisten(manager: ActionManager, action: Action) {
    for (const user of this.users) {
      manager.unsubscribe(action, user);
    }
  }
}
// 行为管理者
// 发布订阅行为的实际管理者
class ActionManager {
  private listeners: { [index: string]: User[] } = {};
  // 订阅
  subscribe(action: Action, listener: User) {
    this.listeners[action] = (this.listeners[action] || []).concat(listener);
  }
  // 取消订阅
  unsubscribe(action: Action, listener: User) {
    const users = this.listeners[action] || [];
    const index = users.findIndex((cur: User) => cur.id === listener.id);
    if (index > -1) {
      users.splice(index, 1);
    }
  }
  // 发布消息
  notify(action: Action, data: Data) {
    const users = this.listeners[action] || [];
    const change: Change = { action, data };
    for (const user of users) {
      user.receive(change);
    }
  }
}
// 用户订阅管理
class Feed {
  private readonly manager: ActionManager;
  constructor() {
    this.manager = new ActionManager();
  }
  addUser(action: Action, ...users: UserFactory[]) {
    for (const user of users) {
      user.listen(this.manager, action);
    }
  }
  removeUser(action: Action, ...users: UserFactory[]) {
    for (const user of users) {
      user.unlisten(this.manager, action);
    }
  }
  add(data: Data) {
    this.manager.notify("add", data);
  }
  modify(data: Data) {
    this.manager.notify("modify", data);
  }
  delete(data: Data) {
    this.manager.notify("delete", data);
  }
}

const feed = new Feed();
const user1 = new UserFactory(1, "jxz_211@163.com");
const user2 = new UserFactory(2);
feed.addUser("add", user1, user2);
feed.addUser("modify", user1);
feed.addUser("delete", user2);
feed.add([
  {
    url: "www.so.com",
    title: "新增文章",
  },
]);
feed.modify([
  {
    url: "hao.360.cn",
    title: "修改文章",
  },
]);
feed.delete([
  {
    url: "kuai360.com",
    title: "删除文章",
  },
]);
```
