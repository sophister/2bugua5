# [译]5个技巧：避免React Hooks 常见问题



**原文**： [https://kentcdodds.com/blog/react-hooks-pitfalls](https://kentcdodds.com/blog/react-hooks-pitfalls) 

在这篇文章里，我们来探索下 React Hooks 的常见问题，以及怎么来避免这些问题。

[React Hooks](https://reactjs.org/hooks) 是在 [2018年10月提出](https://www.youtube.com/watch?v=dpw9EHDh2bM) ，并且在2019年2月 [发布](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html) 。自从 React Hooks 发布以后，很多开发者都在项目中使用了Hooks，因为Hooks确实在很大程度上简化了我们对组件 `state` 和 `副作用` 的管理。

毫无疑问，React Hooks 目前是react生态中一个热点，越来越多的开发者以及开源库，都引入了Hooks。尽管React Hooks 现在收到追捧，但是它的引入，也需要开发者改变自己对于组件生命周期、state以及副作用的思考方式；如果你没有很好的理解 React Hooks，盲目的使用它将会给你带来一些意想不到的bug。OK，接下来我们就来看看使用Hooks可能有哪些坑，以及怎么改变我们的思考方式来避免这些坑。



## 问题1：在理解之前就急于使用Hooks

[React Hooks官方文档](https://reactjs.org/hooks) 写得非常详尽，我强烈建议你在使用 Hooks 之前，把官方文档 **通读** 一遍，尤其是 [FAQ](https://reactjs.org/docs/hooks-faq.html) 部分，里面包含了很多实际开发中会遇到的问题及解决办法。给你自己一两个小时，通读一下官方文档吧，这将对你理解 Hooks 有很大的帮助，并且在将来的实际开发中帮你节省很多(找bug和改bug的)时间。

与此同时，建议你也看一看 [Sophie, Dan 和 Ryan介绍Hooks的分享](https://www.youtube.com/watch?v=dpw9EHDh2bM) 。

第一个问题的解决办法：**仔细研读官方文档以及FAQ** **📚**



## 问题2：不使用(或忽视)React Hooks的ESLint插件

在 React Hooks发布的同时，[eslint-plugin-react-hooks](https://github.com/facebook/react/tree/master/packages/eslint-plugin-react-hooks) 这个 `ESLint `的插件也发布了。这个插件包含两个校验规则：`rules of hooks` 和 `exhaustive deps`。默认的推荐配置，是将 `rules of hooks`设置为 `error`级别，将 `exhaustive deps`设置为 `warning`级别。

我强烈建议你安装、使用并遵守这两条规则。它不仅仅能帮你发现容易被忽略的bug，在这个过程中，还能教你一些代码和hooks的知识，当然了，还有它提供的代码自动修复功能，超cool。

在我和很多开发者交流中，发现很多人开发者对 `exhaustive deps` 这条规则感到困惑。因此，我写了一个简单的demo，来展示如果忽略了这条规则，将会导致什么bug。

假设我们有2个页面：一个是 狗狗🐶 列表页 `List` ，展示一系列狗狗的名字；一个是某一只狗狗的详情页`Detail` 。在列表页上，点击某个狗狗的名字，就会打开对应狗狗的详情页。

OK，在狗狗详情页上，我们有一个展示狗狗详情的组件 `DogInfo`，它接收狗狗的 `dogId`，并且根据 `dogId` 请求API获取对应的详情：

```javascript
function DogInfo({dogId}) {
  const [dog, setDog] = useState(null)
  // imagine you also have loading/error states. omitting to save space...
  useEffect(() => {
    getDog(dogId).then(d => setDog(d))
  }, []) // 😱
  return <div>{/* render the dog info here */}</div>
}
```

上面的代码里，我们 `useEffect` 的依赖列表是一个空数组，因为我们只希望在组件 `mount` 的时候才去发起一次请求。到目前为止，这段代码没什么问题。现在，假设我们的狗狗详情页UI有了一点改动，增加了一个 “相关狗狗”的列表。这时我们的代码就有bug了，点击 “相关狗狗” 列表中的某一只，我们的 `DogInfo` 组件并不会更新到对应的狗狗详情，尽管 `DogInfo` 组件已经重新 `render` 了。

现在的情况是，点击 “相关狗狗” 列表中的某一项，触发了详情页的重新 `render`，并且会把点击的狗狗 `dogId` 传给 `DogInfo` ，但是由于我们在 `DogInfo` 的 `useEffect` 依赖项中，写的空数组，导致这个 `useEffect` 不会重新执行。

嗯，下面是修改之后的代码：

```javascript
function DogInfo({dogId}) {
  const [dog, setDog] = useState(null)
  // imagine you also have loading/error states. omitting to save space...
  useEffect(() => {
    getDog(dogId).then(d => setDog(d))
  }, [dogId]) // ✅
  return <div>{/* render the dog info here */}</div>
}
```

通过这个栗子，我们可以得出这个关键结论：如果 `useEffect` 的某个依赖项真的永远不会改变，那把这个依赖项添加到 `uesEffect` 的依赖数组里，也没有任何问题。同时，如果你认为某个依赖项不会改变，但实际上这个依赖项却变了，这正好帮助你发现了代码里的bug。

和这个例子相比，还有很多其他的场景更难辨别和分析，比如，你在 `useEffect` 里调用了某个函数(函数定义在 useEffect 外面)，但是却 **没有** 在依赖项里添加这个函数，那么代码很可能有bug了。相信我，每次在我忽略了这条规则之后，我都会后悔当初为什么没有遵守规则。

请注意，受限于 `ESLint` 在代码静态分析的一些限制，这条规则 (`exhaustive deps`)有时候不能正确的分析出你代码中的问题。可能这就是它为什么默认设置级别是 `warning` 而不是 `error` 的原因吧。当它不会正确的分析你的代码时，它会给出你一些warning信息，在这种情况下，我建议你稍微重构下你的代码，来保证能正确的被分析。如果重构代码之后，依然不能被正确的分析，那么可能局部的关闭这条规则，也是一个办法吧，为了能继续coding而不至于延期。

第二个问题的解决办法：**安装、使用并且遵守 ESLint ** 。



## 问题3：(错误地)从组件生命周期角度来思考Hooks

在 React Hooks 出现之前，我们可以在类组件里，通过内置的组件生命周期方法，来告诉react，什么时候，它应该做什么操作：

```javascript
class LifecycleComponent extends React.Component {
  constructor() {
    // initialize component instance
  }
  componentDidMount() {
    // run this code when the component is first added to the page
  }
  componentDidUpdate(prevProps, prevState) {
    // run this code when the component is updated on the page
  }
  componentWillUnmount() {
    // run this code when the component is removed from the page
  }
  render() {
    // call me anytime you need some react elements...
  }
}
```

在 Hooks 发布之后，像上面这些写类组件同样没问题(在可预见的将来，这样写也没有任何问题)，这种类组件的方式已经存在了许多年。Hooks 带来了一系列的好处，其中我最喜欢的一个好处是(`useEffect`)，Hooks 使得组件更加的符合**声明式** 语法。有了 Hooks，我们可以不用去分辨“某个操作应该在组件的哪一个生命周期执行”，而是更加直观的告诉 React，“当哪些变化发生时，我希望执行对应的操作”。

因此，现在我们的代码长这样：

```javascript
function HookComponent() {
  React.useEffect(() => {
    // This side effect code is here to synchronize the state of the world
    // with the state of this component.
    return function cleanup() {
      // And I need to cleanup the previous side-effect before running a new one
    }
    // So I need this side-effect and it's cleanup to be re-run...
  }, [when, any, ofThese, change])
  React.useEffect(() => {
    // this side effect will re-run on every single time this component is
    // re-rendered to make sure that what it does is never stale.
  })
  React.useEffect(() => {
    // this side effect can never get stale because
    // it legitimately has no dependencies
  }, [])
  return /* some beautiful react elements */
}
```

Ryan Florence [从另一个角度来解释思考方式上的变化](https://twitter.com/ryanflorence/status/1125040005477425152) 。

我喜欢这个特性(useEffect)的一个主要原因是，它帮助我避免了很多bug。在过去基于类组件的开发过程中，我发现引入bug的很多情形，都是我忘记了在 `componentDidUpdate` 里处理某个 `prop` 或者 `state` 的变化；另一种情况是，我在 `componentDidUpdate` 里处理了 `prop` 或 `state` 的变化，但是却忘记了取消掉上一次变化所引起的副作用。举个栗子，你发起了某次 HTTP 请求，但是在 HTTP 完成之前，组件的某个 prop 或 state 发生了变化，那么你通常应该取消掉这个HTTP请求。

在使用 React Hooks 的场景下，你仍然需要思考你的副作用在什么时机执行，但是你不用再纠结副作用是在哪个生命周期里执行，你思考的是，怎么保持副作用的结果和组件的状态同步。要理解这个点，需要付出一些努力，但是你一旦理解到了，你将避免很多的bug。

因此，你可以给 `useEffect` 的依赖项设置为 **空数组** 的惟一理由，是它确实没有依赖任何外部变量，而 **不是** 你认为这个副作用只需要在组件mount的时候执行一次。

第三个问题解决办法：**不要以组件生命周期的方式来思考 React Hooks，应该是思考，如果让你的副作用和组件状态保持一致** 



## 问题4：过于担心性能

一些开发者看到下面的代码时，他们吓坏了：

```javascript
function MyComponent() {
  function handleClick() {
    console.log('clicked some other component')
  }
  return <SomeOtherComponent onClick={handleClick} />
}
```

他们通常因为下面2个原因而感到担忧：

1. 我们在 `MyComponent` 内部定义了函数`handleClick`，这意味着，每次 `MyComponent` render时，都会重新定义一个不同的`handleClick`
2. 每次render，我们都将一个新的`handleClick`传给了 `SomeOtherComponent`，这意味着我们不能通过 `React.memo`，`React.PureComponent` 或者 `shouldComponentUpdate` 来优化 `SomeOtherComponent`  的性能，这会引起`SomeOtherComponent`许多不必要的重新render

针对第一个问题，JavaScript引擎（即使是在很低端的手机上）定义一个新函数的执行是非常快的。你基本上不会遇到由于重复定义函数而导致你的APP性能低下。

第二个问题来讲，不必要的重复render，也不是一定会引起性能问题。仅仅因为组件重新render了，并不代表实际的DOM会被修改，通常修改DOM才是慢的地方。React 在性能优化方面做的非常好，通常你没有必要为了提升性能去引入一些额外的工作。

如果这些额外的重复render导致你的APP慢，首先应该明确为什么重复render会这么慢。如果一次render本身都很慢，导致额外的重复render引起APP卡顿，那么即使你避免了额外的重复render，你很可能仍然面临性能问题。当你修复了导致render慢的原因之后，你或许会发现，那些重复的render也不会引起APP卡顿了。

如果你真的确认，是额外的重复render导致了APP性能问题，那么你可以使用 React 内置的一些性能优化 API，比如 `React.memo`，`React.useMemo`以及 `React.useCallback`。你可以从我的这篇博客，了解到 [useMemo和useCallback](https://kentcdodds.com/blog/usememo-and-usecallback) 。注意：有的时候，你采取了性能优化措施之后，你的APP反而更卡顿了……因此，务必在性能优化的前后，做好性能检测对比。

同时记住，[production版本的React性能比development版本高很多](https://reactjs.org/docs/optimizing-performance.html#use-the-production-build) 。

第四个问题解决办法：**记住一点，React本来就执行很快，不要过早的担心或者优化你的性能** 。



## 问题5：对Hooks的测试过于重视

我注意到有些开发者担心，如果他们讲组件迁移到 React Hooks，他们需要重写对应的所有测试代码。根据你的测试代码实现方式，这个担忧可能有道理，也可能没有道理。

引用我自己文章 [使用React Hooks，测试代码怎么办？](https://kentcdodds.com/blog/react-hooks-whats-going-to-happen-to-my-tests) ，如果你的测试代码长这样：

```javascript
test('setOpenIndex sets the open index state properly', () => {
  // using enzyme
  const wrapper = mount(<Accordion items={[]} />)
  expect(wrapper.state('openIndex')).toBe(0)
  wrapper.instance().setOpenIndex(1)
  expect(wrapper.state('openIndex')).toBe(1)
})
```

如果是这种情况，那么你正好借着重写测试代码的机会，去优化这些测试代码。毫无疑问，你应该废弃掉上面这样的代码，改成下面这样的：

```javascript
test('can open accordion items to see the contents', () => {
  const hats = {title: 'Favorite Hats', contents: 'Fedoras are classy'}
  const footware = {
    title: 'Favorite Footware',
    contents: 'Flipflops are the best',
  }
  const items = [hats, footware]
  // using React Testing Library
  const {getByText, queryByText} = render(<Accordion items={items} />)
  expect(getByText(hats.contents)).toBeInTheDocument()
  expect(queryByText(footware.contents)).toBeNull()
  fireEvent.click(getByText(footware.title))
  expect(getByText(footware.contents)).toBeInTheDocument()
  expect(queryByText(hats.contents)).toBeNull()
})
```

这两份测试代码的关键区别是，旧的代码是在测试 **组件的具体实现**，新的测试却不是这样。不管组件是基于类的实现方式，还是基于 Hooks 的方式，都是组件内部的具体实现细节。因此，如果你的测试代码，会放到到被测试组件的一些具体实现细节，(比如 `.state()` 或者 `.instance()`)，那么将组件重构为 Hooks 版本，确实会让你的测试代码失效。

但是使用你组件的开发者，是不关心你的组件是基于类实现，还是基于 Hooks 实现。他们只关心你的组件能够正确的实现业务逻辑，或者说只关心你的组件渲染到屏幕上的内容。因此，如果你的测试代码，是检查组件渲染到屏幕上的内容，那么不管你的组件是基于类还是Hooks实现，都不影响测试代码的运行。

你可以从这两篇文章了解更多关于测试方面的内容：[对实现细节的测试](https://kentcdodds.com/blog/testing-implementation-details) 和 [Avoid the Test User](https://kentcdodds.com/blog/avoid-the-test-user) 。

OK，解决这个问题的办法是：**避免去测试组件的实现细节** 。



## 总结



说了这么多，总结起来就是下面这些建议，帮你避免常见的 Hooks 问题的解决办法：

1. 仔细研读官方 Hooks 文档，以及 FAQ 部分
2. 安装、使用并且遵守 [eslint-plugin-react-hooks](https://github.com/facebook/react/tree/master/packages/eslint-plugin-react-hooks) 这个 ESLint 插件
3. 忘掉组件生命周期的思考方式吧。正确的姿势：怎么保持副作用和组件状态的同步。
4. React 本身执行很快，在过早的性能优化之前，一定做好相关的知识点调研
5. 避免测试组件的实现细节，应该关注组件的输入和输出







