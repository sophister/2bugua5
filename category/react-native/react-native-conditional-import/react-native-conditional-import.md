# React Native 打production包过滤掉测试代码



**先说结论**：可以通过 `React Native` 中提供的 `__DEV__` 变量，在产出 `production` 环境的代码包时，过滤掉一些测试的页面。

**环境说明**：

```shell
"react": "16.8.3",
"react-native": "0.59.3",
```

## 问题

我们的RN代码目录大概是这样：

```shell
./
├── common    公共代码目录
├── entry     
├── index.js  
├── page      页面代码目录
```

在我们的RN代码里，有一些公共组件，比如 `Button` `LoadingDialog` 之类的，放置在 `common` 目录下。组件开发完之后，一般会在页面中，增加一个对应的测试页面，比如要测试 `LoadingDialog`，一般会增加一个 `page/test/loadingTest.tsx` 文件，在这个页面里，可以测试下组件的各种功能是否OK。

在 `page/index.ts` 里，之前会有工具，自动扫描 `page/**/**.tsx` 生成所有的页面信息，当然里面也会包含很多在开发中用到的测试页面。

这个 `page/index.ts`的内容如下：

```javascript
import accountBindCardBindCard from './account/bindCard/BindCard';
import testMainMain from './test/main/Main';
import testMarqueeMarquee from './test/marquee/Marquee';
import testStyleTestLoading from './test/styleTest/Loading';
import testStyleTestModal from './test/styleTest/Modal';
import testStyleTestTextAnimation from './test/styleTest/TextAnimation';
import testStyleTestTextTest from './test/styleTest/TextTest';
import testStyleTestWithdrawResult from './test/styleTest/WithdrawResult';
import uplanInvestTypeInvestType from './uplan/investType/InvestType';
import uplanJoinJoin from './uplan/join/Join';

export default {
    'account/bindCard/BindCard': accountBindCardBindCard,
    'Main': testMainMain,
  	 // 下面这些 test 目录下的页面，都是测试页面
    'test/marquee/Marquee': testMarqueeMarquee,
    'test/styleTest/Loading': testStyleTestLoading,
    'test/styleTest/Modal': testStyleTestModal,
    'test/styleTest/TextAnimation': testStyleTestTextAnimation,
    'test/styleTest/TextTest': testStyleTestTextTest,
    'test/styleTest/WithdrawResult': testStyleTestWithdrawResult,
     // 目标就是，在打production包的时候，干掉这些测试页面，减小包大小
    'uplan/investType/InvestType': uplanInvestTypeInvestType,
    'uplan/join/Join': uplanJoinJoin,
};
```

之前的处理方式，是在打包时候，调用一个自己封装的打包JS脚本，这个脚本会先扫描 `page` 目录，如果当前是 `production` 模式，即 `--dev false` ，那么在生成上面的 `page/index.ts` 文件内容时，会过滤掉 `page/test` 目录下的页面。

我们准备在下一次RN升级的时候，干掉自己开发的JS脚本，改回到直接调用RN官方的打包脚本。

因此，需要有一种方式，能够在 `development` 和 `production` 环境下，允许我们有选择地加载某些页面。



## 使用 `__DEV__ `

很容易想到，RN官方就提供了一个全局变量，`__DEV__` 来表明当前是否production模式。

想法很简单，不能直接在 `page/index.ts` 里 `import` 测试页面了，需要根据 `__DEV__` 来决定是否去import。但是在实际测试过程中，还是发现了一些问题，最后总结下来，我大概测试了以下几种情况：

```javascript
export default {
    get 'account/bindCard/BindCard'() { return require('./account/bindCard/BindCard').default; },
    // 下面是几种写法
    get 'test/story/StoryDisplay'() {
        if (__DEV__) {
            return require('./test/story/StoryDisplay').default;
        }
        return null;
    },
    get 'test/styleTest/Loading'() {
        if (__DEV__ !== true) {
            return null;
        }
        return require('./test/styleTest/Loading').default;
    },
    get 'test/styleTest/Modal'() {
        if (!__DEV__) {
            return null;
        }
        return require('./test/styleTest/Modal').default;
    },
    get 'test/styleTest/TextAnimation'() {
        if (__DEV__) {
            return require('./test/styleTest/TextAnimation').default;
        } else {
            return null;
        }
    },
    get 'test/styleTest/TextTest'() {
        if (!__DEV__) {
            return null;
        } else {
            return require('./test/styleTest/TextTest').default;
        }
    },
    // 上面是测试几种写法的实际效果
    get 'test/styleTest/WithdrawResult'() { if (__DEV__) { return require('./test/styleTest/WithdrawResult').default; } return null; },
    get 'uplan/investType/InvestType'() { return require('./uplan/investType/InvestType').default; },
    get 'uplan/join/Join'() { return require('./uplan/join/Join').default; },
};

```

其实仔细看下来，就是两种写法，`require` 是写在 `if` 内部还是外面。

先来看下 `development` 模式下的bundle代码，访问本地URL `http://localhost:8081/index.bundle?platform=android&dev=true&minify=false` :

```javascript
    get 'test/story/StoryDisplay'() {
      if (__DEV__) {
        return _$$_REQUIRE(_dependencyMap[58], "./test/story/StoryDisplay").default;
      }
      return null;
    },
      
    get 'test/styleTest/Loading'() {
      if (__DEV__ !== true) {
        return null;
      }
      return _$$_REQUIRE(_dependencyMap[59], "./test/styleTest/Loading").default;
    },

    get 'test/styleTest/Modal'() {
      if (!__DEV__) {
        return null;
      }
      return _$$_REQUIRE(_dependencyMap[60], "./test/styleTest/Modal").default;
    },

    get 'test/styleTest/TextAnimation'() {
      if (__DEV__) {
        return _$$_REQUIRE(_dependencyMap[61], "./test/styleTest/TextAnimation").default;
      } else {
        return null;
      }
    },

    get 'test/styleTest/TextTest'() {
      if (!__DEV__) {
        return null;
      } else {
        return _$$_REQUIRE(_dependencyMap[62], "./test/styleTest/TextTest").default;
      }
    },

```

可以看出，在 `dev=true` 模式下，所有的测试页面都被RN打包了，上面几种源码的写法，产出没什么差别。

下面再看看 `production` 模式的产出bundle，访问URL `http://localhost:8081/index.bundle?platform=android&dev=false&minify=false`：

```javascript
    get 'test/story/StoryDisplay'() {
      return null;
    },

    get 'test/styleTest/Loading'() {
      {
        return null;
      }
      return _$$_REQUIRE(_dependencyMap[57]).default;
    },

    get 'test/styleTest/Modal'() {
      {
        return null;
      }
      return _$$_REQUIRE(_dependencyMap[58]).default;
    },

    get 'test/styleTest/TextAnimation'() {
      {
        return null;
      }
    },
      
    get 'test/styleTest/TextTest'() {
      {
        return null;
      }
    },

```

Hmm, interesting🤔，可以明显看出，有的写法最终打包的结果，并不是我们意料之中的。

显然，上面的 `test/styleTest/Loading` 和 `test/styleTest/Modal` 是 **有问题** 的。这时候在bundle里搜索这两个页面的代码，可以发现这两个页面代码已经被打包到了bundle里(废话，都可以看到 `_$$_REQUIRE(_dependencyMap[58]).default`  这样的字眼了…… ) 

仔细对比这两种有问题的代码，和其他几种写法，很容易发现，有问题的代码， `require` 都 **没有** 放到 `if/else` 的分支里。 

## 结论







