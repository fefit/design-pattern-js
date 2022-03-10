# 备忘录模式 Memento

对象包含了各种状态数据，且部分数据不便于对外暴露细节，于是将状态数据封装在对象内部，从而实现对象状态的保存和恢复，这种模式被称为备忘录模式。

## 常见的使用场景

- 数据库里的事务

- 编辑器里的撤销、恢复

- 代码版本管理

### 代码示例

现在假设我们要制作一个简单的编辑器，编辑器里包含了光标的位置数据、编辑器的内容等。

初始的代码如下所示：

```typescript
interface Cursor {
  x: number;
  y: number;
}
class Editor {
  private cursor: Cursor = {
    x: 0,
    y: 0,
  };
  private content: string = "";
  setCursor(x: number, y: number) {
    this.cursor = {
      x,
      y,
    };
  }
  setContent(content: string) {
    this.content = content;
  }
}
```

现在我们想要给编辑器加一个撤销和恢复的功能，该怎么做呢？

```typescript
// 快照类
class Snapshot {
  private cursor: Cursor = null;
  private content: string = "";
  private prev: Snapshot = null;
  private next: Snapshot = null;
  private created: boolean = false;
  constructor(private editor: Editor) {}
  // 创建快照内容
  create(cursor: Cursor, content: string): Snapshot {
    if (!this.created) {
      this.created = true;
      this.cursor = cursor;
      this.content = content;
      return this;
    } else {
      const next = new Snapshot(this.editor);
      this.next = next;
      next.prev = this;
      next.create(cursor, content);
      return next;
    }
  }
  // 使用快照重置数据
  restore() {
    this.editor.setCursor(this.cursor);
    this.editor.setContent(this.content);
  }
  // 是否允许前进
  allowForward(): boolean {
    return this.next !== null;
  }
  // 是否允许回退
  allowBackward() {
    return this.prev !== null;
  }
  // 前进快照
  forward(): Snapshot {
    if (this.allowForward()) {
      this.next.restore();
      return this.next;
    }
    return this;
  }
  // 后退快照
  backward(): Snapshot {
    if (this.allowBackward()) {
      this.prev.restore();
      return this.prev;
    }
    return this;
  }
}
// 编辑器类
class Editor {
  private cursor: Cursor = {
    x: 0,
    y: 0,
  };
  private content: string = "";
  private history: Snapshot;
  // 增加构造函数，初始化快照
  constructor() {
    // 创建快照
    this.history = new Snapshot(this);
    this.createSnapshot();
  }
  // 创建快照
  createSnapshot() {
    this.history = this.history.create({ ...this.cursor }, this.content);
  }
  // 修改内容并创建快照
  changeContent(content: string) {
    this.setContent(content);
    this.createSnapshot();
  }
  // 修改光标并创建快照
  changeCursor(cursor: Cursor) {
    this.setCursor(cursor);
    this.createSnapshot();
  }
  // 设置光标位置
  setCursor(cursor: Cursor) {
    this.cursor = cursor;
  }
  // 设置内容
  setContent(content: string) {
    this.content = content;
  }
  // 回退
  undo() {
    this.history = this.history.backward();
  }
  // 重做
  redo() {
    this.history = this.history.forward();
  }
  // 打印信息
  debug() {
    console.log(
      `cursor:{x:${this.cursor.x},y:${this.cursor.y}},content:"${this.content}"`
    );
  }
}
// 测试代码
const editor = new Editor();
editor.debug();
editor.changeCursor({ x: 111, y: 111 });
editor.debug();
editor.changeContent("hello:111");
editor.debug();
editor.undo();
editor.debug();
editor.changeContent("hello: 222");
editor.undo();
editor.debug();
editor.redo();
editor.debug();
```

以上的例子中，针对编辑器的操作都封装在了编辑器的内部，实际上，编辑器都会提供一系列的编辑命令，而编辑器自身更应关注编辑时自身要处理的数据，因此将撤销和重做等功能进行拆分更符合单一职责原则。

于是可以对上面的代码再进行优化：

```typescript
interface Cursor {
  x: number;
  y: number;
}
// 历史记录类
class Historys {
  private snapshots: Snapshot[] = [];
  private currentIndex: number = -1;
  // 获取当前快照
  getCurrentSnapshot() {
    return this.currentIndex >= 0 ? this.snapshots[this.currentIndex] : null;
  }
  // 增加快照
  addSnapshot(snapshot: Snapshot) {
    const lastIndex = this.snapshots.length - 1;
    if (this.currentIndex === lastIndex) {
      this.snapshots.push(snapshot);
      this.currentIndex = lastIndex + 1;
    } else {
      this.snapshots = this.snapshots
        .slice(0, this.currentIndex + 1)
        .concat(snapshot);
      this.currentIndex++;
    }
  }
  // 是否允许前进
  allowForward(): boolean {
    return (
      this.currentIndex >= 0 && this.currentIndex < this.snapshots.length - 1
    );
  }
  // 是否允许回退
  allowBackward(): boolean {
    return this.currentIndex > 0 && this.currentIndex < this.snapshots.length;
  }
  // 前进快照
  forward(): Snapshot {
    if (this.allowForward()) {
      this.currentIndex++;
      return this.getCurrentSnapshot();
    }
    return null;
  }
  // 后退快照
  backward(): Snapshot {
    if (this.allowBackward()) {
      this.currentIndex--;
      return this.getCurrentSnapshot();
    }
    return null;
  }
}
// 快照类
// Memento: 原发器对象的快照值数据，通过构造函数一次性创建
class Snapshot {
  constructor(
    private editor: Editor,
    private cursor: Cursor = null,
    private content: string = ""
  ) {}
  // 使用快照重置数据
  restore() {
    this.editor.setCursor(this.cursor);
    this.editor.setContent(this.content);
  }
  // 打印信息
  display() {
    console.log(
      `cursor:{x:${this.cursor.x},y:${this.cursor.y}},content:"${this.content}"`
    );
  }
}
// 命令类
// Caretaker: 负责人，简化原发器类，将原发器的备忘数据管理起来
class Commander {
  private readonly history: Historys;
  private editor: Editor = null;
  constructor() {
    this.history = new Historys();
  }
  // 初始化
  init(editor: Editor) {
    this.editor = editor;
    this.addBackup();
  }
  // 打印信息
  debug() {
    this.history.getCurrentSnapshot()?.display();
  }
  // 增加快照备份
  addBackup() {
    this.history.addSnapshot(this.editor.createSnapshot());
  }
  // 重做
  redo() {
    this.history.forward();
  }
  // 撤销
  undo() {
    this.history.backward();
  }
}
// 编辑器类
// Originator: 原发器类
class Editor {
  private cursor: Cursor = {
    x: 0,
    y: 0,
  };
  private content: string = "";
  // 构造函数
  constructor(private commander: Commander) {}
  // 创建快照
  createSnapshot() {
    return new Snapshot(this, { ...this.cursor }, this.content);
  }
  // 修改内容并创建快照
  changeContent(content: string) {
    this.setContent(content);
    this.commander.addBackup();
  }
  // 修改光标并创建快照
  changeCursor(cursor: Cursor) {
    this.setCursor(cursor);
    this.commander.addBackup();
  }
  // 设置光标位置
  setCursor(cursor: Cursor) {
    this.cursor = cursor;
  }
  // 设置内容
  setContent(content: string) {
    this.content = content;
  }
}
// 测试
const commander = new Commander();
const editor = new Editor(commander);
commander.init(editor);
commander.debug();
editor.changeCursor({ x: 111, y: 111 });
commander.debug();
editor.changeContent("hello:111");
commander.debug();
commander.undo();
commander.debug();
editor.changeContent("hello: 222");
commander.undo();
commander.debug();
commander.redo();
commander.debug();
commander.redo();
```

由以上的示例可以看出，一个常见的备忘录模式的流程是：

Originator原发器(`Editor`) -> 封装获取备忘数据方法(`createSnapshot`) 
  