# Material Page 与 PageRoute 实现解析

## 概述

`page.dart` 文件实现了 Flutter 中 Material Design 风格的页面路由系统，提供了两种主要的方式来创建和导航页面：

1. **基于 Route 的方式**：使用 `MaterialPageRoute<T>` 直接创建路由
2. **基于 Page 的方式**：使用 `MaterialPage<T>` 通过 Page 系统创建路由

这两种方式最终都使用相同的过渡动画系统（`MaterialRouteTransitionMixin`），提供了跨平台的适配过渡效果。

## 核心组件

### 1. `MaterialPageRoute<T>`

`MaterialPageRoute<T>` 是 Material Design 风格的模态路由类，用于替换整个屏幕并显示新的页面。

```dart 35:62:packages/flutter/lib/src/material/page.dart
class MaterialPageRoute<T> extends PageRoute<T> with MaterialRouteTransitionMixin<T> {
  /// Construct a MaterialPageRoute whose contents are defined by [builder].
  MaterialPageRoute({
    required this.builder,
    super.settings,
    super.requestFocus,
    this.maintainState = true,
    super.fullscreenDialog,
    super.allowSnapshotting = true,
    super.barrierDismissible = false,
    super.traversalEdgeBehavior,
    super.directionalTraversalEdgeBehavior,
  }) {
    assert(opaque);
  }

  /// Builds the primary contents of the route.
  final WidgetBuilder builder;

  @override
  Widget buildContent(BuildContext context) => builder(context);

  @override
  final bool maintainState;

  @override
  String get debugLabel => '${super.debugLabel}(${settings.name})';
}
```

#### 主要特性

- **继承关系**：继承自 `PageRoute<T>`，混入了 `MaterialRouteTransitionMixin<T>`
- **构建方式**：通过 `builder` 回调函数构建页面内容
- **状态保持**：默认保持状态（`maintainState = true`），页面被替换后仍保留在内存中
- **不透明性**：在构造函数中断言 `opaque` 为 true，确保页面完全覆盖底层内容

#### 使用示例

```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => SecondScreen(),
    settings: RouteSettings(name: '/second'),
  ),
);
```

### 2. `MaterialRouteTransitionMixin<T>`

`MaterialRouteTransitionMixin<T>` 是一个 mixin，提供了平台自适应的过渡动画效果。

```dart 85:111:packages/flutter/lib/src/material/page.dart
mixin MaterialRouteTransitionMixin<T> on PageRoute<T> {
  /// Builds the primary contents of the route.
  @protected
  Widget buildContent(BuildContext context);

  @override
  Duration get transitionDuration =>
      _getPageTransitionBuilder(navigator!.context)?.transitionDuration ??
      const Duration(microseconds: 300);

  @override
  Duration get reverseTransitionDuration =>
      _getPageTransitionBuilder(navigator!.context)?.reverseTransitionDuration ??
      const Duration(microseconds: 300);

  PageTransitionsBuilder? _getPageTransitionBuilder(BuildContext context) {
    final TargetPlatform platform = Theme.of(context).platform;
    final PageTransitionsTheme pageTransitionsTheme = Theme.of(context).pageTransitionsTheme;
    return pageTransitionsTheme.builders[platform] ??
        switch (platform) {
          TargetPlatform.iOS || TargetPlatform.macOS => const CupertinoPageTransitionsBuilder(),
          TargetPlatform.android ||
          TargetPlatform.fuchsia ||
          TargetPlatform.windows ||
          TargetPlatform.linux => const ZoomPageTransitionsBuilder(),
        };
  }
```

#### 平台适配逻辑

该 mixin 根据当前平台选择不同的过渡动画：

- **iOS/macOS**：使用 `CupertinoPageTransitionsBuilder`，页面从右侧滑入，带有视差效果
- **Android/其他平台**：使用 `ZoomPageTransitionsBuilder`，页面缩放并淡入淡出

过渡动画也可以通过 `Theme.of(context).pageTransitionsTheme` 进行自定义。

#### 过渡动画持续时间

```dart 90:98:packages/flutter/lib/src/material/page.dart
  @override
  Duration get transitionDuration =>
      _getPageTransitionBuilder(navigator!.context)?.transitionDuration ??
      const Duration(microseconds: 300);

  @override
  Duration get reverseTransitionDuration =>
      _getPageTransitionBuilder(navigator!.context)?.reverseTransitionDuration ??
      const Duration(microseconds: 300);
```

默认过渡持续时间为 300 微秒，但会优先使用主题中配置的过渡构建器的持续时间。

#### 动画控制器更新机制

由于 `AnimationController` 只在创建时构建一次，当过渡持续时间发生变化时，需要手动更新控制器：

```dart 118:133:packages/flutter/lib/src/material/page.dart
  @override
  TickerFuture didPush() {
    controller?.duration = transitionDuration;
    return super.didPush();
  }

  // The reverseTransitionDuration is used to create the AnimationController
  // which is only built once, so when page transition builder is updated and
  // reverseTransitionDuration has a new value, the AnimationController cannot
  // be updated automatically. So we manually update its reverseDuration here.
  // TODO(quncCccccc): Clean up this override method when controller can beupdated as the reverseTransitionDuration is changed.
  @override
  bool didPop(T? result) {
    controller?.reverseDuration = reverseTransitionDuration;
    return super.didPop(result);
  }
```

这种设计是临时方案，代码注释中也提到了未来需要清理这部分逻辑。

#### 过渡协调机制

mixin 实现了 `canTransitionTo` 和 `canTransitionFrom` 方法来协调多个路由之间的过渡：

```dart 161:185:packages/flutter/lib/src/material/page.dart
  @override
  bool canTransitionTo(TransitionRoute<dynamic> nextRoute) {
    // Don't perform outgoing animation if the next route is a fullscreen dialog,
    // or there is no matching transition to use.
    // Don't perform outgoing animation if the next route is a fullscreen dialog.
    final bool nextRouteIsNotFullscreen =
        (nextRoute is! PageRoute<T>) || !nextRoute.fullscreenDialog;

    // If the next route has a delegated transition, then this route is able to
    // use that delegated transition to smoothly sync with the next route's
    // transition.
    final bool nextRouteHasDelegatedTransition =
        nextRoute is ModalRoute<T> && nextRoute.delegatedTransition != null;

    // Otherwise if the next route has the same route transition mixin as this
    // one, then this route will already be synced with its transition.
    return nextRouteIsNotFullscreen &&
        ((nextRoute is MaterialRouteTransitionMixin) || nextRouteHasDelegatedTransition);
  }

  @override
  bool canTransitionFrom(TransitionRoute<dynamic> previousRoute) {
    // Suppress previous route from transitioning if this is a fullscreenDialog route.
    return previousRoute is PageRoute && !fullscreenDialog;
  }
```

- **canTransitionTo**：判断是否可以向下一个路由执行过渡动画，当下一路由是全屏对话框时不会执行出栈动画
- **canTransitionFrom**：判断前一个路由是否可以执行过渡，全屏对话框会阻止前一个路由的过渡

#### 页面构建

```dart 187:195:packages/flutter/lib/src/material/page.dart
  @override
  Widget buildPage(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
  ) {
    final Widget result = buildContent(context);
    return Semantics(scopesRoute: true, explicitChildNodes: true, child: result);
  }
```

`buildPage` 方法构建页面的主要内容，并包装在 `Semantics` widget 中以提供语义信息。

#### 过渡构建

```dart 197:206:packages/flutter/lib/src/material/page.dart
  @override
  Widget buildTransitions(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
    Widget child,
  ) {
    final PageTransitionsTheme theme = Theme.of(context).pageTransitionsTheme;
    return theme.buildTransitions<T>(this, context, animation, secondaryAnimation, child);
  }
```

`buildTransitions` 方法委托给主题的 `buildTransitions` 方法来构建实际的过渡动画效果。

### 3. `MaterialPage<T>`

`MaterialPage<T>` 是基于 `Page<T>` 的 Material Design 页面实现，用于声明式导航系统。

```dart 229:260:packages/flutter/lib/src/material/page.dart
class MaterialPage<T> extends Page<T> {
  /// Creates a material page.
  const MaterialPage({
    required this.child,
    this.maintainState = true,
    this.fullscreenDialog = false,
    this.allowSnapshotting = true,
    super.key,
    super.canPop,
    super.onPopInvoked,
    super.name,
    super.arguments,
    super.restorationId,
  });

  /// The content to be shown in the [Route] created by this page.
  final Widget child;

  /// {@macro flutter.widgets.ModalRoute.maintainState}
  final bool maintainState;

  /// {@macro flutter.widgets.PageRoute.fullscreenDialog}
  final bool fullscreenDialog;

  /// {@macro flutter.widgets.TransitionRoute.allowSnapshotting}
  final bool allowSnapshotting;

  @override
  Route<T> createRoute(BuildContext context) {
    return _PageBasedMaterialPageRoute<T>(page: this, allowSnapshotting: allowSnapshotting);
  }
}
```

#### 与 MaterialPageRoute 的区别

- **MaterialPageRoute**：命令式导航，通过 `Navigator.push` 直接创建和推送路由
- **MaterialPage**：声明式导航，作为 `Page` 对象传递给 `Navigator.pages`，由 Navigator 管理路由的生命周期

#### 使用示例

```dart
Navigator(
  pages: [
    MaterialPage(
      key: ValueKey('home'),
      child: HomeScreen(),
    ),
    MaterialPage(
      key: ValueKey('detail'),
      child: DetailScreen(),
    ),
  ],
  onPopPage: (route, result) {
    if (!route.didPop(result)) {
      return false;
    }
    // 更新 pages 列表
    return true;
  },
)
```

### 4. `_PageBasedMaterialPageRoute<T>`

`_PageBasedMaterialPageRoute<T>` 是内部实现类，将 `MaterialPage` 转换为实际的 `PageRoute`。

```dart 266:287:packages/flutter/lib/src/material/page.dart
class _PageBasedMaterialPageRoute<T> extends PageRoute<T> with MaterialRouteTransitionMixin<T> {
  _PageBasedMaterialPageRoute({required MaterialPage<T> page, super.allowSnapshotting})
    : super(settings: page) {
    assert(opaque);
  }

  MaterialPage<T> get _page => settings as MaterialPage<T>;

  @override
  Widget buildContent(BuildContext context) {
    return _page.child;
  }

  @override
  bool get maintainState => _page.maintainState;

  @override
  bool get fullscreenDialog => _page.fullscreenDialog;

  @override
  String get debugLabel => '${super.debugLabel}(${_page.name})';
}
```

#### 设计目的

这个类确保了基于 `Page` 的路由系统能够使用与 `MaterialPageRoute` 相同的过渡动画和行为。它从关联的 `MaterialPage` 中读取配置，并构建相应的路由。

#### 关键特性

- **设置关联**：通过 `super(settings: page)` 将 `MaterialPage` 作为路由设置
- **内容构建**：直接使用 `_page.child` 作为页面内容
- **配置代理**：将 `maintainState` 和 `fullscreenDialog` 等属性从 Page 代理到 Route

## 过渡动画详解

### Android 风格过渡

在 Android 平台上，默认使用 `ZoomPageTransitionsBuilder`：

- **进入动画**：页面放大并淡入，同时旧页面缩小并淡出
- **退出动画**：相反的过程

这种效果提供了视觉上的深度感。

### iOS 风格过渡

在 iOS/macOS 平台上，默认使用 `CupertinoPageTransitionsBuilder`：

- **进入动画**：新页面从右侧滑入
- **视差效果**：旧页面向左移动，产生视差效果
- **退出动画**：相反的过程

在从右到左（RTL）的环境中，方向会反转。

### 自定义过渡

可以通过 `PageTransitionsTheme` 来自定义不同平台的过渡动画：

```dart
MaterialApp(
  theme: ThemeData(
    pageTransitionsTheme: PageTransitionsTheme(
      builders: {
        TargetPlatform.android: FadeUpwardsPageTransitionsBuilder(),
        TargetPlatform.iOS: CupertinoPageTransitionsBuilder(),
      },
    ),
  ),
)
```

## 委托过渡（Delegated Transition）

`MaterialRouteTransitionMixin` 支持委托过渡机制，允许多个路由协同工作以创建更复杂的过渡效果：

```dart 141:159:packages/flutter/lib/src/material/page.dart
  @override
  DelegatedTransitionBuilder? get delegatedTransition => _delegatedTransition;

  static Widget? _delegatedTransition(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
    bool allowSnapshotting,
    Widget? child,
  ) {
    final PageTransitionsTheme theme = Theme.of(context).pageTransitionsTheme;
    final TargetPlatform platform = Theme.of(context).platform;
    final DelegatedTransitionBuilder? themeDelegatedTransition = theme.delegatedTransition(
      platform,
    );
    return themeDelegatedTransition != null
        ? themeDelegatedTransition(context, animation, secondaryAnimation, allowSnapshotting, child)
        : null;
  }
```

委托过渡允许当前路由使用下一个路由的过渡效果，实现更平滑的视觉衔接。

## 无障碍支持

页面内容被包装在 `Semantics` widget 中：

```dart 187:195:packages/flutter/lib/src/material/page.dart
  @override
  Widget buildPage(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
  ) {
    final Widget result = buildContent(context);
    return Semantics(scopesRoute: true, explicitChildNodes: true, child: result);
  }
```

- `scopesRoute: true`：标识这是一个路由范围
- `explicitChildNodes: true`：确保子节点的语义信息被正确暴露

这确保了屏幕阅读器能够正确地识别和导航页面。

## 状态管理

### maintainState 属性

```dart 57:58:packages/flutter/lib/src/material/page.dart
  @override
  final bool maintainState;
```

- **true**（默认）：页面被替换后仍然保留在内存中，状态得以保持
- **false**：页面被替换后会被销毁，释放资源

当页面包含大量状态或资源时，设置为 `false` 可以帮助减少内存使用。

### 全屏对话框

```dart 19:21:packages/flutter/lib/src/material/page.dart
/// The `fullscreenDialog` property specifies whether the incoming route is a
/// fullscreen modal dialog. On iOS, those routes animate from the bottom to the
/// top rather than horizontally.
```

全屏对话框在 iOS 上会使用不同的过渡动画（从底部滑入），并且会阻止前一个路由的过渡动画。

## 调试支持

### debugLabel

```dart 60:61:packages/flutter/lib/src/material/page.dart
  @override
  String get debugLabel => '${super.debugLabel}(${settings.name})';
```

调试标签包含路由的名称（如果有的话），便于在开发工具中识别路由。

## 总结

`page.dart` 文件提供了完整的 Material Design 风格页面路由实现，包括：

1. **两种使用方式**：命令式的 `MaterialPageRoute` 和声明式的 `MaterialPage`
2. **平台自适应**：根据平台自动选择合适的过渡动画
3. **灵活定制**：支持通过主题自定义过渡效果
4. **性能优化**：支持状态管理和资源释放控制
5. **无障碍支持**：内置语义化支持
6. **过渡协调**：智能协调多个路由之间的过渡动画

这些特性使得 Material Design 路由系统既易于使用，又足够灵活以满足各种需求。
