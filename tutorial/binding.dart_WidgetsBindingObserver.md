# binding.dart：`WidgetsBindingObserver`（56-389）讲解

这段代码定义了 Widgets 层的“观察者”接口：任何对象只要实现（或混入） `WidgetsBindingObserver`，并通过 `WidgetsBinding.instance.addObserver` 注册，就能在系统环境变化时收到回调，例如路由请求、窗口尺寸变化、生命周期变化、低内存等。

```dart 56:88:packages/flutter/lib/src/widgets/binding.dart
/// Interface for classes that register with the Widgets layer binding.
///
/// This can be used by any class, not just widgets. It provides an interface
/// which is used by [WidgetsBinding.addObserver] and
/// [WidgetsBinding.removeObserver] to notify objects of changes in the
/// environment, such as changes to the device metrics or accessibility
/// settings. It is used to implement features such as [MediaQuery].
///
/// This class can be extended directly, or mixed in, to get default behaviors
/// for all of the handlers. Alternatively it can be used with the
/// `implements` keyword, in which case all the handlers must be implemented
/// (and the analyzer will list those that have been omitted).
abstract mixin class WidgetsBindingObserver {
  /// Called when the system tells the app to pop the current route, such as
```

## 这段接口解决了什么问题

- **统一入口**：把来自 `dart:ui`（`PlatformDispatcher`）和 `SystemChannels` 的各种“系统事件”，统一抽象成可覆写的 Dart 方法。
- **解耦**：Widgets 层的核心（`WidgetsBinding`）只负责收集事件并广播给观察者；业务组件（例如 `State`、路由、过渡动画等）只需要实现自己关心的回调。
- **默认安全**：大多数方法都有默认实现（空实现或返回 `false`），使“只覆写关心的部分”成为常态，同时保持二进制兼容与 API 演进空间。

## 为什么是 `abstract mixin class`

`abstract mixin class WidgetsBindingObserver` 同时具备“mixin”与“class”的特性：

- **作为 mixin 使用**：`class MyState extends State<Foo> with WidgetsBindingObserver { ... }`，可以直接继承到默认实现，因此只覆写少数方法即可。
- **作为基类继承**：也可以 `extends WidgetsBindingObserver` （仍然是抽象的）。
- **作为接口实现**：若使用 `implements WidgetsBindingObserver`，即使它有默认方法体，`implements` 也要求你显式实现所有成员（注释里也明确提示 analyzer 会列出遗漏项）。这能强制“完全自定义行为”，但通常会更啰嗦。

## 注册与反注册：避免泄漏

这类观察者通常绑定生命周期注册与释放：

- **注册**：在 `initState` 中调用 `WidgetsBinding.instance.addObserver(this)`。
- **反注册**：在 `dispose` 中调用 `WidgetsBinding.instance.removeObserver(this)`，避免对象已经销毁但仍被回调导致泄漏或异常。

（你提供的片段里已经在文档示例中展示了这一模式。）

## 导航与系统返回：`didPopRoute` 与预测式返回

### `didPopRoute`：系统请求"弹出当前路由"

```dart 85:123:packages/flutter/lib/src/widgets/binding.dart
abstract mixin class WidgetsBindingObserver {
  /// Called when the system tells the app to pop the current route, such as
  /// after a system back button press or back gesture.
  ///
  /// Observers are notified in registration order until one returns
  /// true. If none return true, the application quits.
  ///
  /// Observers are expected to return true if they were able to
  /// handle the notification, for example by closing an active dialog
  /// box, and false otherwise. The [WidgetsApp] widget uses this
  /// mechanism to notify the [Navigator] widget that it should pop
  /// its current route if possible.
  ///
  /// This method exposes the `popRoute` notification from
  /// [SystemChannels.navigation].
  ///
  /// {@macro flutter.widgets.AndroidPredictiveBack}
  Future<bool> didPopRoute() => Future<bool>.value(false);

  /// Called at the start of a predictive back gesture.
  ///
  /// Observers are notified in registration order until one returns true or all
```

要点：

- **按注册顺序通知**：`WidgetsBinding` 会按 `addObserver` 的先后顺序调用。
- **返回值是“是否已处理”**：只要有观察者返回 `true`，后续观察者就不会再收到这次 `didPopRoute`；如果都返回 `false`，默认行为是“应用退出”（对 `WidgetsApp` 来说通常意味着无法再 pop 任何路由）。
- **默认实现**：返回 `false`，表示“我不处理”。

### 预测式返回（Android）：`handleStartBackGesture` / `handleUpdateBackGestureProgress` / `handleCommitBackGesture` / `handleCancelBackGesture`

```dart 104:153:packages/flutter/lib/src/widgets/binding.dart
  /// Called at the start of a predictive back gesture.
  ///
  /// Observers are notified in registration order until one returns true or all
  /// observers have been notified. If an observer returns true then that
  /// observer, and only that observer, will be notified of subsequent events in
  /// this same gesture (for example [handleUpdateBackGestureProgress], etc.).
  ///
  /// Observers are expected to return true if they were able to handle the
  /// notification, for example by starting a predictive back animation, and
  /// false otherwise. [PredictiveBackPageTransitionsBuilder] uses this
  /// mechanism to listen for predictive back gestures.
  ///
  /// If all observers indicate they are not handling this back gesture by
  /// returning false, then a navigation pop will result when
  /// [handleCommitBackGesture] is called, as in a non-predictive system back
  /// gesture.
  ///
  /// Currently, this is only used on Android devices that support the
  /// predictive back feature.
  bool handleStartBackGesture(PredictiveBackEvent backEvent) => false;

  /// Called when a predictive back gesture moves.
  ///
  /// The observer which was notified of this gesture's [handleStartBackGesture]
  /// is the same observer notified for this.
  ///
  /// Currently, this is only used on Android devices that support the
  /// predictive back feature.
  void handleUpdateBackGestureProgress(PredictiveBackEvent backEvent) {}

  /// Called when a predictive back gesture is finished successfully, indicating
  /// that the current route should be popped.
  ///
  /// The observer which was notified of this gesture's [handleStartBackGesture]
  /// is the same observer notified for this. If there is none, then a
  /// navigation pop will result, as in a non-predictive system back gesture.
  ///
  /// Currently, this is only used on Android devices that support the
  /// predictive back feature.
  void handleCommitBackGesture() {}

  /// Called when a predictive back gesture is canceled, indicating that no
  /// navigation should occur.
  ///
  /// The observer which was notified of this gesture's [handleStartBackGesture]
  /// is the same observer notified for this.
  ///
  /// Currently, this is only used on Android devices that support the
  /// predictive back feature.
  void handleCancelBackGesture() {}
```

要点：

- **“谁接管，谁独占后续事件”**：`handleStartBackGesture` 返回 `true` 的那个观察者，会独占本次手势的后续回调（进度、提交、取消）。
- **没观察者接管时的默认行为**：如果所有观察者都返回 `false`，`handleCommitBackGesture` 触发时会走“像普通返回一样的 pop”。
- **用途**：给路由过渡（例如预测式返回动画）提供精细的手势驱动入口。

## 推入新路由：`didPushRoute`（已废弃）与 `didPushRouteInformation`

这段代码展示了 API 演进的典型做法：保留旧接口但标记废弃，并在新接口中提供“向后兼容的默认实现”，以便现有观察者不必立刻改动。

```dart 155:195:packages/flutter/lib/src/widgets/binding.dart
  /// Called when the host tells the application to push a new route onto the
  /// navigator.
  ///
  /// Observers are expected to return true if they were able to
  /// handle the notification. Observers are notified in registration
  /// order until one returns true.
  ///
  /// This method exposes the `pushRoute` notification from
  /// [SystemChannels.navigation].
  @Deprecated(
    'Use didPushRouteInformation instead. '
    'This feature was deprecated after v3.8.0-14.0.pre.',
  )
  Future<bool> didPushRoute(String route) => Future<bool>.value(false);

  /// Called when the host tells the application to push a new
  /// [RouteInformation] and a restoration state onto the router.
  ///
  /// Observers are expected to return true if they were able to
  /// handle the notification. Observers are notified in registration
  /// order until one returns true.
  ///
  /// This method exposes the `pushRouteInformation` notification from
  /// [SystemChannels.navigation].
  ///
  /// The default implementation is to call the [didPushRoute] directly with the
  /// string constructed from [RouteInformation.uri]'s path and query parameters.
  // TODO(chunhtai): remove the default implementation once `didPushRoute` is
  // removed.
  Future<bool> didPushRouteInformation(RouteInformation routeInformation) {
    final Uri uri = routeInformation.uri;
    return didPushRoute(
      Uri.decodeComponent(
        Uri(
          path: uri.path.isEmpty ? '/' : uri.path,
          queryParameters: uri.queryParametersAll.isEmpty ? null : uri.queryParametersAll,
          fragment: uri.fragment.isEmpty ? null : uri.fragment,
        ).toString(),
      ),
    );
  }
```

要点：

- **和 `didPopRoute` 一样的处理模型**：按注册顺序通知，直到有人返回 `true`。
- **为何引入 `RouteInformation`**：相比纯 `String route`，它能携带更结构化的 URI 信息（路径、查询参数、片段）以及与 Router/恢复（restoration）更一致的语义。
- **默认实现的细节**：
  - `uri.path` 为空时兜底为 `/`，避免构造出空路径。
  - 若没有查询参数或 fragment，则传 `null`，避免生成多余的 `?` 或 `#`。
  - `Uri.decodeComponent(...)` 是为了把 URI 字符串解码成更可读的路由字符串再交给旧接口处理。

## 视图与系统环境变化：从哪里来、什么时候触发

这些方法大多是“把底层通知暴露成 Dart 回调”，默认实现为空（或返回默认值），你按需覆写即可：

```dart 197:388:packages/flutter/lib/src/widgets/binding.dart
  /// Called when the application's dimensions change. For example,
  /// when a phone is rotated.
  ///
  /// This method exposes notifications from
  /// [dart:ui.PlatformDispatcher.onMetricsChanged].
  void didChangeMetrics() {}

  /// Called when the platform's text scale factor changes.
  ///
  /// This method exposes notifications from
  /// [dart:ui.PlatformDispatcher.onTextScaleFactorChanged].
  void didChangeTextScaleFactor() {}

  /// Called when the platform brightness changes.
  ///
  /// This method exposes notifications from
  /// [dart:ui.PlatformDispatcher.onPlatformBrightnessChanged].
  void didChangePlatformBrightness() {}

  /// Called when the system tells the app that the user's locale has
  /// changed.
  ///
  /// This method exposes notifications from
  /// [dart:ui.PlatformDispatcher.onLocaleChanged].
  void didChangeLocales(List<Locale>? locales) {}

  /// Called when the system puts the app in the background or returns
  /// the app to the foreground.
  ///
  /// This method exposes notifications from [SystemChannels.lifecycle].
  void didChangeAppLifecycleState(AppLifecycleState state) {}

  /// Called whenever the [PlatformDispatcher] receives a notification that the
  /// focus state on a view has changed.
  void didChangeViewFocus(ViewFocusEvent event) {}

  /// Called when a request is received from the system to exit the application.
  Future<AppExitResponse> didRequestAppExit() async {
    return AppExitResponse.exit;
  }

  /// Called when the system is running low on memory.
  ///
  /// This method exposes the `memoryPressure` notification from
  /// [SystemChannels.system].
  void didHaveMemoryPressure() {}

  /// Called when the system changes the set of currently active accessibility
  /// features.
  ///
  /// This method exposes notifications from
  /// [dart:ui.PlatformDispatcher.onAccessibilityFeaturesChanged].
  void didChangeAccessibilityFeatures() {}
}
```

快速对照表（按你覆写时最关心的维度组织）：

- **窗口/视图几何与渲染相关**：`didChangeMetrics`（屏幕旋转、分屏、窗口缩放等）。
- **可读性与无障碍相关**：`didChangeTextScaleFactor`、`didChangePlatformBrightness`、`didChangeAccessibilityFeatures`。
- **国际化相关**：`didChangeLocales`。
- **生命周期相关**：`didChangeAppLifecycleState`（前后台切换等）。注释中也提示了替代 API：`AppLifecycleListener`。
- **多视图焦点相关**：`didChangeViewFocus`（适用于多 `FlutterView` 的场景，`event` 里包含 view ID）。
- **系统资源压力相关**：`didHaveMemoryPressure`（低内存信号）。
- **退出相关**：`didRequestAppExit`（默认允许退出；如果某个观察者返回 `AppExitResponse.cancel`，会取消退出，并且“会询问所有观察者后再退出”的规则在注释里写得很清楚）。

## 实战建议

- **只覆写你需要的**：使用 `with WidgetsBindingObserver` 通常最省事。
- **谨慎返回 `true`**：在 `didPopRoute` / `didPushRouteInformation` 里返回 `true` 会“截断”后续观察者，因此仅在你确实完全处理了事件时返回 `true`。
- **始终成对注册/反注册**：把 `addObserver` 放进 `initState`，把 `removeObserver` 放进 `dispose`，这是最常见也最可靠的模式。
