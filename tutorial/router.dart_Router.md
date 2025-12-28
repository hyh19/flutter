# Router 与 _RouterState 详解

## 概述

`Router` 是 Flutter 中用于处理应用路由的核心组件，它负责协调路由信息的获取、解析和导航。`Router` 是一个 `StatefulWidget`，其状态由 `_RouterState` 管理。这两个类共同实现了 Flutter 的声明式路由系统。

`Router` 的核心职责是：

- 监听来自操作系统的路由信息（如应用启动时的初始路由、接收到的 intent、系统返回按钮通知等）
- 将路由信息解析为类型 `T` 的配置数据
- 将配置数据转换为 `Page` 对象并传递给 `Navigator`

## Router 类详解

### 类定义

```dart 358:358:packages/flutter/lib/src/widgets/router.dart
class Router<T> extends StatefulWidget {
```

`Router` 是一个泛型 `StatefulWidget`，类型参数 `T` 表示路由配置的数据类型。

### 核心组件

`Router` 通过以下四个核心组件协同工作：

#### 1. RouteInformationProvider

```dart 404:413:packages/flutter/lib/src/widgets/router.dart
  /// The route information provider for the router.
  ///
  /// The value at the time of first build will be used as the initial route.
  /// The [Router] listens to this provider and rebuilds with new names when
  /// it notifies.
  ///
  /// This can be null if this router does not rely on the route information
  /// to build its content. In such case, the [routeInformationParser] must also
  /// be null.
  final RouteInformationProvider? routeInformationProvider;
```

`RouteInformationProvider` 负责提供路由信息。它会在首次构建时提供初始路由，并在路由信息变化时通知 `Router`。

#### 2. RouteInformationParser

```dart 415:424:packages/flutter/lib/src/widgets/router.dart
  /// The route information parser for the router.
  ///
  /// When the [Router] gets a new route information from the [routeInformationProvider],
  /// the [Router] uses this delegate to parse the route information and produce a
  /// configuration. The configuration will be used by [routerDelegate] and
  /// eventually rebuilds the [Router] widget.
  ///
  /// Since this delegate is the primary consumer of the [routeInformationProvider],
  /// it must not be null if [routeInformationProvider] is not null.
  final RouteInformationParser<T>? routeInformationParser;
```

`RouteInformationParser` 负责将 `RouteInformation` 解析为类型 `T` 的配置对象。如果提供了 `routeInformationProvider`，则必须同时提供 `routeInformationParser`。

#### 3. RouterDelegate

```dart 426:437:packages/flutter/lib/src/widgets/router.dart
  /// The router delegate for the router.
  ///
  /// This delegate consumes the configuration from [routeInformationParser] and
  /// builds a navigating widget for the [Router].
  ///
  /// It is also the primary respondent for the [backButtonDispatcher]. The
  /// [Router] relies on [RouterDelegate.popRoute] to handle the back
  /// button.
  ///
  /// If the [RouterDelegate.currentConfiguration] returns a non-null object,
  /// this [Router] will opt for URL updates.
  final RouterDelegate<T> routerDelegate;
```

`RouterDelegate` 是核心组件，它：

- 接收来自 `RouteInformationParser` 的配置
- 构建导航 widget（通常是 `Navigator`）
- 处理返回按钮事件
- 实现 `Listenable` 接口，通知变化时触发 `Router` 重建

#### 4. BackButtonDispatcher

```dart 439:443:packages/flutter/lib/src/widgets/router.dart
  /// The back button dispatcher for the router.
  ///
  /// The two common alternatives are the [RootBackButtonDispatcher] for root
  /// router, or the [ChildBackButtonDispatcher] for other routers.
  final BackButtonDispatcher? backButtonDispatcher;
```

`BackButtonDispatcher` 处理系统返回按钮事件。根路由通常使用 `RootBackButtonDispatcher`，嵌套路由使用 `ChildBackButtonDispatcher`。

### 构造函数

`Router` 提供了两个构造函数：

#### 标准构造函数

```dart 367:377:packages/flutter/lib/src/widgets/router.dart
  const Router({
    super.key,
    this.routeInformationProvider,
    this.routeInformationParser,
    required this.routerDelegate,
    this.backButtonDispatcher,
    this.restorationScopeId,
  }) : assert(
         routeInformationProvider == null || routeInformationParser != null,
         'A routeInformationParser must be provided when a routeInformationProvider is specified.',
       );
```

标准构造函数要求：如果提供了 `routeInformationProvider`，则必须同时提供 `routeInformationParser`。

#### 使用 RouterConfig 的工厂构造函数

```dart 389:402:packages/flutter/lib/src/widgets/router.dart
  factory Router.withConfig({
    Key? key,
    required RouterConfig<T> config,
    String? restorationScopeId,
  }) {
    return Router<T>(
      key: key,
      routeInformationProvider: config.routeInformationProvider,
      routeInformationParser: config.routeInformationParser,
      routerDelegate: config.routerDelegate,
      backButtonDispatcher: config.backButtonDispatcher,
      restorationScopeId: restorationScopeId,
    );
  }
```

这个工厂构造函数允许通过 `RouterConfig` 对象统一配置所有组件。

### 静态方法

#### Router.of

```dart 479:492:packages/flutter/lib/src/widgets/router.dart
  static Router<T> of<T extends Object?>(BuildContext context) {
    final _RouterScope? scope = context.dependOnInheritedWidgetOfExactType<_RouterScope>();
    assert(() {
      if (scope == null) {
        throw FlutterError(
          'Router operation requested with a context that does not include a Router.\n'
          'The context used to retrieve the Router must be that of a widget that '
          'is a descendant of a Router widget.',
        );
      }
      return true;
    }());
    return scope!.routerState.widget as Router<T>;
  }
```

从 `BuildContext` 中获取最近的 `Router` 祖先。如果不存在，会在 debug 模式下断言失败，在 release 模式下抛出异常。

#### Router.maybeOf

```dart 507:510:packages/flutter/lib/src/widgets/router.dart
  static Router<T>? maybeOf<T extends Object?>(BuildContext context) {
    final _RouterScope? scope = context.dependOnInheritedWidgetOfExactType<_RouterScope>();
    return scope?.routerState.widget as Router<T>?;
  }
```

与 `of` 类似，但如果不存在 `Router` 祖先则返回 `null`，不会抛出异常。

#### Router.navigate

```dart 536:543:packages/flutter/lib/src/widgets/router.dart
  static void navigate(BuildContext context, VoidCallback callback) {
    final _RouterScope scope =
        context.getElementForInheritedWidgetOfExactType<_RouterScope>()!.widget as _RouterScope;
    scope.routerState._setStateWithExplicitReportStatus(
      RouteInformationReportingType.navigate,
      callback,
    );
  }
```

强制 `Router` 执行回调并创建新的浏览器历史记录条目，即使 URL 没有变化。这对于在相同 URL 下支持浏览器前进/后退按钮很有用（例如保存滚动位置）。

#### Router.neglect

```dart 567:574:packages/flutter/lib/src/widgets/router.dart
  static void neglect(BuildContext context, VoidCallback callback) {
    final _RouterScope scope =
        context.getElementForInheritedWidgetOfExactType<_RouterScope>()!.widget as _RouterScope;
    scope.routerState._setStateWithExplicitReportStatus(
      RouteInformationReportingType.neglect,
      callback,
    );
  }
```

强制 `Router` 执行回调但不创建新的浏览器历史记录条目，即使检测到 URL 变化。这仍然会更新当前历史记录条目的 URL 和状态。

## _RouterState 类详解

### 类定义

```dart 608:608:packages/flutter/lib/src/widgets/router.dart
class _RouterState<T> extends State<Router<T>> with RestorationMixin {
```

`_RouterState` 是 `Router` 的状态类，混入了 `RestorationMixin` 以支持状态恢复。

### 关键状态变量

```dart 609:612:packages/flutter/lib/src/widgets/router.dart
  Object? _currentRouterTransaction;
  RouteInformationReportingType? _currentIntentionToReport;
  final _RestorableRouteInformation _routeInformation = _RestorableRouteInformation();
  late bool _routeParsePending;
```

- `_currentRouterTransaction`：用于跟踪当前正在进行的路由事务，防止异步操作冲突
- `_currentIntentionToReport`：记录路由信息报告的类型（navigate、neglect 或 none）
- `_routeInformation`：可恢复的路由信息对象
- `_routeParsePending`：标记是否有待处理的路由解析

### 生命周期方法

#### initState

```dart 617:623:packages/flutter/lib/src/widgets/router.dart
  @override
  void initState() {
    super.initState();
    widget.routeInformationProvider?.addListener(_handleRouteInformationProviderNotification);
    widget.backButtonDispatcher?.addCallback(_handleBackButtonDispatcherNotification);
    widget.routerDelegate.addListener(_handleRouterDelegateNotification);
  }
```

在初始化时注册监听器：

- 监听 `RouteInformationProvider` 的路由信息变化
- 注册返回按钮回调
- 监听 `RouterDelegate` 的变化通知

#### restoreState

```dart 625:640:packages/flutter/lib/src/widgets/router.dart
  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    registerForRestoration(_routeInformation, 'route');
    if (_routeInformation.value != null) {
      assert(widget.routeInformationParser != null);
      _processRouteInformation(
        _routeInformation.value!,
        () => widget.routerDelegate.setRestoredRoutePath,
      );
    } else if (widget.routeInformationProvider != null) {
      _processRouteInformation(
        widget.routeInformationProvider!.value,
        () => widget.routerDelegate.setInitialRoutePath,
      );
    }
  }
```

状态恢复时：

- 如果有已保存的路由信息，使用 `setRestoredRoutePath` 恢复
- 否则使用 `RouteInformationProvider` 的初始值调用 `setInitialRoutePath`

#### didChangeDependencies

```dart 712:729:packages/flutter/lib/src/widgets/router.dart
  @override
  void didChangeDependencies() {
    _routeParsePending = true;
    super.didChangeDependencies();
    // The super.didChangeDependencies may have parsed the route information.
    // This can happen if the didChangeDependencies is triggered by state
    // restoration or first build.
    final RouteInformation? currentRouteInformation =
        _routeInformation.value ?? widget.routeInformationProvider?.value;
    if (currentRouteInformation != null && _routeParsePending) {
      _processRouteInformation(
        currentRouteInformation,
        () => widget.routerDelegate.setNewRoutePath,
      );
    }
    _routeParsePending = false;
    _maybeNeedToReportRouteInformation();
  }
```

在依赖变化时：

- 检查是否有待处理的路由信息需要解析
- 如果有，调用 `_processRouteInformation` 处理
- 最后检查是否需要报告路由信息

#### didUpdateWidget

```dart 731:758:packages/flutter/lib/src/widgets/router.dart
  @override
  void didUpdateWidget(Router<T> oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.routeInformationProvider != oldWidget.routeInformationProvider ||
        widget.backButtonDispatcher != oldWidget.backButtonDispatcher ||
        widget.routeInformationParser != oldWidget.routeInformationParser ||
        widget.routerDelegate != oldWidget.routerDelegate) {
      _currentRouterTransaction = Object();
    }
    if (widget.routeInformationProvider != oldWidget.routeInformationProvider) {
      oldWidget.routeInformationProvider?.removeListener(
        _handleRouteInformationProviderNotification,
      );
      widget.routeInformationProvider?.addListener(_handleRouteInformationProviderNotification);
      if (oldWidget.routeInformationProvider?.value != widget.routeInformationProvider?.value) {
        _handleRouteInformationProviderNotification();
      }
    }
    if (widget.backButtonDispatcher != oldWidget.backButtonDispatcher) {
      oldWidget.backButtonDispatcher?.removeCallback(_handleBackButtonDispatcherNotification);
      widget.backButtonDispatcher?.addCallback(_handleBackButtonDispatcherNotification);
    }
    if (widget.routerDelegate != oldWidget.routerDelegate) {
      oldWidget.routerDelegate.removeListener(_handleRouterDelegateNotification);
      widget.routerDelegate.addListener(_handleRouterDelegateNotification);
      _maybeNeedToReportRouteInformation();
    }
  }
```

当 widget 更新时：

- 如果任何委托发生变化，创建新的事务对象（丢弃旧的异步操作）
- 更新监听器注册
- 如果 `RouteInformationProvider` 的值发生变化，触发通知处理

### 核心处理方法

#### _processRouteInformation

```dart 770:780:packages/flutter/lib/src/widgets/router.dart
  void _processRouteInformation(
    RouteInformation information,
    ValueGetter<_RouteSetter<T>> delegateRouteSetter,
  ) {
    assert(_routeParsePending);
    _routeParsePending = false;
    _currentRouterTransaction = Object();
    widget.routeInformationParser!
        .parseRouteInformationWithDependencies(information, context)
        .then<void>(_processParsedRouteInformation(_currentRouterTransaction, delegateRouteSetter));
  }
```

处理路由信息：

1. 创建新的事务对象
2. 调用 `RouteInformationParser` 解析路由信息
3. 解析完成后调用 `_processParsedRouteInformation` 处理结果

#### _processParsedRouteInformation

```dart 782:795:packages/flutter/lib/src/widgets/router.dart
  _RouteSetter<T> _processParsedRouteInformation(
    Object? transaction,
    ValueGetter<_RouteSetter<T>> delegateRouteSetter,
  ) {
    return (T data) async {
      if (_currentRouterTransaction != transaction) {
        return;
      }
      await delegateRouteSetter()(data);
      if (_currentRouterTransaction == transaction) {
        _rebuild();
      }
    };
  }
```

处理解析后的路由信息：

- 检查事务是否仍然有效（防止被新的事务覆盖）
- 调用委托的路由设置方法（`setNewRoutePath`、`setInitialRoutePath` 或 `setRestoredRoutePath`）
- 如果事务仍然有效，触发重建

#### _handleRouteInformationProviderNotification

```dart 797:803:packages/flutter/lib/src/widgets/router.dart
  void _handleRouteInformationProviderNotification() {
    _routeParsePending = true;
    _processRouteInformation(
      widget.routeInformationProvider!.value,
      () => widget.routerDelegate.setNewRoutePath,
    );
  }
```

当 `RouteInformationProvider` 通知新路由信息时，使用 `setNewRoutePath` 处理。

#### _handleBackButtonDispatcherNotification

```dart 805:810:packages/flutter/lib/src/widgets/router.dart
  Future<bool> _handleBackButtonDispatcherNotification() {
    _currentRouterTransaction = Object();
    return widget.routerDelegate.popRoute().then<bool>(
      _handleRoutePopped(_currentRouterTransaction),
    );
  }
```

处理返回按钮事件：

- 创建新事务
- 调用 `RouterDelegate.popRoute()`
- 处理返回结果

#### _handleRoutePopped

```dart 812:822:packages/flutter/lib/src/widgets/router.dart
  _AsyncPassthrough<bool> _handleRoutePopped(Object? transaction) {
    return (bool data) {
      if (transaction != _currentRouterTransaction) {
        // A rebuilt was trigger from a different source. Returns true to
        // prevent bubbling.
        return SynchronousFuture<bool>(true);
      }
      _rebuild();
      return SynchronousFuture<bool>(data);
    };
  }
```

处理路由弹出结果：

- 如果事务已失效，返回 `true` 防止事件冒泡
- 否则触发重建并返回原始结果

#### _rebuild

```dart 824:830:packages/flutter/lib/src/widgets/router.dart
  Future<void> _rebuild([void value]) {
    setState(() {
      /* routerDelegate is ready to rebuild */
    });
    _maybeNeedToReportRouteInformation();
    return SynchronousFuture<void>(value);
  }
```

触发重建并检查是否需要报告路由信息。

#### _handleRouterDelegateNotification

```dart 832:837:packages/flutter/lib/src/widgets/router.dart
  void _handleRouterDelegateNotification() {
    setState(() {
      /* routerDelegate wants to rebuild */
    });
    _maybeNeedToReportRouteInformation();
  }
```

当 `RouterDelegate` 通知变化时，触发重建并检查路由信息报告。

### 路由信息报告机制

#### _scheduleRouteInformationReportingTask

```dart 644:654:packages/flutter/lib/src/widgets/router.dart
  void _scheduleRouteInformationReportingTask() {
    if (_routeInformationReportingTaskScheduled || widget.routeInformationProvider == null) {
      return;
    }
    assert(_currentIntentionToReport != null);
    _routeInformationReportingTaskScheduled = true;
    SchedulerBinding.instance.addPostFrameCallback(
      _reportRouteInformation,
      debugLabel: 'Router.reportRouteInfo',
    );
  }
```

调度路由信息报告任务，使用 `addPostFrameCallback` 确保在帧结束后执行。

#### _reportRouteInformation

```dart 656:673:packages/flutter/lib/src/widgets/router.dart
  void _reportRouteInformation(Duration timestamp) {
    if (!mounted) {
      return;
    }

    assert(_routeInformationReportingTaskScheduled);
    _routeInformationReportingTaskScheduled = false;

    if (_routeInformation.value != null) {
      final RouteInformation currentRouteInformation = _routeInformation.value!;
      assert(_currentIntentionToReport != null);
      widget.routeInformationProvider!.routerReportsNewRouteInformation(
        currentRouteInformation,
        type: _currentIntentionToReport!,
      );
    }
    _currentIntentionToReport = RouteInformationReportingType.none;
  }
```

报告路由信息给 `RouteInformationProvider`，使用之前设置的报告类型。

#### _maybeNeedToReportRouteInformation

```dart 706:710:packages/flutter/lib/src/widgets/router.dart
  void _maybeNeedToReportRouteInformation() {
    _routeInformation.value = _retrieveNewRouteInformation();
    _currentIntentionToReport ??= RouteInformationReportingType.none;
    _scheduleRouteInformationReportingTask();
  }
```

检查是否需要报告路由信息：

- 从 `RouterDelegate` 获取当前配置并转换为 `RouteInformation`
- 如果没有明确的报告意图，默认为 `none`
- 调度报告任务

#### _retrieveNewRouteInformation

```dart 675:681:packages/flutter/lib/src/widgets/router.dart
  RouteInformation? _retrieveNewRouteInformation() {
    final T? configuration = widget.routerDelegate.currentConfiguration;
    if (configuration == null) {
      return null;
    }
    return widget.routeInformationParser?.restoreRouteInformation(configuration);
  }
```

从 `RouterDelegate` 获取当前配置，并通过 `RouteInformationParser` 转换为 `RouteInformation`。

#### _setStateWithExplicitReportStatus

```dart 683:704:packages/flutter/lib/src/widgets/router.dart
  void _setStateWithExplicitReportStatus(RouteInformationReportingType status, VoidCallback fn) {
    assert(status.index >= RouteInformationReportingType.neglect.index);
    assert(() {
      if (_currentIntentionToReport != null &&
          _currentIntentionToReport != RouteInformationReportingType.none &&
          _currentIntentionToReport != status) {
        FlutterError.reportError(
          const FlutterErrorDetails(
            exception:
                'Both Router.navigate and Router.neglect have been called in this '
                'build cycle, and the Router cannot decide whether to report the '
                'route information. Please make sure only one of them is called '
                'within the same build cycle.',
          ),
        );
      }
      return true;
    }());
    _currentIntentionToReport = status;
    _scheduleRouteInformationReportingTask();
    fn();
  }
```

设置明确的报告状态（由 `Router.navigate` 或 `Router.neglect` 调用）：

- 检查是否在同一构建周期内调用了冲突的方法
- 设置报告意图
- 调度报告任务
- 执行回调

### build 方法

```dart 839:858:packages/flutter/lib/src/widgets/router.dart
  @override
  Widget build(BuildContext context) {
    return UnmanagedRestorationScope(
      bucket: bucket,
      child: _RouterScope(
        routeInformationProvider: widget.routeInformationProvider,
        backButtonDispatcher: widget.backButtonDispatcher,
        routeInformationParser: widget.routeInformationParser,
        routerDelegate: widget.routerDelegate,
        routerState: this,
        child: Builder(
          // Use a Builder so that the build method below will have a
          // BuildContext that contains the _RouterScope. This also prevents
          // dependencies look ups in routerDelegate from rebuilding Router
          // widget that may result in re-parsing the route information.
          builder: widget.routerDelegate.build,
        ),
      ),
    );
  }
```

构建 widget 树：

- 使用 `UnmanagedRestorationScope` 提供恢复桶
- 使用 `_RouterScope` 提供路由相关的上下文（通过 `InheritedWidget`）
- 使用 `Builder` 确保 `RouterDelegate.build` 能够访问 `_RouterScope`
- 最终调用 `RouterDelegate.build` 构建导航 widget

## 工作流程

### 初始化流程

1. `Router` widget 创建，`_RouterState.initState` 被调用
2. 注册各种监听器
3. `didChangeDependencies` 被调用
4. 如果有初始路由信息，调用 `_processRouteInformation` 处理
5. `RouteInformationParser` 解析路由信息
6. 解析完成后，调用 `RouterDelegate.setInitialRoutePath`
7. `RouterDelegate` 更新状态并通知监听器
8. `Router` 重建，调用 `RouterDelegate.build` 构建导航 widget

### 路由变化流程

1. `RouteInformationProvider` 通知新的路由信息
2. `_handleRouteInformationProviderNotification` 被调用
3. 调用 `_processRouteInformation` 处理新路由
4. `RouteInformationParser` 异步解析路由信息
5. 解析完成后，调用 `RouterDelegate.setNewRoutePath`
6. `RouterDelegate` 更新状态并通知
7. `Router` 重建
8. 检查是否需要报告路由信息（用于 Web 平台的 URL 更新）

### 返回按钮处理流程

1. 用户按下系统返回按钮
2. `BackButtonDispatcher` 触发回调
3. `_handleBackButtonDispatcherNotification` 被调用
4. 调用 `RouterDelegate.popRoute()`
5. `RouterDelegate` 处理返回逻辑并返回结果
6. 如果事务仍然有效，触发重建

### 路由信息报告流程（Web 平台）

1. `Router` 重建后，`_maybeNeedToReportRouteInformation` 被调用
2. 从 `RouterDelegate.currentConfiguration` 获取当前配置
3. 通过 `RouteInformationParser.restoreRouteInformation` 转换为 `RouteInformation`
4. 调度报告任务（使用 `addPostFrameCallback`）
5. 帧结束后，`_reportRouteInformation` 被调用
6. 调用 `RouteInformationProvider.routerReportsNewRouteInformation` 报告
7. `PlatformRouteInformationProvider` 更新浏览器 URL 和历史记录

## 关键概念

### 异步操作处理

`Router` 使用事务对象（`_currentRouterTransaction`）来跟踪异步操作：

- 当新的路由操作开始时，创建新的事务对象
- 如果旧的操作完成时事务已变化，结果会被丢弃
- 这防止了竞态条件，确保只有最新的操作结果被应用

### 路由信息报告类型

`RouteInformationReportingType` 枚举定义了三种报告类型：

- `none`：默认类型，根据 URI 是否变化决定是否创建历史记录
- `navigate`：强制创建新的历史记录条目（即使 URI 未变化）
- `neglect`：强制替换当前历史记录条目（即使 URI 变化）

### 状态恢复

`Router` 支持状态恢复：

- 如果提供了 `restorationScopeId`，`Router` 会持久化当前配置
- 应用重启后，通过 `setRestoredRoutePath` 恢复状态
- 使用 `RouteInformationParser` 进行序列化和反序列化

### 多 Router 架构

应用可以有多个 `Router` widget：

- 根 `Router` 处理整个路由解析
- 子 `Router` 可以基于父 `Router` 的结果构建子路由
- 只有根 `Router` 应该更新 URL（Web 平台）

## 使用场景

### 标准 Web 应用

```dart
Router<MyRouteConfig>(
  routeInformationProvider: PlatformRouteInformationProvider(
    initialRouteInformation: RouteInformation(uri: Uri.parse('/')),
  ),
  routeInformationParser: MyRouteInformationParser(),
  routerDelegate: MyRouterDelegate(),
)
```

### 不依赖 URL 的子路由

```dart
Router<SubRouteConfig>(
  // routeInformationProvider 和 routeInformationParser 为 null
  routerDelegate: SubRouterDelegate(),
)
```

### 使用 RouterConfig

```dart
Router.withConfig(
  config: MyRouterConfig(),
  restorationScopeId: 'router',
)
```

## 总结

`Router` 和 `_RouterState` 共同实现了 Flutter 的声明式路由系统。它们通过协调 `RouteInformationProvider`、`RouteInformationParser`、`RouterDelegate` 和 `BackButtonDispatcher` 四个核心组件，实现了从路由信息到导航 widget 的完整流程。系统设计考虑了异步操作、状态恢复、Web 平台 URL 同步等复杂场景，为 Flutter 应用提供了强大而灵活的路由能力。
