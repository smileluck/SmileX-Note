[toc]

---

# 单例使用

> 只在第一次使用时生效，后续调用都返回同样结果

要实现一个 `Promise` 只调用一次，而其他地方调用时返回同样的结果，可以使用一个变量来存储已经解析的 `Promise` 结果。以下是一个实现示例：

```javascript
class SingletonPromise {
  constructor(executor) {
    this._promise = null;
    this._resolve = null;
    this._reject = null;

    const wrappedExecutor = (resolve, reject) => {
      if (this._promise) {
        // 如果已经有Promise，则直接返回
        return;
      }
      this._resolve = resolve;
      this._reject = reject;
      executor(resolve, reject);
    };

    this._promise = new Promise(wrappedExecutor);
  }

  getPromise() {
    return this._promise;
  }
}

// 使用示例
const singletonPromise = new SingletonPromise((resolve, reject) => {
  console.log('Promise执行中...');
  setTimeout(() => {
    resolve('成功的结果');
  }, 1000);
});

// 多次调用getPromise，都会返回同一个Promise
singletonPromise.getPromise().then(result => {
  console.log(result); // 成功的结果
});

singletonPromise.getPromise().then(result => {
  console.log(result); // 成功的结果
});
```

### 解释

1. **SingletonPromise 类**:
   
   - 构造函数接受一个 `executor` 函数，类似于 `new Promise` 的用法。
   - 使用 `_promise` 来存储已经创建的 `Promise`。
   - 使用 `_resolve` 和 `_reject` 来存储 `Promise` 的解决和拒绝函数。

2. **wrappedExecutor**:
   
   - 检查 `_promise` 是否已经存在。如果存在，说明 `Promise` 已经在执行或已经完成，直接返回。
   - 如果 `_promise` 不存在，初始化 `_resolve` 和 `_reject`，并执行传入的 `executor`。

3. **getPromise 方法**:
   
   - 返回存储的 `_promise`，确保每次调用都返回同一个 `Promise`。

### 优点

- **延迟执行**: `Promise` 只有在第一次调用 `getPromise` 时才会执行。
- **单例模式**: 无论调用多少次 `getPromise`，都会返回同一个 `Promise` 实例。

### 注意事项

- 如果需要在 `Promise` 被拒绝后重新尝试，可以扩展这个类，添加重试逻辑。
- 确保 `executor` 函数不会抛出同步错误，否则可能会导致未捕获的异常。
