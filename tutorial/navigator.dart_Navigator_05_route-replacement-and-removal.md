# Navigator 路由替换与移除方法详解

## 功能概述

本文档详细讲解 `Navigator` 类中用于替换和移除路由的静态方法。这些方法提供了更细粒度的路由管理控制，适用于构建非线性用户体验。

## 方法概览

路由替换与移除方法主要分为以下几类：

1. **替换路由**：`replace` / `restorableReplace`
2. **替换下方路由**：`replaceRouteBelow` / `restorableReplaceRouteBelow`
3. **移除路由**：`removeRoute` / `removeRouteBelow`

每个替换方法都有对应的 `restorable` 版本，支持状态恢复功能。

## replace

替换最紧密包围给定 context 的 navigator 上的路由。

```dart 2533:2571:packages/flutter/lib/src/widgets/navigator.dart
  /// Replaces a route on the navigator that most tightly encloses the given
  /// context with a new route.
  ///
  /// {@template flutter.widgets.navigator.replace}
  /// The old route must not be currently visible, as this method skips the
  /// animations and therefore the removal would be jarring if it was visible.
  /// To replace the top-most route, consider [pushReplacement] instead, which
  /// _does_ animate the new route, and delays removing the old route until the
  /// new route has finished animating.
  ///
  /// The removed route is removed and completed with a `null` value.
  ///
  /// The new route, the route below the new route (if any), and the route above
  /// the new route, are all notified (see [Route.didReplace],
  /// [Route.didChangeNext], and [Route.didChangePrevious]). If the [Navigator]
  /// has any [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didReplace]). The removed route is disposed with its
  /// future completed.
  ///
  /// This can be useful in combination with [removeRouteBelow] when building a
  /// non-linear user experience.
  ///
  /// The `T` type argument is the type of the return value of the new route.
  /// {@endtemplate}
  ///
  /// See also:
  ///
  ///  * [replaceRouteBelow], which is the same but identifies the route to be
  ///    removed by reference to the route above it, rather than directly.
  ///  * [restorableReplace], which adds a replacement route that can be
  ///    restored during state restoration.
  @optionalTypeArgs
  static void replace<T extends Object?>(
    BuildContext context, {
    required Route<dynamic> oldRoute,
    required Route<T> newRoute,
  }) {
    return Navigator.of(context).replace<T>(oldRoute: oldRoute, newRoute: newRoute);
  }
```

**关键特性**：

- **旧路由必须不可见**：旧路由当前不能可见，因为此方法跳过动画，如果可见，移除会显得突兀
- **无动画**：此方法不运行动画，直接替换
- **被移除的路由以 null 值完成**：被移除的路由以 null 作为结果完成
- **通知机制**：新路由、新路由下方的路由（如果有）和新路由上方的路由都会被通知
- **非线性用户体验**：与 [removeRouteBelow] 结合使用时，可用于构建非线性用户体验

**与 pushReplacement 的区别**：

- `replace` 不运行动画，旧路由必须不可见
- `pushReplacement` 运行动画，新路由进入时动画，旧路由在新路由完成动画后移除

**使用场景**：

- 在后台替换不可见的路由
- 构建非线性导航体验

## restorableReplace

替换一个可恢复的路由。

```dart 2573:2599:packages/flutter/lib/src/widgets/navigator.dart
  /// Replaces a route on the navigator that most tightly encloses the given
  /// context with a new route.
  ///
  /// {@template flutter.widgets.navigator.restorableReplace}
  /// Unlike [Route]s added via [replace], [Route]s added with this method are
  /// restored during state restoration according to the rules outlined in the
  /// "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.replace}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePush}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  @optionalTypeArgs
  static String restorableReplace<T extends Object?>(
    BuildContext context, {
    required Route<dynamic> oldRoute,
    required RestorableRouteBuilder<T> newRouteBuilder,
    Object? arguments,
  }) {
    return Navigator.of(context).restorableReplace<T>(
      oldRoute: oldRoute,
      newRouteBuilder: newRouteBuilder,
      arguments: arguments,
    );
  }
```

**关键特性**：

- 使用此方法替换的路由可以在状态恢复期间恢复
- 接受 [RestorableRouteBuilder] 作为参数，必须是静态函数
- `arguments` 必须是可通过 [StandardMessageCodec] 序列化的对象
- 返回一个不透明的 ID（String）

## replaceRouteBelow

替换最紧密包围给定 context 的 navigator 上的路由。要替换的路由是给定 `anchorRoute` 下方的路由。

```dart 2601:2637:packages/flutter/lib/src/widgets/navigator.dart
  /// Replaces a route on the navigator that most tightly encloses the given
  /// context with a new route. The route to be replaced is the one below the
  /// given `anchorRoute`.
  ///
  /// {@template flutter.widgets.navigator.replaceRouteBelow}
  /// The old route must not be current visible, as this method skips the
  /// animations and therefore the removal would be jarring if it was visible.
  /// To replace the top-most route, consider [pushReplacement] instead, which
  /// _does_ animate the new route, and delays removing the old route until the
  /// new route has finished animating.
  ///
  /// The removed route is removed and completed with a `null` value.
  ///
  /// The new route, the route below the new route (if any), and the route above
  /// the new route, are all notified (see [Route.didReplace],
  /// [Route.didChangeNext], and [Route.didChangePrevious]). If the [Navigator]
  /// has any [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didReplace]). The removed route is disposed with its
  /// future completed.
  ///
  /// The `T` type argument is the type of the return value of the new route.
  /// {@endtemplate}
  ///
  /// See also:
  ///
  ///  * [replace], which is the same but identifies the route to be removed
  ///    directly.
  ///  * [restorableReplaceRouteBelow], which adds a replacement route that can
  ///    be restored during state restoration.
  @optionalTypeArgs
  static void replaceRouteBelow<T extends Object?>(
    BuildContext context, {
    required Route<dynamic> anchorRoute,
    required Route<T> newRoute,
  }) {
    return Navigator.of(context).replaceRouteBelow<T>(anchorRoute: anchorRoute, newRoute: newRoute);
  }
```

**关键特性**：

- 通过引用上方的路由（`anchorRoute`）来标识要替换的路由，而不是直接引用
- 旧路由必须不可见
- 不运行动画
- 被移除的路由以 null 值完成

**与 replace 的区别**：

- `replace` 直接指定要替换的路由
- `replaceRouteBelow` 通过指定锚点路由来标识要替换的路由（锚点路由下方的路由）

**使用场景**：

- 当你知道锚点路由但不知道具体要替换哪个路由时
- 构建复杂的导航结构

## restorableReplaceRouteBelow

替换一个可恢复的路由（通过锚点路由标识）。

```dart 2639:2666:packages/flutter/lib/src/widgets/navigator.dart
  /// Replaces a route on the navigator that most tightly encloses the given
  /// context with a new route. The route to be replaced is the one below the
  /// given `anchorRoute`.
  ///
  /// {@template flutter.widgets.navigator.restorableReplaceRouteBelow}
  /// Unlike [Route]s added via [restorableReplaceRouteBelow], [Route]s added
  /// with this method are restored during state restoration according to the
  /// rules outlined in the "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.replaceRouteBelow}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePush}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  @optionalTypeArgs
  static String restorableReplaceRouteBelow<T extends Object?>(
    BuildContext context, {
    required Route<dynamic> anchorRoute,
    required RestorableRouteBuilder<T> newRouteBuilder,
    Object? arguments,
  }) {
    return Navigator.of(context).restorableReplaceRouteBelow<T>(
      anchorRoute: anchorRoute,
      newRouteBuilder: newRouteBuilder,
      arguments: arguments,
    );
  }
```

## removeRoute

立即从最紧密包围给定 context 的 navigator 中移除 `route` 并 [Route.dispose] 它。

```dart 2806:2839:packages/flutter/lib/src/widgets/navigator.dart
  /// Immediately remove `route` from the navigator that most tightly encloses
  /// the given context, and [Route.dispose] it.
  ///
  /// {@template flutter.widgets.navigator.removeRoute}
  /// No animations are run as a result of this method call.
  ///
  /// The routes below and above the removed route are notified (see
  /// [Route.didChangeNext] and [Route.didChangePrevious]). If the [Navigator]
  /// has any [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didRemove]). The removed route is disposed with its
  /// future completed.
  ///
  /// The given `route` must be in the history; this method will throw an
  /// exception if it is not.
  ///
  /// If non-null, `result` will be used as the result of the route that is
  /// removed; the future that had been returned from pushing the removed route
  /// will complete with `result`. If provided, must match the type argument of
  /// the class of the popped route (`T`).
  ///
  /// The `T` type argument is the type of the return value of the popped route.
  ///
  /// The type of `result`, if provided, must match the type argument of the
  /// class of the removed route (`T`).
  ///
  /// Ongoing gestures within the current route are canceled.
  /// {@endtemplate}
  ///
  /// This method is used, for example, to instantly dismiss dropdown menus that
  /// are up when the screen's orientation changes.
  @optionalTypeArgs
  static void removeRoute<T extends Object?>(BuildContext context, Route<T> route, [T? result]) {
    return Navigator.of(context).removeRoute<T>(route, result);
  }
```

**关键特性**：

- **无动画**：此方法调用不会运行动画
- **立即移除**：路由立即被移除和释放
- **必须存在于历史中**：给定的 `route` 必须在历史中，否则会抛出异常
- **结果参数**：如果 `result` 不为 null，它将用作被移除路由的结果
- **手势取消**：当前路由中的持续手势会被取消

**使用场景**：

- 屏幕方向改变时立即关闭下拉菜单
- 需要立即移除路由而不运行动画的场景

## removeRouteBelow

立即从最紧密包围给定 context 的 navigator 中移除路由并 [Route.dispose] 它。要移除的路由是给定 `anchorRoute` 下方的路由。

```dart 2841:2876:packages/flutter/lib/src/widgets/navigator.dart
  /// Immediately remove a route from the navigator that most tightly encloses
  /// the given context, and [Route.dispose] it. The route to be removed is the
  /// one below the given `anchorRoute`.
  ///
  /// {@template flutter.widgets.navigator.removeRouteBelow}
  /// No animations are run as a result of this method call.
  ///
  /// The routes below and above the removed route are notified (see
  /// [Route.didChangeNext] and [Route.didChangePrevious]). If the [Navigator]
  /// has any [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didRemove]). The removed route is disposed with its
  /// future completed.
  ///
  /// The given `anchorRoute` must be in the history and must have a route below
  /// it; this method will throw an exception if it is not or does not.
  ///
  /// If non-null, `result` will be used as the result of the route that is
  /// removed; the future that had been returned from pushing the removed route
  /// will complete with `result`. If provided, must match the type argument of
  /// the class of the popped route (`T`).
  ///
  /// The `T` type argument is the type of the return value of the popped route.
  ///
  /// The type of `result`, if provided, must match the type argument of the
  /// class of the removed route (`T`).
  ///
  /// Ongoing gestures within the current route are canceled.
  /// {@endtemplate}
  @optionalTypeArgs
  static void removeRouteBelow<T extends Object?>(
    BuildContext context,
    Route<T> anchorRoute, [
    T? result,
  ]) {
    return Navigator.of(context).removeRouteBelow<T>(anchorRoute, result);
  }
```

**关键特性**：

- **通过锚点路由标识**：通过指定锚点路由来标识要移除的路由（锚点路由下方的路由）
- **锚点路由必须存在下方路由**：给定的 `anchorRoute` 必须在历史中，并且必须有一个下方的路由，否则会抛出异常
- **无动画**：不运行动画
- **立即移除**：路由立即被移除和释放

**与 removeRoute 的区别**：

- `removeRoute` 直接指定要移除的路由
- `removeRouteBelow` 通过指定锚点路由来标识要移除的路由（锚点路由下方的路由）

**使用场景**：

- 当你知道锚点路由但不知道具体要移除哪个路由时
- 构建复杂的导航结构

## 方法对比

| 方法 | 功能 | 是否运行动画 | 路由标识方式 |
| --- | --- | --- | --- |
| `replace` | 替换路由 | 否 | 直接指定旧路由 |
| `replaceRouteBelow` | 替换下方路由 | 否 | 通过锚点路由 |
| `removeRoute` | 移除路由 | 否 | 直接指定路由 |
| `removeRouteBelow` | 移除下方路由 | 否 | 通过锚点路由 |

## 注意事项

1. **无动画操作**：所有这些方法都不运行动画，适用于需要立即操作的场景
2. **旧路由必须不可见**：替换方法要求旧路由当前不可见，否则移除会显得突兀
3. **非线性用户体验**：这些方法特别适用于构建非线性用户体验
4. **状态恢复**：只有 `restorable` 版本的方法支持状态恢复
5. **异常处理**：如果指定的路由不存在或不符合条件，这些方法会抛出异常

## 使用建议

1. **何时使用 replace**：
   - 需要在后台替换不可见的路由
   - 构建非线性导航体验
   - 不需要动画效果

2. **何时使用 pushReplacement**：
   - 需要替换当前可见的路由
   - 需要动画效果
   - 用户可以看到过渡

3. **何时使用 removeRoute**：
   - 需要立即移除路由而不运行动画
   - 屏幕方向改变等场景
   - 需要精确控制路由移除

## 相关链接

- [Navigator 类概述与使用指南](navigator.dart_Navigator_1_概述与使用指南.md)
- [Navigator 直接路由操作](navigator.dart_Navigator_4_直接路由操作.md)
- [Navigator 工具方法](navigator.dart_Navigator_6_工具方法.md)
