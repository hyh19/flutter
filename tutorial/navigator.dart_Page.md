# `Page<T>` 类详解

## 概述

`Page<T>` 是 Flutter 中用于描述路由（Route）配置的抽象类。它继承自 `RouteSettings`，提供了更丰富的路由配置能力，特别是页面弹出拦截和状态恢复功能。

## 类定义

```dart 682:682:packages/flutter/lib/src/widgets/navigator.dart
abstract class Page<T> extends RouteSettings {
```

`Page<T>` 是一个泛型抽象类，其中类型参数 `T` 代表对应 `Route` 的返回类型，用于 `Route.currentResult`、`Route.popped` 和 `Route.didPop` 等方法中。

## 继承关系

`Page<T>` 继承自 `RouteSettings`，因此它包含了路由的基本配置信息：

```dart 643:662:packages/flutter/lib/src/widgets/navigator.dart
/// Data that might be useful in constructing a [Route].
@immutable
class RouteSettings {
  /// Creates data used to construct routes.
  const RouteSettings({this.name, this.arguments});

  /// The name of the route (e.g., "/settings").
  ///
  /// If null, the route is anonymous.
  final String? name;

  /// The arguments passed to this route.
  ///
  /// May be used when building the route, e.g. in [Navigator.onGenerateRoute].
  final Object? arguments;

  @override
  String toString() =>
      '${objectRuntimeType(this, 'RouteSettings')}(${name == null ? 'none' : '"$name"'}, $arguments)';
}
```

从 `RouteSettings` 继承的属性包括：

- `name`：路由名称（如 "/settings"），如果为 null 则表示匿名路由
- `arguments`：传递给该路由的参数，可以在构建路由时使用（如在 `Navigator.onGenerateRoute` 中）

## 构造函数

```dart 683:691:packages/flutter/lib/src/widgets/navigator.dart
  /// Creates a page and initializes [key] for subclasses.
  const Page({
    this.key,
    super.name,
    super.arguments,
    this.restorationId,
    this.canPop = true,
    this.onPopInvoked = _defaultPopInvokedHandler,
  });
```

构造函数接受以下参数：

- `key`：与该页面关联的键，用于页面比较
- `super.name`：从父类 `RouteSettings` 继承的路由名称
- `super.arguments`：从父类 `RouteSettings` 继承的路由参数
- `restorationId`：用于状态恢复的 ID
- `canPop`：默认为 `true`，控制是否可以弹出该路由
- `onPopInvoked`：默认使用 `_defaultPopInvokedHandler`，在弹出操作被处理后调用

默认的弹出处理程序是一个空函数：

```dart 693:693:packages/flutter/lib/src/widgets/navigator.dart
  static void _defaultPopInvokedHandler(bool didPop, Object? result) {}
```

## 主要属性

### key

```dart 695:698:packages/flutter/lib/src/widgets/navigator.dart
  /// The key associated with this page.
  ///
  /// This key will be used for comparing pages in [canUpdate].
  final LocalKey? key;
```

与该页面关联的键，用于在 `canUpdate` 方法中比较页面。如果两个页面的 `runtimeType` 和 `key` 都相同，则被认为是可更新的。

### restorationId

```dart 700:709:packages/flutter/lib/src/widgets/navigator.dart
  /// Restoration ID to save and restore the state of the [Route] configured by
  /// this page.
  ///
  /// If no restoration ID is provided, the [Route] will not restore its state.
  ///
  /// See also:
  ///
  ///  * [RestorationManager], which explains how state restoration works in
  ///    Flutter.
  final String? restorationId;
```

用于保存和恢复由该页面配置的路由状态的恢复 ID。如果没有提供恢复 ID，路由将不会恢复其状态。关于状态恢复的详细说明，请参考 `RestorationManager`。

### onPopInvoked

```dart 711:721:packages/flutter/lib/src/widgets/navigator.dart
  /// Called after a pop on the associated route was handled.
  ///
  /// It's not possible to prevent the pop from happening at the time that this
  /// method is called; the pop has already happened. Use [canPop] to
  /// disable pops in advance.
  ///
  /// This will still be called even when the pop is canceled. A pop is canceled
  /// when the associated [Route.popDisposition] returns false, or when
  /// [canPop] is set to false. The `didPop` parameter indicates whether or not
  /// the back navigation actually happened successfully.
  final PopInvokedWithResultCallback<T> onPopInvoked;
```

当关联路由的弹出操作被处理后会被调用。**重要**：在这个方法被调用时，已经无法阻止弹出操作的发生，因为弹出已经发生了。如果需要禁用弹出，请使用 `canPop` 属性。

即使弹出被取消，这个回调仍然会被调用。弹出被取消的情况包括：

- 关联的 `Route.popDisposition` 返回 false
- `canPop` 被设置为 false

`didPop` 参数指示返回导航是否实际成功发生。

`PopInvokedWithResultCallback<T>` 的类型定义如下：

```dart 2781:2781:packages/flutter/lib/src/widgets/routes.dart
typedef PopInvokedWithResultCallback<T> = void Function(bool didPop, T? result);
```

这是一个接受两个参数的函数：

- `bool didPop`：指示是否成功弹出
- `T? result`：弹出时返回的结果

### canPop

```dart 723:731:packages/flutter/lib/src/widgets/navigator.dart
  /// When false, blocks the associated route from being popped.
  ///
  /// If this is set to false for first page in the Navigator. It prevents
  /// Flutter app from exiting.
  ///
  /// If there are any [PopScope] widgets in a route's widget subtree,
  /// each of their `canPop` must be `true`, in addition to this canPop, in
  /// order for the route to be able to pop.
  final bool canPop;
```

当设置为 `false` 时，会阻止关联的路由被弹出。这个属性提供了一种在弹出发生之前就阻止弹出的机制。

**重要使用场景**：

1. **防止应用退出**：如果 Navigator 的第一个页面（首页）的 `canPop` 设置为 `false`，可以防止 Flutter 应用退出。

2. **与 PopScope 协同工作**：如果路由的 widget 子树中有任何 `PopScope` widget，除了这个 `canPop` 必须为 `true` 外，每个 `PopScope` 的 `canPop` 也必须为 `true`，路由才能被弹出。

## 核心方法

### canUpdate

```dart 733:739:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether this page can be updated with the [other] page.
  ///
  /// Two pages are consider updatable if they have same the [runtimeType] and
  /// [key].
  bool canUpdate(Page<dynamic> other) {
    return other.runtimeType == runtimeType && other.key == key;
  }
```

判断该页面是否可以用 `other` 页面更新。如果两个页面的 `runtimeType` 和 `key` 都相同，则认为它们是可更新的。这个方法在 Navigator 更新页面列表时用于判断是否应该更新现有路由，还是创建新的路由。

### createRoute

```dart 741:745:packages/flutter/lib/src/widgets/navigator.dart
  /// Creates the [Route] that corresponds to this page.
  ///
  /// The created [Route] must have its [Route.settings] property set to this [Page].
  @factory
  Route<T> createRoute(BuildContext context);
```

创建与该页面对应的 `Route`。这是一个抽象方法，子类必须实现。创建的 `Route` 必须将其 `Route.settings` 属性设置为该 `Page`。

`@factory` 注解表示这是一个工厂方法，用于创建路由实例。

### toString

```dart 747:748:packages/flutter/lib/src/widgets/navigator.dart
  @override
  String toString() => '${objectRuntimeType(this, 'Page')}("$name", $key, $arguments)';
```

返回页面的字符串表示，包括运行时类型、名称、键和参数。

## 使用场景

### 拦截返回操作

`Page<T>` 提供了两种拦截返回操作的方式：

1. **`canPop`**：在弹出发生之前阻止弹出
2. **`onPopInvoked`**：在弹出被处理后执行回调（无法阻止弹出，但可以执行清理操作）

```dart
Page(
  canPop: false,  // 阻止弹出
  onPopInvoked: (didPop, result) {
    // 弹出操作已处理，执行清理工作
    if (didPop) {
      print('页面已成功弹出，返回结果：$result');
    } else {
      print('弹出被取消');
    }
  },
)
```

### 与 Navigator.pages 配合使用

`Page<T>` 主要用于 `Navigator.pages` API，这是 Flutter 的声明式导航方式：

```dart
Navigator(
  pages: [
    MaterialPage(key: ValueKey('home'), child: HomePage()),
    MaterialPage(key: ValueKey('detail'), child: DetailPage()),
  ],
  onPopPage: (route, result) {
    if (!route.didPop(result)) {
      return false;
    }
    // 更新页面列表
    return true;
  },
)
```

### 状态恢复

通过 `restorationId` 可以实现路由状态的保存和恢复，这对于应用在后台被系统杀死后恢复状态非常有用。

## 相关类型

- `RouteSettings`：`Page<T>` 的父类，提供基本的路由配置信息
- `Route<T>`：由 `Page<T>` 创建的实际路由对象
- `Navigator`：管理路由堆栈的 widget
- `PopScope`：可以在 widget 子树层面拦截返回操作的 widget
- `RestorationManager`：管理状态恢复的系统

## 示例代码参考

根据文档注释，可以参考以下示例代码：

- `examples/api/lib/widgets/page/page_can_pop.0.dart`：演示如何使用 `canPop` 和 `onPopInvoked` 拦截弹出操作

## 总结

`Page<T>` 是 Flutter 声明式导航的核心组件，它：

1. **扩展了路由配置**：在 `RouteSettings` 基础上增加了更多功能
2. **提供了弹出拦截机制**：通过 `canPop` 和 `onPopInvoked` 控制返回行为
3. **支持状态恢复**：通过 `restorationId` 实现状态持久化
4. **支持页面更新**：通过 `canUpdate` 判断页面是否可更新，优化路由管理
5. **定义了路由创建接口**：通过 `createRoute` 方法由子类实现具体的路由创建逻辑

在使用 `Navigator.pages` 进行声明式导航时，`Page<T>` 是不可或缺的基础类。
