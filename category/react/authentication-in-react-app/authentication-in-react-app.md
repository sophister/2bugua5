# [译]React中的用户认证(登录态管理)



**原文地址** ： [https://kentcdodds.com/blog/authentication-in-react-applications](https://kentcdodds.com/blog/authentication-in-react-applications) 

本文主要展示在当下 `React` 应用开发中，怎么使用 `Context` 和 `Hooks` 来管理用户的认证(也就是登录态)。



## 先说结论

下面是本文最终要实现的的简化版，方便大佬们直接看最后的效果：

```javascript
import React from 'react'
import {useUser} from './context/auth'
import AuthenticatedApp from './authenticated-app'
import UnauthenticatedApp from './unauthenticated-app'
function App() {
  const user = useUser()
  return user ? <AuthenticatedApp /> : <UnauthenticatedApp />
}
export App
```

嗯，最终的代码大概就长这样。[大多数](https://twitter.com/kentcdodds/status/1131184429169168387) 需要进行用户认证管理的应用，都可以使用类似上面的逻辑来管理用户登录状态。当用户访问我们应用中的某个需要登录后才能访问的页面时，我们可以将用户重定向到登陆页，大多数情况下也是这样做的，除此之外，我们还可以不进行跳转，直接在该页面上展示未登录用户看到的界面。为了提高用户体验，我们也可以这样：

```javascript
import React from 'react'
import {useUser} from './context/auth'
const AuthenticatedApp = React.lazy(() => import('./authenticated-app'))
const UnauthenticatedApp = React.lazy(() => import('./unauthenticated-app'))
function App() {
  const user = useUser()
  return user ? <AuthenticatedApp /> : <UnauthenticatedApp />
}
export App
```

亲，代码 *懒加载* 就实现了：未登录用户访问我们页面，只会加载渲染未登录界面的代码；已登录用户访问同一个页面，同样只会加载渲染已登录界面的代码。

具体在 `AuthenticatedApp` 和 `UnauthenticatedApp` 里渲染什么界面，完全是开发者来决定。或许他们会渲染一些 `router` ，甚至会复用一些公共组件。无论具体渲染什么，我们都不用关心用户的登录态了，因为在渲染这两个组件之一的时候，已经明确知道了用户的登录状态。



## 怎么一步步实现上面的逻辑

如果你想看看最终整个APP的实现，可以到 [这个github](https://github.com/kentcdodds/bookshelf) 查看。

OK，我们接下来看看怎么一步步来实现上面的逻辑。首先，我们看下我们应用的入口代码：

```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import App from './app'
import AppProviders from './context'
ReactDOM.render(
  <AppProviders>
    <App />
  </AppProviders>,
  document.getElementById('root'),
)
```

下面是 `AppProviders`组件的代码：

```javascript
import React from 'react'
import {AuthProvider} from './auth-context'
import {UserProvider} from './user-context'
function AppProviders({children}) {
  return (
    <AuthProvider>
      <UserProvider>{children}</UserProvider>
    </AuthProvider>
  )
}
export default AppProviders
```

OK，我们有两个 `Provider`： 一个是维护应用的认证状态；另一个是维护当前登录用户的数据。按照字面意思，`AppProvider` 负责初始化整个APP的数据(如果在`localStorage`中存在用户认证后的`token`，那么我们可以直接从`token`中获取一些用户数据)。另一方面，`UserProvider`负责将我们对用户数据的一些修改(比如email地址、履历等)保持和服务器端的同步。

[完整的auth-context.js](https://github.com/kentcdodds/bookshelf/blob/69bde2c117660bd988ffbc60f387165d2f852c62/src/context/auth-context.js) 包含一些和本文主题无关的逻辑，因此下面我们来看下简化版的 `auth-context.js` ：

```javascript
import React from 'react'
import {FullPageSpinner} from '../components/lib'
const AuthContext = React.createContext()
function AuthProvider(props) {
  // 如果我们还不确定当前用户是否登录，比如还在请求后端接口查询登录状态，
  // 那么我们就渲染一个全局的加载中，而不是加载真正的页面组件
  if (weAreStillWaitingToGetTheUserData) {
    return <FullPageSpinner />
  }
  const login = () => {} // make a login request
  const register = () => {} // register the user
  const logout = () => {} // clear the token in localStorage and the user data
 
  // 注意：这里我并没有使用 `React.useMemo` 来优化 provider 的 `value`。
  // 因为这是我们应用里最顶级的组件，很少会在这个顶级组件上触发 重新render
  return (
    <AuthContext.Provider value={{data, login, logout, register}} {...props} />
  )
}
const useAuth = () => React.useContext(AuthContext)
export {AuthProvider, useAuth}
// user-context.js 文件里的 `UserProvider` 大概长这样：
// const UserProvider = props => (
//   <UserContext.Provider value={useAuth().data.user} {...props} />
// )
// and the useUser hook is basically this:
// const useUser = () => React.useContext(UserContext)
```

简化我们应用里的认证管理的关键点是：

**负责维护用户登录态的组件，在获取到当前用户的登录状态之前，不会渲染页面的主体内容，可以渲染一个加载中的全局loading。只有当从服务器端获取到用户的登录状态之后，才去渲染页面的主体：已登录就渲染登录后的组件；未登录渲染未登录的。** 



## 结论

在实际开发中，很多APP面临的场景都是不同的。如果你采用了服务端渲染技术(`SSR`)，那么你很可能不需要一个加载中的loading，因为你在服务端已经明确知道了，当前用户是否已登录。即使在这个场景下，同样可以将用户的登录状态提出到全局的 `Provider` 来管理，这样会增强我们代码的可维护性。



**PS** ：

有些同学问到了相同的问题：如果我们的应用，已登录用户和未登录用户看到的很多界面都相同(比如twitter)，而不是像gmail那样，已登录用户和未登录用户看到的完全不同，我们应该怎么来维护用户登录状态呢?

如果是这种情况，那么代码里很多组件，都会用到 `useUser`这个hook，为了能在一个公共组件中，根据是否登录而执行不同逻辑。因为我们的公共组件可能值关心用户是否登录，你甚至可以再封装出一个 `useIsAuthenticated` hook，返回一个 `boolean` 值，表示当前用户是否已登录。多亏了 `Context ` 和 `Hooks`，要实现这样的逻辑非常的简单。





​      ———时2019年5月26日下午 18:18 竣工于帝都霍营