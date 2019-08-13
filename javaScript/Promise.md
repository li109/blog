# JavaScript夯实基础系列（六）：Promise
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
### 一、构造函数
&emsp;&emsp;<br/>
```js
function Promise(resolver) {
  this[PROMISE_ID] = nextId();
  this._result = this._state = undefined;
  this._subscribers = [];

  if(typeof resolver === 'function') {
    if(this instanceof Promise) {
      try {
        resolver(function resolvePromise(value) {
            resolve(promise, value);
          }, function rejectPromise(reason) {
            reject(promise, reason);
        });
      } catch (e) {
        reject(promise, e);
      }
    } else {
      throw new TypeError("Failed to construct 'Promise': Please use the 'new' operator, this object constructor cannot be called as a function.");
    }
  } else {
    throw new TypeError('You must pass a resolver function as the first argument to the promise constructor');
  }
}
```
&emsp;&emsp;<br/>
### 二、Promise.prototype.then()
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
### 三、存在的问题
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
### 四、总结
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>