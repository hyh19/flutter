# WidgetsApp 构造函数详解

## 概述

`WidgetsApp` 提供了两个构造函数：普通构造函数和 `router` 构造函数。普通构造函数用于基于 `Navigator` 的路由系统，而 `router` 构造函数用于基于 `Router` 的路由系统。两个构造函数都包含大量的参数验证逻辑，确保配置的正确性。

## 普通构造函数

```dart 330:418:packages/flutter/lib/src/widgets/app.dart
  WidgetsApp({
    // can't be const because the asserts use methods on Iterable :-(
    super.key,
    this.navigatorKey,
    this.onGenerateRoute,
    this.onGenerateInitialRoutes,
    this.onUnknownRoute,
    this.onNavigationNotification,
    List<NavigatorObserver> this.navigatorObservers = const <NavigatorObserver>[],
    this.initialRoute,
    this.pageRouteBuilder,
    this.home,
    Map<String, WidgetBuilder> this.routes = const <String, WidgetBuilder>{},
    this.builder,
    this.title,
    this.onGenerateTitle,
    this.textStyle,
    required this.color,
    this.locale,
    this.localizationsDelegates,
    this.localeListResolutionCallback,
    this.localeResolutionCallback,
    this.supportedLocales = const <Locale>[Locale('en', 'US')],
    this.showPerformanceOverlay = false,
    this.showSemanticsDebugger = false,
    this.debugShowWidgetInspector = false,
    this.debugShowCheckedModeBanner = true,
    this.exitWidgetSelectionButtonBuilder,
    this.moveExitWidgetSelectionButtonBuilder,
    this.tapBehaviorButtonBuilder,
    this.shortcuts,
    this.actions,
    this.restorationScopeId,
    @Deprecated(
      'Remove this parameter as it is now ignored. '
      'WidgetsApp never introduces its own MediaQuery; the View widget takes care of that. '
      'This feature was deprecated after v3.7.0-29.0.pre.',
    )
    this.useInheritedMediaQuery = false,
  }) : assert(
         home == null || onGenerateInitialRoutes == null,
         'If onGenerateInitialRoutes is specified, the home argument will be '
         'redundant.',
       ),
       assert(
         home == null || !routes.containsKey(Navigator.defaultRouteName),
         'If the home property is specified, the routes table '
         'cannot include an entry for "/", since it would be redundant.',
       ),
       assert(
         builder != null ||
             home != null ||
             routes.containsKey(Navigator.defaultRouteName) ||
             onGenerateRoute != null ||
             onUnknownRoute != null,
         'Either the home property must be specified, '
         'or the routes table must include an entry for "/", '
         'or there must be on onGenerateRoute callback specified, '
         'or there must be an onUnknownRoute callback specified, '
         'or the builder property must be specified, '
         'because otherwise there is nothing to fall back on if the '
         'app is started with an intent that specifies an unknown route.',
       ),
       assert(
         (home != null || routes.isNotEmpty || onGenerateRoute != null || onUnknownRoute != null) ||
             (builder != null &&
                 navigatorKey == null &&
                 initialRoute == null &&
                 navigatorObservers.isEmpty),
         'If no route is provided using '
         'home, routes, onGenerateRoute, or onUnknownRoute, '
         'a non-null callback for the builder property must be provided, '
         'and the other navigator-related properties, '
         'navigatorKey, initialRoute, and navigatorObservers, '
         'must have their initial values '
         '(null, null, and the empty list, respectively).',
       ),
       assert(
         builder != null || onGenerateRoute != null || pageRouteBuilder != null,
         'If neither builder nor onGenerateRoute are provided, the '
         'pageRouteBuilder must be specified so that the default handler '
         'will know what kind of PageRoute transition to build.',
       ),
       assert(supportedLocales.isNotEmpty),
       routeInformationProvider = null,
       routeInformationParser = null,
       routerDelegate = null,
       backButtonDispatcher = null,
       routerConfig = null;
```

### 参数分类

普通构造函数的参数可以分为以下几类：

1. **路由相关**：`navigatorKey`, `onGenerateRoute`, `onGenerateInitialRoutes`, `onUnknownRoute`, `pageRouteBuilder`, `home`, `routes`, `initialRoute`, `navigatorObservers`
2. **构建相关**：`builder`
3. **应用信息**：`title`, `onGenerateTitle`, `color`
4. **样式相关**：`textStyle`
5. **本地化相关**：`locale`, `localizationsDelegates`, `localeListResolutionCallback`, `localeResolutionCallback`, `supportedLocales`
6. **调试相关**：`showPerformanceOverlay`, `showSemanticsDebugger`, `debugShowWidgetInspector`, `debugShowCheckedModeBanner`
7. **工具相关**：`exitWidgetSelectionButtonBuilder`, `moveExitWidgetSelectionButtonBuilder`, `tapBehaviorButtonBuilder`
8. **交互相关**：`shortcuts`, `actions`
9. **其他**：`onNavigationNotification`, `restorationScopeId`, `useInheritedMediaQuery`（已废弃）

### 断言验证逻辑

#### 断言 1：home 与 onGenerateInitialRoutes 互斥

```dart
assert(
  home == null || onGenerateInitialRoutes == null,
  'If onGenerateInitialRoutes is specified, the home argument will be redundant.',
)
```

**目的**：`home` 和 `onGenerateInitialRoutes` 都用于指定初始路由，如果同时指定会产生冗余。

#### 断言 2：home 与 routes 中的 "/" 互斥

```dart
assert(
  home == null || !routes.containsKey(Navigator.defaultRouteName),
  'If the home property is specified, the routes table cannot include an entry for "/", since it would be redundant.',
)
```

**目的**：`home` 属性隐式定义了 "/" 路由，因此 `routes` 表中不能再包含 "/" 条目。

#### 断言 3：必须提供某种路由机制

```dart
assert(
  builder != null ||
      home != null ||
      routes.containsKey(Navigator.defaultRouteName) ||
      onGenerateRoute != null ||
      onUnknownRoute != null,
  'Either the home property must be specified, ...',
)
```

**目的**：确保应用有某种方式来处理路由，否则当应用启动时遇到未知路由将无法处理。

#### 断言 4：builder 与其他导航属性的关系

```dart
assert(
  (home != null || routes.isNotEmpty || onGenerateRoute != null || onUnknownRoute != null) ||
      (builder != null &&
          navigatorKey == null &&
          initialRoute == null &&
          navigatorObservers.isEmpty),
  'If no route is provided using ...',
)
```

**目的**：如果没有提供路由（通过 `home`、`routes`、`onGenerateRoute` 或 `onUnknownRoute`），则必须提供 `builder`，并且其他导航相关属性必须使用默认值。

#### 断言 5：pageRouteBuilder 的必要性

```dart
assert(
  builder != null || onGenerateRoute != null || pageRouteBuilder != null,
  'If neither builder nor onGenerateRoute are provided, the pageRouteBuilder must be specified ...',
)
```

**目的**：如果既没有提供 `builder` 也没有提供 `onGenerateRoute`，则必须提供 `pageRouteBuilder`，以便默认路由处理器知道如何构建 `PageRoute`。

#### 断言 6：supportedLocales 不能为空

```dart
assert(supportedLocales.isNotEmpty)
```

**目的**：确保应用至少支持一种语言环境。

### Router 相关属性的初始化

在普通构造函数中，所有 Router 相关的属性都被初始化为 `null`：

```dart
routeInformationProvider = null,
routeInformationParser = null,
routerDelegate = null,
backButtonDispatcher = null,
routerConfig = null;
```

这确保了普通构造函数不会使用 Router 相关的功能。

## Router 构造函数

```dart 420:492:packages/flutter/lib/src/widgets/app.dart
  /// Creates a [WidgetsApp] that uses the [Router] instead of a [Navigator].
  ///
  /// {@template flutter.widgets.WidgetsApp.router}
  /// If the [routerConfig] is provided, the other router related delegates,
  /// [routeInformationParser], [routeInformationProvider], [routerDelegate],
  /// and [backButtonDispatcher], must all be null.
  /// {@endtemplate}
  WidgetsApp.router({
    super.key,
    this.routeInformationProvider,
    this.routeInformationParser,
    this.routerDelegate,
    this.routerConfig,
    this.backButtonDispatcher,
    this.builder,
    this.title,
    this.onGenerateTitle,
    this.onNavigationNotification,
    this.textStyle,
    required this.color,
    this.locale,
    this.localizationsDelegates,
    this.localeListResolutionCallback,
    this.localeResolutionCallback,
    this.supportedLocales = const <Locale>[Locale('en', 'US')],
    this.showPerformanceOverlay = false,
    this.showSemanticsDebugger = false,
    this.debugShowWidgetInspector = false,
    this.debugShowCheckedModeBanner = true,
    this.exitWidgetSelectionButtonBuilder,
    this.moveExitWidgetSelectionButtonBuilder,
    this.tapBehaviorButtonBuilder,
    this.shortcuts,
    this.actions,
    this.restorationScopeId,
    @Deprecated(
      'Remove this parameter as it is now ignored. '
      'WidgetsApp never introduces its own MediaQuery; the View widget takes care of that. '
      'This feature was deprecated after v3.7.0-29.0.pre.',
    )
    this.useInheritedMediaQuery = false,
  }) : assert(() {
         if (routerConfig != null) {
           assert(
             (routeInformationProvider ??
                     routeInformationParser ??
                     routerDelegate ??
                     backButtonDispatcher) ==
                 null,
             'If the routerConfig is provided, all the other router delegates must not be provided',
           );
           return true;
         }
         assert(
           routerDelegate != null,
           'Either one of routerDelegate or routerConfig must be provided',
         );
         assert(
           routeInformationProvider == null || routeInformationParser != null,
           'If routeInformationProvider is provided, routeInformationParser must also be provided',
         );
         return true;
       }()),
       assert(supportedLocales.isNotEmpty),
       navigatorObservers = null,
       navigatorKey = null,
       onGenerateRoute = null,
       pageRouteBuilder = null,
       home = null,
       onGenerateInitialRoutes = null,
       onUnknownRoute = null,
       routes = null,
       initialRoute = null;
```

### 参数特点

`router` 构造函数与普通构造函数的主要区别：

1. **Router 相关参数**：包含 `routeInformationProvider`, `routeInformationParser`, `routerDelegate`, `routerConfig`, `backButtonDispatcher`
2. **缺少 Navigator 相关参数**：不包含 `navigatorKey`, `onGenerateRoute`, `home`, `routes` 等 Navigator 相关参数
3. **共享参数**：包含与普通构造函数相同的应用信息、本地化、调试等参数

### Router 构造函数的断言验证逻辑

#### 断言 1：routerConfig 与其他 Router 委托互斥

```dart
if (routerConfig != null) {
  assert(
    (routeInformationProvider ??
            routeInformationParser ??
            routerDelegate ??
            backButtonDispatcher) ==
        null,
    'If the routerConfig is provided, all the other router delegates must not be provided',
  );
}
```

**目的**：如果提供了 `routerConfig`，则不能同时提供其他 Router 相关的委托（`routeInformationProvider`、`routeInformationParser`、`routerDelegate`、`backButtonDispatcher`）。`routerConfig` 是一个封装了所有 Router 配置的对象。

#### 断言 2：必须提供 routerDelegate 或 routerConfig

```dart
assert(
  routerDelegate != null,
  'Either one of routerDelegate or routerConfig must be provided',
);
```

**目的**：必须提供 `routerDelegate` 或 `routerConfig` 之一，否则无法构建 Router。

#### 断言 3：routeInformationProvider 需要 routeInformationParser

```dart
assert(
  routeInformationProvider == null || routeInformationParser != null,
  'If routeInformationProvider is provided, routeInformationParser must also be provided',
);
```

**目的**：如果提供了 `routeInformationProvider`，则必须同时提供 `routeInformationParser`，因为需要解析器来处理路由信息。

### Navigator 相关属性的初始化

在 `router` 构造函数中，所有 Navigator 相关的属性都被初始化为 `null` 或默认值：

```dart
navigatorObservers = null,
navigatorKey = null,
onGenerateRoute = null,
pageRouteBuilder = null,
home = null,
onGenerateInitialRoutes = null,
onUnknownRoute = null,
routes = null,
initialRoute = null;
```

这确保了 `router` 构造函数不会使用 Navigator 相关的功能。

## 两个构造函数的对比

### 相同点

1. **共享参数**：两个构造函数都包含应用信息、本地化、调试、交互等共享参数
2. **参数验证**：都包含 `supportedLocales.isNotEmpty` 的验证
3. **默认值**：许多参数在两个构造函数中都有相同的默认值

### 不同点

| 特性 | 普通构造函数 | Router 构造函数 |
| --- | --- | --- |
| **路由系统** | Navigator | Router |
| **核心参数** | `navigatorKey`, `home`, `routes`, `onGenerateRoute` | `routerDelegate`, `routeInformationParser`, `routerConfig` |
| **适用场景** | 传统的命名路由系统 | 声明式路由系统 |
| **初始化** | Router 相关属性为 `null` | Navigator 相关属性为 `null` |

## 使用建议

### 使用普通构造函数

适用于：

- 使用传统的命名路由（`Navigator.pushNamed`）
- 简单的路由需求
- 不需要深度链接或 Web URL 支持

```dart
WidgetsApp(
  home: MyHomePage(),
  routes: {
    '/details': (context) => DetailsPage(),
  },
  pageRouteBuilder: (settings, builder) => MaterialPageRoute(
    settings: settings,
    builder: builder,
  ),
  color: Colors.blue,
)
```

### 使用 Router 构造函数

适用于：

- 需要声明式路由管理
- 需要深度链接支持
- 需要 Web URL 同步
- 复杂的路由需求

```dart
WidgetsApp.router(
  routerConfig: routerConfig,
  color: Colors.blue,
)
```

## 总结

第二部分详细介绍了 `WidgetsApp` 的两个构造函数：

1. **普通构造函数**：用于基于 `Navigator` 的路由系统，包含 5 个主要断言验证路由配置的正确性

2. **Router 构造函数**：用于基于 `Router` 的路由系统，包含 3 个主要断言验证 Router 配置的正确性

3. **参数验证**：两个构造函数都包含严格的参数验证，确保配置的正确性和一致性

4. **互斥性**：两个构造函数确保 Navigator 和 Router 相关属性不会同时使用，避免配置冲突

这些验证逻辑确保了 `WidgetsApp` 的正确使用，防止了常见的配置错误。
