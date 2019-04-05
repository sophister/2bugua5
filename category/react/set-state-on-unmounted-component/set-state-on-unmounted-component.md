# [译]避免在unmounted组件上调用setState



**原文** :  [https://www.robinwieruch.de/react-warning-cant-call-setstate-on-an-unmounted-component](https://www.robinwieruch.de/react-warning-cant-call-setstate-on-an-unmounted-component)



很多人在开发 `React` 的过程中，会遇到下面这些警告。github上很多issue都和这些警告相关。因此，我想在这篇文章里，讲讲下面这两个警告的原因和应对方法。



* **Warning: Can only update a mounted or mounting component. This usually means you called setState, replaceState, or forceUpdate on an unmounted component. This is a no-op.**
* **Warning: Can’t call setState (or forceUpdate) on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in the componentWillUnmount method.**



在大多数情况下，警告并不会让你的应用崩溃。但你还是应该关注一下这些警告。比如，如果你不能妥善处理你的组件state问题，上面这些警告将会导致性能问题。让我们进一步来看看这些警告到底是怎么回事。



当你的组件已经unmount之后，如果你还在该组件上调用 `this.setState()`，那么上面的警告就会出现。有下面几种情况，都会触发组件的unmount：



* 根据 [React 的条件渲染](https://www.robinwieruch.de/conditional-rendering-react/) ，你不再渲染某个组件了
* 发生了页面跳转，比如使用了 `React Router`，从一个页面跳转到了另一个页面



当组件被unmount之后，你仍然可能会调用该组件的 `this.setState()` 方法，比如你在该组件内部有异步操作(通常是网络请求)发生，在异步操作结束之后，你可能需要更新该组件的state。下面是这种情况发生的例子：



* 你在组件内，向某个API接口发起了异步请求，在请求结束之前，你的组件被unmount了。在这之后，请求响应，你需要根据响应的内容，调用组件的 `this.setState()` 来更新状态，然而，这时该组件已经被unmount了
* 你在组件内部绑定了一个事件回调函数，但是没有在组件的 `componentWillUnmount` 里取消事件绑定。在组件被unmount之后，该事件回调函数可能会被调用
* 你在组件内部有一个定时器(interval)，你在计时器的回调函数里，调用了 `this.setState()` 来更新组件状态。如果你忘记了在 `componentWillUnmount` 里清除掉定时器，那么就会像上面那样，在unmount的组件上更新state



出现上面的警告时，最坏的情况是什么呢？它将给你的react应用带来性能上的负面影响，因为上述情况会导致你的应用存在内存泄露问题。如果你只是在某个组件里，在组件unmount之后还调用了 `this.setState` ，可能不会有太大的问题。但是，如果你的应用了有很多组件都有这个问题，那你的react应用性能将会收到明显的的影响。当然了，这也不是最坏的情况。最坏的情况发生在你忘记了取消事件绑定或者清除计时器。想象一下，你在组件内启动了定时器，每秒都会更新组件的state，之后组件被unmount了。如果你忘记了清除这个定时器，那么你可能会感受到，你的应用性能会被明显的拖慢。



## 怎样避免在定时器/事件回调里调用unmounted组件的setState



你可能已经注意到了，在大多数情况下，如果我们能够在组件的 `componentWillUnmount`里正确的处理定时器、事件监听，那么上面的警告是能够避免的。比如，在 `componentWillUnmount` 里，清除定时器，取消事件绑定等。



在这个 [定时器demo](https://github.com/the-road-to-learn-react/react-interval-setstate-unmounted-component-performance) 里，你可以试试不清除掉定时器的情况。



## 怎样避免在异步请求里调用unmounted组件的setState



在上面的例子里，我们针对定时器、事件绑定，在 `componentWillUnmount`里，相应的清除了定时器、取消了事件绑定。实时上，我们没有理由不这样做。

那么，在react组件里的异步请求的情况下，该怎么避免上面的问题呢？我们经常会在react组件里发起异步请求，在网络结束之后，调用 `this.setState()` 来更新组件state。然而，在我们异步请求结束之前，组件就已经被unmount了，肿么办？上面的警告会出现在你的浏览器控制台里，因为react不能将state更新到已经unmount的组件上。看个例子：



```javascript
class News extends Component {
  constructor(props) {
    super(props);

    this.state = {
      news: [],
    };
  }

  componentDidMount() {
    axios
      .get('https://hn.algolia.com/api/v1/search?query=react')
      .then(result =>
        this.setState({
          news: result.data.hits,
        }),
      );
  }

  render() {
    return (
      <ul>
        {this.state.news.map(topic => (
          <li key={topic.objectID}>{topic.title}</li>
        ))}
      </ul>
    );
  }
}
```



要想避免这个问题，你可以在组件unmount的时候，中断网络请求，或者在网络结束时，避免调用 `this.setState()` 。然而，大多数基于 `Promise`的网络请求库，都没有提供中断网络请求的功能，因此我们需要自己在组件类上，增加一个类的 **实例属性** 来标记当前组件是否已经mount了。这个标记默认是 `false` 的，之后在组件的 `componentDidMount`里，标记设置为 `true`；在组件的 `componentWillUnmount`里，设置为 `false`。通过这个属性，我们能够知道当前组件是否处于mount之后。这个属性 **不会** 受 `this.setState()`的影响，因为它是 **实例属性**，我们能够直接在组件实例上访问它，**不**需要经过react组件的 `this.state`。因为它不在组件的state上，我们修改这个属性，也就 **不会** 触发组件重新render了。看下修改之后的代码：



```javascript
class News extends Component {
  _isMounted = false;

  constructor(props) {
    super(props);

    this.state = {
      news: [],
    };
  }

  componentDidMount() {
    this._isMounted = true;

    axios
      .get('https://hn.algolia.com/api/v1/search?query=react')
      .then(result => {
        if (this._isMounted) {
          this.setState({
            news: result.data.hits,
          });
        }
      });
  }

  componentWillUnmount() {
    this._isMounted = false;
  }

  render() {
    // 省略相同代码 ...
  }
}
```



现在，即使在组件unmount之后，网络请求才结束，我们通过 `this._isMounted` 这个标记，就能避免在网络请求结束后，调用被unmounted组件的 `this.setState()`方法。这个demo地址的源码，在 [这个github仓库](https://github.com/the-road-to-learn-react/react-asynchronous-request-setstate-unmounted-component) 。



通过引入 `this._isMounted` 这个标记，不会影响到你所使用的网络请求库，不管你是使用浏览器原生的 `fetch` 还是 第三方库比如 `axios`，都没有问题。



**译者注**：几年前接触react的时候，找到的方法，就和文章里的一样，本来以为会有其他更高大上的东东的……



​         ———时2019年4月5日周五下午 17:27 竣工于帝都霍营龙跃苑