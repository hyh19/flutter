# `Route<T>` 类详解

## 概述

`Route<T>` 是 Flutter 导航系统中一个核心的抽象类，它定义了路由（Route）与导航器（Navigator）之间的抽象接口。路由是导航栈中的条目，大多数路由通过一个或多个 `OverlayEntry` 对象在导航器的 `Overlay` 中放置视觉元素。

```dart 138:161:packages/flutter/lib/src/widgets/navigator.dart
/// An abstraction for an entry managed by a [Navigator].
///
/// This class defines an abstract interface between the navigator and the
/// "routes" that are pushed on and popped off the navigator. Most routes have
/// visual affordances, which they place in the navigators [Overlay] using one
/// or more [OverlayEntry] objects.
///
/// See [Navigator] for more explanation of how to use a [Route] with
/// navigation, including code examples.
///
/// See [MaterialPageRoute] for a route that replaces the entire screen with a
/// platform-adaptive transition.
///
/// A route can belong to a page if the [settings] are a subclass of [Page]. A
/// page-based route, as opposed to a pageless route, is created from
/// [Page.createRoute] during [Navigator.pages] updates. The page associated
/// with this route may change during the lifetime of the route. If the
/// [Navigator] updates the page of this route, it calls [changedInternalState]
/// to notify the route that the page has been updated.
///
/// The type argument `T` is the route's return type, as used by
/// [currentResult], [popped], and [didPop]. The type `void` may be used if the
/// route does not return a value.
abstract class Route<T> extends _RoutePlaceholder {
```

### 关键特性

- **泛型类型 `T`**：表示路由的返回类型，用于 `currentResult`、`popped` 和 `didPop` 方法。如果路由不返回值，可以使用 `void`。
- **继承关系**：继承自 `_RoutePlaceholder`，这是一个占位符类，用于内部实现。
- **页面关联**：路由可以属于一个页面（Page），如果 `settings` 是 `Page` 的子类，则这是一个基于页面的路由。

## 构造函数与初始化

```dart 162:175:packages/flutter/lib/src/widgets/navigator.dart
  /// Initialize the [Route].
  ///
  /// If the [settings] are not provided, an empty [RouteSettings] object is
  /// used instead.
  ///
  /// {@template flutter.widgets.navigator.Route.requestFocus}
  /// If [requestFocus] is not provided, the value of [Navigator.requestFocus] is
  /// used instead.
  /// {@endtemplate}
  Route({RouteSettings? settings, bool? requestFocus})
    : _settings = settings ?? const RouteSettings(),
      _requestFocus = requestFocus {
    assert(debugMaybeDispatchCreated('widgets', 'Route<T>', this));
  }
```

构造函数接受两个可选参数：

- **`settings`**：路由设置，如果未提供则使用空的 `RouteSettings` 对象。
- **`requestFocus`**：是否在路由状态更新时请求焦点，如果未提供则使用 `Navigator.requestFocus` 的值。

## 核心属性

### 导航器关联

```dart 183:185:packages/flutter/lib/src/widgets/navigator.dart
  /// The navigator that the route is in, if any.
  NavigatorState? get navigator => _navigator;
  NavigatorState? _navigator;
```

`navigator` 属性返回包含此路由的 `NavigatorState`，如果路由不在任何导航器中则返回 `null`。

### 路由设置

```dart 187:201:packages/flutter/lib/src/widgets/navigator.dart
  /// The settings for this route.
  ///
  /// See [RouteSettings] for details.
  ///
  /// The settings can change during the route's lifetime. If the settings
  /// change, the route's overlays will be marked dirty (see
  /// [changedInternalState]).
  ///
  /// If the route is created from a [Page] in the [Navigator.pages] list, then
  /// this will be a [Page] subclass, and it will be updated each time its
  /// corresponding [Page] in the [Navigator.pages] has changed. Once the
  /// [Route] is removed from the history, this value stops updating (and
  /// remains with its last value).
  RouteSettings get settings => _settings;
  RouteSettings _settings;
```

`settings` 属性包含路由的配置信息，可以在路由的生命周期内改变。如果路由是从 `Navigator.pages` 列表中的 `Page` 创建的，则 `settings` 将是 `Page` 的子类。

### 页面基础路由判断

```dart 203:203:packages/flutter/lib/src/widgets/navigator.dart
  bool get _isPageBased => settings is Page<Object?>;
```

内部属性 `_isPageBased` 用于判断路由是否基于页面。

### 焦点请求

```dart 177:181:packages/flutter/lib/src/widgets/navigator.dart
  /// When the route state is updated, request focus if the current route is at the top.
  ///
  /// If not provided in the constructor, [Navigator.requestFocus] is used instead.
  bool get requestFocus => _requestFocus ?? navigator?.widget.requestFocus ?? false;
  final bool? _requestFocus;
```

`requestFocus` 属性决定当路由状态更新时，如果当前路由在顶部，是否请求焦点。

### 状态恢复

```dart 205:218:packages/flutter/lib/src/widgets/navigator.dart
  /// The restoration scope ID to be used for the [RestorationScope] surrounding
  /// this route.
  ///
  /// The restoration scope ID is null if restoration is currently disabled
  /// for this route.
  ///
  /// If the restoration scope ID changes (e.g. because restoration is enabled
  /// or disabled) during the life of the route, the [ValueListenable] notifies
  /// its listeners. As an example, the ID changes to null while the route is
  /// transitioning off screen, which triggers a notification on this field. At
  /// that point, the route is considered as no longer present for restoration
  /// purposes and its state will not be restored.
  ValueListenable<String?> get restorationScopeId => _restorationScopeId;
  final ValueNotifier<String?> _restorationScopeId = ValueNotifier<String?>(null);
```

`restorationScopeId` 用于状态恢复，当路由在屏幕外过渡时，ID 会变为 `null`，表示路由不再参与状态恢复。

### Overlay 条目

```dart 234:244:packages/flutter/lib/src/widgets/navigator.dart
  /// The overlay entries of this route.
  ///
  /// These are typically populated by [install]. The [Navigator] is in charge
  /// of adding them to and removing them from the [Overlay].
  ///
  /// There must be at least one entry in this list after [install] has been
  /// invoked.
  ///
  /// The [Navigator] will take care of keeping the entries together if the
  /// route is moved in the history.
  List<OverlayEntry> get overlayEntries => const <OverlayEntry>[];
```

`overlayEntries` 返回路由的 Overlay 条目列表，默认返回空列表。子类需要在 `install` 方法中填充此列表。

## 生命周期方法

### 安装（Install）

```dart 246:254:packages/flutter/lib/src/widgets/navigator.dart
  /// Called when the route is inserted into the navigator.
  ///
  /// Uses this to populate [overlayEntries]. There must be at least one entry in
  /// this list after [install] has been invoked. The [Navigator] will be in charge
  /// to add them to the [Overlay] or remove them from it by calling
  /// [OverlayEntry.remove].
  @protected
  @mustCallSuper
  void install() {}
```

`install` 方法在路由被插入到导航器时调用，用于填充 `overlayEntries` 列表。子类必须重写此方法并确保至少有一个条目。

### 推送（Push）

```dart 256:273:packages/flutter/lib/src/widgets/navigator.dart
  /// Called after [install] when the route is pushed onto the navigator.
  ///
  /// The returned value resolves when the push transition is complete.
  ///
  /// The [didAdd] method will be called instead of [didPush] when the route
  /// immediately appears on screen without any push transition.
  ///
  /// The [didChangeNext] and [didChangePrevious] methods are typically called
  /// immediately after this method is called.
  @protected
  @mustCallSuper
  TickerFuture didPush() {
    return TickerFuture.complete()..then<void>((void _) {
      if (requestFocus) {
        navigator!.focusNode.enclosingScope?.requestFocus();
      }
    });
  }
```

`didPush` 方法在路由被推送到导航器后调用，返回的 `TickerFuture` 在推送过渡完成时解析。如果 `requestFocus` 为 `true`，会请求焦点。

### 添加（Add）

```dart 275:311:packages/flutter/lib/src/widgets/navigator.dart
  /// Called after [install] when the route is added to the navigator.
  ///
  /// This method is called instead of [didPush] when the route immediately
  /// appears on screen without any push transition.
  ///
  /// The [didChangeNext] and [didChangePrevious] methods are typically called
  /// immediately after this method is called.
  @protected
  @mustCallSuper
  void didAdd() {
    if (requestFocus) {
      // This TickerFuture serves two purposes. First, we want to make sure that
      // animations triggered by other operations will finish before focusing
      // the navigator. Second, navigator.focusNode might acquire more focused
      // children in Route.install asynchronously. This TickerFuture will wait
      // for it to finish first.
      //
      // The later case can be found when subclasses manage their own focus scopes.
      // For example, ModalRoute creates a focus scope in its overlay entries. The
      // focused child can only be attached to navigator after initState which
      // will be guarded by the asynchronous gap.
      TickerFuture.complete().then<void>((void _) {
        // The route can be disposed before the ticker future completes. This can
        // happen when the navigator is under a TabView that warps from one tab to
        // another, non-adjacent tab, with an animation. The TabView reorders its
        // children before and after the warping completes, and that causes its
        // children to be built and disposed within the same frame. If one of its
        // children contains a navigator, the routes in that navigator are also
        // added and disposed within that frame.
        //
        // Since the reference to the navigator will be set to null after it is
        // disposed, we have to do a null-safe operation in case that happens
        // within the same frame when it is added.
        navigator?.focusNode.enclosingScope?.requestFocus();
      });
    }
  }
```

`didAdd` 方法在路由立即出现在屏幕上（没有推送过渡）时调用，而不是 `didPush`。它使用 `TickerFuture` 来确保动画完成后再请求焦点，并处理路由可能在同一帧内被销毁的情况。

### 替换（Replace）

```dart 313:319:packages/flutter/lib/src/widgets/navigator.dart
  /// Called after [install] when the route replaced another in the navigator.
  ///
  /// The [didChangeNext] and [didChangePrevious] methods are typically called
  /// immediately after this method is called.
  @protected
  @mustCallSuper
  void didReplace(Route<dynamic>? oldRoute) {}
```

`didReplace` 方法在路由替换导航器中的另一个路由后调用。

## 弹出（Pop）相关方法

### 弹出处置（Pop Disposition）

```dart 354:387:packages/flutter/lib/src/widgets/navigator.dart
  /// Returns whether calling [Navigator.maybePop] when this [Route] is current
  /// ([isCurrent]) should do anything.
  ///
  /// [Navigator.maybePop] is usually used instead of [Navigator.pop] to handle
  /// the system back button, when it hasn't been disabled via
  /// [SystemNavigator.setFrameworkHandlesBack].
  ///
  /// By default, if a [Route] is the first route in the history (i.e., if
  /// [isFirst]), it reports that pops should be bubbled
  /// ([RoutePopDisposition.bubble]). This behavior prevents the user from
  /// popping the first route off the history and being stranded at a blank
  /// screen; instead, the larger scope is popped (e.g. the application quits,
  /// so that the user returns to the previous application).
  ///
  /// In other cases, the default behavior is to accept the pop
  /// ([RoutePopDisposition.pop]).
  ///
  /// The third possible value is [RoutePopDisposition.doNotPop], which causes
  /// the pop request to be ignored entirely.
  ///
  /// See also:
  ///
  ///  * [Form], which provides a [Form.canPop] boolean that is similar.
  ///  * [PopScope], a widget that provides a way to intercept the back button.
  ///  * [Page.canPop], a way for [Page] to affect this property.
  RoutePopDisposition get popDisposition {
    if (_isPageBased) {
      final Page<Object?> page = settings as Page<Object?>;
      if (!page.canPop) {
        return RoutePopDisposition.doNotPop;
      }
    }
    return isFirst ? RoutePopDisposition.bubble : RoutePopDisposition.pop;
  }
```

`popDisposition` 属性决定当调用 `Navigator.maybePop` 时应该如何处理：

- **`RoutePopDisposition.pop`**：弹出路由（默认行为，除非是第一个路由）
- **`RoutePopDisposition.doNotPop`**：忽略弹出请求（如果基于页面的路由的 `canPop` 为 `false`）
- **`RoutePopDisposition.bubble`**：将弹出请求委托给下一级导航（第一个路由的默认行为）

### 弹出回调

```dart 400:413:packages/flutter/lib/src/widgets/navigator.dart
  /// {@template flutter.widgets.navigator.onPopInvokedWithResult}
  /// Called after a route pop was handled.
  ///
  /// Even when the pop is canceled, for example by a [PopScope] widget, this
  /// will still be called. The `didPop` parameter indicates whether or not the
  /// back navigation actually happened successfully.
  /// {@endtemplate}
  @mustCallSuper
  void onPopInvokedWithResult(bool didPop, T? result) {
    if (_isPageBased) {
      final Page<T> page = settings as Page<T>;
      page.onPopInvoked(didPop, result);
    }
  }
```

`onPopInvokedWithResult` 方法在路由弹出被处理后调用，即使弹出被取消（例如通过 `PopScope` widget）也会调用。`didPop` 参数表示返回导航是否实际成功发生。

### 处理弹出

```dart 435:458:packages/flutter/lib/src/widgets/navigator.dart
  /// A request was made to pop this route. If the route can handle it
  /// internally (e.g. because it has its own stack of internal state) then
  /// return false, otherwise return true (by returning the value of calling
  /// `super.didPop`). Returning false will prevent the default behavior of
  /// [NavigatorState.pop].
  ///
  /// When this function returns true, the navigator removes this route from
  /// the history but does not yet call [dispose]. Instead, it is the route's
  /// responsibility to call [NavigatorState.finalizeRoute], which will in turn
  /// call [dispose] on the route. This sequence lets the route perform an
  /// exit animation (or some other visual effect) after being popped but prior
  /// to being disposed.
  ///
  /// This method should call [didComplete] to resolve the [popped] future (and
  /// this is all that the default implementation does); routes should not wait
  /// for their exit animation to complete before doing so.
  ///
  /// See [popped], [didComplete], and [currentResult] for a discussion of the
  /// `result` argument.
  @mustCallSuper
  bool didPop(T? result) {
    didComplete(result);
    return true;
  }
```

`didPop` 方法处理弹出请求：

- 如果路由可以内部处理（例如有自己的内部状态栈），返回 `false` 以阻止默认行为
- 否则返回 `true`，导航器会从历史记录中移除路由，但不会立即调用 `dispose`
- 路由负责调用 `NavigatorState.finalizeRoute`，这会在退出动画后调用 `dispose`
- 默认实现会调用 `didComplete` 来完成 `popped` future

### 完成（Complete）

```dart 460:479:packages/flutter/lib/src/widgets/navigator.dart
  /// The route was popped or is otherwise being removed somewhat gracefully.
  ///
  /// This is called by [didPop] and in response to
  /// [NavigatorState.pushReplacement]. If [didPop] was not called, then the
  /// [NavigatorState.finalizeRoute] method must be called immediately, and no exit
  /// animation will run.
  ///
  /// The [popped] future is completed by this method. The `result` argument
  /// specifies the value that this future is completed with, unless it is null,
  /// in which case [currentResult] is used instead.
  ///
  /// This should be called before the pop animation, if any, takes place,
  /// though in some cases the animation may be driven by the user before the
  /// route is committed to being popped; this can in particular happen with the
  /// iOS-style back gesture. See [NavigatorState.didStartUserGesture].
  @protected
  @mustCallSuper
  void didComplete(T? result) {
    _popCompleter.complete(result ?? currentResult);
  }
```

`didComplete` 方法完成 `popped` future，使用提供的 `result` 参数，如果为 `null` 则使用 `currentResult`。

### 弹出结果相关属性

```dart 415:433:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether calling [didPop] would return false.
  bool get willHandlePopInternally => false;

  /// When this route is popped (see [Navigator.pop]) if the result isn't
  /// specified or if it's null, this value will be used instead.
  ///
  /// This fallback is implemented by [didComplete]. This value is used if the
  /// argument to that method is null.
  T? get currentResult => null;

  /// A future that completes when this route is popped off the navigator.
  ///
  /// The future completes with the value given to [Navigator.pop], if any, or
  /// else the value of [currentResult]. See [didComplete] for more discussion
  /// on this topic.
  Future<T?> get popped => _popCompleter.future;
  final Completer<T?> _popCompleter = Completer<T?>();

  final Completer<T?> _disposeCompleter = Completer<T?>();
```

- **`willHandlePopInternally`**：指示调用 `didPop` 是否会返回 `false`（默认返回 `false`）
- **`currentResult`**：当路由被弹出且未指定结果时的默认返回值（默认返回 `null`）
- **`popped`**：一个 Future，在路由从导航器弹出时完成，完成值来自 `Navigator.pop` 或 `currentResult`

## 路由关系变化回调

### 下一个路由弹出

```dart 481:488:packages/flutter/lib/src/widgets/navigator.dart
  /// The given route, which was above this one, has been popped off the
  /// navigator.
  ///
  /// This route is now the current route ([isCurrent] is now true), and there
  /// is no next route.
  @protected
  @mustCallSuper
  void didPopNext(Route<dynamic> nextRoute) {}
```

`didPopNext` 方法在位于此路由上方的路由被弹出后调用，此时此路由成为当前路由。

### 下一个路由变化

```dart 490:501:packages/flutter/lib/src/widgets/navigator.dart
  /// This route's next route has changed to the given new route.
  ///
  /// This is called on a route whenever the next route changes for any reason,
  /// so long as it is in the history, including when a route is first added to
  /// a [Navigator] (e.g. by [Navigator.push]), except for cases when
  /// [didPopNext] would be called.
  ///
  /// The `nextRoute` argument will be null if there's no new next route (i.e.
  /// if [isCurrent] is true).
  @protected
  @mustCallSuper
  void didChangeNext(Route<dynamic>? nextRoute) {}
```

`didChangeNext` 方法在路由的下一个路由发生变化时调用，只要路由在历史记录中。`nextRoute` 参数为 `null` 表示没有下一个路由（即路由是当前路由）。

### 上一个路由变化

```dart 503:514:packages/flutter/lib/src/widgets/navigator.dart
  /// This route's previous route has changed to the given new route.
  ///
  /// This is called on a route whenever the previous route changes for any
  /// reason, so long as it is in the history, except for immediately after the
  /// route itself has been pushed (in which case [didPush] or [didReplace] will
  /// be called instead).
  ///
  /// The `previousRoute` argument will be null if there's no previous route
  /// (i.e. if [isFirst] is true).
  @protected
  @mustCallSuper
  void didChangePrevious(Route<dynamic>? previousRoute) {}
```

`didChangePrevious` 方法在路由的上一个路由发生变化时调用，只要路由在历史记录中。`previousRoute` 参数为 `null` 表示没有上一个路由（即路由是第一个路由）。

## 状态变化通知

### 内部状态变化

```dart 516:529:packages/flutter/lib/src/widgets/navigator.dart
  /// Called whenever the internal state of the route has changed.
  ///
  /// This should be called whenever [willHandlePopInternally], [didPop],
  /// [ModalRoute.offstage], or other internal state of the route changes value.
  /// It is used by [ModalRoute], for example, to report the new information via
  /// its inherited widget to any children of the route.
  ///
  /// See also:
  ///
  ///  * [changedExternalState], which is called when the [Navigator] has
  ///    updated in some manner that might affect the routes.
  @protected
  @mustCallSuper
  void changedInternalState() {}
```

`changedInternalState` 方法在路由的内部状态发生变化时调用，例如 `willHandlePopInternally`、`didPop` 或 `ModalRoute.offstage` 等值改变时。

### 外部状态变化

```dart 531:556:packages/flutter/lib/src/widgets/navigator.dart
  /// Called whenever the [Navigator] has updated in some manner that might
  /// affect routes, to indicate that the route may wish to rebuild as well.
  ///
  /// This is called by the [Navigator] whenever the
  /// [NavigatorState]'s [State.widget] changes (as in [State.didUpdateWidget]),
  /// for example because the [MaterialApp] has been rebuilt. This
  /// ensures that routes that directly refer to the state of the
  /// widget that built the [MaterialApp] will be notified when that
  /// widget rebuilds, since it would otherwise be difficult to notify
  /// the routes that state they depend on may have changed.
  ///
  /// It is also called whenever the [Navigator]'s dependencies change
  /// (as in [State.didChangeDependencies]). This allows routes to use the
  /// [Navigator]'s context ([NavigatorState.context]), for example in
  /// [ModalRoute.barrierColor], and update accordingly.
  ///
  /// The [ModalRoute] subclass overrides this to force the barrier
  /// overlay to rebuild.
  ///
  /// See also:
  ///
  ///  * [changedInternalState], the equivalent but for changes to the internal
  ///    state of the route.
  @protected
  @mustCallSuper
  void changedExternalState() {}
```

`changedExternalState` 方法在 `Navigator` 以可能影响路由的方式更新时调用，例如 `NavigatorState` 的 widget 改变或依赖项改变时。

## 资源清理

### 销毁（Dispose）

```dart 558:576:packages/flutter/lib/src/widgets/navigator.dart
  /// Discards any resources used by the object.
  ///
  /// This method should not remove its [overlayEntries] from the [Overlay]. The
  /// object's owner is in charge of doing that.
  ///
  /// After this is called, the object is not in a usable state and should be
  /// discarded.
  ///
  /// This method should only be called by the object's owner; typically the
  /// [Navigator] owns a route and so will call this method when the route is
  /// removed, after which the route is no longer referenced by the navigator.
  @mustCallSuper
  @protected
  void dispose() {
    _navigator = null;
    _restorationScopeId.dispose();
    _disposeCompleter.complete();
    assert(debugMaybeDispatchDisposed(this));
  }
```

`dispose` 方法用于清理资源，将导航器引用设为 `null`，释放状态恢复 ID，并完成销毁完成器。注意：此方法不应从 `Overlay` 中移除 `overlayEntries`，这由对象的所有者（通常是 `Navigator`）负责。

## 路由状态查询

### 是否为当前路由

```dart 578:592:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether this route is the top-most route on the navigator.
  ///
  /// If this is true, then [isActive] is also true.
  bool get isCurrent {
    if (_navigator == null) {
      return false;
    }
    final _RouteEntry? currentRouteEntry = _navigator!._lastRouteEntryWhereOrNull(
      _RouteEntry.isPresentPredicate,
    );
    if (currentRouteEntry == null) {
      return false;
    }
    return currentRouteEntry.route == this;
  }
```

`isCurrent` 属性判断路由是否是导航器中最顶部的路由。如果为 `true`，则 `isActive` 也为 `true`。

### 是否为第一个路由

```dart 594:609:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether this route is the bottom-most active route on the navigator.
  ///
  /// If [isFirst] and [isCurrent] are both true then this is the only route on
  /// the navigator (and [isActive] will also be true).
  bool get isFirst {
    if (_navigator == null) {
      return false;
    }
    final _RouteEntry? currentRouteEntry = _navigator!._firstRouteEntryWhereOrNull(
      _RouteEntry.isPresentPredicate,
    );
    if (currentRouteEntry == null) {
      return false;
    }
    return currentRouteEntry.route == this;
  }
```

`isFirst` 属性判断路由是否是导航器中最底部的活动路由。如果 `isFirst` 和 `isCurrent` 都为 `true`，则这是导航器中的唯一路由。

### 是否有活动路由在下方

```dart 611:626:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether there is at least one active route underneath this route.
  @protected
  bool get hasActiveRouteBelow {
    if (_navigator == null) {
      return false;
    }
    for (final _RouteEntry entry in _navigator!._history) {
      if (entry.route == this) {
        return false;
    }
      if (_RouteEntry.isPresentPredicate(entry)) {
        return true;
      }
    }
    return false;
  }
```

`hasActiveRouteBelow` 属性判断在此路由下方是否至少有一个活动路由。

### 是否活动

```dart 628:640:packages/flutter/lib/src/widgets/navigator.dart
  /// Whether this route is on the navigator.
  ///
  /// If the route is not only active, but also the current route (the top-most
  /// route), then [isCurrent] will also be true. If it is the first route (the
  /// bottom-most route), then [isFirst] will also be true.
  ///
  /// If a higher route is entirely opaque, then the route will be active but not
  /// rendered. It is even possible for the route to be active but for the stateful
  /// widgets within the route to not be instantiated. See [ModalRoute.maintainState].
  bool get isActive {
    return _navigator?._firstRouteEntryWhereOrNull(_RouteEntry.isRoutePredicate(this))?.isPresent ??
        false;
  }
```

`isActive` 属性判断路由是否在导航器上。如果路由不仅是活动的，还是当前路由（最顶部），则 `isCurrent` 也为 `true`。如果更高的路由完全不透明，则路由可能是活动的但不会被渲染。

## 内部辅助方法

### 更新设置

```dart 220:227:packages/flutter/lib/src/widgets/navigator.dart
  void _updateSettings(RouteSettings newSettings) {
    if (_settings != newSettings) {
      _settings = newSettings;
      if (_navigator != null) {
        changedInternalState();
      }
    }
  }
```

`_updateSettings` 方法用于更新路由设置，如果设置改变且路由在导航器中，则调用 `changedInternalState` 通知状态变化。

### 更新恢复 ID

```dart 229:232:packages/flutter/lib/src/widgets/navigator.dart
  // ignore: use_setters_to_change_properties, (setters can't be private)
  void _updateRestorationId(String? restorationId) {
    _restorationScopeId.value = restorationId;
  }
```

`_updateRestorationId` 方法用于更新状态恢复 ID。

## 已废弃的方法

### willPop

```dart 321:352:packages/flutter/lib/src/widgets/navigator.dart
  /// Returns whether calling [Navigator.maybePop] when this [Route] is current
  /// ([isCurrent]) should do anything.
  ///
  /// [Navigator.maybePop] is usually used instead of [Navigator.pop] to handle
  /// the system back button.
  ///
  /// By default, if a [Route] is the first route in the history (i.e., if
  /// [isFirst]), it reports that pops should be bubbled
  /// ([RoutePopDisposition.bubble]). This behavior prevents the user from
  /// popping the first route off the history and being stranded at a blank
  /// screen; instead, the larger scope is popped (e.g. the application quits,
  /// so that the user returns to the previous application).
  ///
  /// In other cases, the default behavior is to accept the pop
  /// ([RoutePopDisposition.pop]).
  ///
  /// The third possible value is [RoutePopDisposition.doNotPop], which causes
  /// the pop request to be ignored entirely.
  ///
  /// See also:
  ///
  ///  * [Form], which provides a [Form.onWillPop] callback that uses this
  ///    mechanism.
  ///  * [WillPopScope], another widget that provides a way to intercept the
  ///    back button.
  @Deprecated(
    'Use popDisposition instead. '
    'This feature was deprecated after v3.12.0-1.0.pre.',
  )
  Future<RoutePopDisposition> willPop() async {
    return isFirst ? RoutePopDisposition.bubble : RoutePopDisposition.pop;
  }
```

`willPop` 方法已废弃，应使用 `popDisposition` 属性代替。

### onPopInvoked

```dart 389:398:packages/flutter/lib/src/widgets/navigator.dart
  /// Called after a route pop was handled.
  ///
  /// Even when the pop is canceled, for example by a [PopScope] widget, this
  /// will still be called. The `didPop` parameter indicates whether or not the
  /// back navigation actually happened successfully.
  @Deprecated(
    'Override onPopInvokedWithResult instead. '
    'This feature was deprecated after v3.22.0-12.0.pre.',
  )
  void onPopInvoked(bool didPop) {}
```

`onPopInvoked` 方法已废弃，应使用 `onPopInvokedWithResult` 方法代替。

## 总结

`Route<T>` 类是 Flutter 导航系统的核心抽象类，它定义了路由与导航器之间的完整接口。它提供了：

1. **生命周期管理**：从安装到销毁的完整生命周期回调
2. **状态查询**：判断路由在导航栈中的位置和状态
3. **弹出处理**：处理返回按钮和路由弹出的逻辑
4. **关系管理**：跟踪和响应相邻路由的变化
5. **状态恢复**：支持应用状态恢复机制

子类（如 `MaterialPageRoute`、`CupertinoPageRoute`）通过重写这些方法来实现具体的路由行为，包括过渡动画、视觉呈现等。
