# WidgetsApp Router 相关属性详解

## 概述

`WidgetsApp` 提供了 Router 相关属性，用于配置基于 `Router` 的声明式路由系统。这些属性与 Navigator 相关属性互斥，用于实现更现代、更灵活的路由管理方式。

## routeInformationParser

```dart 568:582:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.routeInformationParser}
  /// A delegate to parse the route information from the
  /// [routeInformationProvider] into a generic data type to be processed by
  /// the [routerDelegate] at a later stage.
  ///
  /// This object will be used by the underlying [Router].
  ///
  /// The generic type `T` must match the generic type of the [routerDelegate].
  ///
  /// See also:
  ///
  ///  * [Router.routeInformationParser], which receives this object when this
  ///    widget builds the [Router].
  /// {@endtemplate}
  final RouteInformationParser<Object>? routeInformationParser;
```

**功能**：将来自 `routeInformationProvider` 的路由信息解析为泛型数据类型的委托，供 `routerDelegate` 后续处理。

**作用**：

1. **路由信息解析**：将平台提供的路由信息（如 URL）解析为应用可以理解的数据类型
2. **类型转换**：将字符串形式的路径转换为应用的路由配置对象

**泛型要求**：泛型类型 `T` 必须与 `routerDelegate` 的泛型类型匹配。

**使用场景**：通常与 `go_router` 等路由库一起使用，将 URL 路径解析为路由配置。

## routerDelegate

```dart 584:598:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.routerDelegate}
  /// A delegate that configures a widget, typically a [Navigator], with
  /// parsed result from the [routeInformationParser].
  ///
  /// This object will be used by the underlying [Router].
  ///
  /// The generic type `T` must match the generic type of the
  /// [routeInformationParser].
  ///
  /// See also:
  ///
  ///  * [Router.routerDelegate], which receives this object when this widget
  ///    builds the [Router].
  /// {@endtemplate}
  final RouterDelegate<Object>? routerDelegate;
```

**功能**：使用来自 `routeInformationParser` 的解析结果配置 widget（通常是 `Navigator`）的委托。

**作用**：

1. **路由配置**：根据解析后的路由信息配置 Navigator 或其他 widget
2. **状态管理**：管理路由状态，决定显示哪些页面

**泛型要求**：泛型类型 `T` 必须与 `routeInformationParser` 的泛型类型匹配。

**典型实现**：通常实现为 `Navigator` 的包装，根据路由配置构建相应的页面。

## backButtonDispatcher

```dart 600:613:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.backButtonDispatcher}
  /// A delegate that decide whether to handle the Android back button intent.
  ///
  /// This object will be used by the underlying [Router].
  ///
  /// If this is not provided, the widgets app will create a
  /// [RootBackButtonDispatcher] by default.
  ///
  /// See also:
  ///
  ///  * [Router.backButtonDispatcher], which receives this object when this
  ///    widget builds the [Router].
  /// {@endtemplate}
  final BackButtonDispatcher? backButtonDispatcher;
```

**功能**：决定是否处理 Android 返回按钮意图的委托。

**默认行为**：如果未提供，widgets app 将默认创建 `RootBackButtonDispatcher`。

**用途**：

1. **返回按钮处理**：控制 Android 返回按钮的行为
2. **自定义返回逻辑**：可以实现自定义的返回按钮处理逻辑

**使用场景**：当需要自定义返回按钮行为时，例如在特定页面阻止返回或执行额外操作。

## routeInformationProvider

```dart 615:631:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.routeInformationProvider}
  /// A object that provides route information through the
  /// [RouteInformationProvider.value] and notifies its listener when its value
  /// changes.
  ///
  /// This object will be used by the underlying [Router].
  ///
  /// If this is not provided, the widgets app will create a
  /// [PlatformRouteInformationProvider] with initial route name equal to the
  /// [dart:ui.PlatformDispatcher.defaultRouteName] by default.
  ///
  /// See also:
  ///
  ///  * [Router.routeInformationProvider], which receives this object when this
  ///    widget builds the [Router].
  /// {@endtemplate}
  final RouteInformationProvider? routeInformationProvider;
```

**功能**：通过 `RouteInformationProvider.value` 提供路由信息，并在值改变时通知其监听器的对象。

**默认行为**：如果未提供，widgets app 将默认创建 `PlatformRouteInformationProvider`，初始路由名称等于 `PlatformDispatcher.defaultRouteName`。

**作用**：

1. **路由信息提供**：提供当前的路由信息（如 URL）
2. **路由变化通知**：当路由信息改变时通知监听器

**使用场景**：通常用于 Web 应用，同步浏览器的 URL 与应用的路由状态。

## routerConfig

```dart 633:645:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.routerConfig}
  /// An object to configure the underlying [Router].
  ///
  /// If the [routerConfig] is provided, the other router related delegates,
  /// [routeInformationParser], [routeInformationProvider], [routerDelegate],
  /// and [backButtonDispatcher], must all be null.
  ///
  /// See also:
  ///
  ///  * [Router.withConfig], which receives this object when this
  ///    widget builds the [Router].
  /// {@endtemplate}
  final RouterConfig<Object>? routerConfig;
```

**功能**：配置底层 `Router` 的对象。

**互斥性**：如果提供了 `routerConfig`，其他 Router 相关的委托（`routeInformationParser`、`routeInformationProvider`、`routerDelegate`、`backButtonDispatcher`）必须都为 `null`。

**设计目的**：`routerConfig` 是一个封装了所有 Router 配置的对象，提供了更简洁的配置方式。

**优势**：

1. **简化配置**：将多个独立的委托封装为一个配置对象
2. **类型安全**：确保配置的一致性
3. **易于使用**：对于使用路由库（如 `go_router`）的应用，这是推荐的配置方式

## Router 属性的工作流程

### 基本流程

```text
平台路由信息（URL）
    ↓
routeInformationProvider (提供路由信息)
    ↓
routeInformationParser (解析为应用数据类型)
    ↓
routerDelegate (配置 Navigator/Widget)
    ↓
显示对应页面
```

### 返回按钮处理

```text
用户按下返回按钮
    ↓
backButtonDispatcher (决定是否处理)
    ↓
如果处理 → routerDelegate (执行返回操作)
```

## Router 与 Navigator 的对比

### Router 的优势

1. **声明式**：使用声明式的方式管理路由
2. **URL 同步**：自动同步 Web URL 与路由状态
3. **深度链接**：更好的深度链接支持
4. **状态管理**：更灵活的路由状态管理

### Navigator 的优势

1. **简单直接**：API 简单直观
2. **传统方式**：Flutter 的传统路由方式
3. **轻量级**：不需要额外的配置

## 使用示例

### 使用独立的 Router 属性

```dart
WidgetsApp.router(
  routeInformationParser: MyRouteInformationParser(),
  routerDelegate: MyRouterDelegate(),
  routeInformationProvider: PlatformRouteInformationProvider(
    initialRouteInformation: RouteInformation(uri: Uri.parse('/')),
  ),
  backButtonDispatcher: RootBackButtonDispatcher(),
  color: Colors.blue,
)
```

### 使用 routerConfig（推荐）

```dart
final routerConfig = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomePage(),
    ),
    GoRoute(
      path: '/details/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return DetailsPage(id: id);
      },
    ),
  ],
);

WidgetsApp.router(
  routerConfig: routerConfig,
  color: Colors.blue,
)
```

## 属性之间的关系

### 互斥关系

- `routerConfig` 与 `routeInformationParser`、`routeInformationProvider`、`routerDelegate`、`backButtonDispatcher` 互斥
- 如果使用 `routerConfig`，其他 Router 属性必须为 `null`

### 依赖关系

- 如果提供了 `routeInformationProvider`，必须同时提供 `routeInformationParser`
- `routeInformationParser` 和 `routerDelegate` 的泛型类型必须匹配

### 默认值

- 如果未提供 `backButtonDispatcher`，默认创建 `RootBackButtonDispatcher`
- 如果未提供 `routeInformationProvider`，默认创建 `PlatformRouteInformationProvider`

## 总结

第四部分详细介绍了 `WidgetsApp` 的所有 Router 相关属性：

1. **routeInformationParser**：解析路由信息为应用数据类型的委托
2. **routerDelegate**：配置 widget（通常是 Navigator）的委托
3. **backButtonDispatcher**：处理 Android 返回按钮的委托
4. **routeInformationProvider**：提供路由信息的对象
5. **routerConfig**：封装所有 Router 配置的对象（推荐使用）

这些属性共同构成了完整的 Router 路由系统配置，提供了比 Navigator 更现代、更灵活的路由管理方式。使用 `routerConfig` 是推荐的配置方式，特别是与 `go_router` 等路由库一起使用时。
