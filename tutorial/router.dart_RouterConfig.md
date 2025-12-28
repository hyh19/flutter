# RouterConfig 类解析

## 概述

`RouterConfig<T>` 是一个用于配置 `Router` 组件的便捷配置类。它将配置 `Router` 所需的多个委托对象（delegates）打包成一个单一对象，简化了 `Router` 的配置过程。

## 设计目的

配置一个 `Router` 组件通常需要提供多个委托对象：

- `RouteInformationProvider`：提供路由信息
- `RouteInformationParser`：解析路由信息
- `RouterDelegate`：处理路由导航逻辑
- `BackButtonDispatcher`：处理返回按钮事件

`RouterConfig` 将这些组件打包在一起，提供了一个统一的配置接口。

## 类定义

```dart 135:161:packages/flutter/lib/src/widgets/router.dart
class RouterConfig<T> {
  /// Creates a [RouterConfig].
  ///
  /// The [backButtonDispatcher], [routeInformationProvider], and
  /// [routeInformationParser] are optional.
  ///
  /// The [routeInformationProvider] and [routeInformationParser] must both be
  /// provided or both not provided.
  const RouterConfig({
    this.routeInformationProvider,
    this.routeInformationParser,
    required this.routerDelegate,
    this.backButtonDispatcher,
  }) : assert((routeInformationProvider == null) == (routeInformationParser == null));

  /// The [RouteInformationProvider] that is used to configure the [Router].
  final RouteInformationProvider? routeInformationProvider;

  /// The [RouteInformationParser] that is used to configure the [Router].
  final RouteInformationParser<T>? routeInformationParser;

  /// The [RouterDelegate] that is used to configure the [Router].
  final RouterDelegate<T> routerDelegate;

  /// The [BackButtonDispatcher] that is used to configure the [Router].
  final BackButtonDispatcher? backButtonDispatcher;
}
```

## 构造函数

### 参数说明

构造函数接受以下参数：

- `routeInformationProvider`（可选）：用于提供路由信息的 `RouteInformationProvider` 实例
- `routeInformationParser`（可选）：用于解析路由信息的 `RouteInformationParser<T>` 实例
- `routerDelegate`（必需）：用于处理路由导航的 `RouterDelegate<T>` 实例
- `backButtonDispatcher`（可选）：用于处理返回按钮事件的 `BackButtonDispatcher` 实例

### 约束条件

构造函数中包含一个断言（assert），确保 `routeInformationProvider` 和 `routeInformationParser` 的提供状态一致：

```dart 148:148:packages/flutter/lib/src/widgets/router.dart
  }) : assert((routeInformationProvider == null) == (routeInformationParser == null));
```

这个断言的含义是：

- 如果 `routeInformationProvider` 为 `null`，则 `routeInformationParser` 也必须为 `null`
- 如果 `routeInformationProvider` 不为 `null`，则 `routeInformationParser` 也必须不为 `null`

换句话说，这两个参数必须**同时提供或同时不提供**，不能只提供其中一个。

## 属性说明

### routeInformationProvider

```dart 150:151:packages/flutter/lib/src/widgets/router.dart
  /// The [RouteInformationProvider] that is used to configure the [Router].
  final RouteInformationProvider? routeInformationProvider;
```

- **类型**：`RouteInformationProvider?`（可空）
- **作用**：提供路由信息给 `Router` 组件
- **使用场景**：在 Web 应用中，通常使用 `PlatformRouteInformationProvider` 来从浏览器 URL 获取路由信息

### routeInformationParser

```dart 153:154:packages/flutter/lib/src/widgets/router.dart
  /// The [RouteInformationParser] that is used to configure the [Router].
  final RouteInformationParser<T>? routeInformationParser;
```

- **类型**：`RouteInformationParser<T>?`（可空，泛型类型 `T`）
- **作用**：将路由信息（通常是 URL 字符串）解析为类型 `T` 的数据对象
- **泛型说明**：类型参数 `T` 表示解析后的路由数据类型，例如可以是自定义的路由配置类

### routerDelegate

```dart 156:157:packages/flutter/lib/src/widgets/router.dart
  /// The [RouterDelegate] that is used to configure the [Router].
  final RouterDelegate<T> routerDelegate;
```

- **类型**：`RouterDelegate<T>`（必需，泛型类型 `T`）
- **作用**：核心的路由委托，负责根据解析后的路由数据（类型 `T`）构建导航页面
- **必需性**：这是唯一一个必需的参数，因为它是路由系统的核心组件

### backButtonDispatcher

```dart 159:160:packages/flutter/lib/src/widgets/router.dart
  /// The [BackButtonDispatcher] that is used to configure the [Router].
  final BackButtonDispatcher? backButtonDispatcher;
```

- **类型**：`BackButtonDispatcher?`（可空）
- **作用**：处理系统返回按钮事件
- **使用场景**：
  - 根路由通常使用 `RootBackButtonDispatcher`
  - 嵌套路由通常使用 `ChildBackButtonDispatcher`

## 使用示例

### 基本用法

```dart
// 创建一个简单的 RouterConfig
final routerConfig = RouterConfig<String>(
  routerDelegate: MyRouterDelegate(),
);

// 在 Router 中使用
Router(
  routerConfig: routerConfig,
)
```

### 完整配置（包含所有组件）

```dart
final routerConfig = RouterConfig<MyRouteData>(
  routeInformationProvider: PlatformRouteInformationProvider(
    initialRouteInformation: RouteInformation(
      uri: Uri.parse('/home'),
    ),
  ),
  routeInformationParser: MyRouteInformationParser(),
  routerDelegate: MyRouterDelegate(),
  backButtonDispatcher: RootBackButtonDispatcher(),
);
```

### 错误示例（违反约束）

以下代码会导致断言失败，因为只提供了 `routeInformationProvider` 而没有提供 `routeInformationParser`：

```dart
// ❌ 错误：会导致断言失败
final routerConfig = RouterConfig<String>(
  routeInformationProvider: PlatformRouteInformationProvider(),
  // 缺少 routeInformationParser
  routerDelegate: MyRouterDelegate(),
);
```

## 设计模式

`RouterConfig` 采用了**配置对象模式**（Configuration Object Pattern），将多个相关配置参数封装在一个对象中。这种设计有以下优点：

1. **简化 API**：`Router` 组件只需要一个 `routerConfig` 参数，而不是多个独立的参数
2. **类型安全**：通过泛型 `T` 确保 `routeInformationParser` 和 `routerDelegate` 使用相同的类型
3. **约束验证**：在构造函数中通过断言确保配置的有效性
4. **可扩展性**：未来如果需要添加新的配置项，只需在 `RouterConfig` 中添加属性即可

## 与 Router 的关系

`RouterConfig` 是专门为 `Router` 组件设计的配置类。`Router` 组件使用 `RouterConfig` 中提供的各个委托对象来完成路由功能：

1. `RouteInformationProvider` 提供路由信息（如 URL）
2. `RouteInformationParser` 将路由信息解析为类型 `T` 的数据
3. `RouterDelegate` 根据解析后的数据构建页面
4. `BackButtonDispatcher` 处理返回按钮事件

## 总结

`RouterConfig<T>` 是一个配置类，用于简化 `Router` 组件的配置过程。它通过将多个委托对象打包在一起，提供了更清晰的 API。关键要点包括：

- `routerDelegate` 是唯一必需的参数
- `routeInformationProvider` 和 `routeInformationParser` 必须同时提供或同时不提供
- 通过泛型 `T` 确保类型安全
- 使用断言在运行时验证配置的有效性
