# Navigator 命名路由方法详解

## 功能概述

本文档详细讲解 `Navigator` 类中所有与命名路由相关的静态方法。这些方法允许通过路由名称（字符串）来管理导航，而不是直接使用 Route 对象。

## 方法概览

命名路由方法主要分为以下几类：

1. **推送命名路由**：`pushNamed` / `restorablePushNamed`
2. **替换命名路由**：`pushReplacementNamed` / `restorablePushReplacementNamed`
3. **弹出并推送命名路由**：`popAndPushNamed` / `restorablePopAndPushNamed`
4. **推送并移除直到**：`pushNamedAndRemoveUntil` / `restorablePushNamedAndRemoveUntil`

每个方法都有对应的 `restorable` 版本，支持状态恢复功能。

## pushNamed

将命名路由推送到最紧密包围给定 context 的 navigator 上。

```dart 1804:1906:packages/flutter/lib/src/widgets/navigator.dart
  /// Push a named route onto the navigator that most tightly encloses the given
  /// context.
  ///
  /// {@template flutter.widgets.navigator.pushNamed}
  /// The route name will be passed to the [Navigator.onGenerateRoute]
  /// callback. The returned route will be pushed into the navigator.
  ///
  /// The new route and the previous route (if any) are notified (see
  /// [Route.didPush] and [Route.didChangeNext]). If the [Navigator] has any
  /// [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didPush]).
  ///
  /// Ongoing gestures within the current route are canceled when a new route is
  /// pushed.
  ///
  /// The `T` type argument is the type of the return value of the route.
  ///
  /// To use [pushNamed], an [Navigator.onGenerateRoute] callback must be
  /// provided,
  /// {@endtemplate}
  ///
  /// {@template flutter.widgets.navigator.pushNamed.returnValue}
  /// Returns a [Future] that completes to the `result` value passed to [pop]
  /// when the pushed route is popped off the navigator.
  /// {@endtemplate}
  ///
  /// {@template flutter.widgets.Navigator.pushNamed}
  /// The provided `arguments` are passed to the pushed route via
  /// [RouteSettings.arguments]. Any object can be passed as `arguments` (e.g. a
  /// [String], [int], or an instance of a custom `MyRouteArguments` class).
  /// Often, a Map is used to pass key-value pairs.
  ///
  /// The `arguments` may be used in [Navigator.onGenerateRoute] or
  /// [Navigator.onUnknownRoute] to construct the route.
  /// {@endtemplate}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _didPushButton() {
  ///   Navigator.pushNamed(context, '/settings');
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// {@tool snippet}
  ///
  /// The following example shows how to pass additional `arguments` to the
  /// route:
  ///
  /// ```dart
  /// void _showBerlinWeather() {
  ///   Navigator.pushNamed(
  ///     context,
  ///     '/weather',
  ///     arguments: <String, String>{
  ///       'city': 'Berlin',
  ///       'country': 'Germany',
  ///     },
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// {@tool snippet}
  ///
  /// The following example shows how to pass a custom Object to the route:
  ///
  /// ```dart
  /// class WeatherRouteArguments {
  ///   WeatherRouteArguments({ required this.city, required this.country });
  ///   final String city;
  ///   final String country;
  ///
  ///   bool get isGermanCapital {
  ///     return country == 'Germany' && city == 'Berlin';
  ///   }
  /// }
  ///
  /// void _showWeather() {
  ///   Navigator.pushNamed(
  ///     context,
  ///     '/weather',
  ///     arguments: WeatherRouteArguments(city: 'Berlin', country: 'Germany'),
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushNamed], which pushes a route that can be restored
  ///    during state restoration.
  @optionalTypeArgs
  static Future<T?> pushNamed<T extends Object?>(
    BuildContext context,
    String routeName, {
    Object? arguments,
  }) {
    return Navigator.of(context).pushNamed<T>(routeName, arguments: arguments);
  }
```

**关键特性**：

- 路由名称将传递给 [Navigator.onGenerateRoute] 回调
- 返回的 [Route] 将被推送到 navigator 中
- 新路由和前一路由（如果有）会被通知
- 当前路由中的持续手势在新路由推送时会被取消
- 返回一个 [Future]，当推送的路由被弹出时，该 Future 会完成，值为传递给 [pop] 的 `result`
- 可以通过 `arguments` 参数传递数据给路由

**使用要求**：必须提供 [Navigator.onGenerateRoute] 回调。

## restorablePushNamed

推送一个可恢复的命名路由。

```dart 1908:1961:packages/flutter/lib/src/widgets/navigator.dart
  /// Push a named route onto the navigator that most tightly encloses the given
  /// context.
  ///
  /// {@template flutter.widgets.navigator.restorablePushNamed}
  /// Unlike [Route]s pushed via [pushNamed], [Route]s pushed with this method
  /// are restored during state restoration according to the rules outlined
  /// in the "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed}
  ///
  /// {@template flutter.widgets.Navigator.restorablePushNamed.arguments}
  /// The provided `arguments` are passed to the pushed route via
  /// [RouteSettings.arguments]. Any object that is serializable via the
  /// [StandardMessageCodec] can be passed as `arguments`. Often, a Map is used
  /// to pass key-value pairs.
  ///
  /// The arguments may be used in [Navigator.onGenerateRoute] or
  /// [Navigator.onUnknownRoute] to construct the route.
  /// {@endtemplate}
  ///
  /// {@template flutter.widgets.Navigator.restorablePushNamed.returnValue}
  /// The method returns an opaque ID for the pushed route that can be used by
  /// the [RestorableRouteFuture] to gain access to the actual [Route] object
  /// added to the navigator and its return value. You can ignore the return
  /// value of this method, if you do not care about the route object or the
  /// route's return value.
  /// {@endtemplate}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _showParisWeather() {
  ///   Navigator.restorablePushNamed(
  ///     context,
  ///     '/weather',
  ///     arguments: <String, String>{
  ///       'city': 'Paris',
  ///       'country': 'France',
  ///     },
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  static String restorablePushNamed<T extends Object?>(
    BuildContext context,
    String routeName, {
    Object? arguments,
  }) {
    return Navigator.of(context).restorablePushNamed<T>(routeName, arguments: arguments);
  }
```

**与 pushNamed 的区别**：

- 使用此方法推送的路由可以在状态恢复期间恢复
- `arguments` 必须是可通过 [StandardMessageCodec] 序列化的对象
- 返回一个不透明的 ID（String），可用于 [RestorableRouteFuture] 访问实际的路由对象和返回值

## pushReplacementNamed

通过推送命名路由来替换最紧密包围给定 context 的 navigator 的当前路由，然后在新路由完成动画后释放前一路由。

```dart 1963:2025:packages/flutter/lib/src/widgets/navigator.dart
  /// Replace the current route of the navigator that most tightly encloses the
  /// given context by pushing the route named [routeName] and then disposing
  /// the previous route once the new route has finished animating in.
  ///
  /// {@template flutter.widgets.navigator.pushReplacementNamed}
  /// If non-null, `result` will be used as the result of the route that is
  /// removed; the future that had been returned from pushing that old route
  /// will complete with `result`. Routes such as dialogs or popup menus
  /// typically use this mechanism to return the value selected by the user to
  /// the widget that created their route. The type of `result`, if provided,
  /// must match the type argument of the class of the old route (`TO`).
  ///
  /// The route name will be passed to the [Navigator.onGenerateRoute]
  /// callback. The returned route will be pushed into the navigator.
  ///
  /// The new route and the route below the removed route are notified (see
  /// [Route.didPush] and [Route.didChangeNext]). If the [Navigator] has any
  /// [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didReplace]). The removed route is notified once the
  /// new route has finished animating (see [Route.didComplete]). The removed
  /// route's exit animation is not run (see [popAndPushNamed] for a variant
  /// that animates the removed route).
  ///
  /// Ongoing gestures within the current route are canceled when a new route is
  /// pushed.
  ///
  /// The `T` type argument is the type of the return value of the new route,
  /// and `TO` is the type of the return value of the old route.
  ///
  /// To use [pushReplacementNamed], a [Navigator.onGenerateRoute] callback must
  /// be provided.
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@macro flutter.widgets.Navigator.pushNamed}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _switchToBrightness() {
  ///   Navigator.pushReplacementNamed(context, '/settings/brightness');
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushReplacementNamed], which pushes a replacement route that
  ///    can be restored during state restoration.
  @optionalTypeArgs
  static Future<T?> pushReplacementNamed<T extends Object?, TO extends Object?>(
    BuildContext context,
    String routeName, {
    TO? result,
    Object? arguments,
  }) {
    return Navigator.of(
      context,
    ).pushReplacementNamed<T, TO>(routeName, arguments: arguments, result: result);
  }
```

**关键特性**：

- 如果 `result` 不为 null，它将用作被移除路由的结果
- 被移除路由的退出动画不会运行（与 `popAndPushNamed` 不同，后者会运行动画）
- 新路由和移除路由下方的路由会被通知
- 类型参数 `T` 是新路由的返回值类型，`TO` 是旧路由的返回值类型

**使用场景**：常用于登录后替换登录页面，或设置页面中切换不同设置子页面。

## restorablePushReplacementNamed

推送一个可恢复的替换命名路由。

```dart 2027:2063:packages/flutter/lib/src/widgets/navigator.dart
  /// Replace the current route of the navigator that most tightly encloses the
  /// given context by pushing the route named [routeName] and then disposing
  /// the previous route once the new route has finished animating in.
  ///
  /// {@template flutter.widgets.navigator.restorablePushReplacementNamed}
  /// Unlike [Route]s pushed via [pushReplacementNamed], [Route]s pushed with
  /// this method are restored during state restoration according to the rules
  /// outlined in the "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushReplacementNamed}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.arguments}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _switchToAudioVolume() {
  ///   Navigator.restorablePushReplacementNamed(context, '/settings/volume');
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  static String restorablePushReplacementNamed<T extends Object?, TO extends Object?>(
    BuildContext context,
    String routeName, {
    TO? result,
    Object? arguments,
  }) {
    return Navigator.of(
      context,
    ).restorablePushReplacementNamed<T, TO>(routeName, arguments: arguments, result: result);
  }
```

## popAndPushNamed

弹出最紧密包围给定 context 的 navigator 的当前路由，并在其位置推送命名路由。

```dart 2065:2121:packages/flutter/lib/src/widgets/navigator.dart
  /// Pop the current route off the navigator that most tightly encloses the
  /// given context and push a named route in its place.
  ///
  /// {@template flutter.widgets.navigator.popAndPushNamed}
  /// The popping of the previous route is handled as per [pop].
  ///
  /// The new route's name will be passed to the [Navigator.onGenerateRoute]
  /// callback. The returned route will be pushed into the navigator.
  ///
  /// The new route, the old route, and the route below the old route (if any)
  /// are all notified (see [Route.didPop], [Route.didComplete],
  /// [Route.didPopNext], [Route.didPush], and [Route.didChangeNext]). If the
  /// [Navigator] has any [Navigator.observers], they will be notified as well
  /// (see [NavigatorObserver.didPop] and [NavigatorObserver.didPush]). The
  /// animations for the pop and the push are performed simultaneously, so the
  /// route below may be briefly visible even if both the old route and the new
  /// route are opaque (see [TransitionRoute.opaque]).
  ///
  /// Ongoing gestures within the current route are canceled when a new route is
  /// pushed.
  ///
  /// The `T` type argument is the type of the return value of the new route,
  /// and `TO` is the return value type of the old route.
  ///
  /// To use [popAndPushNamed], a [Navigator.onGenerateRoute] callback must be provided.
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@macro flutter.widgets.Navigator.pushNamed}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _selectAccessibility() {
  ///   Navigator.popAndPushNamed(context, '/settings/accessibility');
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePopAndPushNamed], which pushes a new route that can be
  ///    restored during state restoration.
  @optionalTypeArgs
  static Future<T?> popAndPushNamed<T extends Object?, TO extends Object?>(
    BuildContext context,
    String routeName, {
    TO? result,
    Object? arguments,
  }) {
    return Navigator.of(
      context,
    ).popAndPushNamed<T, TO>(routeName, arguments: arguments, result: result);
  }
```

**关键特性**：

- 弹出和推送的动画同时执行
- 即使旧路由和新路由都是不透明的，下方的路由也可能短暂可见
- 与 `pushReplacementNamed` 的区别在于，此方法会运行动画

**使用场景**：常用于设置页面中切换不同设置子页面，需要看到过渡动画。

## restorablePopAndPushNamed

推送一个可恢复的弹出并推送命名路由。

```dart 2123:2158:packages/flutter/lib/src/widgets/navigator.dart
  /// Pop the current route off the navigator that most tightly encloses the
  /// given context and push a named route in its place.
  ///
  /// {@template flutter.widgets.navigator.restorablePopAndPushNamed}
  /// Unlike [Route]s pushed via [popAndPushNamed], [Route]s pushed with
  /// this method are restored during state restoration according to the rules
  /// outlined in the "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.popAndPushNamed}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.arguments}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _selectNetwork() {
  ///   Navigator.restorablePopAndPushNamed(context, '/settings/network');
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  static String restorablePopAndPushNamed<T extends Object?, TO extends Object?>(
    BuildContext context,
    String routeName, {
    TO? result,
    Object? arguments,
  }) {
    return Navigator.of(
      context,
    ).restorablePopAndPushNamed<T, TO>(routeName, arguments: arguments, result: result);
  }
```

## pushNamedAndRemoveUntil

将给定名称的路由推送到最紧密包围给定 context 的 navigator 上，然后移除所有先前的路由，直到 `predicate` 返回 true。

```dart 2160:2228:packages/flutter/lib/src/widgets/navigator.dart
  /// Push the route with the given name onto the navigator that most tightly
  /// encloses the given context, and then remove all the previous routes until
  /// the `predicate` returns true.
  ///
  /// {@template flutter.widgets.navigator.pushNamedAndRemoveUntil}
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
  /// The new route's name (`routeName`) will be passed to the
  /// [Navigator.onGenerateRoute] callback. The returned route will be pushed
  /// into the navigator.
  ///
  /// The new route and the route below the bottommost removed route (which
  /// becomes the route below the new route) are notified (see [Route.didPush]
  /// and [Route.didChangeNext]). If the [Navigator] has any
  /// [Navigator.observers], they will be notified as well (see
  /// [NavigatorObserver.didPush] and [NavigatorObserver.didRemove]). The
  /// removed routes are disposed, once the new route has finished animating,
  /// and the futures that had been returned from pushing those routes
  /// will complete.
  ///
  /// Ongoing gestures within the current route are canceled when a new route is
  /// pushed.
  ///
  /// The `T` type argument is the type of the return value of the new route.
  ///
  /// To use [pushNamedAndRemoveUntil], an [Navigator.onGenerateRoute] callback
  /// must be provided.
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@macro flutter.widgets.Navigator.pushNamed}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _resetToCalendar() {
  ///   Navigator.pushNamedAndRemoveUntil(context, '/calendar', ModalRoute.withName('/'));
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushNamedAndRemoveUntil], which pushes a new route that can
  ///    be restored during state restoration.
  @optionalTypeArgs
  static Future<T?> pushNamedAndRemoveUntil<T extends Object?>(
    BuildContext context,
    String newRouteName,
    RoutePredicate predicate, {
    Object? arguments,
  }) {
    return Navigator.of(
      context,
    ).pushNamedAndRemoveUntil<T>(newRouteName, predicate, arguments: arguments);
  }
```

**关键特性**：

- 如果 [Route.willHandlePopInternally] 为 true，predicate 可能对同一路由应用多次
- 要移除直到具有特定名称的路由，请使用 [ModalRoute.withName] 返回的 [RoutePredicate]
- 要移除推送路由下方的所有路由，请使用始终返回 false 的 [RoutePredicate]（例如 `(Route<dynamic> route) => false`）
- 被移除的路由不会被完成，因此此方法不接受返回值参数

**使用场景**：

- 登录后清除所有登录相关页面，跳转到主页
- 重置导航栈到特定页面

## restorablePushNamedAndRemoveUntil

推送一个可恢复的命名路由并移除直到满足条件。

```dart 2230:2266:packages/flutter/lib/src/widgets/navigator.dart
  /// Push the route with the given name onto the navigator that most tightly
  /// encloses the given context, and then remove all the previous routes until
  /// the `predicate` returns true.
  ///
  /// {@template flutter.widgets.navigator.restorablePushNamedAndRemoveUntil}
  /// Unlike [Route]s pushed via [pushNamedAndRemoveUntil], [Route]s pushed with
  /// this method are restored during state restoration according to the rules
  /// outlined in the "State Restoration" section of [Navigator].
  /// {@endtemplate}
  ///
  /// {@macro flutter.widgets.navigator.pushNamedAndRemoveUntil}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.arguments}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _resetToOverview() {
  ///   Navigator.restorablePushNamedAndRemoveUntil(context, '/overview', ModalRoute.withName('/'));
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  static String restorablePushNamedAndRemoveUntil<T extends Object?>(
    BuildContext context,
    String newRouteName,
    RoutePredicate predicate, {
    Object? arguments,
  }) {
    return Navigator.of(
      context,
    ).restorablePushNamedAndRemoveUntil<T>(newRouteName, predicate, arguments: arguments);
  }
```

## 方法对比

| 方法 | 功能 | 是否运行动画 | 返回值类型 |
| --- | --- | --- | --- |
| `pushNamed` | 推送命名路由 | 是 | `Future<T?>` |
| `pushReplacementNamed` | 替换当前路由 | 是（新路由进入） | `Future<T?>` |
| `popAndPushNamed` | 弹出并推送 | 是（同时执行） | `Future<T?>` |
| `pushNamedAndRemoveUntil` | 推送并移除直到 | 是 | `Future<T?>` |

## 注意事项

1. **必须提供 onGenerateRoute**：所有命名路由方法都需要 [Navigator.onGenerateRoute] 回调来生成路由
2. **参数序列化**：`restorable` 版本的 `arguments` 必须可通过 [StandardMessageCodec] 序列化
3. **状态恢复**：只有 `restorable` 版本的方法支持状态恢复
4. **类型安全**：使用泛型参数 `T` 来指定路由的返回值类型

## 相关链接

- [Navigator 类概述与使用指南](navigator.dart_Navigator_1_概述与使用指南.md)
- [Navigator 构造函数与属性](navigator.dart_Navigator_2_构造函数与属性.md)
- [Navigator 直接路由操作](navigator.dart_Navigator_4_直接路由操作.md)
