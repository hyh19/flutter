# ModalRoute 类详解 - 第五部分：委托过渡与生命周期方法

## 概述

`ModalRoute` 提供了委托过渡机制，允许路由之间协调过渡动画。同时，它还实现了生命周期方法，用于管理路由的安装、推入和添加等操作。

## 委托过渡机制

### delegatedTransition

```dart 1577:1610:packages/flutter/lib/src/widgets/routes.dart
  /// The [DelegatedTransitionBuilder] provided to the route below this one in the
  /// navigation stack.
  ///
  /// {@template flutter.widgets.delegatedTransition}
  /// Used for the purposes of coordinating transitions between two routes with
  /// different route transitions. When a route is added to the stack, the original
  /// topmost route will look for this transition, and if available, it will use
  /// the `delegatedTransition` from the incoming transition to animate off the
  /// screen.
  ///
  /// If the return of the [DelegatedTransitionBuilder] is null, then by default
  /// the original transition of the routes will be used. This is useful if a
  /// route can conditionally provide a transition based on the [BuildContext].
  /// {@endtemplate}
  ///
  /// The [ModalRoute] receiving this transition will set it to their
  /// [receivedTransition] property.
  ///
  /// {@tool dartpad}
  /// This sample shows an app that uses three different page transitions, a
  /// Material Zoom transition, the standard Cupertino sliding transition, and a
  /// custom vertical transition. All of the page routes are able to inform the
  /// previous page how to transition off the screen to sync with the new page.
  ///
  /// ** See code in examples/api/lib/widgets/routes/flexible_route_transitions.0.dart **
  /// {@end-tool}
  ///
  /// {@tool dartpad}
  /// This sample shows an app that uses the same transitions as the previous
  /// sample, this time in a [MaterialApp.router].
  ///
  /// ** See code in examples/api/lib/widgets/routes/flexible_route_transitions.1.dart **
  /// {@end-tool}
  DelegatedTransitionBuilder? get delegatedTransition => null;
```

**功能**：提供委托过渡构建器，用于告诉下方的路由如何过渡退出屏幕。

**使用场景**：当两个路由使用不同的过渡动画时，新路由可以告诉旧路由如何过渡，以实现协调的动画效果。

### receivedTransition

```dart 1612:1622:packages/flutter/lib/src/widgets/routes.dart
  /// The [DelegatedTransitionBuilder] received from the route above this one in
  /// the navigation stack.
  ///
  /// {@macro flutter.widgets.delegatedTransition}
  ///
  /// The `receivedTransition` will use the above route's [delegatedTransition] in
  /// order to show the right route transition when the above route either enters
  /// or leaves the navigation stack. If not null, the `receivedTransition` will
  /// wrap the route content.
  @visibleForTesting
  DelegatedTransitionBuilder? receivedTransition;
```

**功能**：接收来自上层路由的委托过渡构建器。

**机制**：当上层路由提供了 `delegatedTransition` 时，当前路由会将其存储在 `receivedTransition` 中，用于包装自己的过渡内容。

### _buildFlexibleTransitions

```dart 1624:1657:packages/flutter/lib/src/widgets/routes.dart
  // Wraps the transitions of this route with a DelegatedTransitionBuilder, when
  // _receivedTransition is not null.
  Widget _buildFlexibleTransitions(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
    Widget child,
  ) {
    if (receivedTransition == null || secondaryAnimation.isDismissed) {
      return buildTransitions(context, animation, secondaryAnimation, child);
    }

    // Create a static proxy animation to suppress the original secondary transition.
    final ProxyAnimation proxyAnimation = ProxyAnimation();

    final Widget proxiedOriginalTransitions = buildTransitions(
      context,
      animation,
      proxyAnimation,
      child,
    );

    // If receivedTransitions return null, then we want to return the original transitions,
    // but with the secondary animation still proxied. This keeps a desynched
    // animation from playing.
    return receivedTransition!(
          context,
          animation,
          secondaryAnimation,
          allowSnapshotting,
          proxiedOriginalTransitions,
        ) ??
        proxiedOriginalTransitions;
  }
```

**功能**：当 `receivedTransition` 不为 `null` 时，使用委托过渡构建器包装路由的过渡效果。

**实现逻辑**：

1. **条件检查**：如果 `receivedTransition` 为 `null` 或 `secondaryAnimation` 已被解除，直接返回 `buildTransitions` 的结果

2. **创建代理动画**：创建一个静态的 `ProxyAnimation` 来抑制原始的次动画

3. **构建原始过渡**：使用代理动画调用 `buildTransitions`，得到原始的过渡效果

4. **应用委托过渡**：
   - 如果 `receivedTransition` 返回 `null`，返回原始过渡（但次动画已被代理）
   - 否则返回委托过渡构建器的结果

**设计考虑**：这种设计允许路由在保持自己过渡效果的同时，还能响应上层路由的委托过渡需求。

## 生命周期方法

### install

```dart 1659:1664:packages/flutter/lib/src/widgets/routes.dart
  @override
  void install() {
    super.install();
    _animationProxy = ProxyAnimation(super.animation);
    _secondaryAnimationProxy = ProxyAnimation(super.secondaryAnimation);
  }
```

**功能**：安装路由时调用，初始化动画代理。

**实现**：

- 调用父类的 `install()` 方法
- 创建 `_animationProxy` 和 `_secondaryAnimationProxy`，用于管理动画状态

**为什么需要代理**：代理动画允许 `ModalRoute` 在动画运行时动态改变动画的父级（例如在 `offstage` 改变时），而不会影响原始的动画对象。

### didPush

```dart 1666:1672:packages/flutter/lib/src/widgets/routes.dart
  @override
  TickerFuture didPush() {
    if (_scopeKey.currentState != null && navigator!.widget.requestFocus) {
      navigator!.focusNode.enclosingScope?.setFirstFocus(_scopeKey.currentState!.focusScopeNode);
    }
    return super.didPush();
  }
```

**功能**：当路由被推入导航堆栈时调用。

**实现逻辑**：

- 如果 `_scopeKey.currentState` 不为 `null`（路由的 scope 已构建）且 `navigator.requestFocus` 为 `true`，则设置焦点到路由的 `focusScopeNode`
- 调用父类的 `didPush()` 方法

**焦点管理**：这确保了在路由被推入时，焦点会被正确设置到路由的焦点范围内。

### didAdd

```dart 1674:1680:packages/flutter/lib/src/widgets/routes.dart
  @override
  void didAdd() {
    if (_scopeKey.currentState != null && navigator!.widget.requestFocus) {
      navigator!.focusNode.enclosingScope?.setFirstFocus(_scopeKey.currentState!.focusScopeNode);
    }
    super.didAdd();
  }
```

**功能**：当路由被添加到导航堆栈时调用（与 `didPush` 的区别在于，`didAdd` 通常用于初始路由，不需要动画）。

**实现逻辑**：与 `didPush` 类似，设置焦点并调用父类方法。

**区别**：

- `didPush`：用于推入的路由（可能需要动画）
- `didAdd`：用于添加的路由（通常是初始路由，不需要动画）

## 委托过渡使用示例

虽然默认实现返回 `null`，但子类可以重写 `delegatedTransition` 来提供自定义的委托过渡：

```dart
class CustomRoute extends ModalRoute<void> {
  @override
  DelegatedTransitionBuilder? get delegatedTransition {
    return (BuildContext context, Animation<double> animation,
        Animation<double> secondaryAnimation, bool allowSnapshotting,
        Widget child) {
      // 返回自定义的过渡效果，用于协调下方路由的退出动画
      return FadeTransition(
        opacity: secondaryAnimation,
        child: child,
      );
    };
  }

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      body: Center(child: Text('Custom Route')),
    );
  }

  // ... 其他必需的方法实现
}
```

## 生命周期调用顺序

当路由被推入导航堆栈时，生命周期方法的调用顺序如下：

1. **install()**：路由安装，初始化动画代理
2. **didPush()**：路由被推入，设置焦点
3. **buildPage()**：构建页面内容（首次调用）
4. **buildTransitions()**：构建过渡效果（多次调用）

## 总结

第五部分介绍了 `ModalRoute` 的委托过渡机制和生命周期方法：

1. **委托过渡机制**：
   - `delegatedTransition`：提供委托过渡构建器，告诉下方路由如何过渡
   - `receivedTransition`：接收来自上层路由的委托过渡
   - `_buildFlexibleTransitions`：应用委托过渡的实现逻辑

2. **生命周期方法**：
   - `install`：初始化动画代理
   - `didPush`：路由推入时的处理，设置焦点
   - `didAdd`：路由添加时的处理，设置焦点

这些机制和方法为路由之间的协调过渡和生命周期管理提供了完整的支持。
