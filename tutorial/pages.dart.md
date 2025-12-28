# `pages.dart` 文件详解

## 概述

`pages.dart` 文件定义了 Flutter 中用于创建全屏页面路由的核心类。该文件提供了两种创建页面路由的方式：

1. **`PageRoute<T>`**：抽象基类，定义了全屏模态路由的基本行为
2. **`PageRouteBuilder<T>`**：具体实现类，通过回调函数的方式创建页面路由，无需定义新的子类

这些类继承自 `ModalRoute<T>`，用于创建替换整个屏幕的页面路由，是 Flutter 导航系统的重要组成部分。

## 文件结构

```dart 1:11:packages/flutter/lib/src/widgets/pages.dart
// Copyright 2014 The Flutter Authors. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file.

/// @docImport 'navigator.dart';
library;

import 'basic.dart';
import 'framework.dart';
import 'routes.dart';
```

文件导入了以下模块：

- `basic.dart`：基础 widget 类
- `framework.dart`：Flutter 框架核心类
- `routes.dart`：路由相关类（包括 `ModalRoute`、`TransitionRoute` 等）

## `PageRoute<T>` 类

### 类定义

```dart 12:23:packages/flutter/lib/src/widgets/pages.dart
/// A modal route that replaces the entire screen.
///
/// The [PageRouteBuilder] subclass provides a way to create a [PageRoute] using
/// callbacks rather than by defining a new class via subclassing.
///
/// If `barrierDismissible` is true, then pressing the escape key on the keyboard
/// will cause the current route to be popped with null as the value.
///
/// See also:
///
///  * [Route], which documents the meaning of the `T` generic type argument.
abstract class PageRoute<T> extends ModalRoute<T> {
```

`PageRoute<T>` 是一个抽象类，继承自 `ModalRoute<T>`。它表示一个替换整个屏幕的模态路由。

**关键特性**：

- **全屏替换**：与 `PopupRoute`（弹出式路由，如对话框）不同，`PageRoute` 会替换整个屏幕
- **泛型类型 `T`**：表示路由返回值的类型，用于 `Navigator.pop<T>(context, result)` 中的结果传递
- **抽象类**：不能直接实例化，需要通过子类（如 `PageRouteBuilder` 或 `MaterialPageRoute`）来使用

### 构造函数

```dart 24:33:packages/flutter/lib/src/widgets/pages.dart
  /// Creates a modal route that replaces the entire screen.
  PageRoute({
    super.settings,
    super.requestFocus,
    super.traversalEdgeBehavior,
    super.directionalTraversalEdgeBehavior,
    this.fullscreenDialog = false,
    this.allowSnapshotting = true,
    bool barrierDismissible = false,
  }) : _barrierDismissible = barrierDismissible;
```

构造函数参数说明：

- `super.settings`：路由设置信息（`RouteSettings`），包含路由名称和参数
- `super.requestFocus`：是否请求焦点
- `super.traversalEdgeBehavior`：焦点遍历边缘行为
- `super.directionalTraversalEdgeBehavior`：方向性焦点遍历边缘行为
- `fullscreenDialog`：默认为 `false`，表示是否为全屏对话框
- `allowSnapshotting`：默认为 `true`，表示是否允许快照（用于应用切换时的截图）
- `barrierDismissible`：默认为 `false`，表示是否可以通过点击遮罩层关闭路由

### 主要属性

#### fullscreenDialog

```dart 35:44:packages/flutter/lib/src/widgets/pages.dart
  /// {@template flutter.widgets.PageRoute.fullscreenDialog}
  /// Whether this page route is a full-screen dialog.
  ///
  /// In Material and Cupertino, being fullscreen has the effects of making
  /// the app bars have a close button instead of a back button. On
  /// iOS, dialogs transitions animate differently and are also not closeable
  /// with the back swipe gesture.
  /// {@endtemplate}
  @override
  final bool fullscreenDialog;
```

当 `fullscreenDialog` 为 `true` 时，表示这是一个全屏对话框。在 Material 和 Cupertino 设计中，这会产生以下效果：

- **应用栏变化**：应用栏会显示关闭按钮而不是返回按钮
- **iOS 特殊处理**：在 iOS 上，对话框的过渡动画会有所不同，并且不能通过返回滑动手势关闭

#### allowSnapshotting

```dart 46:47:packages/flutter/lib/src/widgets/pages.dart
  @override
  final bool allowSnapshotting;
```

控制是否允许系统在应用切换到后台时对该路由进行快照。这对于敏感页面（如支付页面）很有用，可以防止在应用切换器中显示敏感内容。

#### opaque

```dart 49:50:packages/flutter/lib/src/widgets/pages.dart
  @override
  bool get opaque => true;
```

`opaque` 属性始终返回 `true`，表示 `PageRoute` 是完全不透明的，会完全遮挡底层内容。这与 `PopupRoute` 不同，后者是半透明的。

#### barrierDismissible

```dart 52:54:packages/flutter/lib/src/widgets/pages.dart
  @override
  bool get barrierDismissible => _barrierDismissible;
  final bool _barrierDismissible;
```

控制是否可以通过点击遮罩层（barrier）来关闭路由。对于 `PageRoute`，这个属性通常为 `false`，因为页面路由不应该通过点击外部区域来关闭。

**注意**：如果 `barrierDismissible` 为 `true`，按键盘上的 Escape 键会导致当前路由被弹出，返回值为 `null`。

### 核心方法

#### canTransitionTo

```dart 56:57:packages/flutter/lib/src/widgets/pages.dart
  @override
  bool canTransitionTo(TransitionRoute<dynamic> nextRoute) => nextRoute is PageRoute;
```

判断是否可以过渡到下一个路由。只有当 `nextRoute` 也是 `PageRoute` 类型时，才允许过渡。这确保了页面路由只能与其他页面路由进行过渡，而不能与弹出式路由（如对话框）进行过渡。

#### canTransitionFrom

```dart 59:60:packages/flutter/lib/src/widgets/pages.dart
  @override
  bool canTransitionFrom(TransitionRoute<dynamic> previousRoute) => previousRoute is PageRoute;
```

判断是否可以从上一个路由过渡过来。只有当 `previousRoute` 也是 `PageRoute` 类型时，才允许过渡。

#### popGestureEnabled

```dart 62:66:packages/flutter/lib/src/widgets/pages.dart
  @override
  bool get popGestureEnabled {
    // Fullscreen dialogs aren't dismissible by back swipe.
    return !fullscreenDialog && super.popGestureEnabled;
  }
```

控制是否启用返回手势（iOS 上的侧滑返回）。如果 `fullscreenDialog` 为 `true`，则禁用返回手势，因为全屏对话框不应该通过滑动手势关闭。

## `PageRouteBuilder<T>` 类

### 类定义

```dart 78:89:packages/flutter/lib/src/widgets/pages.dart
/// A utility class for defining one-off page routes in terms of callbacks.
///
/// Callers must define the [pageBuilder] function which creates the route's
/// primary contents. To add transitions define the [transitionsBuilder] function.
///
/// The `T` generic type argument corresponds to the type argument of the
/// created [Route] objects.
///
/// See also:
///
///  * [Route], which documents the meaning of the `T` generic type argument.
class PageRouteBuilder<T> extends PageRoute<T> {
```

`PageRouteBuilder<T>` 是一个实用工具类，用于通过回调函数的方式创建一次性页面路由，而无需定义新的子类。

**设计目的**：

- **简化使用**：不需要为每个页面路由创建新的类
- **灵活性**：通过回调函数自定义页面内容和过渡动画
- **一次性路由**：适合创建临时的、不需要复用的路由

### 构造函数

```dart 90:105:packages/flutter/lib/src/widgets/pages.dart
  /// Creates a route that delegates to builder callbacks.
  PageRouteBuilder({
    super.settings,
    super.requestFocus,
    required this.pageBuilder,
    this.transitionsBuilder = _defaultTransitionsBuilder,
    this.transitionDuration = const Duration(milliseconds: 300),
    this.reverseTransitionDuration = const Duration(milliseconds: 300),
    this.opaque = true,
    this.barrierDismissible = false,
    this.barrierColor,
    this.barrierLabel,
    this.maintainState = true,
    super.fullscreenDialog,
    super.allowSnapshotting = true,
  });
```

构造函数参数说明：

- `super.settings`：路由设置信息
- `super.requestFocus`：是否请求焦点
- `pageBuilder`：**必需参数**，用于构建路由的主要内容
- `transitionsBuilder`：可选，用于构建路由的过渡动画，默认为 `_defaultTransitionsBuilder`（无动画）
- `transitionDuration`：过渡动画持续时间，默认 300 毫秒
- `reverseTransitionDuration`：反向过渡动画持续时间，默认 300 毫秒
- `opaque`：是否不透明，默认为 `true`
- `barrierDismissible`：是否可通过遮罩层关闭，默认为 `false`
- `barrierColor`：遮罩层颜色（可选）
- `barrierLabel`：遮罩层的无障碍标签（可选）
- `maintainState`：是否保持状态，默认为 `true`
- `super.fullscreenDialog`：是否为全屏对话框
- `super.allowSnapshotting`：是否允许快照

### 主要属性

#### pageBuilder

```dart 107:112:packages/flutter/lib/src/widgets/pages.dart
  /// {@template flutter.widgets.pageRouteBuilder.pageBuilder}
  /// Used to build the route's primary contents.
  ///
  /// See [ModalRoute.buildPage] for complete definition of the parameters.
  /// {@endtemplate}
  final RoutePageBuilder pageBuilder;
```

用于构建路由主要内容的回调函数。函数签名如下：

```dart
typedef RoutePageBuilder = Widget Function(
  BuildContext context,
  Animation<double> animation,
  Animation<double> secondaryAnimation,
);
```

参数说明：

- `context`：构建上下文
- `animation`：当前路由的动画对象（从 0.0 到 1.0）
- `secondaryAnimation`：上一个路由的动画对象（从 1.0 到 0.0）

#### transitionsBuilder

```dart 114:121:packages/flutter/lib/src/widgets/pages.dart
  /// {@template flutter.widgets.pageRouteBuilder.transitionsBuilder}
  /// Used to build the route's transitions.
  ///
  /// See [ModalRoute.buildTransitions] for complete definition of the parameters.
  /// {@endtemplate}
  ///
  /// The default transition is a jump cut (i.e. no animation).
  final RouteTransitionsBuilder transitionsBuilder;
```

用于构建路由过渡动画的回调函数。默认情况下使用 `_defaultTransitionsBuilder`，它直接返回子 widget，不添加任何动画（即跳转切换）。

函数签名如下：

```dart
typedef RouteTransitionsBuilder = Widget Function(
  BuildContext context,
  Animation<double> animation,
  Animation<double> secondaryAnimation,
  Widget child,
);
```

#### 其他属性

```dart 123:142:packages/flutter/lib/src/widgets/pages.dart
  @override
  final Duration transitionDuration;

  @override
  final Duration reverseTransitionDuration;

  @override
  final bool opaque;

  @override
  final bool barrierDismissible;

  @override
  final Color? barrierColor;

  @override
  final String? barrierLabel;

  @override
  final bool maintainState;
```

这些属性都是对父类 `PageRoute` 和 `ModalRoute` 中相应属性的重写，用于配置路由的行为。

### 核心方法

#### buildPage

```dart 144:151:packages/flutter/lib/src/widgets/pages.dart
  @override
  Widget buildPage(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
  ) {
    return pageBuilder(context, animation, secondaryAnimation);
  }
```

重写 `ModalRoute.buildPage` 方法，直接调用 `pageBuilder` 回调函数来构建页面内容。

#### buildTransitions

```dart 153:161:packages/flutter/lib/src/widgets/pages.dart
  @override
  Widget buildTransitions(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
    Widget child,
  ) {
    return transitionsBuilder(context, animation, secondaryAnimation, child);
  }
```

重写 `ModalRoute.buildTransitions` 方法，调用 `transitionsBuilder` 回调函数来构建过渡动画。

## 辅助函数

### _defaultTransitionsBuilder

```dart 69:76:packages/flutter/lib/src/widgets/pages.dart
Widget _defaultTransitionsBuilder(
  BuildContext context,
  Animation<double> animation,
  Animation<double> secondaryAnimation,
  Widget child,
) {
  return child;
}
```

默认的过渡构建函数，直接返回子 widget，不添加任何动画效果。这会产生一个"跳转切换"（jump cut）的效果，即页面立即切换，没有过渡动画。

## 使用场景

### 场景 1：创建简单的页面路由

使用 `PageRouteBuilder` 创建简单的页面路由，使用默认的跳转切换动画：

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) {
      return DetailPage();
    },
  ),
);
```

### 场景 2：自定义过渡动画

使用 `PageRouteBuilder` 创建带有自定义过渡动画的页面路由：

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) {
      return DetailPage();
    },
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      // 淡入淡出动画
      return FadeTransition(
        opacity: animation,
        child: child,
      );
    },
    transitionDuration: Duration(milliseconds: 500),
  ),
);
```

### 场景 3：滑动过渡动画

创建带有滑动效果的页面路由：

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) {
      return DetailPage();
    },
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      // 从右侧滑入
      const begin = Offset(1.0, 0.0);
      const end = Offset.zero;
      const curve = Curves.ease;

      var tween = Tween(begin: begin, end: end).chain(
        CurveTween(curve: curve),
      );

      return SlideTransition(
        position: animation.drive(tween),
        child: child,
      );
    },
  ),
);
```

### 场景 4：缩放过渡动画

创建带有缩放效果的页面路由：

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) {
      return DetailPage();
    },
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return ScaleTransition(
        scale: animation,
        child: child,
      );
    },
  ),
);
```

## 与 MaterialPageRoute 的关系

`MaterialPageRoute` 是 `PageRoute` 的一个具体实现，专门用于 Material Design 风格的页面路由。它提供了 Material Design 标准的过渡动画。

```dart
// MaterialPageRoute 内部使用 MaterialRouteTransitionMixin
class MaterialPageRoute<T> extends PageRoute<T> with MaterialRouteTransitionMixin<T> {
  // ...
}
```

**选择建议**：

- **使用 `MaterialPageRoute`**：当需要标准的 Material Design 过渡动画时
- **使用 `PageRouteBuilder`**：当需要自定义过渡动画或特殊效果时
- **创建自定义 `PageRoute` 子类**：当需要创建可复用的、具有特定行为的页面路由时

## 继承关系

```text
Route<T>
  └── TransitionRoute<T>
      └── ModalRoute<T>
          └── PageRoute<T>
              └── PageRouteBuilder<T>
              └── MaterialPageRoute<T> (在 material/page.dart 中)
```

## 相关类型

### RoutePageBuilder

用于构建路由主要内容的回调函数类型：

```dart
typedef RoutePageBuilder = Widget Function(
  BuildContext context,
  Animation<double> animation,
  Animation<double> secondaryAnimation,
);
```

### RouteTransitionsBuilder

用于构建路由过渡动画的回调函数类型：

```dart
typedef RouteTransitionsBuilder = Widget Function(
  BuildContext context,
  Animation<double> animation,
  Animation<double> secondaryAnimation,
  Widget child,
);
```

## 总结

`pages.dart` 文件提供了 Flutter 中创建全屏页面路由的核心类：

1. **`PageRoute<T>`**：抽象基类，定义了全屏模态路由的基本行为和属性
   - 始终不透明（`opaque = true`）
   - 只能与其他 `PageRoute` 进行过渡
   - 支持全屏对话框模式
   - 支持快照控制

2. **`PageRouteBuilder<T>`**：实用工具类，通过回调函数创建页面路由
   - 无需定义新类即可创建路由
   - 支持自定义页面内容和过渡动画
   - 适合创建一次性路由

3. **使用场景**：
   - 标准页面导航：使用 `MaterialPageRoute`
   - 自定义动画：使用 `PageRouteBuilder`
   - 可复用路由：创建自定义 `PageRoute` 子类

这些类共同构成了 Flutter 声明式导航系统的基础，为应用提供了灵活且强大的页面路由能力。
