# Navigator 构造函数与属性详解

## 功能概述

本文档详细讲解 `Navigator` 类的构造函数和所有属性，这些属性控制着 Navigator 的行为和配置。

## 类定义

`Navigator` 是一个 `StatefulWidget`，用于维护基于栈的子 widget 历史记录。

```dart 1558:1558:packages/flutter/lib/src/widgets/navigator.dart
class Navigator extends StatefulWidget {
```

## 构造函数

构造函数创建一个维护基于栈的子 widget 历史的 widget。

```dart 1559:1583:packages/flutter/lib/src/widgets/navigator.dart
  /// Creates a widget that maintains a stack-based history of child widgets.
  ///
  /// If the [pages] is not empty, the [onPopPage] must not be null.
  const Navigator({
    super.key,
    this.pages = const <Page<dynamic>>[],
    @Deprecated(
      'Use onDidRemovePage instead. '
      'This feature was deprecated after v3.16.0-17.0.pre.',
    )
    this.onPopPage,
    this.initialRoute,
    this.onGenerateInitialRoutes = Navigator.defaultGenerateInitialRoutes,
    this.onGenerateRoute,
    this.onUnknownRoute,
    this.transitionDelegate = const DefaultTransitionDelegate<dynamic>(),
    this.reportsRouteUpdateToEngine = false,
    this.clipBehavior = Clip.hardEdge,
    this.observers = const <NavigatorObserver>[],
    this.requestFocus = true,
    this.restorationScopeId,
    this.routeTraversalEdgeBehavior = kDefaultRouteTraversalEdgeBehavior,
    this.routeDirectionalTraversalEdgeBehavior = kDefaultRouteDirectionalTraversalEdgeBehavior,
    this.onDidRemovePage,
  });
```

**重要提示**：如果 [pages] 不为空，则 [onPopPage] 不能为 null（但建议使用 [onDidRemovePage] 替代已废弃的 [onPopPage]）。

## 属性详解

### pages

用于填充历史的页面列表。

```dart 1585:1613:packages/flutter/lib/src/widgets/navigator.dart
  /// The list of pages with which to populate the history.
  ///
  /// Pages are turned into routes using [Page.createRoute] in a manner
  /// analogous to how [Widget]s are turned into [Element]s (and [State]s or
  /// [RenderObject]s) using [Widget.createElement] (and
  /// [StatefulWidget.createState] or [RenderObjectWidget.createRenderObject]).
  ///
  /// When this list is updated, the new list is compared to the previous
  /// list and the set of routes is updated accordingly.
  ///
  /// Some [Route]s do not correspond to [Page] objects, namely, those that are
  /// added to the history using the [Navigator] API ([push] and friends). A
  /// [Route] that does not correspond to a [Page] object is called a pageless
  /// route and is tied to the [Route] that _does_ correspond to a [Page] object
  /// that is below it in the history.
  ///
  /// Pages that are added or removed may be animated as controlled by the
  /// [transitionDelegate]. If a page is removed that had other pageless routes
  /// pushed on top of it using [push] and friends, those pageless routes are
  /// also removed with or without animation as determined by the
  /// [transitionDelegate].
  ///
  /// To use this API, an [onPopPage] callback must also be provided to properly
  /// clean up this list if a page has been popped.
  ///
  /// If [initialRoute] is non-null when the widget is first created, then
  /// [onGenerateInitialRoutes] is used to generate routes that are above those
  /// corresponding to [pages] in the initial history.
  final List<Page<dynamic>> pages;
```

**关键点**：

- 页面通过 [Page.createRoute] 转换为路由，类似于 Widget 转换为 Element 的方式
- 当列表更新时，会与之前的列表进行比较，并相应地更新路由集
- 使用 [Navigator] API（[push] 及其相关方法）添加的路由不对应 [Page] 对象，称为无页面路由（pageless route）
- 添加或删除的页面可能会根据 [transitionDelegate] 进行动画处理
- 如果 [initialRoute] 在首次创建 widget 时不为 null，则使用 [onGenerateInitialRoutes] 生成初始历史中位于 [pages] 对应路由上方的路由

### onPopPage（已废弃）

当 [pop] 被调用但当前 [Route] 对应于 [pages] 列表中找到的 [Page] 时调用。

```dart 1615:1634:packages/flutter/lib/src/widgets/navigator.dart
  /// This is deprecated and replaced by [onDidRemovePage].
  ///
  /// Called when [pop] is invoked but the current [Route] corresponds to a
  /// [Page] found in the [pages] list.
  ///
  /// The `result` argument is the value with which the route is to complete
  /// (e.g. the value returned from a dialog).
  ///
  /// This callback is responsible for calling [Route.didPop] and returning
  /// whether this pop is successful.
  ///
  /// The [Navigator] widget should be rebuilt with a [pages] list that does not
  /// contain the [Page] for the given [Route]. The next time the [pages] list
  /// is updated, if the [Page] corresponding to this [Route] is still present,
  /// it will be interpreted as a new route to display.
  @Deprecated(
    'Use onDidRemovePage instead. '
    'This feature was deprecated after v3.16.0-17.0.pre.',
  )
  final PopPageCallback? onPopPage;
```

**注意**：此属性已在 v3.16.0-17.0.pre 之后废弃，应使用 [onDidRemovePage] 替代。

### onDidRemovePage

当与给定 [Page] 关联的 [Route] 已从 Navigator 中移除时调用。

```dart 1636:1649:packages/flutter/lib/src/widgets/navigator.dart
  /// Called when the [Route] associated with the given [Page] has been removed
  /// from the Navigator.
  ///
  /// This can happen when the route is removed or completed through
  /// [Navigator.pop], [Navigator.pushReplacement], or its friends.
  ///
  /// This callback is responsible for removing the given page from the list of
  /// [pages].
  ///
  /// The [Navigator] widget should be rebuilt with a [pages] list that does not
  /// contain the given page [Page]. The next time the [pages] list
  /// is updated, if the given [Page] is still present, it will be interpreted
  /// as a new page to display.
  final DidRemovePageCallback? onDidRemovePage;
```

**关键点**：

- 当路由通过 [Navigator.pop]、[Navigator.pushReplacement] 或其相关方法移除或完成时，会调用此回调
- 回调负责从 [pages] 列表中移除给定页面
- Navigator widget 应该使用不包含给定页面的 [pages] 列表重建

### transitionDelegate

用于决定在 [pages] 更新期间路由如何进入或退出屏幕的委托。

```dart 1651:1655:packages/flutter/lib/src/widgets/navigator.dart
  /// The delegate used for deciding how routes transition in or off the screen
  /// during the [pages] updates.
  ///
  /// Defaults to [DefaultTransitionDelegate].
  final TransitionDelegate<dynamic> transitionDelegate;
```

**默认值**：[DefaultTransitionDelegate]

### initialRoute

要显示的第一个路由的名称。

```dart 1657:1667:packages/flutter/lib/src/widgets/navigator.dart
  /// The name of the first route to show.
  ///
  /// Defaults to [Navigator.defaultRouteName].
  ///
  /// The value is interpreted according to [onGenerateInitialRoutes], which
  /// defaults to [defaultGenerateInitialRoutes].
  ///
  /// Changing the [initialRoute] will have no effect, as it only controls the
  /// _initial_ route. To change the route while the application is running, use
  /// the static functions on this class, such as [push] or [replace].
  final String? initialRoute;
```

**关键点**：

- 默认值为 [Navigator.defaultRouteName]（即 '/'）
- 值根据 [onGenerateInitialRoutes] 解释，默认为 [defaultGenerateInitialRoutes]
- 更改 [initialRoute] 不会产生任何效果，因为它只控制初始路由
- 要在应用程序运行时更改路由，请使用此类上的静态函数，例如 [push] 或 [replace]

### onGenerateRoute

为给定的 [RouteSettings] 生成路由的回调。

```dart 1669:1670:packages/flutter/lib/src/widgets/navigator.dart
  /// Called to generate a route for a given [RouteSettings].
  final RouteFactory? onGenerateRoute;
```

### onUnknownRoute

当 [onGenerateRoute] 无法生成路由时调用。

```dart 1672:1680:packages/flutter/lib/src/widgets/navigator.dart
  /// Called when [onGenerateRoute] fails to generate a route.
  ///
  /// This callback is typically used for error handling. For example, this
  /// callback might always generate a "not found" page that describes the route
  /// that wasn't found.
  ///
  /// Unknown routes can arise either from errors in the app or from external
  /// requests to push routes, such as from Android intents.
  final RouteFactory? onUnknownRoute;
```

**用途**：

- 通常用于错误处理
- 可以生成一个"未找到"页面来描述未找到的路由
- 未知路由可能来自应用程序中的错误或外部推送路由的请求（例如来自 Android intent）

### observers

此 navigator 的观察者列表。

```dart 1682:1683:packages/flutter/lib/src/widgets/navigator.dart
  /// A list of observers for this navigator.
  final List<NavigatorObserver> observers;
```

观察者可以监听路由的变化，例如路由的推送、弹出、替换等操作。

### restorationScopeId

用于保存和恢复 navigator 状态（包括其历史）的恢复 ID。

```dart 1685:1713:packages/flutter/lib/src/widgets/navigator.dart
  /// Restoration ID to save and restore the state of the navigator, including
  /// its history.
  ///
  /// {@template flutter.widgets.navigator.restorationScopeId}
  /// If a restoration ID is provided, the navigator will persist its internal
  /// state (including the route history as well as the restorable state of the
  /// routes) and restore it during state restoration.
  ///
  /// If no restoration ID is provided, the route history stack will not be
  /// restored and state restoration is disabled for the individual routes as
  /// well.
  ///
  /// The state is persisted in a [RestorationBucket] claimed from
  /// the surrounding [RestorationScope] using the provided restoration ID.
  /// Within that bucket, the [Navigator] also creates a new [RestorationScope]
  /// for its children (the [Route]s).
  ///
  /// See also:
  ///
  ///  * [RestorationManager], which explains how state restoration works in
  ///    Flutter.
  ///  * [RestorationMixin], which contains a runnable code sample showcasing
  ///    state restoration in Flutter.
  ///  * [Navigator], which explains under the heading "state restoration"
  ///    how and under what conditions the navigator restores its state.
  ///  * [Navigator.restorablePush], which includes an example showcasing how
  ///    to push a restorable route onto the navigator.
  /// {@endtemplate}
  final String? restorationScopeId;
```

**关键点**：

- 如果提供了恢复 ID，navigator 将持久化其内部状态（包括路由历史以及路由的可恢复状态）并在状态恢复期间恢复它
- 如果未提供恢复 ID，路由历史栈将不会被恢复，并且各个路由的状态恢复也会被禁用
- 状态保存在使用提供的恢复 ID 从周围的 [RestorationScope] 声明的 [RestorationBucket] 中
- 在该 bucket 内，[Navigator] 还为其子项（[Route]s）创建一个新的 [RestorationScope]

### routeTraversalEdgeBehavior

控制在路由内定义 widget 焦点遍历的焦点范围的第一项和最后一项之外的焦点传输。

```dart 1715:1728:packages/flutter/lib/src/widgets/navigator.dart
  /// Controls the transfer of focus beyond the first and the last items of a
  /// focus scope that defines focus traversal of widgets within a route.
  ///
  /// {@template flutter.widgets.navigator.routeTraversalEdgeBehavior}
  /// The focus inside routes installed in the top of the app affects how
  /// the app behaves with respect to the platform content surrounding it.
  /// For example, on the web, an app is at a minimum surrounded by browser UI,
  /// such as the address bar, browser tabs, and more. The user should be able
  /// to reach browser UI using normal focus shortcuts. Similarly, if the app
  /// is embedded within an `<iframe>` or inside a custom element, it should
  /// be able to participate in the overall focus traversal, including elements
  /// not rendered by Flutter.
  /// {@endtemplate}
  final TraversalEdgeBehavior routeTraversalEdgeBehavior;
```

**用途**：

- 影响应用相对于其周围平台内容的行为
- 在 Web 上，应用至少被浏览器 UI（如地址栏、浏览器标签等）包围
- 用户应该能够使用正常的焦点快捷键到达浏览器 UI
- 如果应用嵌入在 `<iframe>` 内或自定义元素内，它应该能够参与整体焦点遍历，包括 Flutter 未渲染的元素

### routeDirectionalTraversalEdgeBehavior

控制在路由内定义 widget 焦点遍历的焦点范围的第一项和最后一项之外的方向性焦点传输。

```dart 1730:1734:packages/flutter/lib/src/widgets/navigator.dart
  /// Controls the directional transfer of focus beyond the first and the last
  /// items of a focus scope that defines focus traversal of widgets within a route.
  ///
  /// {@macro flutter.widgets.navigator.routeTraversalEdgeBehavior}
  final TraversalEdgeBehavior routeDirectionalTraversalEdgeBehavior;
```

此属性与 [routeTraversalEdgeBehavior] 类似，但专门用于方向性焦点传输。

### defaultRouteName

应用程序的默认路由名称。

```dart 1736:1742:packages/flutter/lib/src/widgets/navigator.dart
  /// The name for the default route of the application.
  ///
  /// See also:
  ///
  ///  * [dart:ui.PlatformDispatcher.defaultRouteName], which reflects the route that the
  ///    application was started with.
  static const String defaultRouteName = '/';
```

**值**：'/'（根路径）

### onGenerateInitialRoutes

当 widget 创建时，如果 [initialRoute] 不为 null，则调用此回调以生成初始 [Route] 对象列表。

```dart 1744:1762:packages/flutter/lib/src/widgets/navigator.dart
  /// Called when the widget is created to generate the initial list of [Route]
  /// objects if [initialRoute] is not null.
  ///
  /// Defaults to [defaultGenerateInitialRoutes].
  ///
  /// The [NavigatorState] and [initialRoute] will be passed to the callback.
  /// The callback must return a list of [Route] objects with which the history
  /// will be primed.
  ///
  /// When parsing the initialRoute, if there's any chance that it may
  /// contain complex characters, it's best to use the
  /// [characters](https://pub.dev/packages/characters) API. This will ensure
  /// that extended grapheme clusters and surrogate pairs are treated as single
  /// characters by the code, the same way that they appear to the user. For
  /// example, the string "👨‍👩‍👦" appears to the user as a single
  /// character and `string.characters.length` intuitively returns 1. On the
  /// other hand, `string.length` returns 8, and `string.runes.length` returns
  /// 5!
  final RouteListFactory onGenerateInitialRoutes;
```

**关键点**：

- 默认值为 [defaultGenerateInitialRoutes]
- [NavigatorState] 和 [initialRoute] 将传递给回调
- 回调必须返回一个 [Route] 对象列表，历史将使用这些对象进行初始化
- 解析 initialRoute 时，如果它可能包含复杂字符，最好使用 [characters](https://pub.dev/packages/characters) API

### reportsRouteUpdateToEngine

此 navigator 是否应在最顶层路由更改时向引擎报告路由更新消息。

```dart 1764:1785:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether this navigator should report route update message back to the
  /// engine when the top-most route changes.
  ///
  /// If the property is set to true, this navigator automatically sends the
  /// route update message to the engine when it detects top-most route changes.
  /// The messages are used by the web engine to update the browser URL bar.
  ///
  /// If the property is set to true when the [Navigator] is first created,
  /// single-entry history mode is requested using
  /// [SystemNavigator.selectSingleEntryHistory]. This means this property
  /// should not be used at the same time as [PlatformRouteInformationProvider]
  /// is used with a [Router] (including when used with [MaterialApp.router],
  /// for example).
  ///
  /// If there are multiple navigators in the widget tree, at most one of them
  /// can set this property to true (typically, the top-most one created from
  /// the [WidgetsApp]). Otherwise, the web engine may receive multiple route
  /// update messages from different navigators and fail to update the URL
  /// bar.
  ///
  /// Defaults to false.
  final bool reportsRouteUpdateToEngine;
```

**关键点**：

- 如果设置为 true，此 navigator 会在检测到最顶层路由更改时自动向引擎发送路由更新消息
- 消息由 Web 引擎用于更新浏览器 URL 栏
- 如果在首次创建 [Navigator] 时将此属性设置为 true，则使用 [SystemNavigator.selectSingleEntryHistory] 请求单条目历史模式
- 此属性不应与 [PlatformRouteInformationProvider] 与 [Router] 一起使用（例如，与 [MaterialApp.router] 一起使用时）
- 如果 widget 树中有多个 navigator，最多只能有一个将此属性设置为 true（通常是从 [WidgetsApp] 创建的最顶层 navigator）
- **默认值**：false

### clipBehavior

裁剪行为。

```dart 1787:1793:packages/flutter/lib/src/widgets/navigator.dart
  /// {@macro flutter.material.Material.clipBehavior}
  ///
  /// In cases where clipping is not desired, consider setting this property to
  /// [Clip.none].
  ///
  /// Defaults to [Clip.hardEdge].
  final Clip clipBehavior;
```

**默认值**：[Clip.hardEdge]

如果不希望裁剪，可以考虑将此属性设置为 [Clip.none]。

### requestFocus

当新路由被推送到 navigator 上时，navigator 及其新的最顶层路由是否应该请求焦点。

```dart 1795:1802:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether or not the navigator and it's new topmost route should request focus
  /// when the new route is pushed onto the navigator.
  ///
  /// If [Route.requestFocus] is set on the topmost route, that will take precedence
  /// over this value.
  ///
  /// Defaults to true.
  final bool requestFocus;
```

**关键点**：

- 如果最顶层路由上设置了 [Route.requestFocus]，则该值优先于此值
- **默认值**：true

## 相关链接

- [Navigator 类概述与使用指南](navigator.dart_Navigator_1_概述与使用指南.md)
- [Navigator 命名路由方法](navigator.dart_Navigator_3_命名路由方法.md)
- [Navigator 直接路由操作](navigator.dart_Navigator_4_直接路由操作.md)
