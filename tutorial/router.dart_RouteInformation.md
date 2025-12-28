# RouteInformation 类详解

## 概述

`RouteInformation` 类表示应用程序的一条路由信息。它是 Flutter 路由系统中用于在不同组件之间传递路由信息的核心数据结构。

```dart 27:49:packages/flutter/lib/src/widgets/router.dart
/// A piece of routing information.
///
/// The route information consists of a location string of the application and
/// a state object that configures the application in that location.
///
/// This information flows two ways, from the [RouteInformationProvider] to the
/// [Router] or from the [Router] to [RouteInformationProvider].
///
/// In the former case, the [RouteInformationProvider] notifies the [Router]
/// widget when a new [RouteInformation] is available. The [Router] widget takes
/// these information and navigates accordingly.
///
/// The latter case happens in web application where the [Router] reports route
/// changes back to the web engine.
///
/// The current [RouteInformation] of an application is also used for state
/// restoration purposes. Before an application is killed, the [Router] converts
/// its current configurations into a [RouteInformation] object utilizing the
/// [RouteInformationProvider]. The [RouteInformation] object is then serialized
/// out and persisted. During state restoration, the object is deserialized and
/// passed back to the [RouteInformationProvider], which turns it into a
/// configuration for the [Router] again to restore its state from.
class RouteInformation {
```

路由信息由两部分组成：

1. **位置信息**（location/uri）：应用程序的位置字符串
2. **状态对象**（state）：在该位置配置应用程序的状态对象

## 数据流向

`RouteInformation` 在路由系统中的数据流向是双向的：

### 从 RouteInformationProvider 到 Router

当 `RouteInformationProvider` 有新的 `RouteInformation` 可用时，它会通知 `Router` widget。`Router` widget 接收这些信息并进行相应的导航。

常见场景：

- 用户点击浏览器后退/前进按钮
- 应用程序接收到深链接（deep link）
- 平台路由信息发生变化

### 从 Router 到 RouteInformationProvider

在 Web 应用程序中，`Router` 会将路由变化报告回 Web 引擎。这确保了浏览器地址栏与应用程序的当前路由保持同步。

## 状态恢复机制

`RouteInformation` 还用于状态恢复目的：

1. **保存状态**：应用程序被终止前，`Router` 会利用 `RouteInformationProvider` 将当前配置转换为 `RouteInformation` 对象
2. **序列化**：`RouteInformation` 对象被序列化并持久化存储
3. **恢复状态**：状态恢复时，对象被反序列化并传回 `RouteInformationProvider`，`RouteInformationProvider` 将其转换为 `Router` 的配置以恢复之前的状态

## 构造函数

```dart 50:63:packages/flutter/lib/src/widgets/router.dart
  /// Creates a route information object.
  ///
  /// Either `location` or `uri` must not be null.
  const RouteInformation({
    @Deprecated(
      'Pass Uri.parse(location) to uri parameter instead. '
      'This feature was deprecated after v3.8.0-3.0.pre.',
    )
    String? location,
    Uri? uri,
    this.state,
  }) : _location = location,
       _uri = uri,
       assert((location != null) != (uri != null));
```

构造函数的参数说明：

- `location`（已废弃）：字符串格式的位置信息，建议使用 `Uri.parse(location)` 传递给 `uri` 参数。该特性在 v3.8.0-3.0.pre 之后被废弃
- `uri`：`Uri` 对象格式的位置信息
- `state`：应用程序在该位置的状态对象，可以是任何可序列化的对象

**重要约束**：`location` 和 `uri` 必须有一个不为 null（使用异或断言确保这一点）。

## 属性详解

### location 属性（已废弃）

```dart 65:82:packages/flutter/lib/src/widgets/router.dart
  /// The location of the application.
  ///
  /// The string is usually in the format of multiple string identifiers with
  /// slashes in between. ex: `/`, `/path`, `/path/to/the/app`.
  @Deprecated(
    'Use uri instead. '
    'This feature was deprecated after v3.8.0-3.0.pre.',
  )
  String get location {
    return _location ??
        Uri.decodeComponent(
          Uri(
            path: uri.path.isEmpty ? '/' : uri.path,
            queryParameters: uri.queryParametersAll.isEmpty ? null : uri.queryParametersAll,
            fragment: uri.fragment.isEmpty ? null : uri.fragment,
          ).toString(),
        );
  }

  final String? _location;
```

`location` 属性返回字符串格式的应用程序位置。

- **格式**：通常是多个字符串标识符，中间用斜杠分隔，例如：`/`、`/path`、`/path/to/the/app`
- **状态**：已废弃，建议使用 `uri` 属性代替
- **实现逻辑**：
  - 如果 `_location` 不为 null，直接返回 `_location`
  - 否则，从 `uri` 属性构造一个位置字符串，包括路径、查询参数和片段（fragment）

### uri 属性

```dart 86:100:packages/flutter/lib/src/widgets/router.dart
  /// The uri location of the application.
  ///
  /// The host and scheme will not be empty if this object is created from a
  /// deep link request. They represents the website that redirect the deep
  /// link.
  ///
  /// In web platform, the host and scheme are always empty.
  Uri get uri {
    if (_uri != null) {
      return _uri;
    }
    return Uri.parse(_location!);
  }

  final Uri? _uri;
```

`uri` 属性返回 `Uri` 对象格式的应用程序位置，这是推荐使用的属性。

**特性说明**：

- **深链接场景**：如果对象是从深链接请求创建的，`host` 和 `scheme` 将不为空，它们代表重定向深链接的网站
- **Web 平台**：在 Web 平台上，`host` 和 `scheme` 始终为空
- **实现逻辑**：
  - 如果 `_uri` 不为 null，直接返回 `_uri`
  - 否则，使用 `Uri.parse(_location!)` 解析 `_location` 字符串

### state 属性

```dart 102:120:packages/flutter/lib/src/widgets/router.dart
  /// The state of the application in the [uri].
  ///
  /// The app can have different states even in the same location. For example,
  /// the text inside a [TextField] or the scroll position in a [ScrollView].
  /// These widget states can be stored in the [state].
  ///
  /// On the web, this information is stored in the browser history when the
  /// [Router] reports this route information back to the web engine
  /// through the [PlatformRouteInformationProvider]. The information
  /// is then passed back, along with the [uri], when the user
  /// clicks the back or forward buttons.
  ///
  /// This information is also serialized and persisted alongside the
  /// [uri] for state restoration purposes. During state restoration,
  /// the information is made available again to the [Router] so it can restore
  /// its configuration to the previous state.
  ///
  /// The state must be serializable.
  final Object? state;
```

`state` 属性存储应用程序在特定 `uri` 位置的状态对象。

**使用场景**：

1. **相同位置的不同状态**：应用程序即使在同一位置也可以有不同的状态，例如：
   - `TextField` 中的文本内容
   - `ScrollView` 中的滚动位置
   - 其他 widget 的状态

2. **Web 浏览器历史**：在 Web 平台上，当 `Router` 通过 `PlatformRouteInformationProvider` 将路由信息报告回 Web 引擎时，这些信息会存储在浏览器历史记录中。当用户点击后退或前进按钮时，信息会连同 `uri` 一起传回。

3. **状态恢复**：这些信息会与 `uri` 一起序列化并持久化，用于状态恢复。在状态恢复期间，信息会再次提供给 `Router`，以便它可以恢复到之前的状态配置。

**重要约束**：`state` 必须是可序列化的对象。

## 使用示例

### 基本用法

```dart
// 创建 RouteInformation 对象（推荐方式）
final routeInfo = RouteInformation(
  uri: Uri.parse('/users/123'),
  state: {'userId': 123, 'scrollPosition': 0.0},
);

// 访问 uri
print(routeInfo.uri.path); // 输出: /users/123

// 访问 state
print(routeInfo.state); // 输出: {userId: 123, scrollPosition: 0.0}
```

### 带查询参数的 URI

```dart
final routeInfo = RouteInformation(
  uri: Uri.parse('/search?q=flutter&page=1'),
  state: {'searchTerm': 'flutter'},
);

print(routeInfo.uri.queryParameters); // 输出: {q: flutter, page: 1}
```

### 状态恢复示例

```dart
// 在状态恢复时使用
final restoredInfo = RouteInformation(
  uri: Uri.parse('/dashboard'),
  state: {
    'selectedTab': 2,
    'scrollOffset': 150.0,
  },
);
```

## 迁移指南

由于 `location` 参数已被废弃，建议进行以下迁移：

**旧代码**：

```dart
final info = RouteInformation(
  location: '/path/to/page',
  state: someState,
);
```

**新代码**：

```dart
final info = RouteInformation(
  uri: Uri.parse('/path/to/page'),
  state: someState,
);
```

## 总结

`RouteInformation` 类是 Flutter 路由系统的核心数据载体，它：

1. **封装路由信息**：包含位置（uri）和状态（state）两部分
2. **支持双向数据流**：在 `RouteInformationProvider` 和 `Router` 之间传递信息
3. **支持状态恢复**：可以序列化保存，用于应用程序状态恢复
4. **适配多平台**：在 Web 平台与浏览器历史记录集成
5. **现代化设计**：推荐使用 `Uri` 而非字符串 `location`

在使用时，应优先使用 `uri` 属性而不是已废弃的 `location` 属性，以确保代码的兼容性和未来的可维护性。
