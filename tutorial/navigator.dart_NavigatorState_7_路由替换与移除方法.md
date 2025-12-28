# NavigatorState 路由替换与移除方法

## 概述

路由替换和移除方法提供了更细粒度的路由栈控制能力。与 `pushReplacement` 不同，这些方法可以替换或移除历史记录中的任意路由，而不仅仅是当前路由。本文档介绍所有路由替换和移除方法及其可恢复版本。

## 1. replace - 替换指定路由

```dart 5305:5323:packages/flutter/lib/src/widgets/navigator.dart
  /// Replaces a route on the navigator with a new route.
  ///
  /// {@macro flutter.widgets.navigator.replace}
  ///
  /// See also:
  ///
  ///  * [replaceRouteBelow], which is the same but identifies the route to be
  ///    removed by reference to the route above it, rather than directly.
  ///  * [restorableReplace], which adds a replacement route that can be
  ///    restored during state restoration.
  @optionalTypeArgs
  void replace<T extends Object?>({required Route<dynamic> oldRoute, required Route<T> newRoute}) {
    assert(!_debugLocked);
    assert(oldRoute._navigator == this);
    _replaceEntry(
      _RouteEntry(newRoute, pageBased: false, initialState: _RouteLifecycle.replace),
      oldRoute,
    );
  }
```

### 功能说明

- **作用**：用新路由替换指定的旧路由
- **参数**：
  - **oldRoute**：要替换的旧路由（必须属于当前导航器）
  - **newRoute**：新路由对象
- **行为**：旧路由被标记为完成（被替换），新路由插入到旧路由之后

### 使用场景

- 替换历史记录中的某个路由
- 更新路由内容而不改变导航栈结构
- 修复路由状态问题

### 使用示例

```dart
// 获取当前路由
final currentRoute = ModalRoute.of(context)!;

// 替换当前路由
Navigator.of(context).replace(
  oldRoute: currentRoute,
  newRoute: MaterialPageRoute(
    builder: (context) => UpdatedPage(),
  ),
);
```

## 2. restorableReplace - 可恢复的替换路由

```dart 5325:5356:packages/flutter/lib/src/widgets/navigator.dart
  /// Replaces a route on the navigator with a new route.
  ///
  /// {@macro flutter.widgets.navigator.restorableReplace}
  ///
  /// {@macro flutter.widgets.navigator.replace}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePush}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  @optionalTypeArgs
  String restorableReplace<T extends Object?>({
    required Route<dynamic> oldRoute,
    required RestorableRouteBuilder<T> newRouteBuilder,
    Object? arguments,
  }) {
    assert(oldRoute._navigator == this);
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
    ).toRouteEntry(this, initialState: _RouteLifecycle.replace);
    _replaceEntry(entry, oldRoute);
    return entry.restorationId!;
  }
```

## 3. _replaceEntry - 内部替换实现

```dart 5358:5386:packages/flutter/lib/src/widgets/navigator.dart
  void _replaceEntry(_RouteEntry entry, Route<dynamic> oldRoute) {
    assert(!_debugLocked);
    if (oldRoute == entry.route) {
      return;
    }
    assert(() {
      _debugLocked = true;
      return true;
    }());
    assert(entry.currentState == _RouteLifecycle.replace);
    assert(entry.route._navigator == null);
    final int index = _history.indexWhere(_RouteEntry.isRoutePredicate(oldRoute));
    assert(index >= 0, 'This Navigator does not contain the specified oldRoute.');
    assert(
      _history[index].isPresent,
      'The specified oldRoute has already been removed from the Navigator.',
    );
    final bool wasCurrent = oldRoute.isCurrent;
    _history.insert(index + 1, entry);
    _history[index].complete(null, isReplaced: true, imperativeRemoval: true);
    _flushHistoryUpdates();
    assert(() {
      _debugLocked = false;
      return true;
    }());
    if (wasCurrent) {
      _afterNavigation(entry.route);
    }
  }
```

### 功能说明

1. **快速返回**：如果新旧路由相同，直接返回
2. **查找旧路由**：在历史记录中查找旧路由的索引
3. **验证状态**：确保旧路由存在且未被移除
4. **插入新路由**：将新路由插入到旧路由之后
5. **完成旧路由**：标记旧路由为完成（被替换）
6. **刷新更新**：应用所有更改
7. **导航后处理**：如果替换的是当前路由，调用 `_afterNavigation`

### 关键点

- **位置保持**：新路由插入到旧路由之后，保持导航栈结构
- **当前路由处理**：如果替换的是当前路由，需要触发导航后处理
- **状态标记**：旧路由被标记为 `isReplaced: true`

## 4. replaceRouteBelow - 替换下方路由

```dart 5388:5410:packages/flutter/lib/src/widgets/navigator.dart
  /// Replaces a route on the navigator with a new route. The route to be
  /// replaced is the one below the given `anchorRoute`.
  ///
  /// {@macro flutter.widgets.navigator.replaceRouteBelow}
  ///
  /// See also:
  ///
  ///  * [replace], which is the same but identifies the route to be removed
  ///    directly.
  ///  * [restorableReplaceRouteBelow], which adds a replacement route that can
  ///    be restored during state restoration.
  @optionalTypeArgs
  void replaceRouteBelow<T extends Object?>({
    required Route<dynamic> anchorRoute,
    required Route<T> newRoute,
  }) {
    assert(newRoute._navigator == null);
    assert(anchorRoute._navigator == this);
    _replaceEntryBelow(
      _RouteEntry(newRoute, pageBased: false, initialState: _RouteLifecycle.replace),
      anchorRoute,
    );
  }
```

### 功能说明

- **作用**：替换指定路由下方的路由
- **参数**：
  - **anchorRoute**：锚点路由（用于定位）
  - **newRoute**：新路由对象
- **行为**：找到锚点路由下方的第一个活动路由并替换它

### 使用场景

- 替换当前路由下方的路由
- 更新导航栈中的特定位置
- 修复历史记录中的路由

### 使用示例

```dart
// 获取当前路由
final currentRoute = ModalRoute.of(context)!;

// 替换当前路由下方的路由
Navigator.of(context).replaceRouteBelow(
  anchorRoute: currentRoute,
  newRoute: MaterialPageRoute(
    builder: (context) => UpdatedPage(),
  ),
);
```

## 5. restorableReplaceRouteBelow - 可恢复的替换下方路由

```dart 5412:5444:packages/flutter/lib/src/widgets/navigator.dart
  /// Replaces a route on the navigator with a new route. The route to be
  /// replaced is the one below the given `anchorRoute`.
  ///
  /// {@macro flutter.widgets.navigator.restorableReplaceRouteBelow}
  ///
  /// {@macro flutter.widgets.navigator.replaceRouteBelow}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePush}
  ///
  /// {@macro flutter.widgets.Navigator.restorablePushNamed.returnValue}
  @optionalTypeArgs
  String restorableReplaceRouteBelow<T extends Object?>({
    required Route<dynamic> anchorRoute,
    required RestorableRouteBuilder<T> newRouteBuilder,
    Object? arguments,
  }) {
    assert(anchorRoute._navigator == this);
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
    ).toRouteEntry(this, initialState: _RouteLifecycle.replace);
    _replaceEntryBelow(entry, anchorRoute);
    return entry.restorationId!;
  }
```

## 6. _replaceEntryBelow - 内部替换下方路由实现

```dart 5446:5473:packages/flutter/lib/src/widgets/navigator.dart
  void _replaceEntryBelow(_RouteEntry entry, Route<dynamic> anchorRoute) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    final int anchorIndex = _history.indexWhere(_RouteEntry.isRoutePredicate(anchorRoute));
    assert(anchorIndex >= 0, 'This Navigator does not contain the specified anchorRoute.');
    assert(
      _history[anchorIndex].isPresent,
      'The specified anchorRoute has already been removed from the Navigator.',
    );
    int index = anchorIndex - 1;
    while (index >= 0) {
      if (_history[index].isPresent) {
        break;
      }
      index -= 1;
    }
    assert(index >= 0, 'There are no routes below the specified anchorRoute.');
    _history.insert(index + 1, entry);
    _history[index].complete(null, isReplaced: true, imperativeRemoval: true);
    _flushHistoryUpdates();
    assert(() {
      _debugLocked = false;
      return true;
    }());
  }
```

### 功能说明

1. **查找锚点路由**：在历史记录中查找锚点路由的索引
2. **查找下方路由**：从锚点路由向前查找第一个活动路由
3. **验证存在**：确保找到了下方路由
4. **插入新路由**：将新路由插入到下方路由之后
5. **完成旧路由**：标记旧路由为完成（被替换）
6. **刷新更新**：应用所有更改

### 关键点

- **跳过非活动路由**：只查找活动路由（`isPresent`）
- **位置保持**：新路由插入到旧路由之后
- **无导航后处理**：替换的不是当前路由，不需要 `_afterNavigation`

## 7. removeRoute - 移除指定路由

```dart 5629:5651:packages/flutter/lib/src/widgets/navigator.dart
  /// Immediately remove `route` from the navigator, and [Route.dispose] it.
  ///
  /// {@macro flutter.widgets.navigator.removeRoute}
  @optionalTypeArgs
  void removeRoute<T extends Object?>(Route<T> route, [T? result]) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    assert(route._navigator == this);
    final bool wasCurrent = route.isCurrent;
    final _RouteEntry entry = _history.firstWhere(_RouteEntry.isRoutePredicate(route));
    entry.complete(result, isReplaced: false, imperativeRemoval: true);
    _flushHistoryUpdates(rearrangeOverlay: false);
    assert(() {
      _debugLocked = false;
      return true;
    }());
    if (wasCurrent) {
      _afterNavigation(_lastRouteEntryWhereOrNull(_RouteEntry.isPresentPredicate)?.route);
    }
  }
```

### 功能说明

- **作用**：立即移除指定路由并销毁它
- **参数**：
  - **route**：要移除的路由（必须属于当前导航器）
  - **result**：传递给路由的结果（可选）
- **行为**：路由被标记为完成并立即移除，不等待动画

### 使用场景

- 移除不需要的路由
- 清理导航栈
- 处理异常情况

### 使用示例

```dart
// 获取当前路由
final currentRoute = ModalRoute.of(context)!;

// 移除当前路由
Navigator.of(context).removeRoute(currentRoute);

// 移除并传递结果
Navigator.of(context).removeRoute(currentRoute, 'removed');
```

## 8. removeRouteBelow - 移除下方路由

```dart 5653:5685:packages/flutter/lib/src/widgets/navigator.dart
  /// Immediately remove a route from the navigator, and [Route.dispose] it. The
  /// route to be removed is the one below the given `anchorRoute`.
  ///
  /// {@macro flutter.widgets.navigator.removeRouteBelow}
  @optionalTypeArgs
  void removeRouteBelow<T extends Object?>(Route<T> anchorRoute, [T? result]) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    assert(anchorRoute._navigator == this);
    final int anchorIndex = _history.indexWhere(_RouteEntry.isRoutePredicate(anchorRoute));
    assert(anchorIndex >= 0, 'This Navigator does not contain the specified anchorRoute.');
    assert(
      _history[anchorIndex].isPresent,
      'The specified anchorRoute has already been removed from the Navigator.',
    );
    int index = anchorIndex - 1;
    while (index >= 0) {
      if (_history[index].isPresent) {
        break;
      }
      index -= 1;
    }
    assert(index >= 0, 'There are no routes below the specified anchorRoute.');
    _history[index].complete(result, isReplaced: false, imperativeRemoval: true);
    _flushHistoryUpdates(rearrangeOverlay: false);
    assert(() {
      _debugLocked = false;
      return true;
    }());
  }
```

### 功能说明

- **作用**：立即移除指定路由下方的路由并销毁它
- **参数**：
  - **anchorRoute**：锚点路由（用于定位）
  - **result**：传递给路由的结果（可选）
- **行为**：找到锚点路由下方的第一个活动路由并移除它

### 使用场景

- 移除当前路由下方的路由
- 清理导航栈中的特定路由
- 处理导航栈结构问题

### 使用示例

```dart
// 获取当前路由
final currentRoute = ModalRoute.of(context)!;

// 移除当前路由下方的路由
Navigator.of(context).removeRouteBelow(currentRoute);
```

## 方法对比表

| 方法 | 功能 | 参数 | 可恢复 |
| --- | --- | --- | --- |
| `replace` | 替换指定路由 | `oldRoute`, `newRoute` | ❌ |
| `restorableReplace` | 替换指定路由 | `oldRoute`, `newRouteBuilder` | ✅ |
| `replaceRouteBelow` | 替换下方路由 | `anchorRoute`, `newRoute` | ❌ |
| `restorableReplaceRouteBelow` | 替换下方路由 | `anchorRoute`, `newRouteBuilder` | ✅ |
| `removeRoute` | 移除指定路由 | `route`, `result?` | ❌ |
| `removeRouteBelow` | 移除下方路由 | `anchorRoute`, `result?` | ❌ |

## replace vs pushReplacement

| 特性 | replace | pushReplacement |
| --- | --- | --- |
| 目标 | 任意路由 | 当前路由 |
| 位置 | 保持原位置 | 栈顶 |
| 使用场景 | 更新历史记录 | 替换当前页面 |

## 注意事项

1. **路由归属**：所有路由参数必须属于当前导航器
2. **路由状态**：路由必须处于活动状态（`isPresent`）
3. **立即执行**：移除操作是立即的，不等待动画
4. **当前路由处理**：移除当前路由时需要触发导航后处理
5. **静态函数**：可恢复方法的 `routeBuilder` 必须是静态函数
6. **参数序列化**：可恢复方法的参数必须可序列化

## 使用建议

1. **替换路由**：使用 `replace` 更新历史记录中的路由
2. **移除路由**：使用 `removeRoute` 清理不需要的路由
3. **下方操作**：使用 `replaceRouteBelow` 或 `removeRouteBelow` 操作下方路由
4. **需要恢复**：使用可恢复版本支持状态恢复

## 相关文档

- [NavigatorState 直接路由操作方法](navigator.dart_NavigatorState_6_直接路由操作方法.md)
- [NavigatorState 路由弹出与工具方法](navigator.dart_NavigatorState_8_路由弹出与工具方法.md)
- [Navigator 路由替换与移除](navigator.dart_Navigator_5_路由替换与移除.md)
