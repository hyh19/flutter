# ModalRoute 类详解 - 第八部分：内部状态管理

## 概述

`ModalRoute` 维护了多个内部状态属性，用于管理路由的显示状态、动画代理和子树上下文。这些状态对于路由的正常运行至关重要。

## offstage

```dart 1914:1940:packages/flutter/lib/src/widgets/routes.dart
  /// Whether this route is currently offstage.
  ///
  /// On the first frame of a route's entrance transition, the route is built
  /// [Offstage] using an animation progress of 1.0. The route is invisible and
  /// non-interactive, but each widget has its final size and position. This
  /// mechanism lets the [HeroController] determine the final local of any hero
  /// widgets being animated as part of the transition.
  ///
  /// The modal barrier, if any, is not rendered if [offstage] is true (see
  /// [barrierColor]).
  ///
  /// Whenever this changes value, [changedInternalState] is called.
  bool get offstage => _offstage;
  bool _offstage = false;
  set offstage(bool value) {
    if (_offstage == value) {
      return;
    }
    setState(() {
      _offstage = value;
    });
    _animationProxy!.parent = _offstage ? kAlwaysCompleteAnimation : super.animation;
    _secondaryAnimationProxy!.parent = _offstage
        ? kAlwaysDismissedAnimation
        : super.secondaryAnimation;
    changedInternalState();
  }
```

**功能**：指示路由当前是否处于 offstage（舞台外）状态。

### offstage 的作用

**Hero 动画支持**：

在路由进入过渡的第一帧，路由使用动画进度 1.0 以 `Offstage` 方式构建。路由不可见且不可交互，但每个 widget 都有其最终大小和位置。这种机制让 `HeroController` 能够确定作为过渡一部分的 hero widgets 的最终位置。

**屏障渲染**：

如果 `offstage` 为 `true`，模态屏障不会被渲染（参见 `barrierColor`）。

### setter 实现

当 `offstage` 值改变时：

1. **状态更新**：调用 `setState` 更新 `_offstage` 值

2. **动画代理更新**：
   - 如果 `offstage` 为 `true`：将 `_animationProxy` 的父级设置为 `kAlwaysCompleteAnimation`（总是完成的动画）
   - 如果 `offstage` 为 `false`：将 `_animationProxy` 的父级设置为 `super.animation`（实际的动画）

3. **次动画代理更新**：
   - 如果 `offstage` 为 `true`：将 `_secondaryAnimationProxy` 的父级设置为 `kAlwaysDismissedAnimation`（总是解除的动画）
   - 如果 `offstage` 为 `false`：将 `_secondaryAnimationProxy` 的父级设置为 `super.secondaryAnimation`（实际的次动画）

4. **通知状态改变**：调用 `changedInternalState()` 通知相关组件状态已改变

**设计原因**：当路由处于 offstage 状态时，使用常量动画可以避免动画更新带来的性能开销，同时保持 Hero 动画所需的布局信息。

## subtreeContext

```dart 1942:1943:packages/flutter/lib/src/widgets/routes.dart
  /// The build context for the subtree containing the primary content of this route.
  BuildContext? get subtreeContext => _subtreeKey.currentContext;
```

**功能**：返回包含路由主要内容的子树的构建上下文。

**实现**：通过 `_subtreeKey.currentContext` 获取。`_subtreeKey` 是一个 `GlobalKey`，用于定位路由子树的根 widget。

**使用场景**：用于在路由外部访问路由子树中的信息，例如发送通知。

## animation 与 secondaryAnimation

```dart 1945:1951:packages/flutter/lib/src/widgets/routes.dart
  @override
  Animation<double>? get animation => _animationProxy;
  ProxyAnimation? _animationProxy;

  @override
  Animation<double>? get secondaryAnimation => _secondaryAnimationProxy;
  ProxyAnimation? _secondaryAnimationProxy;
```

**功能**：提供动画的代理访问。

### 为什么使用代理动画？

**动态父级切换**：

代理动画允许在运行时动态改变动画的父级，而不影响原始的动画对象。这在以下场景中非常有用：

1. **offstage 处理**：当路由进入 offstage 状态时，可以切换到常量动画
2. **动画控制**：可以在不直接修改原始动画的情况下控制动画行为
3. **生命周期管理**：在路由的不同生命周期阶段使用不同的动画源

### 初始化

代理动画在 `install()` 方法中初始化：

```dart
@override
void install() {
  super.install();
  _animationProxy = ProxyAnimation(super.animation);
  _secondaryAnimationProxy = ProxyAnimation(super.secondaryAnimation);
}
```

## 内部字段

### _scopeKey

```dart 2244:2244:packages/flutter/lib/src/widgets/routes.dart
  final GlobalKey<_ModalScopeState<T>> _scopeKey = GlobalKey<_ModalScopeState<T>>();
```

**功能**：用于定位 `_ModalScope` 的 `State` 对象的全局键。

**用途**：

- 访问 `_ModalScopeState` 的方法和属性
- 在 `setState` 中调用 `_routeSetState`
- 在生命周期方法中设置焦点

### _subtreeKey

```dart 2245:2245:packages/flutter/lib/src/widgets/routes.dart
  final GlobalKey _subtreeKey = GlobalKey();
```

**功能**：用于定位路由子树的全局键。

**用途**：获取 `subtreeContext`，用于在路由外部访问路由子树。

### _storageBucket

```dart 2246:2246:packages/flutter/lib/src/widgets/routes.dart
  final PageStorageBucket _storageBucket = PageStorageBucket();
```

**功能**：页面存储桶，用于在路由之间保存和恢复状态。

**用途**：当路由被暂时移除然后恢复时，可以保存和恢复 scroll position 等状态。

## 状态管理流程

### offstage 状态转换

```text
路由进入
    ↓
offstage = true (第一帧，用于 Hero 动画定位)
    ↓
offstage = false (正常显示)
    ↓
路由退出
    ↓
offstage = true (准备移除)
```

### 动画代理切换

```text
offstage = false
    ↓
_animationProxy.parent = super.animation
_secondaryAnimationProxy.parent = super.secondaryAnimation
    ↓
正常动画运行
    ↓
offstage = true
    ↓
_animationProxy.parent = kAlwaysCompleteAnimation
_secondaryAnimationProxy.parent = kAlwaysDismissedAnimation
    ↓
使用常量动画（性能优化）
```

## 使用示例

虽然这些内部状态通常由框架自动管理，但了解它们有助于理解路由的工作原理：

```dart
class CustomRoute extends ModalRoute<void> {
  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    // 使用 animation 代理（通过 getter 访问）
    final currentAnimation = this.animation;

    return Scaffold(
      body: Center(
        child: Text('Animation: ${currentAnimation?.value ?? 0.0}'),
      ),
    );
  }

  // 如果需要访问子树上下文
  void someMethod() {
    final context = subtreeContext;
    if (context != null) {
      // 可以使用 context 进行某些操作
      // 例如：发送通知到子树
      Notification.dispatch(context, SomeNotification());
    }
  }

  // ... 其他必需的方法实现
}
```

## 性能优化考虑

### 1. offstage 优化

当路由处于 offstage 状态时，使用常量动画避免了动画值的计算和更新，提高了性能。

### 2. 代理动画的灵活性

代理动画允许在不需要直接修改原始动画的情况下切换动画源，这提供了更好的封装和灵活性。

### 3. 全局键的使用

使用 `GlobalKey` 允许在路由的不同部分之间进行高效的通信和状态共享。

## 总结

第八部分介绍了 `ModalRoute` 的内部状态管理：

1. **offstage**：控制路由是否处于舞台外状态，用于 Hero 动画支持和性能优化

2. **subtreeContext**：提供路由子树的构建上下文，用于外部访问

3. **animation 和 secondaryAnimation**：通过代理动画提供对动画的访问，支持动态父级切换

4. **内部字段**：`_scopeKey`、`_subtreeKey` 和 `_storageBucket` 用于状态管理和定位

这些内部状态共同确保了路由的正确显示、动画运行和生命周期管理。
