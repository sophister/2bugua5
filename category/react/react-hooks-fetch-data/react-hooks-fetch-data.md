# [译]使用React Hooks请求数据

原文：[How to fetch data with React Hooks?](https://www.robinwieruch.de/react-hooks-fetch-data/)

在这篇文章里，我将演示一下，如果通过使用 [useState](https://reactjs.org/docs/hooks-state.html) [useEffect](https://reactjs.org/docs/hooks-effect.html) 等hooks，在 `React Hook`里请求数据。我们将使用 [Hacker News API](https://hn.algolia.com/api) 来获取最新流行的技术文章。我们将实现一个获取异步数据的自定义hook，能够在我们APP里多个地方进行复用，或者作为单独的包发布到npm上。

如果你还不了解 `React Hooks`，你可以通过我的 [React Hooks 简介](https://www.robinwieruch.de/react-hooks/) 了解下。本文完整的demo代码，在这个 [github仓库](https://github.com/the-road-to-learn-react/react-hooks-introduction) 。

注意：在将来，`React Hooks`的用处，不是请求数据。新的 `Suspense` 特性，将用来请求数据。本文主要是展示我们能用hooks做些什么，来加深我们对hooks的理解。


## 在hooks里请求数据

如果你还不了解怎么在react里请求数据，建议先看看我的 [可扩展的数据请求这篇文章](https://www.robinwieruch.de/react-fetching-data/)，它会介绍在react `class`组件里，怎么进行数据请求，包括怎么通过 [Render Prop Component](https://www.robinwieruch.de/react-render-props-pattern/) 和 [高阶组件HOC](https://www.robinwieruch.de/gentle-introduction-higher-order-components/) 封装可服用的数据请求逻辑，以及怎样进行 加载中和错误 的状态展示。在本文里，我将展示如果在react **函数组件** 里使用 `hooks` 来达到同样的效果。

```javascript
import React, { useState } from 'react';

function App() {
  const [data, setData] = useState({ hits: [] });

  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}

export default App;
```

我们demo里会展示一个hacker news的文章列表。我们使用 `useState` 来维护APP的state以及提供更新state的操作。state的默认值是一个空的数组。

我将使用 [axios](https://github.com/axios/axios) 来处理数据请求，当然你也可以使用自己习惯的其他库，或者使用浏览器原生的 `fetch` 方法。下面我们看下加上 `useEffect` 来请求数据之后的代码：

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });

  useEffect(async () => {
    const result = await axios(
      'http://hn.algolia.com/api/v1/search?query=redux',
    );

    setData(result.data);
  });

  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}

export default App;
```

在 `useEffect` 里，我们使用axios请求到数据之后，调用 `useState` 返回的 `setData` 方法，将新的数据更新到state上，从而触发组件重新render。异步函数我们使用 `async/await` 的语法，来简化代码。

然而，当你运行上面的代码，你会进行死循环。上面的 `useEffect` 函数，在组件初次挂载和每次更新的时候，都会执行；在 `useEffect` 函数里，我们在请求到数据之后，更新了组件的state，导致组件重新渲染，组件渲染之后，又会调用 `useEffect` 函数。结果就是，组件一直在请求数据，刷新，请求数据，刷新……这当然是一个必须要解决掉的bug。我们只希望在组件初次挂载的时候，请求数据。下面，我们将给 `useEffect` 传第二个空数组的参数，来实现这个效果：只在组件mount的时候，调用 `useEffect` 函数。

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });

  useEffect(async () => {
    const result = await axios(
      'http://hn.algolia.com/api/v1/search?query=redux',
    );

    setData(result.data);
  }, []);

  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}

export default App;
```

`useEffect` 的第二个数组参数，用来定义该hook依赖的所有变量。依赖项中只要有一个改变，就会重新调用 `useEffect` 。如果依赖项是空的数组，表明我们的hook不依赖任何变量，因此，该hook只会在组件初次mount的时候执行。

上面的代码还有一个问题。我们使用了 `async/await` 来处理异步操作，根据规范，`async` 函数会返回一个隐式的 `Promise`: " *The async function declaration defines an asynchronous function, which returns an AsyncFunction object. An asynchronous function is a function which operates asynchronously via the event loop, using an implicit Promise to return its result.* " 。然而， effect hook 要么什么都不返回，要么返回一个清理函数。因此，运行上面的代码，你会在 console 里看到这样的警告：*Warning: useEffect function must return a cleanup function or nothing. Promises and useEffect(async () => …) are not supported, but you can call an async function inside an effect.* 因此，不能直接给 `useEffect` 传一个 `async` 函数，我们需要在 `useEffect` 内部，定义一个单独的 `async` 函数。修改之后的代码如下：

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        'http://hn.algolia.com/api/v1/search?query=redux',
      );

      setData(result.data);
    };

    fetchData();
  }, []);

  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}

export default App;
```

到这里，我们就实现了最基本的在react hook里请求数据的功能。如果你想知道怎么进行加载中处理，错误处理，以及如果在 `form` 表单中触发数据请求，如何将数据请求逻辑封装成自定义的hook，就继续往下看吧。


## 怎么手动(或程序)触发一个hook

OK，到目前为止，我们可以在组件mount之后，请求数据并且触发组件更新。但是，怎么实现使用输入框来请求我们输入的话题呢？之前代码里使用“redux” 作为默认的话题。那我们怎么修改这个话题呢，比如我想查询 “react” 相关的文章呢？接下来我们增加一个输入框，来允许用户查询自己感兴趣的话题。为了保存用户输入的内容，我们新增加了一个state：

```javascript
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        'http://hn.algolia.com/api/v1/search?query=redux',
      );

      setData(result.data);
    };

    fetchData();
  }, []);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <ul>
        {data.hits.map(item => (
          <li key={item.objectID}>
            <a href={item.url}>{item.title}</a>
          </li>
        ))}
      </ul>
    </Fragment>
  );
}

export default App;
```



#
