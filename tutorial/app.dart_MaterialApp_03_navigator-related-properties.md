# MaterialApp Navigator 相关属性详解

## 概述

`MaterialApp` 提供了丰富的 Navigator 相关属性，这些属性继承自 `WidgetsApp`，但 `MaterialApp` 对它们进行了特殊处理。最重要的是，`MaterialApp` 使用 `MaterialPageRoute` 作为默认的 `pageRouteBuilder`，并配置了 Material 风格的 Hero 动画。

## navigatorKey

```dart 324:325:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.navigatorKey}
  final GlobalKey<NavigatorState>? navigatorKey;
```

**功能**：用于构建 `Navigator` 的全局键，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：`MaterialApp` 直接传递此属性给底层的 `WidgetsApp`，没有特殊处理。

## scaffoldMessengerKey

```dart 327:333:packages/flutter/lib/src/material/app.dart
  /// A key to use when building the [ScaffoldMessenger].
  ///
  /// If a [scaffoldMessengerKey] is specified, the [ScaffoldMessenger] can be
  /// directly manipulated without first obtaining it from a [BuildContext] via
  /// [ScaffoldMessenger.of]: from the [scaffoldMessengerKey], use the
  /// [GlobalKey.currentState] getter.
  final GlobalKey<ScaffoldMessengerState>? scaffoldMessengerKey;
```

**功能**：用于构建 `ScaffoldMessenger` 的全局键。

**用途**：

1. **直接访问 ScaffoldMessenger**：如果指定了 `scaffoldMessengerKey`，可以直接通过 `scaffoldMessengerKey.currentState` 访问 `ScaffoldMessengerState`，而不需要通过 `ScaffoldMessenger.of(context)` 获取
2. **全局消息控制**：允许在应用的任何地方（包括不在 widget 树中的代码）控制 SnackBar 等 Material 消息

**MaterialApp 特有**：这是 `MaterialApp` 特有的属性，`WidgetsApp` 不包含此属性。

## home

```dart 335:336:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.home}
  final Widget? home;
```

**功能**：应用的默认路由（`Navigator.defaultRouteName`，即 `/`）的 widget，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：`MaterialApp` 直接传递此属性给底层的 `WidgetsApp`，没有特殊处理。

## routes

```dart 338:347:packages/flutter/lib/src/material/app.dart
  /// The application's top-level routing table.
  ///
  /// When a named route is pushed with [Navigator.pushNamed], the route name is
  /// looked up in this map. If the name is present, the associated
  /// [WidgetBuilder] is used to construct a [MaterialPageRoute] that
  /// performs an appropriate transition, including [Hero] animations, to the
  /// new route.
  ///
  /// {@macro flutter.widgets.widgetsApp.routes}
  final Map<String, WidgetBuilder>? routes;
```

**功能**：应用的顶级路由表，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：

- 当使用 `Navigator.pushNamed` 推送命名路由时，`MaterialApp` 使用 `MaterialPageRoute` 构建路由（而不是 `WidgetsApp` 中需要手动指定的 `pageRouteBuilder`）
- `MaterialPageRoute` 执行适当的 Material Design 过渡动画，包括 Hero 动画

## initialRoute

```dart 349:350:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.initialRoute}
  final String? initialRoute;
```

**功能**：如果构建了 `Navigator`，要显示的第一个路由的名称，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：`MaterialApp` 直接传递此属性给底层的 `WidgetsApp`，没有特殊处理。

## onGenerateRoute

```dart 352:353:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.onGenerateRoute}
  final RouteFactory? onGenerateRoute;
```

**功能**：当应用导航到命名路由时使用的路由生成器回调，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：`MaterialApp` 直接传递此属性给底层的 `WidgetsApp`，没有特殊处理。

## onGenerateInitialRoutes

```dart 355:356:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.onGenerateInitialRoutes}
  final InitialRouteListFactory? onGenerateInitialRoutes;
```

**功能**：如果提供了 `initialRoute`，用于生成初始路由的路由生成器回调，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：`MaterialApp` 直接传递此属性给底层的 `WidgetsApp`，没有特殊处理。

## onUnknownRoute

```dart 358:359:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.onUnknownRoute}
  final RouteFactory? onUnknownRoute;
```

**功能**：当 `onGenerateRoute` 无法生成路由时调用，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：`MaterialApp` 直接传递此属性给底层的 `WidgetsApp`，没有特殊处理。

## onNavigationNotification

```dart 361:362:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.onNavigationNotification}
  final NotificationListenerCallback<NavigationNotification>? onNavigationNotification;
```

**功能**：接收 `NavigationNotification` 时使用的回调，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：`MaterialApp` 直接传递此属性给底层的 `WidgetsApp`，没有特殊处理。

## navigatorObservers

```dart 364:365:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.navigatorObservers}
  final List<NavigatorObserver>? navigatorObservers;
```

**功能**：为此应用创建的 `Navigator` 的观察者列表，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：

- `MaterialApp` 会自动添加一个 `HeroController` 到 `navigatorObservers` 列表中（通过 `HeroControllerScope` 实现）
- 这个 `HeroController` 使用 `MaterialRectArcTween` 创建 Material 风格的 Hero 动画效果

## MaterialApp 的默认 pageRouteBuilder

虽然 `MaterialApp` 没有直接暴露 `pageRouteBuilder` 属性，但在构建 `WidgetsApp` 时，它会自动使用 `MaterialPageRoute` 作为默认的 `pageRouteBuilder`：

```dart 1105:1107:packages/flutter/lib/src/material/app.dart
      pageRouteBuilder: <T>(RouteSettings settings, WidgetBuilder builder) {
        return MaterialPageRoute<T>(settings: settings, builder: builder);
      },
```

**功能**：当应用导航到命名路由时使用的 `PageRoute` 生成器回调。

**特点**：

1. **Material 风格过渡**：`MaterialPageRoute` 提供 Material Design 风格的页面过渡动画
2. **Hero 动画支持**：`MaterialPageRoute` 支持 Hero 动画，与 `MaterialApp` 的 `HeroController` 配合工作
3. **自动配置**：开发者不需要手动指定 `pageRouteBuilder`，`MaterialApp` 会自动使用 `MaterialPageRoute`

## Hero 动画配置

`MaterialApp` 通过 `HeroControllerScope` 和 `HeroController` 配置 Hero 动画：

```dart 1165:1165:packages/flutter/lib/src/material/app.dart
      child: HeroControllerScope(controller: _heroController, child: result),
```

`_heroController` 是通过 `MaterialApp.createMaterialHeroController()` 创建的：

```dart 799:805:packages/flutter/lib/src/material/app.dart
  static HeroController createMaterialHeroController() {
    return HeroController(
      createRectTween: (Rect? begin, Rect? end) {
        return MaterialRectArcTween(begin: begin, end: end);
      },
    );
  }
```

**特点**：

- **MaterialRectArcTween**：使用 Material Design 风格的弧形过渡动画
- **自动配置**：`MaterialApp` 自动创建和配置 `HeroController`，开发者无需手动配置

## 属性之间的关系

### 路由查找顺序

`MaterialApp` 的路由查找顺序与 `WidgetsApp` 相同：

```text
Navigator.pushNamed('/details')
    ↓
1. 检查 home（如果路由是 "/"）
    ↓ (未找到)
2. 检查 routes 表
    ↓ (未找到)
3. 调用 onGenerateRoute
    ↓ (返回 null)
4. 调用 onUnknownRoute
```

### MaterialApp 的特殊处理

1. **自动使用 MaterialPageRoute**：`MaterialApp` 自动使用 `MaterialPageRoute` 作为 `pageRouteBuilder`，无需手动指定
2. **Hero 动画支持**：通过 `HeroControllerScope` 和 `MaterialRectArcTween` 提供 Material 风格的 Hero 动画
3. **ScaffoldMessenger 支持**：通过 `scaffoldMessengerKey` 提供全局的 SnackBar 等消息控制

## 使用示例

### 基本使用

```dart
MaterialApp(
  home: MyHomePage(),
  routes: {
    '/details': (context) => DetailsPage(),
    '/settings': (context) => SettingsPage(),
  },
  theme: ThemeData.light(),
)
```

### 使用 onGenerateRoute

```dart
MaterialApp(
  onGenerateRoute: (settings) {
    if (settings.name == '/') {
      return MaterialPageRoute(builder: (_) => HomePage());
    }
    if (settings.name!.startsWith('/user/')) {
      final userId = settings.name!.split('/').last;
      return MaterialPageRoute(
        builder: (_) => UserPage(userId: userId),
      );
    }
    return null; // 将调用 onUnknownRoute
  },
  onUnknownRoute: (settings) {
    return MaterialPageRoute(
      builder: (_) => NotFoundPage(route: settings.name),
    );
  },
  theme: ThemeData.light(),
)
```

### 使用 ScaffoldMessenger

```dart
final GlobalKey<ScaffoldMessengerState> scaffoldMessengerKey =
    GlobalKey<ScaffoldMessengerState>();

MaterialApp(
  scaffoldMessengerKey: scaffoldMessengerKey,
  home: MyHomePage(),
  theme: ThemeData.light(),
)

// 在应用的任何地方使用
scaffoldMessengerKey.currentState?.showSnackBar(
  SnackBar(content: Text('Hello')),
);
```

## 与 WidgetsApp 的对比

### 相同点

- 所有 Navigator 相关属性的功能和行为与 `WidgetsApp` 相同
- 路由查找顺序相同
- 支持相同的路由配置方式

### 不同点

| 特性 | WidgetsApp | MaterialApp |
| --- | --- | --- |
| **pageRouteBuilder** | 必须手动指定 | 自动使用 `MaterialPageRoute` |
| **Hero 动画** | 需要手动配置 | 自动配置 Material 风格 Hero 动画 |
| **ScaffoldMessenger** | 不支持 | 支持（通过 `scaffoldMessengerKey`） |

## 总结

第三部分详细介绍了 `MaterialApp` 的所有 Navigator 相关属性：

1. **继承自 WidgetsApp 的属性**：`navigatorKey`、`home`、`routes`、`initialRoute`、`onGenerateRoute`、`onGenerateInitialRoutes`、`onUnknownRoute`、`onNavigationNotification`、`navigatorObservers`

2. **MaterialApp 特有属性**：`scaffoldMessengerKey`（用于全局控制 SnackBar 等消息）

3. **MaterialApp 的特殊处理**：
   - 自动使用 `MaterialPageRoute` 作为 `pageRouteBuilder`
   - 自动配置 Material 风格的 Hero 动画（通过 `HeroControllerScope` 和 `MaterialRectArcTween`）

4. **Hero 动画**：通过 `createMaterialHeroController()` 静态方法创建 Material 风格的 Hero 控制器

这些特性使得 `MaterialApp` 能够提供完整的 Material Design 路由体验，包括 Material 风格的页面过渡和 Hero 动画。
