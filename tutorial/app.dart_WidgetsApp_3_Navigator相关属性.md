# WidgetsApp Navigator 相关属性详解

## 概述

`WidgetsApp` 提供了丰富的 Navigator 相关属性，用于配置基于 `Navigator` 的路由系统。这些属性共同定义了应用的路由行为，包括路由生成、初始路由、路由观察等。

## navigatorKey

```dart 494:510:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.navigatorKey}
  /// A key to use when building the [Navigator].
  ///
  /// If a [navigatorKey] is specified, the [Navigator] can be directly
  /// manipulated without first obtaining it from a [BuildContext] via
  /// [Navigator.of]: from the [navigatorKey], use the [GlobalKey.currentState]
  /// getter.
  ///
  /// If this is changed, a new [Navigator] will be created, losing all the
  /// application state in the process; in that case, the [navigatorObservers]
  /// must also be changed, since the previous observers will be attached to the
  /// previous navigator.
  ///
  /// The [Navigator] is only built if [onGenerateRoute] is not null; if it is
  /// null, [navigatorKey] must also be null.
  /// {@endtemplate}
  final GlobalKey<NavigatorState>? navigatorKey;
```

**功能**：用于构建 `Navigator` 的全局键。

**用途**：

1. **直接访问 Navigator**：如果指定了 `navigatorKey`，可以直接通过 `navigatorKey.currentState` 访问 `NavigatorState`，而不需要通过 `Navigator.of(context)` 获取。

2. **全局导航控制**：允许在应用的任何地方（包括不在 widget 树中的代码）控制导航。

**注意事项**：

- 如果 `navigatorKey` 改变，会创建新的 `Navigator`，导致应用状态丢失
- 如果 `navigatorKey` 改变，`navigatorObservers` 也必须改变，因为之前的观察者会附加到之前的 Navigator
- 只有当 `onGenerateRoute` 不为 `null` 时，`Navigator` 才会被构建；如果 `onGenerateRoute` 为 `null`，`navigatorKey` 也必须为 `null`

## onGenerateRoute

```dart 512:534:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.onGenerateRoute}
  /// The route generator callback used when the app is navigated to a
  /// named route.
  ///
  /// If this returns null when building the routes to handle the specified
  /// [initialRoute], then all the routes are discarded and
  /// [Navigator.defaultRouteName] is used instead (`/`). See [initialRoute].
  ///
  /// During normal app operation, the [onGenerateRoute] callback will only be
  /// applied to route names pushed by the application, and so should never
  /// return null.
  ///
  /// This is used if [routes] does not contain the requested route.
  ///
  /// The [Navigator] is only built if routes are provided (either via [home],
  /// [routes], [onGenerateRoute], or [onUnknownRoute]); if they are not,
  /// [builder] must not be null.
  /// {@endtemplate}
  ///
  /// If this property is not set, either the [routes] or [home] properties must
  /// be set, and the [pageRouteBuilder] must also be set so that the
  /// default handler will know what routes and [PageRoute]s to build.
  final RouteFactory? onGenerateRoute;
```

**功能**：当应用导航到命名路由时使用的路由生成器回调。

**路由查找顺序**：

1. 首先检查 `routes` 表是否包含请求的路由
2. 如果 `routes` 表中没有，则调用 `onGenerateRoute`
3. 如果 `onGenerateRoute` 返回 `null`，则调用 `onUnknownRoute`

**特殊行为**：

- **初始路由处理**：如果 `onGenerateRoute` 在处理 `initialRoute` 时返回 `null`，所有路由都会被丢弃，使用 `Navigator.defaultRouteName`（`/`）代替
- **正常操作**：在正常应用操作中，`onGenerateRoute` 只应用于应用推送的路由名称，因此不应该返回 `null`

**依赖关系**：

- 如果 `onGenerateRoute` 未设置，则必须设置 `routes` 或 `home` 属性
- 同时必须设置 `pageRouteBuilder`，以便默认处理器知道如何构建 `PageRoute`

## onGenerateInitialRoutes

```dart 536:544:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.onGenerateInitialRoutes}
  /// The routes generator callback used for generating initial routes if
  /// [initialRoute] is provided.
  ///
  /// If this property is not set, the underlying
  /// [Navigator.onGenerateInitialRoutes] will default to
  /// [Navigator.defaultGenerateInitialRoutes].
  /// {@endtemplate}
  final InitialRouteListFactory? onGenerateInitialRoutes;
```

**功能**：如果提供了 `initialRoute`，用于生成初始路由的路由生成器回调。

**默认行为**：如果未设置此属性，底层 `Navigator.onGenerateInitialRoutes` 将默认使用 `Navigator.defaultGenerateInitialRoutes`。

**使用场景**：自定义深度链接的处理方式，例如当应用通过深度链接启动时，可以自定义如何生成初始路由栈。

## pageRouteBuilder

```dart 546:566:packages/flutter/lib/src/widgets/app.dart
  /// The [PageRoute] generator callback used when the app is navigated to a
  /// named route.
  ///
  /// A [PageRoute] represents the page in a [Navigator], so that it can
  /// correctly animate between pages, and to represent the "return value" of
  /// a route (e.g. which button a user selected in a modal dialog).
  ///
  /// This callback can be used, for example, to specify that a [MaterialPageRoute]
  /// or a [CupertinoPageRoute] should be used for building page transitions.
  ///
  /// The [PageRouteFactory] type is generic, meaning the provided function must
  /// itself be generic. For example (with special emphasis on the `<T>` at the
  /// start of the closure):
  ///
  /// ```dart
  /// pageRouteBuilder: <T>(RouteSettings settings, WidgetBuilder builder) => PageRouteBuilder<T>(
  ///   settings: settings,
  ///   pageBuilder: (BuildContext context, Animation<double> animation, Animation<double> secondaryAnimation) => builder(context),
  /// ),
  /// ```
  final PageRouteFactory? pageRouteBuilder;
```

**功能**：当应用导航到命名路由时使用的 `PageRoute` 生成器回调。

**用途**：

1. **定义页面过渡**：指定使用哪种类型的 `PageRoute`（如 `MaterialPageRoute` 或 `CupertinoPageRoute`）来构建页面过渡
2. **自定义动画**：可以创建自定义的页面过渡动画

**泛型要求**：`PageRouteFactory` 类型是泛型的，提供的函数本身必须是泛型的。注意示例中闭包开头的 `<T>`。

## home

```dart 647:678:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.home}
  /// The widget for the default route of the app ([Navigator.defaultRouteName],
  /// which is `/`).
  ///
  /// This is the route that is displayed first when the application is started
  /// normally, unless [initialRoute] is specified. It's also the route that's
  /// displayed if the [initialRoute] can't be displayed.
  ///
  /// To be able to directly call [Theme.of], [MediaQuery.of], etc, in the code
  /// that sets the [home] argument in the constructor, you can use a [Builder]
  /// widget to get a [BuildContext].
  ///
  /// If [home] is specified, then [routes] must not include an entry for `/`,
  /// as [home] takes its place.
  ///
  /// The [Navigator] is only built if routes are provided (either via [home],
  /// [routes], [onGenerateRoute], or [onUnknownRoute]); if they are not,
  /// [builder] must not be null.
  ///
  /// The difference between using [home] and using [builder] is that the [home]
  /// subtree is inserted into the application below a [Navigator] (and thus
  /// below an [Overlay], which [Navigator] uses). With [home], therefore,
  /// dialog boxes will work automatically, the [routes] table will be used, and
  /// APIs such as [Navigator.push] and [Navigator.pop] will work as expected.
  /// In contrast, the widget returned from [builder] is inserted _above_ the
  /// app's [Navigator] (if any).
  /// {@endtemplate}
  ///
  /// If this property is set, the [pageRouteBuilder] property must also be set
  /// so that the default route handler will know what kind of [PageRoute]s to
  /// build.
  final Widget? home;
```

**功能**：应用的默认路由（`Navigator.defaultRouteName`，即 `/`）的 widget。

**行为**：

1. **首次显示**：应用正常启动时首先显示的路由，除非指定了 `initialRoute`
2. **回退路由**：如果 `initialRoute` 无法显示，则显示此路由

**与 routes 的关系**：

- 如果指定了 `home`，则 `routes` 不能包含 `/` 条目，因为 `home` 已经占据了该位置

**与 builder 的区别**：

- **home**：插入到 `Navigator` 下方（因此在 `Overlay` 下方），对话框会自动工作，`routes` 表会被使用，`Navigator.push` 和 `Navigator.pop` 等 API 会按预期工作
- **builder**：返回的 widget 插入到应用的 `Navigator` 上方（如果有的话）

**依赖关系**：如果设置了此属性，还必须设置 `pageRouteBuilder` 属性，以便默认路由处理器知道构建哪种类型的 `PageRoute`。

## routes

```dart 680:707:packages/flutter/lib/src/widgets/app.dart
  /// The application's top-level routing table.
  ///
  /// When a named route is pushed with [Navigator.pushNamed], the route name is
  /// looked up in this map. If the name is present, the associated
  /// [WidgetBuilder] is used to construct a [PageRoute] specified by
  /// [pageRouteBuilder] to perform an appropriate transition, including [Hero]
  /// animations, to the new route.
  ///
  /// {@template flutter.widgets.widgetsApp.routes}
  /// If the app only has one page, then you can specify it using [home] instead.
  ///
  /// If [home] is specified, then it implies an entry in this table for the
  /// [Navigator.defaultRouteName] route (`/`), and it is an error to
  /// redundantly provide such a route in the [routes] table.
  ///
  /// If a route is requested that is not specified in this table (or by
  /// [home]), then the [onGenerateRoute] callback is called to build the page
  /// instead.
  ///
  /// The [Navigator] is only built if routes are provided (either via [home],
  /// [routes], [onGenerateRoute], or [onUnknownRoute]); if they are not,
  /// [builder] must not be null.
  /// {@endtemplate}
  ///
  /// If the routes map is not empty, the [pageRouteBuilder] property must be set
  /// so that the default route handler will know what kind of [PageRoute]s to
  /// build.
  final Map<String, WidgetBuilder>? routes;
```

**功能**：应用的顶级路由表。

**工作原理**：

1. 当使用 `Navigator.pushNamed` 推送命名路由时，在此映射中查找路由名称
2. 如果名称存在，使用关联的 `WidgetBuilder` 构建由 `pageRouteBuilder` 指定的 `PageRoute`
3. 执行适当的过渡，包括 Hero 动画

**路由查找顺序**：

1. 首先检查 `routes` 表（或 `home`）
2. 如果未找到，调用 `onGenerateRoute`
3. 如果 `onGenerateRoute` 返回 `null`，调用 `onUnknownRoute`

**依赖关系**：如果 `routes` 映射不为空，必须设置 `pageRouteBuilder` 属性。

## onUnknownRoute

```dart 709:724:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.onUnknownRoute}
  /// Called when [onGenerateRoute] fails to generate a route, except for the
  /// [initialRoute].
  ///
  /// This callback is typically used for error handling. For example, this
  /// callback might always generate a "not found" page that describes the route
  /// that wasn't found.
  ///
  /// Unknown routes can arise either from errors in the app or from external
  /// requests to push routes, such as from Android intents.
  ///
  /// The [Navigator] is only built if routes are provided (either via [home],
  /// [routes], [onGenerateRoute], or [onUnknownRoute]); if they are not,
  /// [builder] must not be null.
  /// {@endtemplate}
  final RouteFactory? onUnknownRoute;
```

**功能**：当 `onGenerateRoute` 无法生成路由时调用（除了 `initialRoute`）。

**使用场景**：

1. **错误处理**：通常用于错误处理，例如生成一个"未找到"页面，描述未找到的路由
2. **外部请求**：处理来自外部请求的路由推送，例如 Android intents

**调用时机**：只有在 `onGenerateRoute` 失败时才会调用，且不包括 `initialRoute` 的情况。

## onNavigationNotification

```dart 726:736:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.onNavigationNotification}
  /// The callback to use when receiving a [NavigationNotification].
  ///
  /// By default this updates the engine with the navigation status and stops
  /// bubbling the notification.
  ///
  /// See also:
  ///
  ///  * [NotificationListener.onNotification], which uses this callback.
  /// {@endtemplate}
  final NotificationListenerCallback<NavigationNotification>? onNavigationNotification;
```

**功能**：接收 `NavigationNotification` 时使用的回调。

**默认行为**：默认情况下，这会更新引擎的导航状态并停止通知的冒泡。

**用途**：用于监听导航状态变化，例如监听是否可以弹出路由。

## initialRoute

```dart 738:773:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.initialRoute}
  /// The name of the first route to show, if a [Navigator] is built.
  ///
  /// Defaults to [dart:ui.PlatformDispatcher.defaultRouteName], which may be
  /// overridden by the code that launched the application.
  ///
  /// If the route name starts with a slash, then it is treated as a "deep link",
  /// and before this route is pushed, the routes leading to this one are pushed
  /// also. For example, if the route was `/a/b/c`, then the app would start
  /// with the four routes `/`, `/a`, `/a/b`, and `/a/b/c` loaded, in that order.
  /// Even if the route was just `/a`, the app would start with `/` and `/a`
  /// loaded. You can use the [onGenerateInitialRoutes] property to override
  /// this behavior.
  ///
  /// Intermediate routes aren't required to exist. In the example above, `/a`
  /// and `/a/b` could be skipped if they have no matching route. But `/a/b/c` is
  /// required to have a route, else [initialRoute] is ignored and
  /// [Navigator.defaultRouteName] is used instead (`/`). This can happen if the
  /// app is started with an intent that specifies a non-existent route.
  ///
  /// The [Navigator] is only built if routes are provided (either via [home],
  /// [routes], [onGenerateRoute], or [onUnknownRoute]); if they are not,
  /// [initialRoute] must be null and [builder] must not be null.
  ///
  /// Changing the [initialRoute] will have no effect, as it only controls the
  /// _initial_ route. To change the route while the application is running, use
  /// the [Navigator] or [Router] APIs.
  ///
  /// See also:
  ///
  ///  * [Navigator.initialRoute], which is used to implement this property.
  ///  * [Navigator.push], for pushing additional routes.
  ///  * [Navigator.pop], for removing a route from the stack.
  ///
  /// {@endtemplate}
  final String? initialRoute;
```

**功能**：如果构建了 `Navigator`，要显示的第一个路由的名称。

**默认值**：默认为 `PlatformDispatcher.defaultRouteName`，可能被启动应用的代码覆盖。

**深度链接支持**：

- 如果路由名称以斜杠开头，则被视为"深度链接"
- 在推送此路由之前，会先推送导致此路由的路由
- 例如，如果路由是 `/a/b/c`，应用将按顺序加载四个路由：`/`、`/a`、`/a/b` 和 `/a/b/c`

**中间路由**：

- 中间路由不需要存在（如 `/a` 和 `/a/b` 可以跳过）
- 但最终路由（如 `/a/b/c`）必须存在，否则 `initialRoute` 会被忽略，使用 `Navigator.defaultRouteName`（`/`）代替

**限制**：更改 `initialRoute` 不会有任何效果，因为它只控制*初始*路由。要在应用运行时更改路由，请使用 `Navigator` 或 `Router` API。

## navigatorObservers

```dart 775:785:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.navigatorObservers}
  /// The list of observers for the [Navigator] created for this app.
  ///
  /// This list must be replaced by a list of newly-created observers if the
  /// [navigatorKey] is changed.
  ///
  /// The [Navigator] is only built if routes are provided (either via [home],
  /// [routes], [onGenerateRoute], or [onUnknownRoute]); if they are not,
  /// [navigatorObservers] must be the empty list and [builder] must not be null.
  /// {@endtemplate}
  final List<NavigatorObserver>? navigatorObservers;
```

**功能**：为此应用创建的 `Navigator` 的观察者列表。

**用途**：用于监听导航事件，例如路由推送、弹出等。常用于：

- 路由分析（如 Firebase Analytics）
- 路由日志记录
- 自定义导航行为

**注意事项**：如果 `navigatorKey` 改变，必须用新创建的观察者列表替换此列表。

## builder

```dart 787:834:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.builder}
  /// A builder for inserting widgets above the [Navigator] or - when the
  /// [WidgetsApp.router] constructor is used - above the [Router] but below the
  /// other widgets created by the [WidgetsApp] widget, or for replacing the
  /// [Navigator]/[Router] entirely.
  ///
  /// For example, from the [BuildContext] passed to this method, the
  /// [Directionality], [Localizations], [DefaultTextStyle], [MediaQuery], etc,
  /// are all available. They can also be overridden in a way that impacts all
  /// the routes in the [Navigator] or [Router].
  ///
  /// This is rarely useful, but can be used in applications that wish to
  /// override those defaults, e.g. to force the application into right-to-left
  /// mode despite being in English, or to override the [MediaQuery] metrics
  /// (e.g. to leave a gap for advertisements shown by a plugin from OEM code).
  ///
  /// For specifically overriding the [title] with a value based on the
  /// [Localizations], consider [onGenerateTitle] instead.
  ///
  /// The [builder] callback is passed two arguments, the [BuildContext] (as
  /// `context`) and a [Navigator] or [Router] widget (as `child`).
  ///
  /// If no routes are provided to the regular [WidgetsApp] constructor using
  /// [home], [routes], [onGenerateRoute], or [onUnknownRoute], the `child` will
  /// be null, and it is the responsibility of the [builder] to provide the
  /// application's routing machinery.
  ///
  /// If routes _are_ provided to the regular [WidgetsApp] constructor using one
  /// or more of those properties or if the [WidgetsApp.router] constructor is
  /// used, then `child` is not null, and the returned value should include the
  /// `child` in the widget subtree; if it does not, then the application will
  /// have no [Navigator] or [Router] and the routing related properties (i.e.
  /// [navigatorKey], [home], [routes], [onGenerateRoute], [onUnknownRoute],
  /// [initialRoute], [navigatorObservers], [routeInformationProvider],
  /// [backButtonDispatcher], [routerDelegate], and [routeInformationParser])
  /// are ignored.
  /// {@endtemplate}
  final TransitionBuilder? builder;
```

**功能**：用于在 `Navigator` 或 `Router` 上方插入 widget，或完全替换 `Navigator`/`Router` 的构建器。

**使用场景**：

1. **覆盖默认值**：覆盖 `Directionality`、`Localizations`、`DefaultTextStyle`、`MediaQuery` 等默认值
2. **强制布局方向**：例如强制应用进入从右到左模式
3. **覆盖媒体查询**：例如为 OEM 代码插件显示的广告留出间隙

**参数**：

- `context`：`BuildContext`，可以访问 `Directionality`、`Localizations` 等
- `child`：`Navigator` 或 `Router` widget，如果未提供路由则为 `null`

**重要提示**：如果返回的值不包含 `child`，应用将没有 `Navigator` 或 `Router`，路由相关属性将被忽略。

## 属性之间的关系

### 路由查找顺序

```text
Navigator.pushNamed('/details')
    ↓
1. 检查 routes 表
    ↓ (未找到)
2. 调用 onGenerateRoute
    ↓ (返回 null)
3. 调用 onUnknownRoute
```

### 依赖关系

- `home` 和 `routes` 中的 `/` 互斥
- `home` 和 `onGenerateInitialRoutes` 互斥
- 如果使用 `home` 或 `routes`，必须提供 `pageRouteBuilder`
- 如果使用 `builder` 且不提供路由，`child` 为 `null`，需要自己提供路由机制

## 使用示例

### 基本使用

```dart
WidgetsApp(
  home: MyHomePage(),
  routes: {
    '/details': (context) => DetailsPage(),
    '/settings': (context) => SettingsPage(),
  },
  pageRouteBuilder: (settings, builder) => MaterialPageRoute(
    settings: settings,
    builder: builder,
  ),
  color: Colors.blue,
)
```

### 使用 onGenerateRoute

```dart
WidgetsApp(
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
  color: Colors.blue,
)
```

## 总结

第三部分详细介绍了 `WidgetsApp` 的所有 Navigator 相关属性：

1. **navigatorKey**：用于直接访问 Navigator 的全局键
2. **onGenerateRoute**：路由生成器，处理未在 routes 表中定义的路由
3. **onGenerateInitialRoutes**：初始路由生成器，自定义深度链接处理
4. **pageRouteBuilder**：PageRoute 生成器，定义页面过渡动画
5. **home**：默认路由的 widget
6. **routes**：路由表，定义命名路由
7. **onUnknownRoute**：未知路由处理器，用于错误处理
8. **onNavigationNotification**：导航通知回调
9. **initialRoute**：初始路由名称，支持深度链接
10. **navigatorObservers**：Navigator 观察者列表
11. **builder**：自定义构建器，用于覆盖默认值或完全自定义路由

这些属性共同构成了完整的 Navigator 路由系统配置。
