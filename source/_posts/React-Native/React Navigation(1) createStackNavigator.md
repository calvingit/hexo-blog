---
title: React Navigation(1) createStackNavigator
date: 2018-09-25 23:51:53
category: React-Native
---

# 创建导航器的 API

App 的导航有很多种，`React Navigation`也提供了很多接口来创建各种不同的导航器，这些 API 返回的也是`React Component`，可以直接`export`：

- `createStackNavigator` 堆栈导航器，进入新界面时，不释放旧界面
- `createSwitchNavigator` 切换导航器，进入新界面时，释放旧界面
- `createDrawerNavigator` 左侧边栏导航器
- `createBottomTabNavigator` 底部标签栏导航
- `createMaterialTopTabNavigator` 顶部 MD 风格标签栏导航
- `createMaterialBottomTabNavigator` 底部 MD 风格标签栏导航

下面详细说明 API：

## createStackNavigator

```
createStackNavigator(RouteConfigs, StackNavigatorConfig);
```

栈导航器，类似于 iOS 的 `UINavigationController`。

- **RouteConfigs**

  ```
  createStackNavigator({
  // 每当需要创建一个新 screen 时，可以像这样写一个对象
  Profile: {
  // `ProfileScreen` 是包含整屏内容的 React Component.
  screen: ProfileScreen,
  // 当 `ProfileScreen` 被加载时, StackNavigator 会给它一个 `navigation` 属性.

      // 可选: 从浏览器打开时会被调用:
      path: 'people/:name',
      // 可以从path里面解析动作和参数.

      // 可选: 覆盖了screen的 `navigationOptions`
      navigationOptions: ({ navigation }) => ({
      }),

  },

  ...MyOtherRoutes,
  });
  ```

- **StackNavigatorConfig**
  路由的参数： - `initialRouteName` 默认显示的 screen，必须是 RouteConfigs 里面的定义的 key - `initialRouteParams` 初始化路由的参数 - `initialRouteKey` 初始化路由的可选 id - `navigationOptions` 所有 screen 的默认 navigation options - `paths`

      	视觉效果参数：

      	- `mode` ：转场动画样式，有两种`card`和`modal`
      		- `card` 默认，使用原生的转场动画
      		- `modal` 从底部弹出模态效果，只支持iOS
      	- `headerMode` ：
      		- `float` 固定单个导航栏，切换screen的时候有动画，iOS 通用模式。
      		- `screen` 每个screen有自己的导航栏，Android 通用模式
      		- `none` 不显示导航栏
      	- `headerBackTitleVisible` 导航栏是否显示返回按钮标题
      	- `headerTransitionPreset` 导航栏动画
      		- `fade-in-place` 默认，淡入淡出效果，类似于Twitter、Instagram、Facebook
      		- `uikit` iOS 的默认效果
      	- `headerLayoutPreset` 如何布局导航栏组件
      		- `left` 标题放在左边，Android默认。在iOS里面会隐藏返回按钮标题。
      		- `center` 标题居中，iOS里默认
      	- `cardStyle` 覆盖或扩展默认的样式
      	- `transitionConfig` 一个返回合并了默认的转场设置的对象的函数，函数接收三个参数：
      		- `transitionProps` 新的screen的转场属性
      		- `prevTransitionProps` 前一个screen的转场属性
      		- `isModal` 是否模态
      	- `onTransitionStart` 函数，转场动画开始时调用
      	- `onTransitionEnd` 函数，转场动画结束时调用

- **navigationOptions** - `title` 字符串，`headerTitle`的备选。当在 TabNavigator 使用是当做`tabBarLabel`，当在 DrawerNavigator 里面使用时当做`drawerLabel` - `header` React 元素或者是一个接受`HeaderProps`参数并返回 React 元素的函数。当设为`null`时，隐藏导航栏 - `headTitle` 字符串或者 React 组件，默认设置到`title`。当传入组件时，它接受三个属性：`allowFontScaling`, `style` 和 `children`。标题字符串在`children`里面。 - headerTitleAllowFontScaling 标题字体是否自适应，默认`true` - headerBackImage 自定义返回按钮图片的 React 元素或组件。当作为组件时，它接受很多属性（`tintColor`, title）。默认是`Image`组件，图片是`react-navigation/views/assets/back-icon.png`。 - `headerBackTitle` 返回按钮的标题，传`null`禁用标题，默认是前一个 screen 的`headerTitle`。 需要注意的是，`headerBackTitle`在 A 里面定义，但是在下一个 screen B 里面显示。
  `StackNavigator({ A: { screen: AScreen, navigationOptions: () => ({ headerBackTitle: null }), }, B: { screen: BScreen, navigationOptions: () => ({ }), } });` - `headerTruncatedBackTitle` 当`headerBackTitle`显示不了的时候，用这个参数替换，默认是`Back` - `headerRight` 显示在导航栏右边的元素，类似于 iOS 的`UIBarButtonItem`的设置 - `headerLeft` 显示在导航栏左边的元素或组件，当做组件时，接收几个属性：`onPress`, `title`, `titleStyle`等，具体查看`Header.js` - `headerStyle` 导航栏的样式对象 - `headerForceInset` 传递一个`forceInset`对象给 SafeAreaView 内部 - `headerTitleStyle` 标题组件的样式对象 - `headerBackTitleStyle` 返回标题的样式 - `headerLeftContainerStyle` 自定义`headerLeft`组件容器的样式，比如添加`padding` - `headerRightContainerStyle` 自定义`headerRight`组件容器的样式，比如添加`padding` - `headerTitleContainerStyle` 自定义`headerTitle`组件容器的样式，比如添加`padding` - `headerTintColor` 导航栏的 tint color - `headerPressColorAndroid` MD 波纹颜色，支持 Android >= 5.0 - `headerTransparent` 是否透明，默认 false。当为`true`时，导航栏不再有背景，除非你明确的设置`headerStyle`或`headerBackground` - `headerBackground` 这个组件和`headerTransparent`一起使用，用来渲染导航栏的背景 - `gesturesEnabled` 是否使用手势来 dimiss screen，iOS 默认 true，Android 默认 false。 - `gestureResponseDistance` 包含两个属性的对象：`horizontal` 水平方向距离，默认 25；`vertical` 垂直方向距离，默认 135。

**Navigator 属性**

- `screenProps` 这个属性会 传递给子 screen，子 screen 通过`this.props.screenProps`调用
  `const SomeStack = createStackNavigator({ // config }); <SomeStack screenProps={/* this prop will get passed to the screen components as this.props.screenProps */} />`
