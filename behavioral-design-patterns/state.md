# 状态模式 State

对象的某些行为受状态的影响，通过对状态的管理来改变对象行为的模式就是状态模式。


## 常见的使用场景

- 编译器词法分析

- 流程审核

## 代码示例

我们经常会遇到对同一个操作，因为当前所处的状态不一样，产生的行为也不一样。

比如，提交表单的时候：

```html
<button id="btn">提交</button>
<script>
  (function(){
    let isFetching = false;
    document.getElementById('btn').addEventListener('click', function(){
      if(isFetching){
        return false;
      }
      const $this = $(this);
      isFetching = true;
      $.ajax({
        url: "/someurl",
        complete: function(){
          isFetching = false;
          $this.text('提交');
        }
      });
      $this.text('提交中...');
    }, false);
  })();
</script>
```

上面是一个比较常见的示例，提交按钮在点击后发起请求，请求过程中按钮被锁定防止重复提交，这时提交按钮就存在两种状态，锁定和未锁定，并会在两者中进行转换。

现在我们来看一个更复杂点的示例：

```typescript
class VideoPlayer {
  
}
```