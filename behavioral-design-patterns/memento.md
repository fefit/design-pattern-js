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
class Snapshot{
  constructor(private eidtor: Editor, private cursor: Cursor, private content: string){

  }
  
}
class Editor {
  private cursor: Cursor = {
    x: 0,
    y: 0,
  };
  private content: string = "";
  private history: Snapshot;
  // 增加构造函数，初始化快照
  constructor(){
    // 创建快照
    this.history = new Snapshot(this, this.cursor, this.content);
  }
  
  changeCursor(x: number, y: number){
    this.setCursor(x, y);
  }
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

