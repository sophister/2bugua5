# React Hooks useDebounce

前端开发中，经常需要对一些频发触发的事件回调进行 *截流* ，也就是传说中的 `debounce` 函数，这玩意儿和  函数*防抖* `throttle` 也能称得上是前端面试的高频题目了吧。

随着最近业务里越来越多的react代码，开始基于 React Hooks 来实现，就面临，怎么在 React Hooks里来做函数截流。

看了一些代码，大概是这样的

```typescript
function Detail(){
  const [data, setData] = useStatae(null);
  // 用户连续输入过程中，这里能work么？
  const getProductDetail = utils.debounce(function(){
    fetch('xxxx').then((data) => {
      setData(data)
    });
  });
  return (
    <div>
    	<input onChange={getProductDetail} placeholder="input some keyword..." />
    </div>
  );
}
```

感觉，上面这段代码，还是没有理解react函数组件的一个关键东东：**函数组件的每一次render，都生成一份单独的快照，里面的局部变量(包括内部定义的函数，当然也有匿名函数)，都和前一次render是不同的东东** 

回到上面的代码，该组件 **每一次render** ，都会生成新的 `getProductDetail` 函数，因此，上面的 `debounce` 其实是不会生效的。

有一些实现思路，是直接在回调函数上延迟，类似 [这个作者的](https://github.com/samanmohamadi/use-debounced-effect/blob/master/src/index.js) ，代码长这样

```javascript
import React, { useEffect, useRef } from 'react';

export function useDebouncedEffect(callback, delay, deps = []) {
  const firstUpdate = useRef(true);
  useEffect(() => {
      if (firstUpdate.current) {
        firstUpdate.current = false;
        return;
      }
      const handler = setTimeout(() => {
        callback();
      }, delay);

      return () => {
        clearTimeout(handler);
      };
    },
    [delay, ...deps],
  );
}

export default useDebouncedEffect;
```

这种实现，在大多数场景下够用了。但是，考虑如下场景：

如果我们在 `callback` 里是执行一个异步调用，比如API请求之类的。我们第一个请求已经发出去了，在第一个请求返回之前，第二个请求也发出去了，并且第二个请求的结果先收到，这时候，我们需要解决，当收到第一个请求的结果时，应该要抛弃掉，不能使用了。

通常我们在使用 `useEffect`  的时候，都会通过 `cleanup` 函数来清理掉前一个无效effect的异步结果。那么，在添加了 `debounce` 的场景下，是不是也可以这样来实现呢？

这就是第二种思路，这种方法，通常不会直接对原始的 `callback` 进行 `debounce` ，而是引入了一个触发回调执行的额外的state，通过对state的修改进行 `debounce` ，触发 `callback` 的执行。

本来以为这个 [作者的写法是基于这种思路的](https://github.com/xnimorz/use-debounce/blob/master/src/useDebouncedCallback.ts) ，但是仔细一看，好像还是有点差别，作者还是第一种思路。

好吧，没有找到和我想法完全一样的😂

其实参照上面第二个思路，来实现一个 `useDebounce` 也不复杂，我自己手撸一个，大概是这样的

```typescript
import { useState, useEffect, useRef, useCallback } from "react";

export function useDebounceState<T>(initValue: T, delay: number) {
  const [value, setValue] = useState<T>(initValue);
  const timerRef = useRef(null);
  // reset timer when delay changes
  useEffect(
    function () {
      if (timerRef.current) {
        clearTimeout(timerRef.current);
        timerRef.current = null;
      }
    },
    [delay]
  );
  const debounceSetValue = useCallback(
    function (val) {
      if (timerRef.current) {
        clearTimeout(timerRef.current);
        timerRef.current = null;
      }
      timerRef.current = setTimeout(function () {
        setValue(val);
      }, delay);
    },
    [delay]
  );
  return [value, debounceSetValue];
}

interface DebounceOptions {
  imediate?: boolean;
  initArgs?: any[];
}

const INIT_VALUE = -1;
export function useDebounce(fn, delay: number, options: DebounceOptions = {}) {
  const [num, setNum] = useDebounceState(INIT_VALUE, delay);
  // save actual arguments when fn called
  const callArgRef = useRef(options.initArgs || []);
  // save real callback function
  const fnRef = useRef(fn);
  // wrapped function
  const trigger = useCallback(function () {
    callArgRef.current = [].slice.call(arguments);
    setNum((prev) => {
      return prev + 1;
    });
  }, []);
  // update real callback
  useEffect(function () {
    fnRef.current = fn;
  });
  useEffect(
    function () {
      if (num === INIT_VALUE && !options.imediate) {
        // prevent init call
        return;
      }
      return fnRef.current.apply(null, callArgRef.current);
    },
    [num, options.imediate]
  );
  return trigger;
}

```

主要特点大概就是

* 允许首次调用的时候直接执行，而不delay，通过 `options.imediate` 来完成
* 通过在 `useEffect` 里来执行实际的回调，充分利用 `cleanup` 机制，允许回调函数返回一个 `cleanup` 函数

比如在网络请求的情况下，我们可以这样使用

```typescript
import React, { useEffect } from "react";

function apiCall(arg) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(`${arg} resolved`);
    }, 1500);
  });
}

export function NetworkTest() {
  const debouncedApiCall = useDebounce(function (arg: string) {
    console.log(`${Date.now()} : api call start, arg=${arg}`);
    // 通常在 useEffect 里，会用一个标记+cleanup函数来判断异步结果是否过期
    let isObsolete = false;
    // assume this is a network api call
    apiCall(arg).then(function (out) {
      if (isObsolete) {
        console.log(`${Date.now()} : receive data but dropped, arg=${arg}`);
        return;
      }
      console.log(`${Date.now()} : final response is [${out}], arg=${arg}`);
    });
    return function cleanup() {
      console.log(`${Date.now()} : cleanup called for api, arg=${arg}`);
      isObsolete = true;
    };
  }, 500);
  // 这里模仿连续2次事件的触发，并且每次都成功执行了函数
  useEffect(
    function () {
      debouncedApiCall("1");
      // delay 600, make sure the first api call start
      setTimeout(function () {
        debouncedApiCall("2");
      }, 600);
    },
    [debouncedApiCall]
  );
  return (
    <div>
      <h2>test network response with debounce</h2>
      <div>watch console log when page load</div>
    </div>
  );
}

```

代码也放在了 [github gist上](https://gist.github.com/sophister/9cc74bb7f0509bdd6e763edbbd21ba64)  ，也可以在 [codesandebox上试试demo](https://codesandbox.io/s/react-hook-debounce-demo-mgr89?file=/src/App.js) 



## 相关链接

* [lodash库的debounce函数](https://lodash.com/docs/4.17.15#debounce) 
* 