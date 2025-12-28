# MaterialApp 构造函数详解

## 概述

`MaterialApp` 提供了两个构造函数：普通构造函数和 `router` 构造函数。普通构造函数用于基于 `Navigator` 的路由系统，而 `router` 构造函数用于基于 `Router` 的路由系统。两个构造函数都包含大量的参数，用于配置 Material Design 应用的各种功能。

## 普通构造函数

```dart 218:267:packages/flutter/lib/src/material/app.dart
  const MaterialApp({
    super.key,
    this.navigatorKey,
    this.scaffoldMessengerKey,
    this.home,
    Map<String, WidgetBuilder> this.routes = const <String, WidgetBuilder>{},
    this.initialRoute,
    this.onGenerateRoute,
    this.onGenerateInitialRoutes,
    this.onUnknownRoute,
    this.onNavigationNotification,
    List<NavigatorObserver> this.navigatorObservers = const <NavigatorObserver>[],
    this.builder,
    this.title = '',
    this.onGenerateTitle,
    this.color,
    this.theme,
    this.darkTheme,
    this.highContrastTheme,
    this.highContrastDarkTheme,
    this.themeMode = ThemeMode.system,
    this.themeAnimationDuration = kThemeAnimationDuration,
    this.themeAnimationCurve = Curves.linear,
    this.locale,
    this.localizationsDelegates,
    this.localeListResolutionCallback,
    this.localeResolutionCallback,
    this.supportedLocales = const <Locale>[Locale('en', 'US')],
    this.debugShowMaterialGrid = false,
    this.showPerformanceOverlay = false,
    this.checkerboardRasterCacheImages = false,
    this.checkerboardOffscreenLayers = false,
    this.showSemanticsDebugger = false,
    this.debugShowCheckedModeBanner = true,
    this.shortcuts,
    this.actions,
    this.restorationScopeId,
    this.scrollBehavior,
    @Deprecated(
      'Remove this parameter as it is now ignored. '
      'MaterialApp never introduces its own MediaQuery; the View widget takes care of that. '
      'This feature was deprecated after v3.7.0-29.0.pre.',
    )
    this.useInheritedMediaQuery = false,
    this.themeAnimationStyle,
  }) : routeInformationProvider = null,
       routeInformationParser = null,
       routerDelegate = null,
       backButtonDispatcher = null,
       routerConfig = null;
```

### 参数分类

普通构造函数的参数可以分为以下几类：

1. **路由相关（Navigator）**：`navigatorKey`, `home`, `routes`, `initialRoute`, `onGenerateRoute`, `onGenerateInitialRoutes`, `onUnknownRoute`, `navigatorObservers`, `onNavigationNotification`
2. **Material 特有**：`scaffoldMessengerKey`, `theme`, `darkTheme`, `highContrastTheme`, `highContrastDarkTheme`, `themeMode`, `themeAnimationDuration`, `themeAnimationCurve`, `themeAnimationStyle`, `scrollBehavior`, `debugShowMaterialGrid`
3. **构建相关**：`builder`
4. **应用信息**：`title`, `onGenerateTitle`, `color`
5. **本地化相关**：`locale`, `localizationsDelegates`, `localeListResolutionCallback`, `localeResolutionCallback`, `supportedLocales`
6. **调试相关**：`showPerformanceOverlay`, `checkerboardRasterCacheImages`, `checkerboardOffscreenLayers`, `showSemanticsDebugger`, `debugShowCheckedModeBanner`
7. **交互相关**：`shortcuts`, `actions`
8. **其他**：`restorationScopeId`, `useInheritedMediaQuery`（已废弃）

### 默认值

- `routes`：默认为空映射 `const <String, WidgetBuilder>{}`
- `title`：默认为空字符串 `''`
- `themeMode`：默认为 `ThemeMode.system`（跟随系统设置）
- `themeAnimationDuration`：默认为 `kThemeAnimationDuration`
- `themeAnimationCurve`：默认为 `Curves.linear`
- `supportedLocales`：默认为 `[Locale('en', 'US')]`
- `navigatorObservers`：默认为空列表 `const <NavigatorObserver>[]`
- `debugShowCheckedModeBanner`：默认为 `true`（在调试模式下显示横幅）

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

```dart 269:322:packages/flutter/lib/src/material/app.dart
  /// Creates a [MaterialApp] that uses the [Router] instead of a [Navigator].
  ///
  /// {@macro flutter.widgets.WidgetsApp.router}
  const MaterialApp.router({
    super.key,
    this.scaffoldMessengerKey,
    this.routeInformationProvider,
    this.routeInformationParser,
    this.routerDelegate,
    this.routerConfig,
    this.backButtonDispatcher,
    this.builder,
    this.title,
    this.onGenerateTitle,
    this.onNavigationNotification,
    this.color,
    this.theme,
    this.darkTheme,
    this.highContrastTheme,
    this.highContrastDarkTheme,
    this.themeMode = ThemeMode.system,
    this.themeAnimationDuration = kThemeAnimationDuration,
    this.themeAnimationCurve = Curves.linear,
    this.locale,
    this.localizationsDelegates,
    this.localeListResolutionCallback,
    this.localeResolutionCallback,
    this.supportedLocales = const <Locale>[Locale('en', 'US')],
    this.debugShowMaterialGrid = false,
    this.showPerformanceOverlay = false,
    this.checkerboardRasterCacheImages = false,
    this.checkerboardOffscreenLayers = false,
    this.showSemanticsDebugger = false,
    this.debugShowCheckedModeBanner = true,
    this.shortcuts,
    this.actions,
    this.restorationScopeId,
    this.scrollBehavior,
    @Deprecated(
      'Remove this parameter as it is now ignored. '
      'MaterialApp never introduces its own MediaQuery; the View widget takes care of that. '
      'This feature was deprecated after v3.7.0-29.0.pre.',
    )
    this.useInheritedMediaQuery = false,
    this.themeAnimationStyle,
  }) : assert(routerDelegate != null || routerConfig != null),
       navigatorObservers = null,
       navigatorKey = null,
       onGenerateRoute = null,
       home = null,
       onGenerateInitialRoutes = null,
       onUnknownRoute = null,
       routes = null,
       initialRoute = null;
```

### 参数特点

`router` 构造函数与普通构造函数的主要区别：

1. **Router 相关参数**：包含 `routeInformationProvider`, `routeInformationParser`, `routerDelegate`, `routerConfig`, `backButtonDispatcher`
2. **缺少 Navigator 相关参数**：不包含 `navigatorKey`, `home`, `routes`, `onGenerateRoute` 等 Navigator 相关参数
3. **共享参数**：包含与普通构造函数相同的 Material 主题、本地化、调试等参数

### Router 构造函数的断言验证逻辑

#### 断言：必须提供 routerDelegate 或 routerConfig

```dart
assert(routerDelegate != null || routerConfig != null),
```

**目的**：必须提供 `routerDelegate` 或 `routerConfig` 之一，否则无法构建 Router。

### Navigator 相关属性的初始化

在 `router` 构造函数中，所有 Navigator 相关的属性都被初始化为 `null` 或默认值：

```dart
navigatorObservers = null,
navigatorKey = null,
onGenerateRoute = null,
home = null,
onGenerateInitialRoutes = null,
onUnknownRoute = null,
routes = null,
initialRoute = null;
```

这确保了 `router` 构造函数不会使用 Navigator 相关的功能。

## 两个构造函数的对比

### 相同点

1. **共享参数**：两个构造函数都包含 Material 主题、本地化、调试、交互等共享参数
2. **默认值**：许多参数在两个构造函数中都有相同的默认值
3. **Material 功能**：两个构造函数都支持完整的 Material Design 功能

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
MaterialApp(
  home: MyHomePage(),
  routes: {
    '/details': (context) => DetailsPage(),
  },
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  themeMode: ThemeMode.system,
)
```

### 使用 Router 构造函数

适用于：

- 需要声明式路由管理
- 需要深度链接支持
- 需要 Web URL 同步
- 复杂的路由需求

```dart
MaterialApp.router(
  routerConfig: routerConfig,
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  themeMode: ThemeMode.system,
)
```

## 构造函数文档注释

### 普通构造函数的要求

```dart 209:217:packages/flutter/lib/src/material/app.dart
  /// Creates a MaterialApp.
  ///
  /// At least one of [home], [routes], [onGenerateRoute], or [builder] must be
  /// non-null. If only [routes] is given, it must include an entry for the
  /// [Navigator.defaultRouteName] (`/`), since that is the route used when the
  /// application is launched with an intent that specifies an otherwise
  /// unsupported route.
  ///
  /// This class creates an instance of [WidgetsApp].
```

**要求**：

1. 至少提供以下之一：`home`、`routes`、`onGenerateRoute` 或 `builder`
2. 如果只提供了 `routes`，必须包含 `Navigator.defaultRouteName`（`/`）的条目
3. `MaterialApp` 会创建一个 `WidgetsApp` 实例

## 总结

第二部分详细介绍了 `MaterialApp` 的两个构造函数：

1. **普通构造函数**：用于基于 `Navigator` 的路由系统，包含完整的 Material Design 功能配置

2. **Router 构造函数**：用于基于 `Router` 的路由系统，包含断言验证确保 Router 配置的正确性

3. **参数验证**：`router` 构造函数包含断言验证，确保必须提供 `routerDelegate` 或 `routerConfig`

4. **互斥性**：两个构造函数确保 Navigator 和 Router 相关属性不会同时使用，避免配置冲突

5. **Material 功能**：两个构造函数都支持完整的 Material Design 功能，包括主题系统、Hero 动画等

这些验证逻辑确保了 `MaterialApp` 的正确使用，防止了常见的配置错误。
