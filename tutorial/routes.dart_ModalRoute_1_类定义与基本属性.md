# ModalRoute 类定义与基本属性详解

## 概述

`ModalRoute` 是 Flutter 中一个重要的路由抽象类，用于创建阻塞与之前路由交互的模态路由。它继承自 `TransitionRoute<T>` 并混入了 `LocalHistoryRoute<T>`。

## 类定义

```dart 1231:1243:packages/flutter/lib/src/widgets/routes.dart
/// A route that blocks interaction with previous routes.
///
/// [ModalRoute]s cover the entire [Navigator]. They are not necessarily
/// [opaque], however; for example, a pop-up menu uses a [ModalRoute] but only
/// shows the menu in a small box overlapping the previous route.
///
/// The `T` type argument is the return value of the route. If there is no
/// return value, consider using `void` as the return value.
///
/// See also:
///
///  * [Route], which further documents the meaning of the `T` generic type argument.
abstract class ModalRoute<T> extends TransitionRoute<T> with LocalHistoryRoute<T> {
```

### 关键特性

1. **阻塞交互**：`ModalRoute` 覆盖整个 `Navigator`，阻止用户与之前的路由交互
2. **不一定是完全遮挡**：虽然覆盖整个 Navigator，但不一定是完全不透明的（opaque）。例如，弹出菜单使用 `ModalRoute`，但只在覆盖前一个路由的小框中显示菜单
3. **泛型类型参数**：`T` 类型参数是路由的返回值类型。如果没有返回值，建议使用 `void`

## 构造函数

```dart 1244:1251:packages/flutter/lib/src/widgets/routes.dart
  /// Creates a route that blocks interaction with previous routes.
  ModalRoute({
    super.settings,
    super.requestFocus,
    this.filter,
    this.traversalEdgeBehavior,
    this.directionalTraversalEdgeBehavior,
  });
```

构造函数接收以下参数：

- `super.settings`：路由设置，传递给父类 `TransitionRoute`
- `super.requestFocus`：是否请求焦点，传递给父类
- `this.filter`：应用到模态屏障的滤镜
- `this.traversalEdgeBehavior`：焦点遍历边缘行为
- `this.directionalTraversalEdgeBehavior`：方向性焦点遍历边缘行为

## 基本属性

### filter

```dart 1253:1257:packages/flutter/lib/src/widgets/routes.dart
  /// The filter to add to the barrier.
  ///
  /// If given, this filter will be applied to the modal barrier using
  /// [BackdropFilter]. This allows blur effects, for example.
  final ui.ImageFilter? filter;
```

**用途**：如果提供了滤镜，它将通过 `BackdropFilter` 应用到模态屏障上。这允许实现模糊效果等功能。

**示例应用场景**：

- 在对话框后面添加背景模糊效果
- 实现毛玻璃效果（glassmorphism）

### traversalEdgeBehavior

```dart 1259:1263:packages/flutter/lib/src/widgets/routes.dart
  /// Controls the transfer of focus beyond the first and the last items of a
  /// [FocusScopeNode].
  ///
  /// If set to null, [Navigator.routeTraversalEdgeBehavior] is used.
  final TraversalEdgeBehavior? traversalEdgeBehavior;
```

**用途**：控制焦点超出 `FocusScopeNode` 的第一个和最后一个项目时的转移行为。

**默认行为**：如果为 `null`，则使用 `Navigator.routeTraversalEdgeBehavior`。

### directionalTraversalEdgeBehavior

```dart 1265:1269:packages/flutter/lib/src/widgets/routes.dart
  /// Controls the directional transfer of focus beyond the first and the last
  /// items of a [FocusScopeNode].
  ///
  /// If set to null, [Navigator.routeDirectionalTraversalEdgeBehavior] is used.
  final TraversalEdgeBehavior? directionalTraversalEdgeBehavior;
```

**用途**：控制方向性焦点超出 `FocusScopeNode` 的第一个和最后一个项目时的转移行为。

**默认行为**：如果为 `null`，则使用 `Navigator.routeDirectionalTraversalEdgeBehavior`。

## 继承关系

`ModalRoute` 的类层次结构如下：

```text
Route<T>
  └── TransitionRoute<T>
      └── ModalRoute<T> (with LocalHistoryRoute<T>)
```

### TransitionRoute

`TransitionRoute` 提供了路由过渡动画的基础功能，包括：

- 动画控制器管理
- 过渡动画生命周期
- 动画状态管理

### LocalHistoryRoute

`LocalHistoryRoute` Mixin 提供了本地历史记录功能，允许路由在其生命周期内维护一个本地历史堆栈，这对于实现"返回"功能非常有用。

## 设计模式

`ModalRoute` 使用了以下设计模式：

1. **模板方法模式**：定义了一组子类需要重写的方法（如 `buildPage`、`buildTransitions`）
2. **策略模式**：通过 `filter` 和屏障相关的 getter 方法，允许子类自定义行为
3. **观察者模式**：通过 `InheritedModel` 和 `_ModalScopeStatus` 实现状态通知机制

## 总结

第一部分介绍了 `ModalRoute` 的基本概念、构造函数和基本属性。这些属性为后续的复杂功能（如屏障管理、状态管理、过渡动画等）奠定了基础。
