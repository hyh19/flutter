# ModalRoute Pop操作管理详解

## 概述

`ModalRoute` 提供了完整的 Pop 操作管理机制，包括已废弃的 `willPop` 方法和新的 `popDisposition`、`onPopInvokedWithResult` 方法。这些方法允许路由控制是否可以弹出，以及在弹出时执行清理操作。

## 已废弃的方法：willPop

```dart 1959:1991:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [RoutePopDisposition.doNotPop] if any of callbacks added with
  /// [addScopedWillPopCallback] returns either false or null. If they all
  /// return true, the base [Route.willPop]'s result will be returned. The
  /// callbacks will be called in the order they were added, and will only be
  /// called if all previous callbacks returned true.
  ///
  /// Typically this method is not overridden because applications usually
  /// don't create modal routes directly, they use higher level primitives
  /// like [showDialog]. The scoped [WillPopCallback] list makes it possible
  /// for ModalRoute descendants to collectively define the value of [willPop].
  ///
  /// See also:
  ///
  ///  * [Form], which provides an `onWillPop` callback that uses this mechanism.
  ///  * [addScopedWillPopCallback], which adds a callback to the list this
  ///    method checks.
  ///  * [removeScopedWillPopCallback], which removes a callback from the list
  ///    this method checks.
  @Deprecated(
    'Use popDisposition instead. '
    'This feature was deprecated after v3.12.0-1.0.pre.',
  )
  @override
  Future<RoutePopDisposition> willPop() async {
    final _ModalScopeState<T>? scope = _scopeKey.currentState;
    assert(scope != null);
    for (final WillPopCallback callback in List<WillPopCallback>.of(_willPopCallbacks)) {
      if (!await callback()) {
        return RoutePopDisposition.doNotPop;
      }
    }
    return super.willPop();
  }
```

**状态**：已废弃，应使用 `popDisposition` 代替。

**功能**：检查是否可以弹出路由。

**实现逻辑**：

1. 遍历所有通过 `addScopedWillPopCallback` 添加的回调
2. 按添加顺序调用每个回调
3. 如果任何回调返回 `false` 或 `null`，返回 `RoutePopDisposition.doNotPop`
4. 只有当所有回调都返回 `true` 时，才调用父类的 `willPop()` 方法

**设计目的**：允许路由的子 widget（如 `Form`）通过注册回调来阻止路由弹出。

## popDisposition

```dart 1993:2019:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [RoutePopDisposition.doNotPop] if any of the [PopEntry] instances
  /// registered with [registerPopEntry] have [PopEntry.canPopNotifier] set to
  /// false.
  ///
  /// Typically this method is not overridden because applications usually
  /// don't create modal routes directly, they use higher level primitives
  /// like [showDialog]. The scoped [PopEntry] list makes it possible for
  /// ModalRoute descendants to collectively define the value of
  /// [popDisposition].
  ///
  /// See also:
  ///
  ///  * [Form], which provides an `onPopInvokedWithResult` callback that is similar.
  ///  * [registerPopEntry], which adds a [PopEntry] to the list this method
  ///    checks.
  ///  * [unregisterPopEntry], which removes a [PopEntry] from the list this
  ///    method checks.
  @override
  RoutePopDisposition get popDisposition {
    for (final PopEntry<Object?> popEntry in _popEntries) {
      if (!popEntry.canPopNotifier.value) {
        return RoutePopDisposition.doNotPop;
      }
    }

    return super.popDisposition;
  }
```

**功能**：返回路由的弹出处置方式，这是 `willPop` 的现代替代方案。

**实现逻辑**：

1. 遍历所有通过 `registerPopEntry` 注册的 `PopEntry` 实例
2. 检查每个 `PopEntry` 的 `canPopNotifier.value`
3. 如果任何 `PopEntry` 的 `canPopNotifier.value` 为 `false`，返回 `RoutePopDisposition.doNotPop`
4. 否则调用父类的 `popDisposition`

**与 willPop 的区别**：

- `willPop`：使用回调函数（已废弃）
- `popDisposition`：使用 `PopEntry` 和 `ValueNotifier`（现代方式）

**优势**：

- 更灵活：使用 `ValueNotifier` 可以动态改变值
- 更高效：避免异步回调的开销
- 更清晰：明确的状态管理

### PopEntry 数据结构

```dart 1953:1957:packages/flutter/lib/src/widgets/routes.dart
  final List<WillPopCallback> _willPopCallbacks = <WillPopCallback>[];

  // Holding as Object? instead of T so that PopScope in this route can be
  // declared with any supertype of T.
  final Set<PopEntry<Object?>> _popEntries = <PopEntry<Object?>>{};
```

`_popEntries` 是一个 `Set`，存储所有注册的 `PopEntry` 实例。

## onPopInvokedWithResult

```dart 2021:2027:packages/flutter/lib/src/widgets/routes.dart
  @override
  void onPopInvokedWithResult(bool didPop, T? result) {
    for (final PopEntry<Object?> popEntry in _popEntries) {
      popEntry.onPopInvokedWithResult(didPop, result);
    }
    super.onPopInvokedWithResult(didPop, result);
  }
```

**功能**：当路由弹出操作被调用时（无论是否实际弹出）执行清理操作。

**参数**：

- `didPop`：是否实际执行了弹出操作
- `result`：弹出时返回的结果值

**实现逻辑**：

1. 遍历所有注册的 `PopEntry` 实例
2. 调用每个 `PopEntry` 的 `onPopInvokedWithResult` 方法
3. 调用父类的 `onPopInvokedWithResult` 方法

**使用场景**：执行清理操作，例如：

- 保存表单数据
- 取消正在进行的网络请求
- 更新共享状态

## PopEntry 注册机制

### registerPopEntry

```dart 2084:2098:packages/flutter/lib/src/widgets/routes.dart
  /// Registers the existence of a [PopEntry] in the route.
  ///
  /// [PopEntry] instances registered in this way will have their
  /// [PopEntry.onPopInvokedWithResult] callbacks called when a route is popped or a pop
  /// is attempted. They will also be able to block pop operations with
  /// [PopEntry.canPopNotifier] through this route's [popDisposition] method.
  ///
  /// See also:
  ///
  ///  * [unregisterPopEntry], which performs the opposite operation.
  void registerPopEntry(PopEntry<Object?> popEntry) {
    _popEntries.add(popEntry);
    popEntry.canPopNotifier.addListener(_maybeDispatchNavigationNotification);
    _maybeDispatchNavigationNotification();
  }
```

**功能**：注册一个 `PopEntry` 到路由中。

**实现**：

1. 将 `PopEntry` 添加到 `_popEntries` 集合中
2. 为 `PopEntry.canPopNotifier` 添加监听器，监听 `_maybeDispatchNavigationNotification`
3. 调用 `_maybeDispatchNavigationNotification()` 通知状态改变

### unregisterPopEntry

```dart 2100:2109:packages/flutter/lib/src/widgets/routes.dart
  /// Unregisters a [PopEntry] in the route's widget subtree.
  ///
  /// See also:
  ///
  ///  * [registerPopEntry], which performs the opposite operation.
  void unregisterPopEntry(PopEntry<Object?> popEntry) {
    _popEntries.remove(popEntry);
    popEntry.canPopNotifier.removeListener(_maybeDispatchNavigationNotification);
    _maybeDispatchNavigationNotification();
  }
```

**功能**：从路由中注销一个 `PopEntry`。

**实现**：

1. 从 `_popEntries` 集合中移除 `PopEntry`
2. 移除 `canPopNotifier` 的监听器
3. 调用 `_maybeDispatchNavigationNotification()` 通知状态改变

## 已废弃的回调方法

### addScopedWillPopCallback

```dart 2029:2060:packages/flutter/lib/src/widgets/routes.dart
  /// Enables this route to veto attempts by the user to dismiss it.
  ///
  /// This callback runs asynchronously and it's possible that it will be called
  /// after its route has been disposed. The callback should check [State.mounted]
  /// before doing anything.
  ///
  /// A typical application of this callback would be to warn the user about
  /// unsaved [Form] data if the user attempts to back out of the form. In that
  /// case, use the [Form.onWillPop] property to register the callback.
  ///
  /// See also:
  ///
  ///  * [WillPopScope], which manages the registration and unregistration
  ///    process automatically.
  ///  * [Form], which provides an `onWillPop` callback that uses this mechanism.
  ///  * [willPop], which runs the callbacks added with this method.
  ///  * [removeScopedWillPopCallback], which removes a callback from the list
  ///    that [willPop] checks.
  @Deprecated(
    'Use registerPopEntry or PopScope instead. '
    'This feature was deprecated after v3.12.0-1.0.pre.',
  )
  void addScopedWillPopCallback(WillPopCallback callback) {
    assert(
      _scopeKey.currentState != null,
      'Tried to add a willPop callback to a route that is not currently in the tree.',
    );
    _willPopCallbacks.add(callback);
    if (_willPopCallbacks.length == 1) {
      _maybeDispatchNavigationNotification();
    }
  }
```

**状态**：已废弃，应使用 `registerPopEntry` 或 `PopScope` 代替。

### removeScopedWillPopCallback

```dart 2062:2082:packages/flutter/lib/src/widgets/routes.dart
  /// Remove one of the callbacks run by [willPop].
  ///
  /// See also:
  ///
  ///  * [Form], which provides an `onWillPop` callback that uses this mechanism.
  ///  * [addScopedWillPopCallback], which adds callback to the list
  ///    checked by [willPop].
  @Deprecated(
    'Use unregisterPopEntry or PopScope instead. '
    'This feature was deprecated after v3.12.0-1.0.pre.',
  )
  void removeScopedWillPopCallback(WillPopCallback callback) {
    assert(
      _scopeKey.currentState != null,
      'Tried to remove a willPop callback from a route that is not currently in the tree.',
    );
    _willPopCallbacks.remove(callback);
    if (_willPopCallbacks.isEmpty) {
      _maybeDispatchNavigationNotification();
    }
  }
```

**状态**：已废弃，应使用 `unregisterPopEntry` 或 `PopScope` 代替。

## 使用示例

### 现代方式：使用 PopScope

```dart
class FormRoute extends ModalRoute<void> {
  final _formKey = GlobalKey<FormState>();

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      body: Form(
        key: _formKey,
        child: PopScope(
          canPop: _formKey.currentState?.validate() ?? true,
          onPopInvoked: (didPop) {
            if (!didPop) {
              // 显示保存提示
              showDialog(
                context: context,
                builder: (_) => AlertDialog(
                  title: Text('Unsaved changes'),
                  content: Text('Do you want to save your changes?'),
                  actions: [
                    TextButton(
                      onPressed: () => Navigator.pop(context),
                      child: Text('Discard'),
                    ),
                    TextButton(
                      onPressed: () {
                        // 保存数据
                        _saveForm();
                        Navigator.pop(context); // 关闭对话框
                        Navigator.pop(context); // 关闭路由
                      },
                      child: Text('Save'),
                    ),
                  ],
                ),
              );
            }
          },
          child: YourFormContent(),
        ),
      ),
    );
  }

  void _saveForm() {
    // 保存表单逻辑
  }

  // ... 其他必需的方法实现
}
```

### 已废弃方式（仅用于理解）

```dart
// 不要使用这种方式，仅用于理解历史实现
class OldFormRoute extends ModalRoute<void> {
  final _formKey = GlobalKey<FormState>();

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      body: Form(
        key: _formKey,
        onWillPop: () async {
          // 已废弃的方式
          return _formKey.currentState?.validate() ?? true;
        },
        child: YourFormContent(),
      ),
    );
  }

  // ... 其他必需的方法实现
}
```

## Pop 操作流程

### 现代流程

```text
用户触发 Pop 操作
    ↓
检查 popDisposition
    ↓
遍历所有 PopEntry.canPopNotifier.value
    ↓
如果任何值为 false → RoutePopDisposition.doNotPop
    ↓
否则 → 执行 Pop 操作
    ↓
调用 onPopInvokedWithResult(didPop, result)
    ↓
通知所有 PopEntry.onPopInvokedWithResult
```

### 已废弃流程（仅用于参考）

```text
用户触发 Pop 操作
    ↓
调用 willPop()
    ↓
遍历所有 WillPopCallback
    ↓
如果任何回调返回 false → RoutePopDisposition.doNotPop
    ↓
否则 → 执行 Pop 操作
```

## 总结

第九部分介绍了 `ModalRoute` 的 Pop 操作管理：

1. **popDisposition**：现代的方式，通过 `PopEntry` 和 `ValueNotifier` 控制是否可以弹出

2. **onPopInvokedWithResult**：在弹出操作被调用时执行清理操作

3. **registerPopEntry / unregisterPopEntry**：注册和注销 `PopEntry` 实例

4. **已废弃的方法**：`willPop`、`addScopedWillPopCallback`、`removeScopedWillPopCallback` 应使用新的 `PopScope` 和 `PopEntry` 机制代替

新的机制提供了更好的性能、更清晰的 API 和更灵活的状态管理。
