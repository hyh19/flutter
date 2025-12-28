# ModalRoute 类详解 - 第十部分：通知分发与状态变化处理

## 概述

`ModalRoute` 提供了通知分发机制和状态变化处理方法，用于在路由状态改变时通知相关组件，并处理路由之间的交互。

## _maybeDispatchNavigationNotification

```dart 2111:2136:packages/flutter/lib/src/widgets/routes.dart
  void _maybeDispatchNavigationNotification() {
    if (!isCurrent) {
      return;
    }
    final NavigationNotification notification = NavigationNotification(
      // canPop indicates that the originator of the Notification can handle a
      // pop. In the case of PopScope, it handles pops when canPop is
      // false. Hence the seemingly backward logic here.
      canHandlePop: popDisposition == RoutePopDisposition.doNotPop || _willPopCallbacks.isNotEmpty,
    );
    // Avoid dispatching a notification in the middle of a build.
    switch (SchedulerBinding.instance.schedulerPhase) {
      case SchedulerPhase.postFrameCallbacks:
        notification.dispatch(subtreeContext);
      case SchedulerPhase.idle:
      case SchedulerPhase.midFrameMicrotasks:
      case SchedulerPhase.persistentCallbacks:
      case SchedulerPhase.transientCallbacks:
        SchedulerBinding.instance.addPostFrameCallback((Duration timeStamp) {
          if (!(subtreeContext?.mounted ?? false)) {
            return;
          }
          notification.dispatch(subtreeContext);
        }, debugLabel: 'ModalRoute.dispatchNotification');
    }
  }
```

**功能**：可能分发导航通知，通知路由子树关于 Pop 操作的状态。

### 实现逻辑

#### 1. 条件检查

```dart
if (!isCurrent) {
  return;
}
```

只有当路由是当前路由（`isCurrent` 为 `true`）时才分发通知。

#### 2. 创建通知

```dart
final NavigationNotification notification = NavigationNotification(
  canHandlePop: popDisposition == RoutePopDisposition.doNotPop || _willPopCallbacks.isNotEmpty,
);
```

创建一个 `NavigationNotification`，其中 `canHandlePop` 的逻辑看起来是反向的：

- **注释说明**：`canHandlePop` 表示通知的发起者可以处理 Pop 操作。对于 `PopScope`，当 `canPop` 为 `false` 时它会处理 Pop。因此这里的逻辑看起来是反向的。
- **实际逻辑**：如果 `popDisposition` 是 `doNotPop` 或存在 `_willPopCallbacks`，则 `canHandlePop` 为 `true`

#### 3. 调度通知

根据当前的调度器阶段选择不同的分发方式：

- **postFrameCallbacks 阶段**：直接分发通知
- **其他阶段**：通过 `addPostFrameCallback` 在帧结束后分发

**设计原因**：避免在构建过程中分发通知，确保通知在合适的时机被处理。

### 调用时机

`_maybeDispatchNavigationNotification` 在以下情况下被调用：

1. 注册 `PopEntry` 时（`registerPopEntry`）
2. 注销 `PopEntry` 时（`unregisterPopEntry`）
3. 添加 `WillPopCallback` 时（已废弃，`addScopedWillPopCallback`）
4. 移除 `WillPopCallback` 时（已废弃，`removeScopedWillPopCallback`）
5. `didPopNext` 时（当上层路由被弹出后）

## hasScopedWillPopCallback

```dart 2138:2162:packages/flutter/lib/src/widgets/routes.dart
  /// True if one or more [WillPopCallback] callbacks exist.
  ///
  /// This method is used to disable the horizontal swipe pop gesture supported
  /// by [MaterialPageRoute] for [TargetPlatform.iOS] and
  /// [TargetPlatform.macOS]. If a pop might be vetoed, then the back gesture is
  /// disabled.
  ///
  /// The [buildTransitions] method will not be called again if this changes,
  /// since it can change during the build as descendants of the route add or
  /// remove callbacks.
  ///
  /// See also:
  ///
  ///  * [addScopedWillPopCallback], which adds a callback.
  ///  * [removeScopedWillPopCallback], which removes a callback.
  ///  * [willHandlePopInternally], which reports on another reason why
  ///    a pop might be vetoed.
  @Deprecated(
    'Use popDisposition instead. '
    'This feature was deprecated after v3.12.0-1.0.pre.',
  )
  @protected
  bool get hasScopedWillPopCallback {
    return _willPopCallbacks.isNotEmpty;
  }
```

**状态**：已废弃，应使用 `popDisposition` 代替。

**功能**：判断是否存在一个或多个 `WillPopCallback` 回调。

**历史用途**：用于禁用 iOS 和 macOS 上的水平滑动返回手势。如果 Pop 操作可能被否决，则禁用返回手势。

## didChangePrevious

```dart 2164:2168:packages/flutter/lib/src/widgets/routes.dart
  @override
  void didChangePrevious(Route<dynamic>? previousRoute) {
    super.didChangePrevious(previousRoute);
    changedInternalState();
  }
```

**功能**：当前一个路由改变时调用。

**实现**：

1. 调用父类的 `didChangePrevious` 方法
2. 调用 `changedInternalState()` 通知内部状态改变

**使用场景**：当路由栈发生变化，当前路由的前一个路由改变时，需要更新内部状态（例如，更新屏障的显示）。

## didChangeNext

```dart 2170:2181:packages/flutter/lib/src/widgets/routes.dart
  @override
  void didChangeNext(Route<dynamic>? nextRoute) {
    if (nextRoute is ModalRoute<T> &&
        canTransitionTo(nextRoute) &&
        nextRoute.delegatedTransition != delegatedTransition) {
      receivedTransition = nextRoute.delegatedTransition;
    } else {
      receivedTransition = null;
    }
    super.didChangeNext(nextRoute);
    changedInternalState();
  }
```

**功能**：当下一个路由改变时调用。

**实现逻辑**：

1. **检查委托过渡**：
   - 如果下一个路由是 `ModalRoute<T>`
   - 且可以过渡到该路由（`canTransitionTo(nextRoute)`）
   - 且下一个路由的 `delegatedTransition` 与当前路由不同
   - 则将下一个路由的 `delegatedTransition` 设置为 `receivedTransition`
   - 否则将 `receivedTransition` 设置为 `null`

2. **调用父类方法**：调用 `super.didChangeNext(nextRoute)`

3. **通知状态改变**：调用 `changedInternalState()`

**设计目的**：处理路由之间的委托过渡协调，确保过渡动画的连贯性。

## didPopNext

```dart 2183:2195:packages/flutter/lib/src/widgets/routes.dart
  @override
  void didPopNext(Route<dynamic> nextRoute) {
    if (nextRoute is ModalRoute<T> &&
        canTransitionTo(nextRoute) &&
        nextRoute.delegatedTransition != delegatedTransition) {
      receivedTransition = nextRoute.delegatedTransition;
    } else {
      receivedTransition = null;
    }
    super.didPopNext(nextRoute);
    changedInternalState();
    _maybeDispatchNavigationNotification();
  }
```

**功能**：当上层路由被弹出，当前路由重新成为顶部路由时调用。

**实现逻辑**：

1. **处理委托过渡**：与 `didChangeNext` 相同的逻辑
2. **调用父类方法**：调用 `super.didPopNext(nextRoute)`
3. **通知状态改变**：调用 `changedInternalState()`
4. **分发导航通知**：调用 `_maybeDispatchNavigationNotification()`

**额外操作**：与 `didChangeNext` 相比，`didPopNext` 额外调用了 `_maybeDispatchNavigationNotification()`，因为当前路由重新成为活动路由，需要通知子树关于 Pop 操作的状态。

## changedInternalState

```dart 2197:2208:packages/flutter/lib/src/widgets/routes.dart
  @override
  void changedInternalState() {
    super.changedInternalState();
    // No need to mark dirty if this method is called during build phase.
    if (SchedulerBinding.instance.schedulerPhase != SchedulerPhase.persistentCallbacks) {
      setState(() {
        /* internal state already changed */
      });
      _modalBarrier.markNeedsBuild();
    }
    _modalScope.maintainState = maintainState;
  }
```

**功能**：当路由的内部状态改变时调用。

**实现逻辑**：

1. **调用父类方法**：调用 `super.changedInternalState()`

2. **条件更新**：
   - 如果当前不在构建阶段（`persistentCallbacks`），则：
     - 调用 `setState` 触发重建（虽然状态已经改变，但需要触发 `buildTransitions`）
     - 标记 `_modalBarrier` 需要重建

3. **更新状态维护**：更新 `_modalScope.maintainState` 为当前的 `maintainState` 值

**性能优化**：避免在构建阶段调用 `setState`，防止重复构建。

## changedExternalState

```dart 2210:2217:packages/flutter/lib/src/widgets/routes.dart
  @override
  void changedExternalState() {
    super.changedExternalState();
    _modalBarrier.markNeedsBuild();
    if (_scopeKey.currentState != null) {
      _scopeKey.currentState!._forceRebuildPage();
    }
  }
```

**功能**：当路由的外部状态改变时调用（例如，`Navigator` 的依赖改变）。

**实现逻辑**：

1. **调用父类方法**：调用 `super.changedExternalState()`

2. **标记屏障重建**：标记 `_modalBarrier` 需要重建

3. **强制重建页面**：如果 `_scopeKey.currentState` 不为 `null`，调用 `_forceRebuildPage()` 强制重建页面

**使用场景**：当 `Navigator` 的配置改变（例如主题改变）时，需要重建路由的 UI。

## 便捷属性

### canPop

```dart 2219:2227:packages/flutter/lib/src/widgets/routes.dart
  /// Whether this route can be popped.
  ///
  /// A route can be popped if there is at least one active route below it, or
  /// if [willHandlePopInternally] returns true.
  ///
  /// When this changes, if the route is visible, the route will
  /// rebuild, and any widgets that used [ModalRoute.of] will be
  /// notified.
  bool get canPop => hasActiveRouteBelow || willHandlePopInternally;
```

**功能**：判断路由是否可以被弹出。

**实现**：路由可以被弹出，如果：

- 下方至少有一个活动路由（`hasActiveRouteBelow`），或
- 路由会内部处理弹出（`willHandlePopInternally`）

### impliesAppBarDismissal

```dart 2229:2235:packages/flutter/lib/src/widgets/routes.dart
  /// Whether an [AppBar] in the route should automatically add a back button or
  /// close button.
  ///
  /// This getter returns true if there is at least one active route below it,
  /// or there is at least one [LocalHistoryEntry] with [impliesAppBarDismissal]
  /// set to true
  bool get impliesAppBarDismissal => hasActiveRouteBelow || _entriesImpliesAppBarDismissal > 0;
```

**功能**：判断路由中的 `AppBar` 是否应该自动添加返回按钮或关闭按钮。

**实现**：返回 `true`，如果：

- 下方至少有一个活动路由，或
- 至少有一个 `LocalHistoryEntry` 的 `impliesAppBarDismissal` 为 `true`

### fullscreenDialog

```dart 2237:2240:packages/flutter/lib/src/widgets/routes.dart
  /// {@macro flutter.widgets.RawDialogRoute.fullscreenDialog}
  // TODO(dkwingsmt): Rename `ModalRoute.fullscreenDialog` something semantically suitable for a modal.
  // https://github.com/flutter/flutter/issues/168949
  bool get fullscreenDialog => false;
```

**功能**：指示路由是否是全屏对话框。

**默认值**：`false`

**注意**：代码中有 TODO 注释，说明这个属性可能需要重命名为更语义化的名称。

## 状态变化流程

### 内部状态变化流程

```text
内部状态改变
    ↓
changedInternalState()
    ↓
检查是否在构建阶段
    ↓
如果不在构建阶段：
  - 调用 setState()
  - 标记 _modalBarrier 需要重建
    ↓
更新 _modalScope.maintainState
```

### 外部状态变化流程

```text
外部状态改变
    ↓
changedExternalState()
    ↓
标记 _modalBarrier 需要重建
    ↓
如果 _scopeKey.currentState 存在：
  - 调用 _forceRebuildPage()
```

## 总结

第十部分介绍了 `ModalRoute` 的通知分发和状态变化处理机制：

1. **_maybeDispatchNavigationNotification**：分发导航通知，通知路由子树关于 Pop 操作的状态

2. **didChangePrevious / didChangeNext / didPopNext**：处理路由栈变化，协调委托过渡

3. **changedInternalState / changedExternalState**：处理内部和外部状态变化，触发必要的重建

4. **便捷属性**：`canPop`、`impliesAppBarDismissal`、`fullscreenDialog` 提供常用的状态查询

这些机制确保了路由状态变化时的正确响应和 UI 更新。
