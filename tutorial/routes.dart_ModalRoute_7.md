# ModalRoute 类详解 - 第七部分：状态维护与手势支持

## 概述

`ModalRoute` 提供了状态维护机制，用于控制路由在非活动状态时是否保留在内存中。同时，它还支持手势操作（如 iOS 风格的返回手势和 Android 的预测性返回），提供了相关的属性来控制这些手势行为。

## maintainState

```dart 1848:1870:packages/flutter/lib/src/widgets/routes.dart
  /// {@template flutter.widgets.ModalRoute.maintainState}
  /// Whether the route should remain in memory when it is inactive.
  ///
  /// If this is true, then the route is maintained, so that any futures it is
  /// holding from the next route will properly resolve when the next route
  /// pops. If this is not necessary, this can be set to false to allow the
  /// framework to entirely discard the route's widget hierarchy when it is not
  /// visible.
  ///
  /// Setting [maintainState] to false does not guarantee that the route will be
  /// discarded. For instance, it will not be discarded if it is still visible
  /// because the next above it is not opaque (e.g. it is a popup dialog).
  /// {@endtemplate}
  ///
  /// If this getter would ever start returning a different value, the
  /// [changedInternalState] should be invoked so that the change can take
  /// effect.
  ///
  /// See also:
  ///
  ///  * [OverlayEntry.maintainState], which is the underlying implementation
  ///    of this property.
  bool get maintainState;
```

**功能**：控制路由在非活动状态时是否保留在内存中。

**行为**：

- **true**：路由被保留在内存中，即使它不可见。这对于需要从下一个路由接收 future 结果的情况很重要
- **false**：允许框架在路由不可见时完全丢弃路由的 widget 层次结构，释放内存

**重要说明**：

设置 `maintainState` 为 `false` 并不保证路由会被丢弃。例如，如果路由仍然可见（因为上方的路由不是完全不透明的，比如弹出对话框），路由不会被丢弃。

### 使用场景

#### 场景 1：需要接收结果的路由

```dart
class ResultReceiverRoute extends ModalRoute<String> {
  @override
  bool get maintainState => true; // 保持状态以接收结果

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      body: Center(
        child: ElevatedButton(
          onPressed: () async {
            // 导航到下一个路由并等待结果
            final result = await Navigator.push<String>(
              context,
              MaterialPageRoute(builder: (_) => NextPage()),
            );
            // 即使路由被遮挡，仍然可以接收结果
            Navigator.pop(context, result);
          },
          child: Text('Navigate'),
        ),
      ),
    );
  }

  // ... 其他必需的方法实现
}
```

#### 场景 2：不需要保留状态的路由

```dart
class DisposableRoute extends ModalRoute<void> {
  @override
  bool get maintainState => false; // 不保持状态，节省内存

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      body: Center(
        child: Text('This route can be disposed when inactive'),
      ),
    );
  }

  // ... 其他必需的方法实现
}
```

## popGestureInProgress

```dart 1872:1879:packages/flutter/lib/src/widgets/routes.dart
  /// True if a back gesture (iOS-style back swipe or Android predictive back)
  /// is currently underway for this route.
  ///
  /// See also:
  ///
  ///  * [popGestureEnabled], which returns true if a user-triggered pop gesture
  ///    would be allowed.
  bool get popGestureInProgress => navigator!.userGestureInProgress;
```

**功能**：返回当前是否有返回手势正在进行中。

**手势类型**：

- iOS 风格的返回滑动（左边缘向右滑动）
- Android 的预测性返回

**实现**：直接返回 `navigator!.userGestureInProgress`，表示导航器级别的用户手势状态。

**使用场景**：判断当前是否有用户手势正在进行，可以用于调整 UI 或禁用某些交互。

## popGestureEnabled

```dart 1881:1910:packages/flutter/lib/src/widgets/routes.dart
  /// Whether a pop gesture can be started by the user for this route.
  ///
  /// Returns true if the user can edge-swipe to a previous route.
  ///
  /// This should only be used between frames, not during build.
  @override
  bool get popGestureEnabled {
    // If there's nothing to go back to, then obviously we don't support
    // the back gesture.
    if (isFirst) {
      return false;
    }
    // If the route wouldn't actually pop if we popped it, then the gesture
    // would be really confusing (or would skip internal routes), so disallow it.
    if (willHandlePopInternally) {
      return false;
    }
    // If attempts to dismiss this route might be vetoed such as in a page
    // with forms, then do not allow the user to dismiss the route with a swipe.
    if (hasScopedWillPopCallback || popDisposition == RoutePopDisposition.doNotPop) {
      return false;
    }
    // If we're in an animation already, we cannot be manually swiped.
    if (!animation!.isCompleted) {
      return false;
    }

    // Looks like a back gesture would be welcome!
    return true;
  }
```

**功能**：判断用户是否可以为此路由启动弹出手势。

**使用限制**：只应在帧之间使用，不应在 build 过程中使用。

### 判断逻辑

方法通过多个条件来判断是否允许弹出手势：

#### 1. 是否为第一个路由

```dart
if (isFirst) {
  return false;
}
```

如果没有可以返回的路由，显然不支持返回手势。

#### 2. 是否内部处理弹出

```dart
if (willHandlePopInternally) {
  return false;
}
```

如果路由会内部处理弹出操作，则不允许手势。因为这会导致手势行为混乱（或跳过内部路由）。

#### 3. 是否有弹出回调或禁止弹出

```dart
if (hasScopedWillPopCallback || popDisposition == RoutePopDisposition.doNotPop) {
  return false;
}
```

如果尝试关闭路由可能会被拒绝（例如包含表单的页面），则不允许用户通过滑动手势关闭路由。

#### 4. 动画是否完成

```dart
if (!animation!.isCompleted) {
  return false;
}
```

如果已经有动画正在进行，则不能手动滑动。这避免了动画冲突。

#### 5. 允许手势

如果所有条件都满足，则允许返回手势：

```dart
return true;
```

### 使用示例

虽然这个方法通常由框架内部使用，但了解其逻辑有助于理解为什么某些路由不支持手势返回：

```dart
class FormRoute extends ModalRoute<void> {
  final _formKey = GlobalKey<FormState>();

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      body: Form(
        key: _formKey,
        child: Column(
          children: [
            TextFormField(
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return 'Required';
                }
                return null;
              },
            ),
            ElevatedButton(
              onPressed: () {
                if (_formKey.currentState!.validate()) {
                  Navigator.pop(context);
                }
              },
              child: Text('Submit'),
            ),
          ],
        ),
      ),
    );
  }

  // 由于使用了 Form（内部使用 PopScope），popDisposition 可能返回 doNotPop
  // 因此这个路由可能不支持手势返回（由 popGestureEnabled 自动处理）

  // ... 其他必需的方法实现
}
```

## 状态维护与手势的关系

`maintainState` 和手势支持之间没有直接关系，但它们都影响路由的生命周期和用户体验：

- **maintainState**：控制路由的内存管理
- **popGestureEnabled**：控制用户交互行为

### 性能考虑

1. **内存管理**：
   - 对于不需要保留状态的路由，设置 `maintainState` 为 `false` 可以节省内存
   - 对于需要接收异步结果的路由，必须设置 `maintainState` 为 `true`

2. **用户体验**：
   - `popGestureEnabled` 确保只有在安全和合适的情况下才允许手势返回
   - 避免在用户可能丢失数据（如未保存的表单）时允许手势返回

## 最佳实践

### 1. 合理使用 maintainState

```dart
// 好的做法：需要接收结果时保持状态
class DialogRoute extends ModalRoute<bool> {
  @override
  bool get maintainState => true; // 需要接收用户选择结果
  // ...
}

// 好的做法：不需要结果时不保持状态
class InfoRoute extends ModalRoute<void> {
  @override
  bool get maintainState => false; // 纯展示路由，不需要保持状态
  // ...
}
```

### 2. 理解 popGestureEnabled 的限制

不要手动覆盖 `popGestureEnabled`，框架已经根据路由状态自动处理。如果需要阻止手势返回，应该：

```dart
// 使用 PopScope 来控制是否允许弹出
class ProtectedRoute extends ModalRoute<void> {
  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return PopScope(
      canPop: false, // 阻止弹出
      child: Scaffold(
        body: YourContent(),
      ),
    );
  }

  // popGestureEnabled 会自动返回 false，因为 popDisposition 会是 doNotPop
}
```

## 总结

第七部分介绍了 `ModalRoute` 的状态维护和手势支持机制：

1. **maintainState**：控制路由在非活动状态时是否保留在内存中，对于需要接收异步结果的路由很重要

2. **popGestureInProgress**：指示当前是否有返回手势正在进行

3. **popGestureEnabled**：判断是否允许用户启动弹出手势，通过多个条件确保手势返回的安全性和合理性

这些机制共同确保了路由的内存效率和用户交互的安全性。
