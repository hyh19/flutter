# Navigator 概述与使用指南详解

## 功能概述

`Navigator` 是一个管理子 widget 栈的 widget，采用栈式管理方式。许多应用在 widget 树顶部附近放置一个 navigator，以便使用 [Overlay] 显示逻辑历史，最近访问的页面在视觉上位于较旧页面的上方。使用这种模式可以让 navigator 通过在 overlay 中移动 widget 来实现从一个页面到另一个页面的视觉过渡。类似地，navigator 也可以用于显示对话框，通过将对话框 widget 定位在当前页面上方。

```dart 1260:1267:packages/flutter/lib/src/widgets/navigator.dart
/// A widget that manages a set of child widgets with a stack discipline.
///
/// Many apps have a navigator near the top of their widget hierarchy in order
/// to display their logical history using an [Overlay] with the most recently
/// visited pages visually on top of the older pages. Using this pattern lets
/// the navigator visually transition from one page to another by moving the widgets
/// around in the overlay. Similarly, the navigator can be used to show a dialog
/// by positioning the dialog widget above the current page.
```

## 使用 Pages API

`Navigator` 会将它的 [Navigator.pages] 转换为一个 [Route] 栈（如果提供了该属性）。[Navigator.pages] 的变化会触发路由栈的更新。`Navigator` 会更新其路由以匹配新的 [Navigator.pages] 配置。要使用此 API，可以创建一个 [Page] 子类，并为 [Navigator.pages] 定义一个 [Page] 列表。还需要提供一个 [Navigator.onPopPage] 回调，以便在弹出时正确清理输入页面。

默认情况下，`Navigator` 会使用 [DefaultTransitionDelegate] 来决定路由如何进入或退出屏幕。要自定义它，可以定义一个 [TransitionDelegate] 子类，并将其提供给 [Navigator.transitionDelegate]。

```dart 1269:1284:packages/flutter/lib/src/widgets/navigator.dart
/// ## Using the Pages API
///
/// The [Navigator] will convert its [Navigator.pages] into a stack of [Route]s
/// if it is provided. A change in [Navigator.pages] will trigger an update to
/// the stack of [Route]s. The [Navigator] will update its routes to match the
/// new configuration of its [Navigator.pages]. To use this API, one can create
/// a [Page] subclass and defines a list of [Page]s for [Navigator.pages]. A
/// [Navigator.onPopPage] callback is also required to properly clean up the
/// input pages in case of a pop.
///
/// By Default, the [Navigator] will use [DefaultTransitionDelegate] to decide
/// how routes transition in or out of the screen. To customize it, define a
/// [TransitionDelegate] subclass and provide it to the
/// [Navigator.transitionDelegate].
///
/// For more information on using the pages API, see the [Router] widget.
```

## 使用 Navigator API

移动应用通常通过全屏元素（称为"屏幕"或"页面"）来展示内容。在 Flutter 中，这些元素称为路由（routes），由 `Navigator` widget 管理。navigator 管理一个 [Route] 对象栈，并提供两种管理栈的方式：声明式 API [Navigator.pages] 或命令式 API [Navigator.push] 和 [Navigator.pop]。

当用户界面符合栈式范式时（用户应该能够导航回栈中的早期元素），使用路由和 Navigator 是合适的。在某些平台上，例如 Android，系统 UI 会提供一个返回按钮（在应用程序边界之外），允许用户导航回应用程序栈中的早期路由。在没有这种内置导航机制的平台上，可以使用 [AppBar]（通常在 [Scaffold.appBar] 属性中使用）自动为用户导航添加返回按钮。

```dart 1286:1303:packages/flutter/lib/src/widgets/navigator.dart
/// ## Using the Navigator API
///
/// Mobile apps typically reveal their contents via full-screen elements
/// called "screens" or "pages". In Flutter these elements are called
/// routes and they're managed by a [Navigator] widget. The navigator
/// manages a stack of [Route] objects and provides two ways for managing
/// the stack, the declarative API [Navigator.pages] or imperative API
/// [Navigator.push] and [Navigator.pop].
///
/// When your user interface fits this paradigm of a stack, where the user
/// should be able to _navigate_ back to an earlier element in the stack,
/// the use of routes and the Navigator is appropriate. On certain platforms,
/// such as Android, the system UI will provide a back button (outside the
/// bounds of your application) that will allow the user to navigate back
/// to earlier routes in your application's stack. On platforms that don't
/// have this build-in navigation mechanism, the use of an [AppBar] (typically
/// used in the [Scaffold.appBar] property) can automatically add a back
/// button for user navigation.
```

### 显示全屏路由

虽然可以直接创建 navigator，但最常见的是使用由 `Router` 创建的 navigator，它本身由 [WidgetsApp] 或 [MaterialApp] widget 创建和配置。可以使用 [Navigator.of] 引用该 navigator。

[MaterialApp] 是设置的最简单方式。[MaterialApp] 的 home 成为 [Navigator] 栈底部的路由。这是启动应用时看到的内容。

```dart 1305:1320:packages/flutter/lib/src/widgets/navigator.dart
/// ### Displaying a full-screen route
///
/// Although you can create a navigator directly, it's most common to use the
/// navigator created by the `Router` which itself is created and configured by
/// a [WidgetsApp] or a [MaterialApp] widget. You can refer to that navigator
/// with [Navigator.of].
///
/// A [MaterialApp] is the simplest way to set things up. The [MaterialApp]'s
/// home becomes the route at the bottom of the [Navigator]'s stack. It is what
/// you see when the app is launched.
///
/// ```dart
/// void main() {
///   runApp(const MaterialApp(home: MyAppHome()));
/// }
/// ```
```

要在栈上推送新路由，可以创建一个 [MaterialPageRoute] 实例，带有一个创建要在屏幕上显示内容的 builder 函数。例如：

```dart 1322:1342:packages/flutter/lib/src/widgets/navigator.dart
/// To push a new route on the stack you can create an instance of
/// [MaterialPageRoute] with a builder function that creates whatever you
/// want to appear on the screen. For example:
///
/// ```dart
/// Navigator.push(context, MaterialPageRoute<void>(
///   builder: (BuildContext context) {
///     return Scaffold(
///       appBar: AppBar(title: const Text('My Page')),
///       body: Center(
///         child: TextButton(
///           child: const Text('POP'),
///           onPressed: () {
///             Navigator.pop(context);
///           },
///         ),
///       ),
///     );
///   },
/// ));
/// ```
```

路由使用 builder 函数而不是 child widget 来定义其 widget，因为它会根据推送和弹出的时间在不同的上下文中构建和重建。

可以看到，新路由可以通过 Navigator 的 pop 方法弹出，显示应用的主页：

```dart 1348:1353:packages/flutter/lib/src/widgets/navigator.dart
/// As you can see, the new route can be popped, revealing the app's home
/// page, with the Navigator's pop method:
///
/// ```dart
/// Navigator.pop(context);
/// ```
```

在带有 [Scaffold] 的路由中，通常不需要提供弹出 Navigator 的 widget，因为 Scaffold 会自动在其 AppBar 中添加"返回"按钮。按下返回按钮会调用 [Navigator.pop]。在 Android 上，按下系统返回按钮也会执行相同的操作。

```dart 1355:1359:packages/flutter/lib/src/widgets/navigator.dart
/// It usually isn't necessary to provide a widget that pops the Navigator
/// in a route with a [Scaffold] because the Scaffold automatically adds a
/// 'back' button to its AppBar. Pressing the back button causes
/// [Navigator.pop] to be called. On Android, pressing the system back
/// button does the same thing.
```

### 使用命名路由

移动应用通常管理大量路由，按名称引用它们通常是最简单的。路由名称按约定使用类似路径的结构（例如，'/a/b/c'）。应用的首页路由默认命名为 '/'。

[MaterialApp] 可以使用 [Map<String, WidgetBuilder>] 创建，该映射从路由名称映射到创建它的 builder 函数。[MaterialApp] 使用此映射为其 navigator 的 [onGenerateRoute] 回调创建值。

```dart 1361:1390:packages/flutter/lib/src/widgets/navigator.dart
/// ### Using named navigator routes
///
/// Mobile apps often manage a large number of routes and it's often
/// easiest to refer to them by name. Route names, by convention,
/// use a path-like structure (for example, '/a/b/c').
/// The app's home page route is named '/' by default.
///
/// The [MaterialApp] can be created
/// with a [Map<String, WidgetBuilder>] which maps from a route's name to
/// a builder function that will create it. The [MaterialApp] uses this
/// map to create a value for its navigator's [onGenerateRoute] callback.
///
/// ```dart
/// void main() {
///   runApp(MaterialApp(
///     home: const MyAppHome(), // becomes the route named '/'
///     routes: <String, WidgetBuilder> {
///       '/a': (BuildContext context) => const MyPage(title: Text('page A')),
///       '/b': (BuildContext context) => const MyPage(title: Text('page B')),
///       '/c': (BuildContext context) => const MyPage(title: Text('page C')),
///     },
///   ));
/// }
/// ```
///
/// To show a route by name:
///
/// ```dart
/// Navigator.pushNamed(context, '/b');
/// ```
```

### 路由可以返回值

当推送路由以向用户请求值时，可以通过 [pop] 方法的 result 参数返回值。

推送路由的方法返回一个 [Future]。当路由被弹出时，Future 会解析，[Future] 的值是 [pop] 方法的 `result` 参数。

例如，如果我们想要求用户按 'OK' 来确认操作，可以 `await` [Navigator.push] 的结果：

```dart 1392:1419:packages/flutter/lib/src/widgets/navigator.dart
/// ### Routes can return a value
///
/// When a route is pushed to ask the user for a value, the value can be
/// returned via the [pop] method's result parameter.
///
/// Methods that push a route return a [Future]. The Future resolves when the
/// route is popped and the [Future]'s value is the [pop] method's `result`
/// parameter.
///
/// For example if we wanted to ask the user to press 'OK' to confirm an
/// operation we could `await` the result of [Navigator.push]:
///
/// ```dart
/// bool? value = await Navigator.push(context, MaterialPageRoute<bool>(
///   builder: (BuildContext context) {
///     return Center(
///       child: GestureDetector(
///         child: const Text('OK'),
///         onTap: () { Navigator.pop(context, true); }
///       ),
///     );
///   }
/// ));
/// ```
///
/// If the user presses 'OK' then value will be true. If the user backs
/// out of the route, for example by pressing the Scaffold's back button,
/// the value will be null.
```

当路由用于返回值时，路由的类型参数必须与 [pop] 的 result 类型匹配。这就是为什么我们使用 `MaterialPageRoute<bool>` 而不是 `MaterialPageRoute<void>` 或仅仅是 `MaterialPageRoute` 的原因。

```dart 1421:1425:packages/flutter/lib/src/widgets/navigator.dart
/// When a route is used to return a value, the route's type parameter must
/// match the type of [pop]'s result. That's why we've used
/// `MaterialPageRoute<bool>` instead of `MaterialPageRoute<void>` or just
/// `MaterialPageRoute`. (If you prefer to not specify the types, though, that's
/// fine too.)
```

### 弹出路由（Popup routes）

路由不必遮挡整个屏幕。[PopupRoute] 使用 [ModalRoute.barrierColor] 覆盖屏幕，该颜色可以只是部分不透明，以允许当前屏幕显示出来。弹出路由是"模态"的，因为它们阻止对下方 widget 的输入。

有一些函数可以创建和显示弹出路由。例如：[showDialog]、[showMenu] 和 [showModalBottomSheet]。这些函数返回它们推送的路由的 Future，如上所述。调用者可以 await 返回的值，以便在路由弹出时执行操作，或发现路由的值。

还有一些 widget 可以创建弹出路由，例如 [PopupMenuButton] 和 [DropdownButton]。这些 widget 创建 PopupRoute 的内部子类，并使用 Navigator 的 push 和 pop 方法来显示和关闭它们。

```dart 1427:1442:packages/flutter/lib/src/widgets/navigator.dart
/// ### Popup routes
///
/// Routes don't have to obscure the entire screen. [PopupRoute]s cover the
/// screen with a [ModalRoute.barrierColor] that can be only partially opaque to
/// allow the current screen to show through. Popup routes are "modal" because
/// they block input to the widgets below.
///
/// There are functions which create and show popup routes. For
/// example: [showDialog], [showMenu], and [showModalBottomSheet]. These
/// functions return their pushed route's Future as described above.
/// Callers can await the returned value to take an action when the
/// route is popped, or to discover the route's value.
///
/// There are also widgets which create popup routes, like [PopupMenuButton] and
/// [DropdownButton]. These widgets create internal subclasses of PopupRoute
/// and use the Navigator's push and pop methods to show and dismiss them.
```

### 自定义路由

可以创建 widget 库路由类（如 [PopupRoute]、[ModalRoute] 或 [PageRoute]）的子类，以控制用于显示路由的动画过渡、路由模态屏障的颜色和行为，以及路由的其他方面。

[PageRouteBuilder] 类使得可以通过回调定义自定义路由。下面是一个示例，当路由出现或消失时，它会旋转和淡出其子项。此路由不会遮挡整个屏幕，因为它指定了 `opaque: false`，就像弹出路由一样。

```dart 1444:1472:packages/flutter/lib/src/widgets/navigator.dart
/// ### Custom routes
///
/// You can create your own subclass of one of the widget library route classes
/// like [PopupRoute], [ModalRoute], or [PageRoute], to control the animated
/// transition employed to show the route, the color and behavior of the route's
/// modal barrier, and other aspects of the route.
///
/// The [PageRouteBuilder] class makes it possible to define a custom route
/// in terms of callbacks. Here's an example that rotates and fades its child
/// when the route appears or disappears. This route does not obscure the entire
/// screen because it specifies `opaque: false`, just as a popup route does.
///
/// ```dart
/// Navigator.push(context, PageRouteBuilder<void>(
///   opaque: false,
///   pageBuilder: (BuildContext context, _, _) {
///     return const Center(child: Text('My PageRoute'));
///   },
///   transitionsBuilder: (_, Animation<double> animation, _, Widget child) {
///     return FadeTransition(
///       opacity: animation,
///       child: RotationTransition(
///         turns: Tween<double>(begin: 0.5, end: 1.0).animate(animation),
///         child: child,
///       ),
///     );
///   }
/// ));
/// ```
```

页面路由分为两部分构建："页面"和"过渡"。页面成为传递给 `transitionsBuilder` 函数的子项的后代。通常页面只构建一次，因为它不依赖于其动画参数（在此示例中用 `_` 省略）。过渡在其持续时间的每一帧上构建。

```dart 1474:1479:packages/flutter/lib/src/widgets/navigator.dart
/// The page route is built in two parts, the "page" and the
/// "transitions". The page becomes a descendant of the child passed to
/// the `transitionsBuilder` function. Typically the page is only built once,
/// because it doesn't depend on its animation parameters (elided with `_`
/// in this example). The transition is built on every frame
/// for its duration.
```

### 嵌套 Navigator

应用可以使用多个 [Navigator]。将一个 [Navigator] 嵌套在另一个 [Navigator] 下方可用于创建"内部旅程"，例如标签导航、用户注册、商店结账或其他代表应用程序子部分的独立旅程。

#### 示例

iOS 应用的标准做法是使用标签导航，其中每个标签维护自己的导航历史。因此，每个标签都有自己的 [Navigator]，创建一种"并行导航"。

除了标签的并行导航之外，仍然可以启动完全覆盖标签的全屏页面。例如：引导流程或警报对话框。因此，必须存在一个位于标签导航上方的"根" [Navigator]。因此，每个标签的 [Navigator] 实际上是位于单个根 [Navigator] 下方的嵌套 [Navigator]。

```dart 1484:1502:packages/flutter/lib/src/widgets/navigator.dart
/// ### Nesting Navigators
///
/// An app can use more than one [Navigator]. Nesting one [Navigator] below
/// another [Navigator] can be used to create an "inner journey" such as tabbed
/// navigation, user registration, store checkout, or other independent journeys
/// that represent a subsection of your overall application.
///
/// #### Example
///
/// It is standard practice for iOS apps to use tabbed navigation where each
/// tab maintains its own navigation history. Therefore, each tab has its own
/// [Navigator], creating a kind of "parallel navigation."
///
/// In addition to the parallel navigation of the tabs, it is still possible to
/// launch full-screen pages that completely cover the tabs. For example: an
/// on-boarding flow, or an alert dialog. Therefore, there must exist a "root"
/// [Navigator] that sits above the tab navigation. As a result, each of the
/// tab's [Navigator]s are actually nested [Navigator]s sitting below a single
/// root [Navigator].
```

在实践中，用于标签导航的嵌套 [Navigator] 位于 [WidgetsApp] 和 [CupertinoTabView] widget 中，不需要显式创建或管理。

```dart 1504:1506:packages/flutter/lib/src/widgets/navigator.dart
/// In practice, the nested [Navigator]s for tabbed navigation sit in the
/// [WidgetsApp] and [CupertinoTabView] widgets and do not need to be explicitly
/// created or managed.
```

[Navigator.of] 对给定 [BuildContext] 中最接近的祖先 [Navigator] 进行操作。确保在预期的 [Navigator] 下方提供 [BuildContext]，特别是在创建嵌套 [Navigator] 的大型 `build` 方法中。[Builder] widget 可用于在 widget 子树的所需位置访问 [BuildContext]。

```dart 1521:1525:packages/flutter/lib/src/widgets/navigator.dart
/// [Navigator.of] operates on the nearest ancestor [Navigator] from the given
/// [BuildContext]. Be sure to provide a [BuildContext] below the intended
/// [Navigator], especially in large `build` methods where nested [Navigator]s
/// are created. The [Builder] widget can be used to access a [BuildContext] at
/// a desired location in the widget subtree.
```

### 查找封闭路由

在模态路由的常见情况下，可以从 build 方法内部使用 [ModalRoute.of] 获取封闭路由。要确定封闭路由是否是活动路由（例如，以便在路由不活动时使控件变暗），可以检查返回路由的 [Route.isCurrent] 属性。

```dart 1527:1533:packages/flutter/lib/src/widgets/navigator.dart
/// ### Finding the enclosing route
///
/// In the common case of a modal route, the enclosing route can be obtained
/// from inside a build method using [ModalRoute.of]. To determine if the
/// enclosing route is the active route (e.g. so that controls can be dimmed
/// when the route is not active), the [Route.isCurrent] property can be checked
/// on the returned route.
```

## 状态恢复（State Restoration）

如果提供了 [restorationScopeId] 并且被有效的 [RestorationScope] 包围，[Navigator] 将通过重新创建当前的路由历史栈并在状态恢复期间恢复这些路由的内部状态来恢复其状态。但是，栈上的并非所有 [Route] 都可以恢复：

- 如果提供了 [Page.restorationId]，基于 [Page] 的路由会恢复其状态。
- 使用经典命令式 API（[push]、[pushNamed] 及其相关方法）添加的 [Route] 永远无法恢复其状态。
- 使用可恢复命令式 API（[restorablePush]、[restorablePushNamed] 以及名称中包含"restorable"的所有其他命令式方法）添加的 [Route] 如果其下方直到并包括其下方第一个基于 [Page] 的路由的所有路由都被恢复，则恢复其状态。如果其下方没有基于 [Page] 的路由，则仅当其下方的所有路由都恢复其状态时，它才恢复其状态。

```dart 1535:1552:packages/flutter/lib/src/widgets/navigator.dart
/// ## State Restoration
///
/// If provided with a [restorationScopeId] and when surrounded by a valid
/// [RestorationScope] the [Navigator] will restore its state by recreating
/// the current history stack of [Route]s during state restoration and by
/// restoring the internal state of those [Route]s. However, not all [Route]s
/// on the stack can be restored:
///
///  * [Page]-based routes restore their state if [Page.restorationId] is
///    provided.
///  * [Route]s added with the classic imperative API ([push], [pushNamed], and
///    friends) can never restore their state.
///  * A [Route] added with the restorable imperative API ([restorablePush],
///    [restorablePushNamed], and all other imperative methods with "restorable"
///    in their name) restores its state if all routes below it up to and
///    including the first [Page]-based route below it are restored. If there
///    is no [Page]-based route below it, it only restores its state if all
///    routes below it restore theirs.
```

如果 [Route] 被认为是可恢复的，[Navigator] 会将其 [Route.restorationScopeId] 设置为非 null 值。路由可以使用该 ID 来存储和恢复它们自己的状态。例如，[ModalRoute] 将使用此 ID 为其内容 widget 创建 [RestorationScope]。

```dart 1554:1557:packages/flutter/lib/src/widgets/navigator.dart
/// If a [Route] is deemed restorable, the [Navigator] will set its
/// [Route.restorationScopeId] to a non-null value. Routes can use that ID to
/// store and restore their own state. As an example, the [ModalRoute] will
/// use this ID to create a [RestorationScope] for its content widgets.
```

## 相关链接

- [Navigator 构造函数与属性](navigator.dart_Navigator_2_构造函数与属性.md)
- [Navigator 命名路由方法](navigator.dart_Navigator_3_命名路由方法.md)
- [Navigator 直接路由操作](navigator.dart_Navigator_4_直接路由操作.md)
