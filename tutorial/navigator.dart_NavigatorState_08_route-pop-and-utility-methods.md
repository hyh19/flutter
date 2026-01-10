# NavigatorState 路由弹出与工具方法详解

## 概述

本文档介绍 `NavigatorState` 中的路由弹出方法、用户手势处理、工具方法和构建方法。这些方法提供了路由栈的查询、弹出操作、用户交互处理和 UI 构建功能。

## 1. canPop - 检查是否可以弹出

```dart 5475:5498:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether the navigator can be popped.
  ///
  /// {@macro flutter.widgets.navigator.canPop}
  ///
  /// See also:
  ///
  ///  * [Route.isFirst], which returns true for routes for which [canPop]
  ///    returns false.
  bool canPop() {
    final Iterator<_RouteEntry> iterator = _history.where(_RouteEntry.isPresentPredicate).iterator;
    if (!iterator.moveNext()) {
      // We have no active routes, so we can't pop.
      return false;
    }
    if (iterator.current.route.willHandlePopInternally) {
      // The first route can handle pops itself, so we can pop.
      return true;
    }
    if (!iterator.moveNext()) {
      // There's only one route, so we can't pop.
      return false;
    }
    return true; // there's at least two routes, so we can pop
  }
```

### 功能说明

- **作用**：检查导航器是否可以弹出路由
- **返回**：`true` 如果可以弹出，`false` 否则
- **逻辑**：
  1. 如果没有活动路由，返回 `false`
  2. 如果第一个路由可以内部处理弹出，返回 `true`
  3. 如果只有一个路由，返回 `false`
  4. 如果有至少两个路由，返回 `true`

### 使用场景

- 决定是否显示返回按钮
- 检查是否可以执行弹出操作
- 处理系统返回按钮

### 使用示例

```dart
// 检查是否可以弹出
if (Navigator.of(context).canPop()) {
  // 显示返回按钮
  IconButton(
    icon: Icon(Icons.arrow_back),
    onPressed: () => Navigator.of(context).pop(),
  );
} else {
  // 显示关闭按钮或其他操作
}
```

## 2. maybePop - 尝试弹出

```dart 5500:5546:packages/flutter/lib/src/widgets/navigator.dart
  /// Consults the current route's [Route.popDisposition] method, and acts
  /// accordingly, potentially popping the route as a result; returns whether
  /// the pop request should be considered handled.
  ///
  /// {@macro flutter.widgets.navigator.maybePop}
  ///
  /// See also:
  ///
  ///  * [Form], which provides a [Form.canPop] boolean that enables the
  ///    form to prevent any [pop]s initiated by the app's back button.
  ///  * [ModalRoute], which provides a `scopedOnPopCallback` that can be used
  ///    to define the route's `willPop` method.
  @optionalTypeArgs
  Future<bool> maybePop<T extends Object?>([T? result]) async {
    final _RouteEntry? lastEntry = _lastRouteEntryWhereOrNull(_RouteEntry.isPresentPredicate);
    if (lastEntry == null) {
      return false;
    }
    assert(lastEntry.route._navigator == this);

    // TODO(justinmc): When the deprecated willPop method is removed, delete
    // this code and use only popDisposition, below.
    if (await lastEntry.route.willPop() == RoutePopDisposition.doNotPop) {
      return true;
    }
    if (!mounted) {
      // Forget about this pop, we were disposed in the meantime.
      return true;
    }

    final _RouteEntry? newLastEntry = _lastRouteEntryWhereOrNull(_RouteEntry.isPresentPredicate);
    if (lastEntry != newLastEntry) {
      // Forget about this pop, something happened to our history in the meantime.
      return true;
    }

    switch (lastEntry.route.popDisposition) {
      case RoutePopDisposition.bubble:
        return false;
      case RoutePopDisposition.pop:
        pop(result);
        return true;
      case RoutePopDisposition.doNotPop:
        lastEntry.route.onPopInvokedWithResult(false, result);
        return true;
    }
  }
```

### 功能说明

- **作用**：尝试弹出当前路由，根据路由的 `popDisposition` 决定行为
- **返回**：`true` 如果弹出请求已处理，`false` 如果应该向上冒泡
- **行为**：
  - **bubble**：返回 `false`，让系统处理
  - **pop**：执行弹出，返回 `true`
  - **doNotPop**：不弹出，但通知路由，返回 `true`

### 使用场景

- 处理系统返回按钮
- 实现自定义返回逻辑
- 处理表单验证等场景

### 使用示例

```dart
// 在 WillPopScope 中使用
WillPopScope(
  onWillPop: () async {
    final navigator = Navigator.of(context);
    final handled = await navigator.maybePop();
    return handled;
  },
  child: MyPage(),
);
```

## 3. pop - 弹出当前路由

```dart 5548:5602:packages/flutter/lib/src/widgets/navigator.dart
  /// Pop the top-most route off the navigator.
  ///
  /// {@macro flutter.widgets.navigator.pop}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage for closing a route is as follows:
  ///
  /// ```dart
  /// void _handleClose() {
  ///   navigator.pop();
  /// }
  /// ```
  /// {@end-tool}
  /// {@tool snippet}
  ///
  /// A dialog box might be closed with a result:
  ///
  /// ```dart
  /// void _handleAccept() {
  ///   navigator.pop(true); // dialog returns true
  /// }
  /// ```
  /// {@end-tool}
  @optionalTypeArgs
  void pop<T extends Object?>([T? result]) {
    assert(!_debugLocked);
    assert(() {
      _debugLocked = true;
      return true;
    }());
    final _RouteEntry entry = _history.lastWhere(_RouteEntry.isPresentPredicate);
    if (entry.pageBased && widget.onPopPage != null) {
      if (widget.onPopPage!(entry.route, result)) {
        if (entry.currentState.index <= _RouteLifecycle.idle.index) {
          // The entry may have been disposed if the pop finishes synchronously.
          assert(entry.route._popCompleter.isCompleted);
          entry.currentState = _RouteLifecycle.pop;
        }
        entry.route.onPopInvokedWithResult(true, result);
      }
    } else {
      entry.pop<T>(result, imperativeRemoval: true);
      assert(entry.currentState == _RouteLifecycle.pop);
    }
    if (entry.currentState == _RouteLifecycle.pop) {
      _flushHistoryUpdates(rearrangeOverlay: false);
    }
    assert(entry.currentState == _RouteLifecycle.idle || entry.route._popCompleter.isCompleted);
    assert(() {
      _debugLocked = false;
      return true;
    }());
    _afterNavigation(entry.route);
  }
```

### 功能说明

- **作用**：弹出导航器栈顶的路由
- **参数**：`result` - 传递给前一个路由的结果（可选）
- **行为**：
  - 如果是页面路由且提供了 `onPopPage`，调用它
  - 否则直接调用路由的 `pop` 方法
  - 刷新历史记录更新

### 使用示例

```dart
// 基本弹出
Navigator.of(context).pop();

// 弹出并返回结果
Navigator.of(context).pop('success');

// 在对话框中使用
ElevatedButton(
  onPressed: () => Navigator.of(context).pop(true),
  child: Text('确认'),
);
```

## 4. popUntil - 弹出直到条件

```dart 5604:5627:packages/flutter/lib/src/widgets/navigator.dart
  /// Calls [pop] repeatedly until the predicate returns true.
  ///
  /// {@macro flutter.widgets.navigator.popUntil}
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// void _doLogout() {
  ///   navigator.popUntil(ModalRoute.withName('/'));
  /// }
  /// ```
  /// {@end-tool}
  void popUntil(RoutePredicate predicate) {
    _RouteEntry? candidate = _lastRouteEntryWhereOrNull(_RouteEntry.isPresentPredicate);
    while (candidate != null) {
      if (predicate(candidate.route)) {
        return;
      }
      pop();
      candidate = _lastRouteEntryWhereOrNull(_RouteEntry.isPresentPredicate);
    }
  }
```

### 功能说明

- **作用**：重复调用 `pop()` 直到断言返回 `true`
- **参数**：`predicate` - 路由断言函数
- **行为**：从栈顶开始弹出，直到找到满足条件的路由

### 使用场景

- 返回到根路由
- 返回到特定路由
- 清理导航栈

### 使用示例

```dart
// 返回到根路由
Navigator.of(context).popUntil(ModalRoute.withName('/'));

// 返回到登录页
Navigator.of(context).popUntil((route) => route.settings.name == '/login');

// 返回到第一个路由
Navigator.of(context).popUntil((route) => route.isFirst);
```

## 5. finalizeRoute - 完成路由生命周期

```dart 5687:5734:packages/flutter/lib/src/widgets/navigator.dart
  /// Complete the lifecycle for a route that has been popped off the navigator.
  ///
  /// When the navigator pops a route, the navigator retains a reference to the
  /// route in order to call [Route.dispose] if the navigator itself is removed
  /// from the tree. When the route is finished with any exit animation, the
  /// route should call this function to complete its lifecycle (e.g., to
  /// receive a call to [Route.dispose]).
  ///
  /// The given `route` must have already received a call to [Route.didPop].
  /// This function may be called directly from [Route.didPop] if [Route.didPop]
  /// will return true.
  void finalizeRoute(Route<dynamic> route) {
    // FinalizeRoute may have been called while we were already locked as a
    // responds to route.didPop(). Make sure to leave in the state we were in
    // before the call.
    bool? wasDebugLocked;
    assert(() {
      wasDebugLocked = _debugLocked;
      _debugLocked = true;
      return true;
    }());
    assert(_history.where(_RouteEntry.isRoutePredicate(route)).length == 1);
    final int index = _history.indexWhere(_RouteEntry.isRoutePredicate(route));
    final _RouteEntry entry = _history[index];
    // For page-based route with zero transition, the finalizeRoute can be
    // called on any life cycle above pop.
    if (entry.pageBased && entry.currentState.index < _RouteLifecycle.pop.index) {
      _observedRouteDeletions.add(
        _NavigatorPopObservation(
          route,
          _getRouteBefore(index - 1, _RouteEntry.willBePresentPredicate)?.route,
        ),
      );
    } else {
      assert(entry.currentState == _RouteLifecycle.popping);
    }
    entry.finalize();
    // finalizeRoute can be called during _flushHistoryUpdates if a pop
    // finishes synchronously.
    if (!_flushingHistory) {
      _flushHistoryUpdates(rearrangeOverlay: false);
    }

    assert(() {
      _debugLocked = wasDebugLocked!;
      return true;
    }());
  }
```

### 功能说明

- **作用**：完成路由的生命周期，通常在路由退出动画完成后调用
- **调用时机**：路由的 `didPop` 方法返回 `true` 后
- **行为**：
  - 标记路由为最终状态
  - 如果不在刷新过程中，刷新历史记录
  - 保持锁定状态一致

### 使用场景

- 自定义路由实现
- 处理路由生命周期
- 清理路由资源

## 6. 用户手势处理

### didStartUserGesture - 开始用户手势

```dart 5761:5782:packages/flutter/lib/src/widgets/navigator.dart
  /// The navigator is being controlled by a user gesture.
  ///
  /// For example, called when the user beings an iOS back gesture.
  ///
  /// When the gesture finishes, call [didStopUserGesture].
  void didStartUserGesture() {
    _userGesturesInProgress += 1;
    if (_userGesturesInProgress == 1) {
      final int routeIndex = _getIndexBefore(
        _history.length - 1,
        _RouteEntry.willBePresentPredicate,
      );
      final Route<dynamic> route = _history[routeIndex].route;
      Route<dynamic>? previousRoute;
      if (!route.willHandlePopInternally && routeIndex > 0) {
        previousRoute = _getRouteBefore(routeIndex - 1, _RouteEntry.willBePresentPredicate)!.route;
      }
      for (final NavigatorObserver observer in _effectiveObservers) {
        observer.didStartUserGesture(route, previousRoute);
      }
    }
  }
```

### didStopUserGesture - 停止用户手势

```dart 5784:5796:packages/flutter/lib/src/widgets/navigator.dart
  /// A user gesture completed.
  ///
  /// Notifies the navigator that a gesture regarding which the navigator was
  /// previously notified with [didStartUserGesture] has completed.
  void didStopUserGesture() {
    assert(_userGesturesInProgress > 0);
    _userGesturesInProgress -= 1;
    if (_userGesturesInProgress == 0) {
      for (final NavigatorObserver observer in _effectiveObservers) {
        observer.didStopUserGesture();
      }
    }
  }
```

### userGestureInProgress - 用户手势进行中

```dart 5749:5759:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether a route is currently being manipulated by the user, e.g.
  /// as during an iOS back gesture.
  ///
  /// See also:
  ///
  ///  * [userGestureInProgressNotifier], which notifies its listeners if
  ///    the value of [userGestureInProgress] changes.
  bool get userGestureInProgress => userGestureInProgressNotifier.value;

  /// Notifies its listeners if the value of [userGestureInProgress] changes.
  final ValueNotifier<bool> userGestureInProgressNotifier = ValueNotifier<bool>(false);
```

### 功能说明

- **作用**：跟踪用户手势状态（如 iOS 边缘返回手势）
- **计数**：支持多个并发手势（使用计数器）
- **通知**：通过 `ValueNotifier` 通知监听者状态变化

## 7. 指针事件处理

### _handlePointerDown - 处理指针按下

```dart 5800:5802:packages/flutter/lib/src/widgets/navigator.dart
  void _handlePointerDown(PointerDownEvent event) {
    _activePointers.add(event.pointer);
  }
```

### _handlePointerUpOrCancel - 处理指针抬起或取消

```dart 5804:5806:packages/flutter/lib/src/widgets/navigator.dart
  void _handlePointerUpOrCancel(PointerEvent event) {
    _activePointers.remove(event.pointer);
  }
```

### _cancelActivePointers - 取消活动指针

```dart 5808:5823:packages/flutter/lib/src/widgets/navigator.dart
  void _cancelActivePointers() {
    // TODO(abarth): This mechanism is far from perfect. See https://github.com/flutter/flutter/issues/4770
    if (SchedulerBinding.instance.schedulerPhase == SchedulerPhase.idle) {
      // If we're between frames (SchedulerPhase.idle) then absorb any
      // subsequent pointers from this frame. The absorbing flag will be
      // reset in the next frame, see build().
      final RenderAbsorbPointer? absorber = _overlayKey.currentContext
          ?.findAncestorRenderObjectOfType<RenderAbsorbPointer>();
      setState(() {
        absorber?.absorbing = true;
        // We do this in setState so that we'll reset the absorbing value back
        // to false on the next frame.
      });
    }
    _activePointers.toList().forEach(WidgetsBinding.instance.cancelPointer);
  }
```

### 功能说明

- **作用**：在导航过程中取消活动的指针事件，防止交互冲突
- **时机**：在 `_afterNavigation` 中调用
- **机制**：设置 `AbsorbPointer` 吸收后续指针事件

## 8. 工具方法

### _firstRouteEntryWhereOrNull - 查找第一个匹配的路由条目

```dart 5825:5833:packages/flutter/lib/src/widgets/navigator.dart
  /// Gets first route entry satisfying the predicate, or null if not found.
  _RouteEntry? _firstRouteEntryWhereOrNull(_RouteEntryPredicate test) {
    for (final _RouteEntry element in _history) {
      if (test(element)) {
        return element;
      }
    }
    return null;
  }
```

### _lastRouteEntryWhereOrNull - 查找最后一个匹配的路由条目

```dart 5835:5844:packages/flutter/lib/src/widgets/navigator.dart
  /// Gets last route entry satisfying the predicate, or null if not found.
  _RouteEntry? _lastRouteEntryWhereOrNull(_RouteEntryPredicate test) {
    _RouteEntry? result;
    for (final _RouteEntry element in _history) {
      if (test(element)) {
        result = element;
      }
    }
    return result;
  }
```

### _getRouteById - 通过 ID 获取路由

```dart 5736:5740:packages/flutter/lib/src/widgets/navigator.dart
  @optionalTypeArgs
  Route<T>? _getRouteById<T>(String id) {
    return _firstRouteEntryWhereOrNull((_RouteEntry entry) => entry.restorationId == id)?.route
        as Route<T>?;
  }
```

## 9. build - 构建 UI

```dart 5846:5900:packages/flutter/lib/src/widgets/navigator.dart
  @protected
  @override
  Widget build(BuildContext context) {
    assert(!_debugLocked);
    assert(_history.isNotEmpty);

    // Hides the HeroControllerScope for the widget subtree so that the other
    // nested navigator underneath will not pick up the hero controller above
    // this level.
    return HeroControllerScope.none(
      child: NotificationListener<NavigationNotification>(
        onNotification: (NavigationNotification notification) {
          // If the state of this Navigator does not change whether or not the
          // whole framework can pop, propagate the Notification as-is.
          if (notification.canHandlePop || !canPop()) {
            return false;
          }
          // Otherwise, dispatch a new Notification with the correct canPop and
          // stop the propagation of the old Notification.
          const NavigationNotification nextNotification = NavigationNotification(
            canHandlePop: true,
          );
          nextNotification.dispatch(context);
          return true;
        },
        child: Listener(
          onPointerDown: _handlePointerDown,
          onPointerUp: _handlePointerUpOrCancel,
          onPointerCancel: _handlePointerUpOrCancel,
          child: AbsorbPointer(
            absorbing: false, // it's mutated directly by _cancelActivePointers above
            child: FocusTraversalGroup(
              policy: FocusTraversalGroup.maybeOf(context),
              child: Focus(
                focusNode: focusNode,
                autofocus: true,
                skipTraversal: true,
                includeSemantics: false,
                child: UnmanagedRestorationScope(
                  bucket: bucket,
                  child: Overlay(
                    key: _overlayKey,
                    clipBehavior: widget.clipBehavior,
                    initialEntries: overlay == null
                        ? _allRouteOverlayEntries.toList(growable: false)
                        : const <OverlayEntry>[],
                  ),
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
```

### 功能说明

1. **HeroControllerScope.none**：隐藏 Hero 控制器作用域，防止嵌套导航器继承
2. **NavigationNotification 处理**：处理导航通知，决定是否可以弹出
3. **指针事件监听**：监听指针事件，用于手势处理
4. **焦点管理**：管理导航器的焦点状态
5. **状态恢复**：提供恢复作用域
6. **Overlay**：渲染所有路由的 Overlay 条目

### Widget 树结构

```text
HeroControllerScope.none
└── NotificationListener<NavigationNotification>
    └── Listener (指针事件)
        └── AbsorbPointer
            └── FocusTraversalGroup
                └── Focus
                    └── UnmanagedRestorationScope
                        └── Overlay
                            └── 所有路由的 OverlayEntry
```

## 方法总结

| 方法 | 功能 | 返回类型 |
| --- | --- | --- |
| `canPop()` | 检查是否可以弹出 | `bool` |
| `maybePop()` | 尝试弹出 | `Future<bool>` |
| `pop()` | 弹出当前路由 | `void` |
| `popUntil()` | 弹出直到条件 | `void` |
| `finalizeRoute()` | 完成路由生命周期 | `void` |
| `didStartUserGesture()` | 开始用户手势 | `void` |
| `didStopUserGesture()` | 停止用户手势 | `void` |
| `userGestureInProgress` | 用户手势进行中 | `bool` |

## 注意事项

1. **canPop 检查**：在执行弹出操作前应该检查 `canPop()`
2. **maybePop 使用**：处理系统返回按钮时使用 `maybePop()`
3. **结果传递**：使用 `pop(result)` 传递结果给前一个路由
4. **手势处理**：用户手势处理主要用于 iOS 边缘返回手势
5. **指针取消**：导航时会自动取消活动指针，防止交互冲突

## 相关文档

- [NavigatorState 路由替换与移除方法](navigator.dart_NavigatorState_7_路由替换与移除方法.md)
- [NavigatorState 历史记录刷新机制](navigator.dart_NavigatorState_4_历史记录刷新机制.md)
- [Navigator 工具方法](navigator.dart_Navigator_6_工具方法.md)
