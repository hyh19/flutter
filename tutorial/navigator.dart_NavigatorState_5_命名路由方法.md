# NavigatorState 命名路由方法

## 概述

命名路由方法允许通过路由名称（字符串）进行导航，而不需要直接创建 `Route` 对象。这些方法通过 `onGenerateRoute` 回调将路由名称转换为实际的 `Route` 对象。本文档介绍所有命名路由方法及其可恢复版本。

## 核心方法：_routeNamed

所有命名路由方法都依赖于 `_routeNamed` 方法来解析路由名称：

```dart 4627:4687:packages/flutter/lib/src/widgets/navigator.dart
  Route<T?>? _routeNamed<T>(String name, {required Object? arguments, bool allowNull = false}) {
    assert(!_debugLocked);
    if (allowNull && widget.onGenerateRoute == null) {
      return null;
    }
    assert(() {
      if (widget.onGenerateRoute == null) {
        throw FlutterError(
          'Navigator.onGenerateRoute was null, but the route named "$name" was referenced.\n'
          'To use the Navigator API with named routes (pushNamed, pushReplacementNamed, or '
          'pushNamedAndRemoveUntil), the Navigator must be provided with an '
          'onGenerateRoute handler.\n'
          'The Navigator was:\n'
          '  $this',
        );
      }
      return true;
    }());
    final RouteSettings settings = RouteSettings(name: name, arguments: arguments);
    Route<T?>? route = widget.onGenerateRoute!(settings) as Route<T?>?;
    if (route == null && !allowNull) {
      assert(() {
        if (widget.onUnknownRoute == null) {
          throw FlutterError.fromParts(<DiagnosticsNode>[
            ErrorSummary(
              'Navigator.onGenerateRoute returned null when requested to build route "$name".',
            ),
            ErrorDescription(
              'The onGenerateRoute callback must never return null, unless an onUnknownRoute '
              'callback is provided as well.',
            ),
            DiagnosticsProperty<NavigatorState>(
              'The Navigator was',
              this,
              style: DiagnosticsTreeStyle.errorProperty,
            ),
          ]);
        }
        return true;
      }());
      route = widget.onUnknownRoute!(settings) as Route<T?>?;
      assert(() {
        if (route == null) {
          throw FlutterError.fromParts(<DiagnosticsNode>[
            ErrorSummary(
              'Navigator.onUnknownRoute returned null when requested to build route "$name".',
            ),
            ErrorDescription('The onUnknownRoute callback must never return null.'),
            DiagnosticsProperty<NavigatorState>(
              'The Navigator was',
              this,
              style: DiagnosticsTreeStyle.errorProperty,
            ),
          ]);
        }
        return true;
      }());
    }
    assert(route != null || allowNull);
    return route;
  }
```

### 功能说明

1. **检查 onGenerateRoute**：确保已提供路由生成回调
2. **创建 RouteSettings**：使用路由名称和参数创建设置对象
3. **生成路由**：调用 `onGenerateRoute` 生成路由
4. **处理未知路由**：如果生成失败，尝试使用 `onUnknownRoute`
5. **返回路由**：返回生成的路由对象

### 参数说明

- **name**：路由名称（如 `'/home'`）
- **arguments**：传递给路由的参数
- **allowNull**：是否允许返回 `null`（用于某些特殊场景）

## 1. pushNamed - 推送命名路由

```dart 4689:4715:packages/flutter/lib/src/widgets/navigator.dart
  /// Push a named route onto the navigator.
  ///
  /// {@macro flutter.widgets.navigator.pushNamed}
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
  /// void _aaronBurrSir() {
  ///   navigator.pushNamed('/nyc/1776');
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushNamed], which pushes a route that can be restored
  ///    during state restoration.
  @optionalTypeArgs
  Future<T?> pushNamed<T extends Object?>(String routeName, {Object? arguments}) {
    return push<T?>(_routeNamed<T>(routeName, arguments: arguments)!);
  }
```

### 功能说明

- **作用**：将命名路由推送到导航器栈顶
- **返回**：`Future<T?>`，路由弹出时返回结果
- **实现**：调用 `_routeNamed` 解析路由，然后调用 `push`

### 使用示例

```dart
// 基本用法
Navigator.of(context).pushNamed('/details');

// 传递参数
Navigator.of(context).pushNamed('/details', arguments: {'id': 123});

// 获取返回值
final result = await Navigator.of(context).pushNamed<bool>('/dialog');
if (result == true) {
  // 用户确认
}
```

## 2. restorablePushNamed - 可恢复的推送命名路由

```dart 4717:4750:packages/flutter/lib/src/widgets/navigator.dart
  /// Push a named route onto the navigator.
  ///
  /// {@macro flutter.widgets.navigator.restorablePushNamed}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed}
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
  /// void _openDetails() {
  ///   navigator.restorablePushNamed('/nyc/1776');
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  String restorablePushNamed<T extends Object?>(String routeName, {Object? arguments}) {
    assert(
      debugIsSerializableForRestoration(arguments),
      'The arguments object must be serializable via the StandardMessageCodec.',
    );
    final _RouteEntry entry = _RestorationInformation.named(
      name: routeName,
      arguments: arguments,
      restorationScopeId: _nextPagelessRestorationScopeId,
    ).toRouteEntry(this, initialState: _RouteLifecycle.push);
    _pushEntry(entry);
    return entry.restorationId!;
  }
```

### 功能说明

- **作用**：推送可恢复的命名路由
- **返回**：`String`，恢复 ID，可用于后续恢复
- **要求**：参数必须可序列化

### 与 pushNamed 的区别

| 特性 | pushNamed | restorablePushNamed |
| --- | --- | --- |
| 返回值 | `Future<T?>` | `String` (恢复 ID) |
| 状态恢复 | 不支持 | 支持 |
| 参数要求 | 任意 | 必须可序列化 |
| 使用场景 | 临时导航 | 需要恢复的导航 |

## 3. pushReplacementNamed - 替换当前路由

```dart 4752:4787:packages/flutter/lib/src/widgets/navigator.dart
  /// Replace the current route of the navigator by pushing the route named
  /// [routeName] and then disposing the previous route once the new route has
  /// finished animating in.
  ///
  /// {@macro flutter.widgets.navigator.pushReplacementNamed}
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
  /// void _startBike() {
  ///   navigator.pushReplacementNamed('/jouett/1781');
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushReplacementNamed], which pushes a replacement route that
  ///  can be restored during state restoration.
  @optionalTypeArgs
  Future<T?> pushReplacementNamed<T extends Object?, TO extends Object?>(
    String routeName, {
    TO? result,
    Object? arguments,
  }) {
    return pushReplacement<T?, TO>(
      _routeNamed<T>(routeName, arguments: arguments)!,
      result: result,
    );
  }
```

### 功能说明

- **作用**：用新路由替换当前路由
- **行为**：新路由动画完成后，旧路由被销毁
- **参数**：
  - **result**：传递给旧路由的结果
  - **arguments**：传递给新路由的参数

### 使用场景

- 登录后跳转到主页（替换登录页）
- 完成向导后进入主应用
- 替换当前页面为新页面

### 使用示例

```dart
// 替换当前路由
Navigator.of(context).pushReplacementNamed('/home');

// 替换并传递结果
Navigator.of(context).pushReplacementNamed('/home', result: 'logged_in');
```

## 4. restorablePushReplacementNamed - 可恢复的替换路由

```dart 4789:4828:packages/flutter/lib/src/widgets/navigator.dart
  /// Replace the current route of the navigator by pushing the route named
  /// [routeName] and then disposing the previous route once the new route has
  /// finished animating in.
  ///
  /// {@macro flutter.widgets.navigator.restorablePushReplacementNamed}
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
  /// void _startCar() {
  ///   navigator.restorablePushReplacementNamed('/jouett/1781');
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  String restorablePushReplacementNamed<T extends Object?, TO extends Object?>(
    String routeName, {
    TO? result,
    Object? arguments,
  }) {
    assert(
      debugIsSerializableForRestoration(arguments),
      'The arguments object must be serializable via the StandardMessageCodec.',
    );
    final _RouteEntry entry = _RestorationInformation.named(
      name: routeName,
      arguments: arguments,
      restorationScopeId: _nextPagelessRestorationScopeId,
    ).toRouteEntry(this, initialState: _RouteLifecycle.pushReplace);
    _pushReplacementEntry(entry, result);
    return entry.restorationId!;
  }
```

## 5. popAndPushNamed - 弹出并推送

```dart 4830:4862:packages/flutter/lib/src/widgets/navigator.dart
  /// Pop the current route off the navigator and push a named route in its
  /// place.
  ///
  /// {@macro flutter.widgets.navigator.popAndPushNamed}
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
  /// void _begin() {
  ///   navigator.popAndPushNamed('/nyc/1776');
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePopAndPushNamed], which pushes a new route that can be
  ///    restored during state restoration.
  @optionalTypeArgs
  Future<T?> popAndPushNamed<T extends Object?, TO extends Object?>(
    String routeName, {
    TO? result,
    Object? arguments,
  }) {
    pop<TO>(result);
    return pushNamed<T>(routeName, arguments: arguments);
  }
```

### 功能说明

- **作用**：先弹出当前路由，再推送新路由
- **行为**：等价于 `pop()` + `pushNamed()`，但更高效
- **使用场景**：需要替换当前页面但保留返回栈

### 与 pushReplacementNamed 的区别

| 特性 | popAndPushNamed | pushReplacementNamed |
| --- | --- | --- |
| 返回栈 | 保留 | 不保留（替换） |
| 动画 | 先弹出，再推入 | 直接替换 |
| 使用场景 | 导航到同级页面 | 替换当前页面 |

## 6. restorablePopAndPushNamed - 可恢复的弹出并推送

```dart 4864:4893:packages/flutter/lib/src/widgets/navigator.dart
  /// Pop the current route off the navigator and push a named route in its
  /// place.
  ///
  /// {@macro flutter.widgets.navigator.restorablePopAndPushNamed}
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
  /// void _end() {
  ///   navigator.restorablePopAndPushNamed('/nyc/1776');
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  String restorablePopAndPushNamed<T extends Object?, TO extends Object?>(
    String routeName, {
    TO? result,
    Object? arguments,
  }) {
    pop<TO>(result);
    return restorablePushNamed(routeName, arguments: arguments);
  }
```

## 7. pushNamedAndRemoveUntil - 推送并移除直到条件

```dart 4895:4926:packages/flutter/lib/src/widgets/navigator.dart
  /// Push the route with the given name onto the navigator, and then remove all
  /// the previous routes until the `predicate` returns true.
  ///
  /// {@macro flutter.widgets.navigator.pushNamedAndRemoveUntil}
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
  /// void _handleOpenCalendar() {
  ///   navigator.pushNamedAndRemoveUntil('/calendar', ModalRoute.withName('/'));
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [restorablePushNamedAndRemoveUntil], which pushes a new route that can
  ///    be restored during state restoration.
  @optionalTypeArgs
  Future<T?> pushNamedAndRemoveUntil<T extends Object?>(
    String newRouteName,
    RoutePredicate predicate, {
    Object? arguments,
  }) {
    return pushAndRemoveUntil<T?>(_routeNamed<T>(newRouteName, arguments: arguments)!, predicate);
  }
```

### 功能说明

- **作用**：推送新路由，并移除所有满足条件的旧路由
- **参数**：
  - **newRouteName**：新路由名称
  - **predicate**：路由断言，返回 `true` 时停止移除
- **使用场景**：重置导航栈（如登录后清空登录相关页面）

### 使用示例

```dart
// 重置到根路由
Navigator.of(context).pushNamedAndRemoveUntil(
  '/home',
  ModalRoute.withName('/'),
);

// 重置到特定路由
Navigator.of(context).pushNamedAndRemoveUntil(
  '/dashboard',
  (route) => route.settings.name == '/login',
);
```

## 8. restorablePushNamedAndRemoveUntil - 可恢复的推送并移除

```dart 4928:4966:packages/flutter/lib/src/widgets/navigator.dart
  /// Push the route with the given name onto the navigator, and then remove all
  /// the previous routes until the `predicate` returns true.
  ///
  /// {@macro flutter.widgets.navigator.restorablePushNamedAndRemoveUntil}
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
  /// void _openCalendar() {
  ///   navigator.restorablePushNamedAndRemoveUntil('/calendar', ModalRoute.withName('/'));
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  String restorablePushNamedAndRemoveUntil<T extends Object?>(
    String newRouteName,
    RoutePredicate predicate, {
    Object? arguments,
  }) {
    assert(
      debugIsSerializableForRestoration(arguments),
      'The arguments object must be serializable via the StandardMessageCodec.',
    );
    final _RouteEntry entry = _RestorationInformation.named(
      name: newRouteName,
      arguments: arguments,
      restorationScopeId: _nextPagelessRestorationScopeId,
    ).toRouteEntry(this, initialState: _RouteLifecycle.push);
    _pushEntryAndRemoveUntil(entry, predicate);
    return entry.restorationId!;
  }
```

## 方法对比表

| 方法 | 功能 | 返回类型 | 可恢复 |
| --- | --- | --- | --- |
| `pushNamed` | 推送路由 | `Future<T?>` | ❌ |
| `restorablePushNamed` | 推送路由 | `String` | ✅ |
| `pushReplacementNamed` | 替换路由 | `Future<T?>` | ❌ |
| `restorablePushReplacementNamed` | 替换路由 | `String` | ✅ |
| `popAndPushNamed` | 弹出并推送 | `Future<T?>` | ❌ |
| `restorablePopAndPushNamed` | 弹出并推送 | `String` | ✅ |
| `pushNamedAndRemoveUntil` | 推送并移除 | `Future<T?>` | ❌ |
| `restorablePushNamedAndRemoveUntil` | 推送并移除 | `String` | ✅ |

## 路由配置示例

### 基本配置

```dart
MaterialApp(
  routes: {
    '/': (context) => HomePage(),
    '/details': (context) => DetailsPage(),
    '/settings': (context) => SettingsPage(),
  },
  onGenerateRoute: (settings) {
    // 处理动态路由
    if (settings.name == '/user/:id') {
      final id = settings.arguments as String;
      return MaterialPageRoute(
        builder: (context) => UserPage(id: id),
      );
    }
    return null; // 使用默认路由
  },
  onUnknownRoute: (settings) {
    // 处理未知路由
    return MaterialPageRoute(
      builder: (context) => NotFoundPage(),
    );
  },
)
```

## 注意事项

1. **必须提供 onGenerateRoute**：使用命名路由时必须提供路由生成回调
2. **参数序列化**：可恢复方法的参数必须可序列化
3. **路由名称格式**：建议使用绝对路径（如 `'/home'`）
4. **返回值处理**：使用 `await` 等待路由返回结果
5. **状态恢复**：可恢复方法需要启用状态恢复功能

## 相关文档

- [NavigatorState 直接路由操作方法](navigator.dart_NavigatorState_6_直接路由操作方法.md)
- [NavigatorState 路由替换与移除方法](navigator.dart_NavigatorState_7_路由替换与移除方法.md)
- [Navigator 命名路由方法](navigator.dart_Navigator_3_命名路由方法.md)
