Transaction
--------

位于源码：`<root>/src/utils/Transaction.js`。

作用
------

用于在执行一个函数时，为其封装一定的不变量。即当函数执行失败时，能正确地恢复到初始状态。

使用
------

```
// 1. 定义一个Transaction. 
function MyTransaction() {}
extends(MyTransaction.prototype, Transaction.Mixin)

// 2. 实现getTransactionWrappers
//    定义任意多数量的wrappers
MyTransaction.prototype.getTransactionWrappers = function () {
  return [
    {
      initialize: func1,
      close: func2,
    },
    {
      initialize: func3,
      close: func4,
    }
  ];
}

// 3. perform特定的函数，传入context及arguments
(new MyTransaction()).perform(mainFn);
````

执行顺序
----------
上面的函数执行顺序为：

```
func1 -> func3 -> mainFn -> func2 -> func4
```

其wrapper机制如下图：
```
 *                       wrappers (injected at creation time)
 *                                      +        +
 *                                      |        |
 *                    +-----------------|--------|--------------+
 *                    |                 v        |              |
 *                    |      +---------------+   |              |
 *                    |   +--|    wrapper1   |---|----+         |
 *                    |   |  +---------------+   v    |         |
 *                    |   |          +-------------+  |         |
 *                    |   |     +----|   wrapper2  |--------+   |
 *                    |   |     |    +-------------+  |     |   |
 *                    |   |     |                     |     |   |
 *                    |   v     v                     v     v   | wrapper
 *      call          | +---+ +---+   +---------+   +---+ +---+ | invariants
 * perform(anyMethod) | |   | |   |   |         |   |   | |   | | maintained
 * +----------------->|-|---|-|---|-->|anyMethod|---|---|-|---|-|-------->
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | +---+ +---+   +---------+   +---+ +---+ |
 *                    |  initialize                    close    |
 *                    +-----------------------------------------+
```


如果任意的initialize函数抛出异常了，那么：

- 其对应的close函数将不会被执行
- mainFn将不会被执行
- 第一个抛出异常的函数的异常将抛出来，后面的将被忽略


如果只有mainFn抛异常了，那么：

- initialize和close按上面的顺序执行
- 异常将抛出

如果只有close函数抛出异常了，那么：

- mainFn将正常执行，因为mainFn总是先于close执行的
- 异常将抛出
