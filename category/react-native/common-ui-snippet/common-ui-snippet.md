# React Native 常见UI代码实现

为了统一编码习惯，增强 `React Native`代码可维护性，收集一些常见的UI样式，统一提供实现方案。

## <Text> 常见样式

相关链接:

* [https://snack.expo.io/@sophister/text-code-snippet](https://snack.expo.io/@sophister/text-code-snippet)
* [nested Text vertical align not working](https://stackoverflow.com/a/50469114/2583885)

1. 单行文字垂直居中

![单行文字垂直居中](./assets/text-single-line-vertical.png)

经常会有单行文字垂直居中的需求，文字全部同一种样式，**不存在嵌套**，基本上，默认给 `Text` 设置 `lineHeight` 即可，比如这样：

```javascript
const style = {
    lineHeight: 40,
    fontSize: xxx,
};
<Text style={style}>some single line text to vertical align</Text>
```

如果遇到在 `Android` 下，文字顶部间距过大，可以使用 `View` 来嵌套 `Text` 去除掉顶部间距，像这样：

```javascript
const styles = {
    wrap: {
        height: 40,
        flexDirection: 'column',
        justifyContent: 'center',
    },
    text: {
        fontSize: xxx,
        includeFontPadding: false,
        textAlignVertical: 'center',
    }
};
<View style={styles.wrap}>
    <Text style={styles.text}>some single line text to vertical align</Text>
</View>
```

2. 不同样式的文字，单行内垂直居中

![不同样式的文字，单行内垂直居中](./assets/text-multiple-vertical.png)

上面这种效果，经常能见到(先不考虑图标，只考虑文字的情况)，尝试了 `Text` 嵌套来实现垂直居中，但是效果不甚理想，需要用上 `View` 包含各个 `Text` 来实现：

```javascript
const style2 = {
    wrap: {
    height: 40,
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 15,
    justifyContent: 'space-between',
    borderBottomWidth: 1,
    borderBottomColor: '#ccc',
    backgroundColor: '#fff',
  },
  main: {
    fontSize: 16,
    color: '#000',
  },
  sub: {
    fontSize: 12,
    color: 'orange',
  }
};
<View style={style2.wrap}>
    <Text style={style2.main}>U享自动投标服务</Text>
    <Text style={style2.sub}>岁末限时福利</Text>
</View>
```

如果要考虑右侧的小 icon，实现也简单直接，右侧用 `View` 包起来，作为一个整体来和左侧文字布局：

```javascript
const style3 = {
  wrap: {
    height: 40,
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 15,
    justifyContent: 'space-between',
    borderBottomWidth: 1,
    borderBottomColor: '#ccc',
    backgroundColor: '#fff',
  },
  main: {
    fontSize: 16,
    color: '#000',
  },
  right: {
    alignSelf: 'stretch',
    flexDirection: 'row',
    alignItems: 'center',
  },
  icon: {
    width: 16,
    height: 16,
  },
  sub: {
    fontSize: 12,
    color: 'orange',
    marginLeft: 3,
  }
};
<View style={style3.wrap}>
    <Text style={style3.main}>U享自动投标服务</Text>
    <View style={style3.right}>
        <Image style={style3.icon} source={{uri: 'https://m.we.com/cms/5864b392e0286a10854d8766/dev/loginRedPacket@2x.png'}}></Image>
        <Text style={style3.sub}>岁末限时福利</Text>
    </View>
</View>
```

3. 文字固定在图片某个位置
