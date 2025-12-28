# ModalRoute buildPage与buildTransitions详解

## 概述

`ModalRoute` 提供了两个关键的方法供子类重写：`buildPage` 和 `buildTransitions`。这两个方法分别负责构建路由的内容和过渡动画，是 `ModalRoute` 架构的核心组成部分。

## buildPage 方法

```dart 1407:1436:packages/flutter/lib/src/widgets/routes.dart
  /// Override this method to build the primary content of this route.
  ///
  /// The arguments have the following meanings:
  ///
  ///  * `context`: The context in which the route is being built.
  ///  * [animation]: The animation for this route's transition. When entering,
  ///    the animation runs forward from 0.0 to 1.0. When exiting, this animation
  ///    runs backwards from 1.0 to 0.0.
  ///  * [secondaryAnimation]: The animation for the route being pushed on top of
  ///    this route. This animation lets this route coordinate with the entrance
  ///    and exit transition of routes pushed on top of this route.
  ///
  /// This method is only called when the route is first built, and rarely
  /// thereafter. In particular, it is not automatically called again when the
  /// route's state changes unless it uses [ModalRoute.of]. For a builder that
  /// is called every time the route's state changes, consider
  /// [buildTransitions]. For widgets that change their behavior when the
  /// route's state changes, consider [ModalRoute.of] to obtain a reference to
  /// the route; this will cause the widget to be rebuilt each time the route
  /// changes state.
  ///
  /// In general, [buildPage] should be used to build the page contents, and
  /// [buildTransitions] for the widgets that change as the page is brought in
  /// and out of view. Avoid using [buildTransitions] for content that never
  /// changes; building such content once from [buildPage] is more efficient.
  Widget buildPage(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
  );
```

### 方法签名

`buildPage` 是一个抽象方法，子类必须实现它。它接收三个参数：

- `context`：构建上下文
- `animation`：路由的过渡动画，进入时从 0.0 到 1.0，退出时从 1.0 到 0.0
- `secondaryAnimation`：上层路由的动画，用于协调与上层路由的过渡

### 调用时机

**重要特性**：

1. **首次构建时调用**：`buildPage` 只在路由首次构建时调用
2. **很少重新调用**：之后很少再次调用，除非路由完全重建
3. **不会自动响应状态变化**：当路由状态改变时，不会自动调用 `buildPage`

### 设计原则

**性能优化**：

- `buildPage` 应该用于构建**不经常变化**的页面内容
- 对于需要响应状态变化的内容，应该：
  - 在 `buildPage` 返回的 widget 中使用 `ModalRoute.of` 来获取路由引用
  - 使用 `buildTransitions` 来处理需要频繁更新的过渡效果

### 使用示例

```dart
class MyModalRoute extends ModalRoute<void> {
  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    // 构建页面的主要内容，这部分内容通常不会频繁变化
    return Scaffold(
      appBar: AppBar(
        title: Text('My Page'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Static Content'),
            // 如果需要响应路由状态，使用 ModalRoute.of
            Builder(
              builder: (context) {
                final route = ModalRoute.of(context);
                return Text('Can Pop: ${route?.canPop ?? false}');
              },
            ),
          ],
        ),
      ),
    );
  }

  // ... 其他必需的方法实现
}
```

## buildTransitions 方法

```dart 1438:1575:packages/flutter/lib/src/widgets/routes.dart
  /// Override this method to wrap the [child] with one or more transition
  /// widgets that define how the route arrives on and leaves the screen.
  ///
  /// By default, the child (which contains the widget returned by [buildPage])
  /// is not wrapped in any transition widgets.
  ///
  /// The [buildTransitions] method, in contrast to [buildPage], is called each
  /// time the [Route]'s state changes while it is visible (e.g. if the value of
  /// [canPop] changes on the active route).
  ///
  /// The [buildTransitions] method is typically used to define transitions
  /// that animate the new topmost route's comings and goings. When the
  /// [Navigator] pushes a route on the top of its stack, the new route's
  /// primary [animation] runs from 0.0 to 1.0. When the Navigator pops the
  /// topmost route, e.g. because the use pressed the back button, the
  /// primary animation runs from 1.0 to 0.0.
  ///
  /// {@tool snippet}
  /// The following example uses the primary animation to drive a
  /// [SlideTransition] that translates the top of the new route vertically
  /// from the bottom of the screen when it is pushed on the Navigator's
  /// stack. When the route is popped the SlideTransition translates the
  /// route from the top of the screen back to the bottom.
  ///
  /// We've used [PageRouteBuilder] to demonstrate the [buildTransitions] method
  /// here. The body of an override of the [buildTransitions] method would be
  /// defined in the same way.
  ///
  /// ```dart
  /// PageRouteBuilder<void>(
  ///   pageBuilder: (BuildContext context,
  ///       Animation<double> animation,
  ///       Animation<double> secondaryAnimation,
  ///   ) {
  ///     return Scaffold(
  ///       appBar: AppBar(title: const Text('Hello')),
  ///       body: const Center(
  ///         child: Text('Hello World'),
  ///       ),
  ///     );
  ///   },
  ///   transitionsBuilder: (
  ///       BuildContext context,
  ///       Animation<double> animation,
  ///       Animation<double> secondaryAnimation,
  ///       Widget child,
  ///    ) {
  ///     return SlideTransition(
  ///       position: Tween<Offset>(
  ///         begin: const Offset(0.0, 1.0),
  ///         end: Offset.zero,
  ///       ).animate(animation),
  ///       child: child, // child is the value returned by pageBuilder
  ///     );
  ///   },
  /// )
  /// ```
  /// {@end-tool}
  ///
  /// When the [Navigator] pushes a route on the top of its stack, the
  /// [secondaryAnimation] can be used to define how the route that was on
  /// the top of the stack leaves the screen. Similarly when the topmost route
  /// is popped, the secondaryAnimation can be used to define how the route
  /// below it reappears on the screen. When the Navigator pushes a new route
  /// on the top of its stack, the old topmost route's secondaryAnimation
  /// runs from 0.0 to 1.0. When the Navigator pops the topmost route, the
  /// secondaryAnimation for the route below it runs from 1.0 to 0.0.
  ///
  /// {@tool snippet}
  /// The example below adds a transition that's driven by the
  /// [secondaryAnimation]. When this route disappears because a new route has
  /// been pushed on top of it, it translates in the opposite direction of
  /// the new route. Likewise when the route is exposed because the topmost
  /// route has been popped off.
  ///
  /// ```dart
  /// PageRouteBuilder<void>(
  ///   pageBuilder: (BuildContext context,
  ///       Animation<double> animation,
  ///       Animation<double> secondaryAnimation,
  ///   ) {
  ///     return Scaffold(
  ///       appBar: AppBar(title: const Text('Hello')),
  ///       body: const Center(
  ///         child: Text('Hello World'),
  ///       ),
  ///     );
  ///   },
  ///   transitionsBuilder: (
  ///       BuildContext context,
  ///       Animation<double> animation,
  ///       Animation<double> secondaryAnimation,
  ///       Widget child,
  ///    ) {
  ///     return SlideTransition(
  ///       position: Tween<Offset>(
  ///         begin: const Offset(0.0, 1.0),
  ///         end: Offset.zero,
  ///       ).animate(animation),
  ///       child: SlideTransition(
  ///         position: Tween<Offset>(
  ///           begin: Offset.zero,
  ///           end: const Offset(0.0, 1.0),
  ///         ).animate(secondaryAnimation),
  ///         child: child,
  ///       ),
  ///      );
  ///   },
  /// )
  /// ```
  /// {@end-tool}
  ///
  /// In practice the `secondaryAnimation` is used pretty rarely.
  ///
  /// The arguments to this method are as follows:
  ///
  ///  * `context`: The context in which the route is being built.
  ///  * [animation]: When the [Navigator] pushes a route on the top of its stack,
  ///    the new route's primary [animation] runs from 0.0 to 1.0. When the [Navigator]
  ///    pops the topmost route this animation runs from 1.0 to 0.0.
  ///  * [secondaryAnimation]: When the Navigator pushes a new route
  ///    on the top of its stack, the old topmost route's [secondaryAnimation]
  ///    runs from 0.0 to 1.0. When the [Navigator] pops the topmost route, the
  ///    [secondaryAnimation] for the route below it runs from 1.0 to 0.0.
  ///  * `child`, the page contents, as returned by [buildPage].
  ///
  /// See also:
  ///
  ///  * [buildPage], which is used to describe the actual contents of the page,
  ///    and whose result is passed to the `child` argument of this method.
  Widget buildTransitions(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
    Widget child,
  ) {
    return child;
  }
```

### 方法签名

`buildTransitions` 有一个默认实现（直接返回 `child`），子类可以重写它来自定义过渡效果。它接收四个参数：

- `context`：构建上下文
- `animation`：主动画，推入路由时从 0.0 到 1.0，弹出时从 1.0 到 0.0
- `secondaryAnimation`：次动画，用于协调与相邻路由的过渡
- `child`：由 `buildPage` 返回的 widget

### 调用时机

**关键特性**：

1. **频繁调用**：与 `buildPage` 不同，`buildTransitions` 在路由可见时，每当路由状态改变时都会被调用（例如 `canPop` 改变时）
2. **状态响应**：可以响应路由状态的动态变化

### animation 参数

**主动画（animation）**：

- **推入路由时**：从 0.0 运行到 1.0
- **弹出路由时**：从 1.0 运行到 0.0
- **用途**：控制当前路由的进入和退出动画

### secondaryAnimation 参数

**次动画（secondaryAnimation）**：

- **推入新路由时**：旧路由的 `secondaryAnimation` 从 0.0 运行到 1.0
- **弹出当前路由时**：下方路由的 `secondaryAnimation` 从 1.0 运行到 0.0
- **用途**：协调与相邻路由的过渡效果
- **注意**：在实际使用中很少用到

### 使用示例

#### 示例 1：基本的滑动过渡

```dart
class SlideRoute extends ModalRoute<void> {
  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      appBar: AppBar(title: Text('Slide Route')),
      body: Center(child: Text('Hello World')),
    );
  }

  @override
  Widget buildTransitions(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation, Widget child) {
    return SlideTransition(
      position: Tween<Offset>(
        begin: const Offset(0.0, 1.0), // 从底部开始
        end: Offset.zero, // 移动到正常位置
      ).animate(animation),
      child: child,
    );
  }

  // ... 其他必需的方法实现
}
```

#### 示例 2：淡入淡出过渡

```dart
class FadeRoute extends ModalRoute<void> {
  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      appBar: AppBar(title: Text('Fade Route')),
      body: Center(child: Text('Fading In')),
    );
  }

  @override
  Widget buildTransitions(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation, Widget child) {
    return FadeTransition(
      opacity: animation,
      child: child,
    );
  }

  // ... 其他必需的方法实现
}
```

#### 示例 3：缩放过渡

```dart
class ScaleRoute extends ModalRoute<void> {
  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      appBar: AppBar(title: Text('Scale Route')),
      body: Center(child: Text('Scaling')),
    );
  }

  @override
  Widget buildTransitions(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation, Widget child) {
    return ScaleTransition(
      scale: Tween<double>(
        begin: 0.0,
        end: 1.0,
      ).animate(CurvedAnimation(
        parent: animation,
        curve: Curves.easeOut,
      )),
      child: child,
    );
  }

  // ... 其他必需的方法实现
}
```

#### 示例 4：组合过渡（使用 secondaryAnimation）

```dart
class CombinedRoute extends ModalRoute<void> {
  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      appBar: AppBar(title: Text('Combined Route')),
      body: Center(child: Text('Combined Animation')),
    );
  }

  @override
  Widget buildTransitions(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation, Widget child) {
    return SlideTransition(
      position: Tween<Offset>(
        begin: const Offset(0.0, 1.0),
        end: Offset.zero,
      ).animate(animation),
      child: SlideTransition(
        position: Tween<Offset>(
          begin: Offset.zero,
          end: const Offset(0.0, -1.0), // 向上滑出
        ).animate(secondaryAnimation),
        child: child,
      ),
    );
  }

  // ... 其他必需的方法实现
}
```

## buildPage 与 buildTransitions 的区别

### 对比表

| 特性 | buildPage | buildTransitions |
| --- | --- | --- |
| **调用频率** | 首次构建时调用，很少重新调用 | 路由状态改变时频繁调用 |
| **用途** | 构建页面内容（静态部分） | 构建过渡动画（动态部分） |
| **性能** | 适合构建复杂但不变的内容 | 适合构建简单但频繁更新的动画 |
| **参数** | 接收 animation 和 secondaryAnimation | 接收 animation、secondaryAnimation 和 child |
| **返回值** | 返回页面内容的 Widget | 返回包装了 child 的过渡 Widget |

### 最佳实践

1. **职责分离**：
   - `buildPage` 负责构建页面内容
   - `buildTransitions` 负责添加过渡效果

2. **性能优化**：
   - 在 `buildPage` 中构建不经常变化的内容
   - 在 `buildTransitions` 中只添加必要的过渡 Widget

3. **避免重复构建**：
   - 不要在 `buildTransitions` 中构建复杂的内容
   - 使用 `child` 参数（即 `buildPage` 的返回值）

## 总结

第四部分详细介绍了 `ModalRoute` 的两个核心构建方法：

1. **buildPage**：用于构建路由的主要内容，只在首次构建时调用，适合构建静态内容

2. **buildTransitions**：用于包装内容并添加过渡效果，在路由状态改变时频繁调用，适合构建动画效果

这两个方法的设计体现了 Flutter 框架对性能的优化考虑：将静态内容的构建与动态过渡效果的构建分离，避免不必要的重复构建。
