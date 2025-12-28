# NavigatorState 直接路由操作方法详解

## 概述

直接路由操作方法允许直接操作 `Route` 对象进行导航，而不需要通过路由名称。这些方法提供了更灵活的路由控制，适用于需要自定义路由或动态创建路由的场景。本文档介绍所有直接路由操作方法及其可恢复版本。

## 1. push - 推送路由

```dart 4968:4997:packages/flutter/lib/src/widgets/navigator.dart
  /// Push the given route onto the navigator.
  ///
  /// {@macro flutter.widgets.navigator.push}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _openPage() {
  ///   navigator.push<void>(
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
  Future<T?> push<T extends Object?>(Route<T> route) {
    _pushEntry(_RouteEntry(route, pageBased: false, initialState: _RouteLifecycle.push));
    return route.popped;
  }
```

### 功能说明

- **作用**：将路由推送到导航器栈顶
- **返回**：`Future<T?>`，路由弹出时返回结果
- **实现**：创建 `_RouteEntry` 并调用 `_pushEntry`

### 使用示例

```dart
// 基本用法
Navigator.of(context).push(
  MaterialPageRoute(
    builder: (context) => DetailsPage(),
  ),
);

// 自定义转场动画
Navigator.of(context).push(
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => DetailsPage(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return FadeTransition(opacity: animation, child: child);
    },
  ),
);

// 获取返回值
final result = await Navigator.of(context).push<bool>(
  MaterialPageRoute(
    builder: (context) => DialogPage(),
  ),
);
```

## 2. restorablePush - 可恢复的推送路由

```dart 5009:5044:packages/flutter/lib/src/widgets/navigator.dart
  /// Push a new route onto the navigator.
  ///
  /// {@macro flutter.widgets.navigator.restorablePush}
  ///
  /// {@macro flutter.widgets.navigator.push}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePush}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  ///
  /// {@tool dartpad}
  /// Typical usage is as follows:
  ///
  /// ** See code in examples/api/lib/widgets/navigator/navigator_state.restorable_push.0.dart **
  /// {@end-tool}
  @optionalTypeArgs
  String restorablePush<T extends Object?>(
    RestorableRouteBuilder<T> routeBuilder, {
    Object? arguments,
  }) {
    assert(
      _debugIsStaticCallback(routeBuilder),
      'The provided routeBuilder must be a static function.',
    );
    assert(
      debugIsSerializableForRestoration(arguments),
      'The arguments object must be serializable via the StandardMessageCodec.',
    );
    final _RouteEntry entry = _RestorationInformation.anonymous(
      routeBuilder: routeBuilder,
      arguments: arguments,
      restorationScopeId: _nextPagelessRestorationScopeId,
    ).toRouteEntry(this, initialState: _RouteLifecycle.push);
    _pushEntry(entry);
    return entry.restorationId!;
  }
```

### 功能说明

- **作用**：推送可恢复的路由
- **返回**：`String`，恢复 ID
- **要求**：
  - `routeBuilder` 必须是静态函数
  - `arguments` 必须可序列化

### 与 push 的区别

| 特性 | push | restorablePush |
| --- | --- | --- |
| 参数 | `Route<T>` | `RestorableRouteBuilder<T>` |
| 返回值 | `Future<T?>` | `String` (恢复 ID) |
| 状态恢复 | 不支持 | 支持 |
| 函数要求 | 任意 | 必须是静态函数 |

### 使用示例

```dart
// 可恢复的路由推送
final restorationId = Navigator.of(context).restorablePush(
  (context, arguments) {
    final id = arguments as int? ?? 0;
    return MaterialPageRoute(
      builder: (context) => DetailsPage(id: id),
    );
  },
  arguments: 123,
);
```

## 3. _pushEntry - 内部推送实现

```dart 5046:5061:packages/flutter/lib/src/widgets/navigator.dart
  void _pushEntry(_RouteEntry entry) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    assert(entry.route._navigator == null);
    assert(entry.currentState == _RouteLifecycle.push);
    _history.add(entry);
    _flushHistoryUpdates();
    assert(() {
      _debugLocked = false;
      return true;
    }());
    _afterNavigation(entry.route);
  }
```

### 功能说明

1. **锁定导航器**：防止并发操作
2. **验证路由**：确保路由未被其他导航器使用
3. **添加到历史**：将路由条目添加到历史记录
4. **刷新更新**：调用 `_flushHistoryUpdates` 应用更改
5. **导航后处理**：调用 `_afterNavigation` 处理导航后事件

## 4. _afterNavigation - 导航后处理

```dart 5063:5096:packages/flutter/lib/src/widgets/navigator.dart
  void _afterNavigation(Route<dynamic>? route) {
    if (!kReleaseMode) {
      // Among other uses, performance tools use this event to ensure that perf
      // stats reflect the time interval since the last navigation event
      // occurred, ensuring that stats only reflect the current page.

      Map<String, dynamic>? routeJsonable;
      if (route != null) {
        routeJsonable = <String, dynamic>{};

        final String description;
        if (route is TransitionRoute<dynamic>) {
          final TransitionRoute<dynamic> transitionRoute = route;
          description = transitionRoute.debugLabel;
        } else {
          description = '$route';
        }
        routeJsonable['description'] = description;

        final RouteSettings settings = route.settings;
        final Map<String, dynamic> settingsJsonable = <String, dynamic>{'name': settings.name};
        if (settings.arguments != null) {
          settingsJsonable['arguments'] = jsonEncode(
            settings.arguments,
            toEncodable: (Object? object) => '$object',
          );
        }
        routeJsonable['settings'] = settingsJsonable;
      }

      developer.postEvent('Flutter.Navigation', <String, dynamic>{'route': routeJsonable});
    }
    _cancelActivePointers();
  }
```

### 功能说明

1. **性能工具事件**：在调试模式下发送导航事件给性能工具
2. **取消活动指针**：取消所有活动的指针事件，防止导航过程中的交互冲突

## 5. pushReplacement - 替换当前路由

```dart 5098:5136:packages/flutter/lib/src/widgets/navigator.dart
  /// Replace the current route of the navigator by pushing the given route and
  /// then disposing the previous route once the new route has finished
  /// animating in.
  ///
  /// {@macro flutter.widgets.navigator.pushReplacement}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _doOpenPage() {
  ///   navigator.pushReplacement<void, void>(
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
  Future<T?> pushReplacement<T extends Object?, TO extends Object?>(
    Route<T> newRoute, {
    TO? result,
  }) {
    assert(newRoute._navigator == null);
    _pushReplacementEntry(
      _RouteEntry(newRoute, pageBased: false, initialState: _RouteLifecycle.pushReplace),
      result,
    );
    return newRoute.popped;
  }
```

### 功能说明

- **作用**：用新路由替换当前路由
- **行为**：新路由动画完成后，旧路由被销毁
- **参数**：
  - **newRoute**：新路由对象
  - **result**：传递给旧路由的结果

### 使用场景

- 登录后跳转到主页
- 完成向导后进入主应用
- 替换当前页面为新页面

### 使用示例

```dart
// 替换当前路由
Navigator.of(context).pushReplacement(
  MaterialPageRoute(
    builder: (context) => HomePage(),
  ),
);

// 替换并传递结果
Navigator.of(context).pushReplacement(
  MaterialPageRoute(
    builder: (context) => HomePage(),
  ),
  result: 'logged_in',
);
```

## 6. restorablePushReplacement - 可恢复的替换路由

```dart 5138:5176:packages/flutter/lib/src/widgets/navigator.dart
  /// Replace the current route of the navigator by pushing a new route and
  /// then disposing the previous route once the new route has finished
  /// animating in.
  ///
  /// {@macro flutter.widgets.navigator.restorablePushReplacement}
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
  /// ** See code in examples/api/lib/widgets/navigator/navigator_state.restorable_push_replacement.0.dart **
  /// {@end-tool}
  @optionalTypeArgs
  String restorablePushReplacement<T extends Object?, TO extends Object?>(
    RestorableRouteBuilder<T> routeBuilder, {
    TO? result,
    Object? arguments,
  }) {
    assert(
      _debugIsStaticCallback(routeBuilder),
      'The provided routeBuilder must be a static function.',
    );
    assert(
      debugIsSerializableForRestoration(arguments),
      'The arguments object must be serializable via the StandardMessageCodec.',
    );
    final _RouteEntry entry = _RestorationInformation.anonymous(
      routeBuilder: routeBuilder,
      arguments: arguments,
      restorationScopeId: _nextPagelessRestorationScopeId,
    ).toRouteEntry(this, initialState: _RouteLifecycle.pushReplace);
    _pushReplacementEntry(entry, result);
    return entry.restorationId!;
  }
```

## 7. _pushReplacementEntry - 内部替换实现

```dart 5178:5201:packages/flutter/lib/src/widgets/navigator.dart
  void _pushReplacementEntry<TO extends Object?>(_RouteEntry entry, TO? result) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    assert(entry.route._navigator == null);
    assert(_history.isNotEmpty);
    assert(
      _history.any(_RouteEntry.isPresentPredicate),
      'Navigator has no active routes to replace.',
    );
    assert(entry.currentState == _RouteLifecycle.pushReplace);
    _history
        .lastWhere(_RouteEntry.isPresentPredicate)
        .complete(result, isReplaced: true, imperativeRemoval: true);
    _history.add(entry);
    _flushHistoryUpdates();
    assert(() {
      _debugLocked = false;
      return true;
    }());
    _afterNavigation(entry.route);
  }
```

### 功能说明

1. **验证历史记录**：确保有活动路由可以替换
2. **完成旧路由**：标记最后一个活动路由为完成（被替换）
3. **添加新路由**：将新路由添加到历史记录
4. **刷新更新**：应用所有更改

## 8. pushAndRemoveUntil - 推送并移除直到条件

```dart 5203:5238:packages/flutter/lib/src/widgets/navigator.dart
  /// Push the given route onto the navigator, and then remove all the previous
  /// routes until the `predicate` returns true.
  ///
  /// {@macro flutter.widgets.navigator.pushAndRemoveUntil}
  ///
  /// {@macro flutter.widgets.navigator.pushNamed.returnValue}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _resetAndOpenPage() {
  ///   navigator.pushAndRemoveUntil<void>(
  ///     MaterialPageRoute<void>(builder: (BuildContext context) => const MyHomePage()),
  ///     ModalRoute.withName('/'),
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  ///
  ///
  /// See also:
  ///
  ///  * [restorablePushAndRemoveUntil], which pushes a route that can be
  ///    restored during state restoration.
  @optionalTypeArgs
  Future<T?> pushAndRemoveUntil<T extends Object?>(Route<T> newRoute, RoutePredicate predicate) {
    assert(newRoute._navigator == null);
    assert(newRoute.overlayEntries.isEmpty);
    _pushEntryAndRemoveUntil(
      _RouteEntry(newRoute, pageBased: false, initialState: _RouteLifecycle.push),
      predicate,
    );
    return newRoute.popped;
  }
```

### 功能说明

- **作用**：推送新路由，并移除所有满足条件的旧路由
- **参数**：
  - **newRoute**：新路由对象
  - **predicate**：路由断言，返回 `true` 时停止移除
- **使用场景**：重置导航栈

### 使用示例

```dart
// 重置到根路由
Navigator.of(context).pushAndRemoveUntil(
  MaterialPageRoute(
    builder: (context) => HomePage(),
  ),
  (route) => route.isFirst,
);

// 重置到特定路由
Navigator.of(context).pushAndRemoveUntil(
  MaterialPageRoute(
    builder: (context) => DashboardPage(),
  ),
  ModalRoute.withName('/login'),
);
```

## 9. restorablePushAndRemoveUntil - 可恢复的推送并移除

```dart 5240:5277:packages/flutter/lib/src/widgets/navigator.dart
  /// Push a new route onto the navigator, and then remove all the previous
  /// routes until the `predicate` returns true.
  ///
  /// {@macro flutter.widgets.navigator.restorablePushAndRemoveUntil}
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
  /// ** See code in examples/api/lib/widgets/navigator/navigator_state.restorable_push_and_remove_until.0.dart **
  /// {@end-tool}
  @optionalTypeArgs
  String restorablePushAndRemoveUntil<T extends Object?>(
    RestorableRouteBuilder<T> newRouteBuilder,
    RoutePredicate predicate, {
    Object? arguments,
  }) {
    assert(
      _debugIsStaticCallback(newRouteBuilder),
      'The provided routeBuilder must be a static function.',
    );
    assert(
      debugIsSerializableForRestoration(arguments),
      'The arguments object must be serializable via the StandardMessageCodec.',
    );
    final _RouteEntry entry = _RestorationInformation.anonymous(
      routeBuilder: newRouteBuilder,
      arguments: arguments,
      restorationScopeId: _nextPagelessRestorationScopeId,
    ).toRouteEntry(this, initialState: _RouteLifecycle.push);
    _pushEntryAndRemoveUntil(entry, predicate);
    return entry.restorationId!;
  }
```

## 10. _pushEntryAndRemoveUntil - 内部推送并移除实现

```dart 5279:5303:packages/flutter/lib/src/widgets/navigator.dart
  void _pushEntryAndRemoveUntil(_RouteEntry entry, RoutePredicate predicate) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    assert(entry.route._navigator == null);
    assert(entry.route.overlayEntries.isEmpty);
    assert(entry.currentState == _RouteLifecycle.push);
    int index = _history.length - 1;
    _history.add(entry);
    while (index >= 0 && !predicate(_history[index].route)) {
      if (_history[index].isPresent) {
        _history[index].complete(null, isReplaced: false, imperativeRemoval: true);
      }
      index -= 1;
    }
    _flushHistoryUpdates();

    assert(() {
      _debugLocked = false;
      return true;
    }());
    _afterNavigation(entry.route);
  }
```

### 功能说明

1. **添加新路由**：先将新路由添加到历史记录
2. **移除旧路由**：从后往前遍历，移除所有不满足条件的路由
3. **刷新更新**：应用所有更改

## 方法对比表

| 方法 | 功能 | 返回类型 | 可恢复 |
| --- | --- | --- | --- |
| `push` | 推送路由 | `Future<T?>` | ❌ |
| `restorablePush` | 推送路由 | `String` | ✅ |
| `pushReplacement` | 替换路由 | `Future<T?>` | ❌ |
| `restorablePushReplacement` | 替换路由 | `String` | ✅ |
| `pushAndRemoveUntil` | 推送并移除 | `Future<T?>` | ❌ |
| `restorablePushAndRemoveUntil` | 推送并移除 | `String` | ✅ |

## 命名路由 vs 直接路由

### 命名路由的优势

- **解耦**：路由名称与实现分离
- **配置集中**：路由配置集中管理
- **易于维护**：修改路由实现不影响调用代码

### 直接路由的优势

- **灵活性**：可以动态创建路由
- **自定义**：可以完全控制路由行为
- **性能**：不需要路由查找过程

## 使用建议

1. **简单导航**：使用命名路由（`pushNamed`）
2. **动态路由**：使用直接路由（`push`）
3. **需要恢复**：使用可恢复版本
4. **自定义转场**：使用直接路由配合 `PageRouteBuilder`

## 注意事项

1. **路由唯一性**：一个路由不能同时属于多个导航器
2. **Overlay 条目**：`pushAndRemoveUntil` 要求新路由的 `overlayEntries` 为空
3. **静态函数**：可恢复方法的 `routeBuilder` 必须是静态函数
4. **参数序列化**：可恢复方法的参数必须可序列化

## 相关文档

- [NavigatorState 命名路由方法](navigator.dart_NavigatorState_5_命名路由方法.md)
- [NavigatorState 路由替换与移除方法](navigator.dart_NavigatorState_7_路由替换与移除方法.md)
- [Navigator 直接路由操作](navigator.dart_Navigator_4_直接路由操作.md)
