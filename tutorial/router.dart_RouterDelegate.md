# RouterDelegate 详解

## 概述

`RouterDelegate` 是 Flutter 路由系统的核心组件之一，它是一个抽象类，用于构建和配置导航组件。`RouterDelegate` 是 `Router` widget 的核心部分，它响应来自引擎的路由推送和弹出意图，并通知 `Router` 进行重建。同时，它也作为 `Router` widget 的构建器，在 `Router` widget 构建时构建导航组件（通常是 `Navigator`）。

```dart 1341:1341:packages/flutter/lib/src/widgets/router.dart
abstract class RouterDelegate<T> extends Listenable {
```

`RouterDelegate` 是一个泛型抽象类，类型参数 `T` 表示路由配置的数据类型。它继承自 `Listenable`，这意味着它可以通知监听者状态变化。

## 核心职责

`RouterDelegate` 的主要职责包括：

1. **响应路由推送意图**：当引擎推送新路由时，路由信息会被 `RouteInformationParser` 解析为类型 `T` 的配置，然后通过 `setInitialRoutePath` 或 `setNewRoutePath` 传递给路由委托，路由委托据此配置自身

2. **构建导航组件**：当 `Router` widget 调用 `build` 方法时，路由委托会构建最新的导航组件

3. **响应路由弹出意图**：当操作系统请求弹出当前路由时，`popRoute` 方法会被调用

4. **通知状态变化**：作为 `Listenable`，当应用状态发生变化或接收到路由相关的引擎意图时，它会通知监听者

## 实现建议

在实现 `RouterDelegate` 的子类时，建议：

- 定义一个 `Listenable` 应用状态对象用于构建导航组件
- 当应用状态发生变化或接收到路由相关引擎意图时，更新应用状态并通知监听者

所有子类必须实现 `setNewRoutePath`、`popRoute` 和 `build` 方法。

## 核心方法详解

### setInitialRoutePath

```dart 1358:1360:packages/flutter/lib/src/widgets/router.dart
  Future<void> setInitialRoutePath(T configuration) {
    return setNewRoutePath(configuration);
  }
```

在应用启动时，`Router` 会调用此方法，传入 `RouteInformationParser` 从解析初始路由得到的配置结构。

此方法应该配置 `RouterDelegate`，使得当 `build` 方法被调用时，它会创建一个与初始路由匹配的 widget 树。

**默认实现**：默认情况下，此方法将 `configuration` 转发给 `setNewRoutePath`。

**性能建议**：如果结果可以同步计算，考虑使用 `SynchronousFuture`，这样 `Router` 就不需要等待下一个微任务来调度构建。

**状态恢复**：在状态恢复期间，会调用 `setRestoredRoutePath` 而不是此方法。

### setRestoredRoutePath

```dart 1372:1374:packages/flutter/lib/src/widgets/router.dart
  Future<void> setRestoredRoutePath(T configuration) {
    return setNewRoutePath(configuration);
  }
```

在状态恢复期间，`Router` 会调用此方法（而不是 `setInitialRoutePath`）将之前的配置传回给委托。

**状态恢复机制**：

- 当 `Router` 配置了状态恢复时，它会在状态序列化期间持久化 `currentConfiguration` 的值
- 在状态恢复期间，`Router` 调用此方法将之前的配置传回给委托
- 委托有责任根据提供的配置信息恢复其内部状态

**默认实现**：默认情况下，此方法将 `configuration` 转发给 `setNewRoutePath`。

### setNewRoutePath

```dart 1376:1382:packages/flutter/lib/src/widgets/router.dart
  /// Called by the [Router] when the [Router.routeInformationProvider] reports that a
  /// new route has been pushed to the application by the operating system.
  ///
  /// Consider using a [SynchronousFuture] if the result can be computed
  /// synchronously, so that the [Router] does not need to wait for the next
  /// microtask to schedule a build.
  Future<void> setNewRoutePath(T configuration);
```

当 `Router.routeInformationProvider` 报告操作系统向应用推送了新路由时，`Router` 会调用此方法。

**性能建议**：如果结果可以同步计算，考虑使用 `SynchronousFuture`，这样 `Router` 就不需要等待下一个微任务来调度构建。

**注意**：这是一个抽象方法，子类必须实现。

### popRoute

```dart 1384:1394:packages/flutter/lib/src/widgets/router.dart
  /// Called by the [Router] when the [Router.backButtonDispatcher] reports that
  /// the operating system is requesting that the current route be popped.
  ///
  /// The method should return a boolean [Future] to indicate whether this
  /// delegate handles the request. Returning false will cause the entire app
  /// to be popped.
  ///
  /// Consider using a [SynchronousFuture] if the result can be computed
  /// synchronously, so that the [Router] does not need to wait for the next
  /// microtask to schedule a build.
  Future<bool> popRoute();
```

当 `Router.backButtonDispatcher` 报告操作系统请求弹出当前路由时，`Router` 会调用此方法。

**返回值**：此方法应返回一个布尔值 `Future`，指示此委托是否处理了该请求。返回 `false` 将导致整个应用被弹出。

**性能建议**：如果结果可以同步计算，考虑使用 `SynchronousFuture`，这样 `Router` 就不需要等待下一个微任务来调度构建。

**注意**：这是一个抽象方法，子类必须实现。

### currentConfiguration

```dart 1396:1424:packages/flutter/lib/src/widgets/router.dart
  /// Called by the [Router] when it detects a route information may have
  /// changed as a result of rebuild.
  ///
  /// If this getter returns non-null, the [Router] will start to report new
  /// route information back to the engine. In web applications, the new
  /// route information is used for populating browser history in order to
  /// support the forward and the backward buttons.
  ///
  /// When overriding this method, the configuration returned by this getter
  /// must be able to construct the current app state and build the widget
  /// with the same configuration in the [build] method if it is passed back
  /// to the [setNewRoutePath]. Otherwise, the browser backward and forward
  /// buttons will not work properly.
  ///
  /// By default, this getter returns null, which prevents the [Router] from
  /// reporting the route information. To opt in, a subclass can override this
  /// getter to return the current configuration.
  ///
  /// At most one [Router] can opt in to route information reporting. Typically,
  /// only the top-most [Router] created by [WidgetsApp.router] should opt for
  /// route information reporting.
  ///
  /// ## State Restoration
  ///
  /// This getter is also used by the [Router] to implement state restoration.
  /// During state serialization, the [Router] will persist the current
  /// configuration and during state restoration pass it back to the delegate
  /// by calling [setRestoredRoutePath].
  T? get currentConfiguration => null;
```

当 `Router` 检测到路由信息可能因重建而发生变化时，会调用此 getter。

**功能**：

- 如果此 getter 返回非 null 值，`Router` 将开始向引擎报告新的路由信息
- 在 Web 应用中，新的路由信息用于填充浏览器历史记录，以支持前进和后退按钮

**重要约束**：

- 重写此方法时，此 getter 返回的配置必须能够构造当前应用状态
- 如果该配置被传回给 `setNewRoutePath`，在 `build` 方法中使用相同配置必须能够构建相同的 widget
- 否则，浏览器的后退和前进按钮将无法正常工作

**默认行为**：默认情况下，此 getter 返回 `null`，这会阻止 `Router` 报告路由信息。要启用此功能，子类可以重写此 getter 以返回当前配置。

**限制**：最多只有一个 `Router` 可以选择加入路由信息报告。通常，只有由 `WidgetsApp.router` 创建的最顶层 `Router` 应该选择加入路由信息报告。

**状态恢复**：此 getter 也用于实现状态恢复。在状态序列化期间，`Router` 会持久化当前配置，在状态恢复期间，通过调用 `setRestoredRoutePath` 将其传回给委托。

### build

```dart 1426:1445:packages/flutter/lib/src/widgets/router.dart
  /// Called by the [Router] to obtain the widget tree that represents the
  /// current state.
  ///
  /// This is called whenever the [Future]s returned by [setInitialRoutePath],
  /// [setNewRoutePath], or [setRestoredRoutePath] complete as well as when this
  /// notifies its clients (see the [Listenable] interface, which this interface
  /// includes). In addition, it may be called at other times. It is important,
  /// therefore, that the methods above do not update the state that the [build]
  /// method uses before they complete their respective futures.
  ///
  /// Typically this method returns a suitably-configured [Navigator]. If you do
  /// plan to create a navigator, consider using the
  /// [PopNavigatorRouterDelegateMixin]. If state restoration is enabled for the
  /// [Router] using this delegate, consider providing a non-null
  /// [Navigator.restorationScopeId] to the [Navigator] returned by this method.
  ///
  /// This method must not return null.
  ///
  /// The `context` is the [Router]'s build context.
  Widget build(BuildContext context);
```

`Router` 调用此方法以获取表示当前状态的 widget 树。

**调用时机**：

- 当 `setInitialRoutePath`、`setNewRoutePath` 或 `setRestoredRoutePath` 返回的 `Future` 完成时
- 当此委托通知其客户端时（参见 `Listenable` 接口）
- 在其他时候也可能被调用

**重要注意事项**：

- 上面的方法在完成各自的 `Future` 之前，不应该更新 `build` 方法使用的状态

**典型实现**：

- 通常此方法返回一个适当配置的 `Navigator`
- 如果计划创建导航器，考虑使用 `PopNavigatorRouterDelegateMixin`
- 如果使用此委托的 `Router` 启用了状态恢复，考虑为返回的 `Navigator` 提供非 null 的 `Navigator.restorationScopeId`

**约束**：此方法不能返回 `null`。

**参数**：`context` 是 `Router` 的构建上下文。

**注意**：这是一个抽象方法，子类必须实现。

## 状态恢复

如果拥有此委托的 `Router` 配置了状态恢复，它将使用以下机制持久化和恢复此 `RouterDelegate` 的配置：

1. **序列化阶段**：在应用被操作系统终止之前，`currentConfiguration` 的值被序列化并持久化
2. **恢复阶段**：应用重启后，该值被反序列化并通过调用 `setRestoredRoutePath`（默认情况下只是调用 `setNewRoutePath`）传回给 `RouterDelegate`
3. **状态恢复**：`RouterDelegate` 有责任使用提供的配置信息恢复其内部状态

## 相关组件

### RouteInformationParser

`RouteInformationParser` 负责在传递给路由委托之前，将路由信息解析为配置。它与 `RouterDelegate` 协同工作：

1. `RouteInformationParser` 将路由信息解析为类型 `T` 的配置
2. `RouterDelegate` 接收配置并更新其内部状态
3. `RouterDelegate` 通过 `build` 方法构建导航组件

### Router

`Router` 是将所有委托组合在一起以提供完整路由解决方案的 widget。它协调 `RouteInformationProvider`、`RouteInformationParser` 和 `RouterDelegate` 的工作。

## 实现示例

典型的 `RouterDelegate` 实现模式：

1. 定义一个 `Listenable` 应用状态对象
2. 在 `setNewRoutePath` 中更新应用状态
3. 在 `popRoute` 中处理路由弹出逻辑
4. 在 `build` 中根据应用状态构建 `Navigator`
5. 当状态发生变化时，通知监听者（调用 `notifyListeners()`）

## 总结

`RouterDelegate` 是 Flutter 声明式路由系统的核心组件，它负责：

- 接收并处理路由配置
- 管理应用的路由状态
- 构建导航组件（通常是 `Navigator`）
- 响应系统路由意图（推送、弹出）
- 支持状态恢复

通过正确实现 `RouterDelegate` 的子类，开发者可以完全控制应用的路由行为和导航逻辑。
