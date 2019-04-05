

# [译]使用React Hooks请求数据

原文：[How to fetch data with React Hooks?](https://www.robinwieruch.de/react-hooks-fetch-data/)

在这篇文章里，我将演示一下，如果通过使用 [useState](https://reactjs.org/docs/hooks-state.html) [useEffect](https://reactjs.org/docs/hooks-effect.html) 等hooks，在 `React Hook`里请求数据。我们将使用 [Hacker News API](https://hn.algolia.com/api) 来获取最新流行的技术文章。我们将实现一个获取异步数据的自定义hook，能够在我们APP里多个地方进行复用，或者作为单独的包发布到npm上。

如果你还不了解 `React Hooks`，你可以通过我的 [React Hooks 简介](https://www.robinwieruch.de/react-hooks/) 了解下。本文完整的demo代码，在这个 [github仓库](https://github.com/the-road-to-learn-react/react-hooks-introduction) 。

*注意*：在将来，`React Hooks`的用处，不是请求数据。新的 `Suspense` 特性，将用来请求数据。本文主要是展示我们能用hooks做些什么，来加深我们对hooks的理解。


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

上面的代码里，两个state相互是独立的，怎么实现每次请求用户输入的主题数据呢？改动之后的代码如下：

```javascript
// 省略 ...

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        `http://hn.algolia.com/api/v1/search?query=${query}`,
      );

      setData(result.data);
    };

    fetchData();
  }, []);

  return (
    // 省略 ...
  );
}

export default App;
```

一个新的问题：在组件mount之后，你在输入框里输入不同的值，不会触发重新获取相应的数据。因为我们在 `useEffect` 的第二个参数，是一个空数组，表明这个hook不依赖任何状态，它只会在组件mount的时候执行一次。事实上，从上面代码可以看出，我们的 `useEffect` 里，是依赖了 `query` 这个变量，因此，我们需要把  `query` 添加到 `useEffect` 的依赖里。每当 `query` 改变的时候，都会触发effect重新执行：

```javascript
// 省略相同代码 ...

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        `http://hn.algolia.com/api/v1/search?query=${query}`,
      );

      setData(result.data);
    };

    fetchData();
  }, [query]);

  return (
    // 省略相同代码 ...
  );
}

export default App;
```

运行上面的代码，每当你在输入框里输入内容时，都会触发hook重新请求数据。但这会带来一个新的问题：我们在输入过程中，每输入一个字符，都会触发hook的执行，导致重新请求数据。更理想的情况，我们应该提供一个 **提交** 按钮来触发数据的刷新：

```javascript
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [search, setSearch] = useState('redux');

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        `http://hn.algolia.com/api/v1/search?query=${search}`,
      );

      setData(result.data);
    };

    fetchData();
}, [search]);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button type="button" onClick={() => setSearch(query)}>
        Search
      </button>

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
```

我们新增了一个 state `search` 来保存当前要检索的topic。当用户点击搜索按钮的是，将输入框的值更新到 `search` 中，触发effect重新执行来请求相应数据。同时，我们将 `search` 的默认值设置为和 `query` 一样，都是 `redux`，因为effect会在组件mount的时候执行一次，这时候拿到的 `query` 就是默认值。你可能会想，`query` 和 `search` 要表达的几乎是同一个东西，用两个 state 似乎容易混淆，那我们可以把实际要请求的 `url` 作为第二个state，而不是 `search`，比如下面这样：

```javascript
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'http://hn.algolia.com/api/v1/search?query=redux',
  );

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(url);

      setData(result.data);
    };

    fetchData();
  }, [url]);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button
        type="button"
        onClick={() =>
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        Search
      </button>

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
```

OK，到这里，我们实现了通过事件来隐式的触发effect执行，从而重新请求数据。下面我们看看，怎么处理 **加载中** 这种状态呢。


## React hooks 数据请求实现加载中

我们再引用一个新的 `state` 来保存 **加载中** 这个状态，通常我们会渲染一个加载中的指示器，来提示用户网络请求正在处理中：

```javascript
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'http://hn.algolia.com/api/v1/search?query=redux',
  );
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);

      const result = await axios(url);

      setData(result.data);
      setIsLoading(false);
    };

    fetchData();
  }, [url]);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button
        type="button"
        onClick={() =>
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        Search
      </button>

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>
  );
}

export default App;
```

当我们的effect执行时，会设置 `isLoading` 为 `true`，当请求结束时，设置为false。


## React hooks 数据请求的错误处理

通常在网络请求时，都必须要考虑到网络异常的情况，那我们在hook里怎么来处理网络异常呢？和上面的加载中类似，我们只需要额外增加一个state就行了。我们在代码里使用了 `async/await`，因此可以使用 `try-catch` 来处理异步操作的异常：

```javascript
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'http://hn.algolia.com/api/v1/search?query=redux',
  );
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setIsError(false);
      setIsLoading(true);

      try {
        const result = await axios(url);

        setData(result.data);
      } catch (error) {
        setIsError(true);
      }

      setIsLoading(false);
    };

    fetchData();
  }, [url]);

  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button
        type="button"
        onClick={() =>
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        Search
      </button>

      {isError && <div>Something went wrong ...</div>}

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>
  );
}

export default App;
```

每当我们effect重新执行的时候，都会重置错误状态。在通常情况下，用户第一次遇到网络错误之后，可以重新发起一次请求，第二次请求是有可能成功的，因此需要在每次请求开始时，重置错误状态。


## 结合 form 提交数据

在大多数时候，我们都会把用户输入项，放在一个 `form` 表单里，结合 `form` 表单之后的代码如下：

```javascript
function App() {
  // 省略相同代码 ...

  const doFetch = () => {
    setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`);
  };

  return (
    <Fragment>
      <form onSubmit={event => {
        doFetch();

        // 阻止浏览器默认刷新页面
        event.preventDefault();
      }}>
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>

      {isError && <div>Something went wrong ...</div>}

      // 省略相同代码 ...
    </Fragment>
  );
}
```


## 自定义数据请求hook

 到目前为止，我们所有的hook代码，都写在 **函数组件** 内部，这可能让我们的函数组件显得很臃肿，因此，我们可以把数据请求的hook，单独提取出来，作为自定义的hook，就像下面这样：

 ```javascript
 import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';

const useDataApi = (initialUrl, initialData) => {
  const [data, setData] = useState(initialData);
  const [url, setUrl] = useState(initialUrl);
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setIsError(false);
      setIsLoading(true);

      try {
        const result = await axios(url);

        setData(result.data);
      } catch (error) {
        setIsError(true);
      }

      setIsLoading(false);
    };

    fetchData();
  }, [url]);

  const doFetch = url => {
    setUrl(url);
  };

  return { data, isLoading, isError, doFetch };
};

function App() {
  const [query, setQuery] = useState('redux');
  const { data, isLoading, isError, doFetch } = useDataApi(
    'http://hn.algolia.com/api/v1/search?query=redux',
    { hits: [] },
  );

  return (
    <Fragment>
      <form
        onSubmit={event => {
          doFetch(
            `http://hn.algolia.com/api/v1/search?query=${query}`,
          );

          event.preventDefault();
        }}
      >
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>

      {isError && <div>Something went wrong ...</div>}

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>
  );
}

export default App;
 ```

自定义hook没什么特别的，也是一个普通的函数，里面可以调用其他的hook，返回一些内部的state以及修改state的方法。


## Reducer Hook

到这里，我们实现了使用多个state来维护我们组件的各种状态：加载中，网络异常以及请求成功。但是，这3种状态，我们使用了3个独立的state，然而他们本质上是相互关联的。正如你所看到的，这3个状态都在我们的hook里维护，那我们何不通过 `useReducer` 这个hook来把这3个状态组合成一个呢？

一个 `useReducer` hook，接收一个 `reducer` 函数以及初始状态 ，返回当前的状态以及修改状态的 `dispatch` 函数。

```javascript
import React, {
  Fragment,
  useState,
  useEffect,
  useReducer,
} from 'react';
import axios from 'axios';

const dataFetchReducer = (state, action) => {
  switch (action.type) {
    case 'FETCH_INIT':
      return {
        ...state,
        isLoading: true,
        isError: false
      };
    case 'FETCH_SUCCESS':
      return {
        ...state,
        isLoading: false,
        isError: false,
        data: action.payload,
      };
    case 'FETCH_FAILURE':
      return {
        ...state,
        isLoading: false,
        isError: true,
      };
    default:
      throw new Error();
  }
};

const useDataApi = (initialUrl, initialData) => {
  const [url, setUrl] = useState(initialUrl);

  const [state, dispatch] = useReducer(dataFetchReducer, {
    isLoading: false,
    isError: false,
    data: initialData,
  });

  useEffect(() => {
    const fetchData = async () => {
      dispatch({ type: 'FETCH_INIT' });

      try {
        const result = await axios(url);

        dispatch({ type: 'FETCH_SUCCESS', payload: result.data });
      } catch (error) {
        dispatch({ type: 'FETCH_FAILURE' });
      }
    };

    fetchData();
  }, [url]);

  return { ...state, doFetch };
};

```

总的说来，通过Reducer Hook，我们能够把相关联的状态管理封装在一起。通过 `dispatch` 事件的方式来触发状态改变，让我们的状态更加可预测。


## Effect Hook 里中断网络请求

在开发React应用中，经过遇到这样一种情况，我们在 `componentDidMount` 里发起异步请求，在请求回来 **之前**，我们的组件被 `unmount` 了，等异步请求完成时，再调用 `setState`，这时候会触发警告，因为我们在一个已经销毁的组件上更新state。接下来我们看看，如何在自定义hook里，变在组件unmount之后，还会更新state的问题:

```javascript
const useDataApi = (initialUrl, initialData) => {
  const [url, setUrl] = useState(initialUrl);

  const [state, dispatch] = useReducer(dataFetchReducer, {
    isLoading: false,
    isError: false,
    data: initialData,
  });

  useEffect(() => {
    let didCancel = false;

    const fetchData = async () => {
      dispatch({ type: 'FETCH_INIT' });

      try {
        const result = await axios(url);

        if (!didCancel) {
          dispatch({ type: 'FETCH_SUCCESS', payload: result.data });
        }
      } catch (error) {
        if (!didCancel) {
          dispatch({ type: 'FETCH_FAILURE' });
        }
      }
    };

    fetchData();

    return () => {
      didCancel = true;
    };
  }, [url]);

  const doFetch = url => {
    setUrl(url);
  };

  return { ...state, doFetch };
};
```

每一个 effect hook都 *可以* 返回一个清理函数，这个函数会在组件unmount的时候被调用。在上面的代码里，我们在effect里添加了一个标记，表明当前组件是否被unmount了，默认是false的，在effect清理函数里，会设置为true。当网络结束时，我们会先判断这个标记，如果组件已经被unmount了，那么就不用更新state了。

*注意*： 我们这里并没有真正的 **中断** 网络请求，网络请求仍然完成了，我们只是在网络结束之后，不会去更新被unmount的组件状态。


## 相关文档

* [react fetching data](https://www.robinwieruch.de/react-fetching-data/)
* [react-warning-cant-call-setstate-on-an-unmounted-component](https://www.robinwieruch.de/react-warning-cant-call-setstate-on-an-unmounted-component/)


        ----- 时2019年4月5日周五下午 15:54 竣工于帝都霍营龙跃苑
