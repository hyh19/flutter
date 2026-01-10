# Navigator 直接路由操作方法详解

## 功能概述

本文档详细讲解 `Navigator` 类中所有直接操作 Route 对象的静态方法。这些方法允许直接使用 Route 对象来管理导航，而不是通过路由名称。

## 方法概览

直接路由操作方法主要分为以下几类：

1. **推送路由**：`push` / `restorablePush`
2. **替换路由**：`pushReplacement` / `restorablePushReplacement`
3. **推送并移除直到**：`pushAndRemoveUntil` / `restorablePushAndRemoveUntil`
4. **弹出路由**：`pop` / `popUntil` / `maybePop`

每个推送方法都有对应的 `restorable` 版本，支持状态恢复功能。

## push

将给定路由推送到最紧密包围给定 context 的 navigator 上。

```dart 2268:2308:packages/flutter/lib/src/widgets/navigator.dart
  /// Push the given route onto the navigator that most tightly encloses the
  /// given context.
  ///
  /// {@template flutter.widgets.navigator.push}
  /// The new route and the previous route (if any) are notified (see
  /// [Route.didPush] and [Route.didChangeNext]). If the [Navigator] has any
  /// [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didPush]).
  ///
  /// Ongoing gestures within the current route are canceled when a new route is
  /// pushed.
  ///
  /// The `T` type argument is the type of the return value of the route.
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _openMyPage() {
  ///   Navigator.push<void>(
  ///     context,
  ///     MaterialPageRoute<void>(
  ///       builder: (BuildContext context) => const MyPage(),
  ///     ),
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePush], which pushes a route that can be restored during
  ///    state restoration.
  @optionalTypeArgs
  static Future<T?> push<T extends Object?>(BuildContext context, Route<T> route) {
    return Navigator.of(context).push(route);
  }
```

**关键特性**：

- 新路由和前一路由（如果有）会被通知
- 当前路由中的持续手势在新路由推送时会被取消
- 返回一个 [Future]，当推送的路由被弹出时，该 Future 会完成，值为传递给 [pop] 的 `result`
- 类型参数 `T` 是路由的返回值类型

**使用场景**：最常用的路由推送方法，适用于需要直接创建 Route 对象的场景。

## restorablePush

推送一个可恢复的路由。

```dart 2310:2347:packages/flutter/lib/src/widgets/navigator.dart
  /// Push a new route onto the navigator that most tightly encloses the
  /// given context.
  ///
  /// {@template flutter.widgets.navigator.restorablePush}
  /// Unlike [Route]s pushed via [push], [Route]s pushed with this method are
  /// restored during state restoration according to the rules outlined in the
  /// "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.push}
  ///
  /// {@template flutter.widgets.Navigator.restorablePush}
  /// The method takes a [RestorableRouteBuilder] as argument, which must be a
  /// _static_ function annotated with `@pragma('vm:entry-point')`. It must
  /// instantiate and return a new [Route] object that will be added to the
  /// navigator. The provided `arguments` object is passed to the
  /// `routeBuilder`. The navigator calls the static `routeBuilder` function
  /// again during state restoration to re-create the route object.
  ///
  /// Any object that is serializable via the [StandardMessageCodec] can be
  /// passed as `arguments`. Often, a Map is used to pass key-value pairs.
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  ///
  /// {@tool dartpad}
  /// Typical usage is as follows:
  ///
  /// ** See code in examples/api/lib/widgets/navigator/navigator.restorable_push.0.dart **
  /// {@end-tool}
  @optionalTypeArgs
  static String restorablePush<T extends Object?>(
    BuildContext context,
    RestorableRouteBuilder<T> routeBuilder, {
    Object? arguments,
  }) {
    return Navigator.of(context).restorablePush(routeBuilder, arguments: arguments);
  }
```

**关键特性**：

- 使用此方法推送的路由可以在状态恢复期间恢复
- 方法接受一个 [RestorableRouteBuilder] 作为参数，必须是标注了 `@pragma('vm:entry-point')` 的静态函数
- `arguments` 必须是可通过 [StandardMessageCodec] 序列化的对象
- 返回一个不透明的 ID（String），可用于访问路由对象和返回值

## pushReplacement

通过推送给定路由来替换最紧密包围给定 context 的 navigator 的当前路由，然后在新路由完成动画后释放前一路由。

```dart 2349:2403:packages/flutter/lib/src/widgets/navigator.dart
  /// Replace the current route of the navigator that most tightly encloses the
  /// given context by pushing the given route and then disposing the previous
  /// route once the new route has finished animating in.
  ///
  /// {@template flutter.widgets.navigator.pushReplacement}
  /// If non-null, `result` will be used as the result of the route that is
  /// removed; the future that had been returned from pushing that old route will
  /// complete with `result`. Routes such as dialogs or popup menus typically
  /// use this mechanism to return the value selected by the user to the widget
  /// that created their route. The type of `result`, if provided, must match
  /// the type argument of the class of the old route (`TO`).
  ///
  /// The new route and the route below the removed route are notified (see
  /// [Route.didPush] and [Route.didChangeNext]). If the [Navigator] has any
  /// [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didReplace]). The removed route is notified once the
  /// new route has finished animating (see [Route.didComplete]).
  ///
  /// Ongoing gestures within the current route are canceled when a new route is
  /// pushed.
  ///
  /// The `T` type argument is the type of the return value of the new route,
  /// and `TO` is the type of the return value of the old route.
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _completeLogin() {
  ///   Navigator.pushReplacement<void, void>(
  ///     context,
  ///     MaterialPageRoute<void>(
  ///       builder: (BuildContext context) => const MyHomePage(),
  ///     ),
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushReplacement], which pushes a replacement route that can
  ///    be restored during state restoration.
  @optionalTypeArgs
  static Future<T?> pushReplacement<T extends Object?, TO extends Object?>(
    BuildContext context,
    Route<T> newRoute, {
    TO? result,
  }) {
    return Navigator.of(context).pushReplacement<T, TO>(newRoute, result: result);
  }
```

**关键特性**：

- 如果 `result` 不为 null，它将用作被移除路由的结果
- 被移除路由的退出动画不会运行
- 新路由和移除路由下方的路由会被通知
- 类型参数 `T` 是新路由的返回值类型，`TO` 是旧路由的返回值类型

**使用场景**：常用于登录后替换登录页面，或完成某个流程后替换当前页面。

## restorablePushReplacement

推送一个可恢复的替换路由。

```dart 2405:2436:packages/flutter/lib/src/widgets/navigator.dart
  /// Replace the current route of the navigator that most tightly encloses the
  /// given context by pushing a new route and then disposing the previous
  /// route once the new route has finished animating in.
  ///
  /// {@template flutter.widgets.navigator.restorablePushReplacement}
  /// Unlike [Route]s pushed via [pushReplacement], [Route]s pushed with this
  /// method are restored during state restoration according to the rules
  /// outlined in the "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushReplacement}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePush}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  ///
  /// {@tool dartpad}
  /// Typical usage is as follows:
  ///
  /// ** See code in examples/api/lib/widgets/navigator/navigator.restorable_push_replacement.0.dart **
  /// {@end-tool}
  @optionalTypeArgs
  static String restorablePushReplacement<T extends Object?, TO extends Object?>(
    BuildContext context,
    RestorableRouteBuilder<T> routeBuilder, {
    TO? result,
    Object? arguments,
  }) {
    return Navigator.of(
      context,
    ).restorablePushReplacement<T, TO>(routeBuilder, result: result, arguments: arguments);
  }
```

## pushAndRemoveUntil

将给定路由推送到最紧密包围给定 context 的 navigator 上，然后移除所有先前的路由，直到 `predicate` 返回 true。

```dart 2438:2498:packages/flutter/lib/src/widgets/navigator.dart
  /// Push the given route onto the navigator that most tightly encloses the
  /// given context, and then remove all the previous routes until the
  /// `predicate` returns true.
  ///
  /// {@template flutter.widgets.navigator.pushAndRemoveUntil}
  /// The predicate may be applied to the same route more than once if
  /// [Route.willHandlePopInternally] is true.
  ///
  /// To remove routes until a route with a certain name, use the
  /// [RoutePredicate] returned from [ModalRoute.withName].
  ///
  /// To remove all the routes below the pushed route, use a [RoutePredicate]
  /// that always returns false (e.g. `(Route<dynamic> route) => false`).
  ///
  /// The removed routes are removed without being completed, so this method
  /// does not take a return value argument.
  ///
  /// The newly pushed route and its preceding route are notified for
  /// [Route.didPush]. After removal, the new route and its new preceding route,
  /// (the route below the bottommost removed route) are notified through
  /// [Route.didChangeNext]). If the [Navigator] has any [Navigator.observers],
  /// they will be notified as well (see [NavigatorObserver.didPush] and
  /// [NavigatorObserver.didRemove]). The removed routes are disposed of and
  /// notified, once the new route has finished animating. The futures that had
  /// been returned from pushing those routes will complete.
  ///
  /// Ongoing gestures within the current route are canceled when a new route is
  /// pushed.
  ///
  /// The `T` type argument is the type of the return value of the new route.
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _finishAccountCreation() {
  ///   Navigator.pushAndRemoveUntil<void>(
  ///     context,
  ///     MaterialPageRoute<void>(builder: (BuildContext context) => const MyHomePage()),
  ///     ModalRoute.withName('/'),
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushAndRemoveUntil], which pushes a route that can be
  ///    restored during state restoration.
  @optionalTypeArgs
  static Future<T?> pushAndRemoveUntil<T extends Object?>(
    BuildContext context,
    Route<T> newRoute,
    RoutePredicate predicate,
  ) {
    return Navigator.of(context).pushAndRemoveUntil<T>(newRoute, predicate);
  }
```

**关键特性**：

- 如果 [Route.willHandlePopInternally] 为 true，predicate 可能对同一路由应用多次
- 要移除直到具有特定名称的路由，请使用 [ModalRoute.withName] 返回的 [RoutePredicate]
- 要移除推送路由下方的所有路由，请使用始终返回 false 的 [RoutePredicate]
- 被移除的路由不会被完成，因此此方法不接受返回值参数

**使用场景**：

- 登录后清除所有登录相关页面，跳转到主页
- 完成注册流程后，清除注册相关页面，跳转到主页

## restorablePushAndRemoveUntil

推送一个可恢复的路由并移除直到满足条件。

```dart 2500:2531:packages/flutter/lib/src/widgets/navigator.dart
  /// Push a new route onto the navigator that most tightly encloses the
  /// given context, and then remove all the previous routes until the
  /// `predicate` returns true.
  ///
  /// {@template flutter.widgets.navigator.restorablePushAndRemoveUntil}
  /// Unlike [Route]s pushed via [pushAndRemoveUntil], [Route]s pushed with this
  /// method are restored during state restoration according to the rules
  /// outlined in the "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushAndRemoveUntil}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePush}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  ///
  /// {@tool dartpad}
  /// Typical usage is as follows:
  ///
  /// ** See code in examples/api/lib/widgets/navigator/navigator.restorable_push_and_remove_until.0.dart **
  /// {@end-tool}
  @optionalTypeArgs
  static String restorablePushAndRemoveUntil<T extends Object?>(
    BuildContext context,
    RestorableRouteBuilder<T> newRouteBuilder,
    RoutePredicate predicate, {
    Object? arguments,
  }) {
    return Navigator.of(
      context,
    ).restorablePushAndRemoveUntil<T>(newRouteBuilder, predicate, arguments: arguments);
  }
```

## pop

弹出最紧密包围给定 context 的 navigator 的最顶层路由。

```dart 2727:2775:packages/flutter/lib/src/widgets/navigator.dart
  /// Pop the top-most route off the navigator that most tightly encloses the
  /// given context.
  ///
  /// {@template flutter.widgets.navigator.pop}
  /// The current route's [Route.didPop] method is called first. If that method
  /// returns false, then the route remains in the [Navigator]'s history (the
  /// route is expected to have popped some internal state; see e.g.
  /// [LocalHistoryRoute]). Otherwise, the rest of this description applies.
  ///
  /// If non-null, `result` will be used as the result of the route that is
  /// popped; the future that had been returned from pushing the popped route
  /// will complete with `result`. Routes such as dialogs or popup menus
  /// typically use this mechanism to return the value selected by the user to
  /// the widget that created their route. The type of `result`, if provided,
  /// must match the type argument of the class of the popped route (`T`).
  ///
  /// The popped route and the route below it are notified (see [Route.didPop],
  /// [Route.didComplete], and [Route.didPopNext]). If the [Navigator] has any
  /// [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didPop]).
  ///
  /// The `T` type argument is the type of the return value of the popped route.
  ///
  /// The type of `result`, if provided, must match the type argument of the
  /// class of the popped route (`T`).
  /// {@endtemplate}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage for closing a route is as follows:
  ///
  /// ```dart
  /// void _close() {
  ///   Navigator.pop(context);
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// A dialog box might be closed with a result:
  ///
  /// ```dart
  /// void _accept() {
  ///   Navigator.pop(context, true); // dialog returns true
  /// }
  /// ```
  @optionalTypeArgs
  static void pop<T extends Object?>(BuildContext context, [T? result]) {
    Navigator.of(context).pop<T>(result);
  }
```

**关键特性**：

- 首先调用当前路由的 [Route.didPop] 方法
- 如果 [Route.didPop] 返回 false，则路由保留在 [Navigator] 的历史中（路由应该已经弹出了一些内部状态）
- 如果 `result` 不为 null，它将用作被弹出路由的结果
- 被弹出的路由和其下方的路由会被通知
- 类型参数 `T` 是被弹出路由的返回值类型

**使用场景**：

- 关闭当前页面
- 对话框返回用户选择的值

## popUntil

重复调用 [pop] 直到 `predicate` 返回 true。

```dart 2777:2804:packages/flutter/lib/src/widgets/navigator.dart
  /// Calls [pop] repeatedly on the navigator that most tightly encloses the
  /// given context until the predicate returns true.
  ///
  /// {@template flutter.widgets.navigator.popUntil}
  /// The predicate may be applied to the same route more than once if
  /// [Route.willHandlePopInternally] is true.
  ///
  /// To pop until a route with a certain name, use the [RoutePredicate]
  /// returned from [ModalRoute.withName].
  ///
  /// The routes are closed with null as their `return` value.
  ///
  /// See [pop] for more details of the semantics of popping a route.
  /// {@endtemplate}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _logout() {
  ///   Navigator.popUntil(context, ModalRoute.withName('/login'));
  /// }
  /// ```
  /// {@end-tool}
  static void popUntil(BuildContext context, RoutePredicate predicate) {
    Navigator.of(context).popUntil(predicate);
  }
```

**关键特性**：

- 如果 [Route.willHandlePopInternally] 为 true，predicate 可能对同一路由应用多次
- 要弹出直到具有特定名称的路由，请使用 [ModalRoute.withName] 返回的 [RoutePredicate]
- 路由以 null 作为返回值关闭

**使用场景**：

- 登出时返回到登录页面
- 返回到应用的根页面

## maybePop

查询当前路由的 [Route.popDisposition] getter 或 [Route.willPop] 方法，并相应地采取行动，可能会弹出路由；返回是否应认为弹出请求已处理。

```dart 2691:2725:packages/flutter/lib/src/widgets/navigator.dart
  /// Consults the current route's [Route.popDisposition] getter or
  /// [Route.willPop] method, and acts accordingly, potentially popping the
  /// route as a result; returns whether the pop request should be considered
  /// handled.
  ///
  /// {@template flutter.widgets.navigator.maybePop}
  /// If the [RoutePopDisposition] is [RoutePopDisposition.pop], then the [pop]
  /// method is called, and this method returns true, indicating that it handled
  /// the pop request.
  ///
  /// If the [RoutePopDisposition] is [RoutePopDisposition.doNotPop], then this
  /// method returns true, but does not do anything beyond that.
  ///
  /// If the [RoutePopDisposition] is [RoutePopDisposition.bubble], then this
  /// method returns false, and the caller is responsible for sending the
  /// request to the containing scope (e.g. by closing the application).
  ///
  /// This method is typically called for a user-initiated [pop]. For example on
  /// Android it's called by the binding for the system's back button.
  ///
  /// The `T` type argument is the type of the return value of the current
  /// route. (Typically this isn't known; consider specifying `dynamic` or
  /// `Null`.)
  /// {@endtemplate}
  ///
  /// See also:
  ///
  ///  * [Form], which provides an `onWillPop` callback that enables the form
  ///    to veto a [pop] initiated by the app's back button.
  ///  * [ModalRoute], which provides a `scopedWillPopCallback` that can be used
  ///    to define the route's `willPop` method.
  @optionalTypeArgs
  static Future<bool> maybePop<T extends Object?>(BuildContext context, [T? result]) {
    return Navigator.of(context).maybePop<T>(result);
  }
```

**关键特性**：

- 如果 [RoutePopDisposition] 是 [RoutePopDisposition.pop]，则调用 [pop] 方法，此方法返回 true
- 如果 [RoutePopDisposition] 是 [RoutePopDisposition.doNotPop]，则此方法返回 true，但不执行任何操作
- 如果 [RoutePopDisposition] 是 [RoutePopDisposition.bubble]，则此方法返回 false，调用者负责将请求发送到包含范围（例如关闭应用程序）
- 此方法通常用于用户发起的 [pop]，例如在 Android 上由系统返回按钮的绑定调用

**使用场景**：

- 处理系统返回按钮
- 在弹出前进行确认或验证

## 方法对比

| 方法 | 功能 | 是否运行动画 | 返回值类型 |
| --- | --- | --- | --- |
| `push` | 推送路由 | 是 | `Future<T?>` |
| `pushReplacement` | 替换当前路由 | 是（新路由进入） | `Future<T?>` |
| `pushAndRemoveUntil` | 推送并移除直到 | 是 | `Future<T?>` |
| `pop` | 弹出路由 | 是 | `void` |
| `popUntil` | 弹出直到 | 是 | `void` |
| `maybePop` | 可能弹出 | 可能 | `Future<bool>` |

## 注意事项

1. **直接使用 Route 对象**：这些方法需要直接创建 Route 对象，而不是使用路由名称
2. **类型安全**：使用泛型参数 `T` 来指定路由的返回值类型
3. **状态恢复**：只有 `restorable` 版本的方法支持状态恢复
4. **手势取消**：推送新路由时，当前路由中的持续手势会被取消

## 相关链接

- [Navigator 类概述与使用指南](navigator.dart_Navigator_1_概述与使用指南.md)
- [Navigator 命名路由方法](navigator.dart_Navigator_3_命名路由方法.md)
- [Navigator 路由替换与移除](navigator.dart_Navigator_5_路由替换与移除.md)
