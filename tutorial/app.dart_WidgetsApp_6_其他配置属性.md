# WidgetsApp 其他配置属性详解

## 概述

`WidgetsApp` 除了路由和本地化相关的属性外，还提供了许多其他配置属性，包括应用标题、颜色、文本样式、快捷键、操作、调试工具、状态恢复等。这些属性共同构成了应用的完整配置。

## title 与 onGenerateTitle

### title

```dart 836:847:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.title}
  /// A one-line description used by the device to identify the app for the user.
  ///
  /// On Android the titles appear above the task manager's app snapshots which are
  /// displayed when the user presses the "recent apps" button. On iOS this
  /// value cannot be used. `CFBundleDisplayName` from the app's `Info.plist` is
  /// referred to instead whenever present, `CFBundleName` otherwise.
  /// On the web it is used as the page title, which shows up in the browser's list of open tabs.
  ///
  /// To provide a localized title instead, use [onGenerateTitle].
  /// {@endtemplate}
  final String? title;
```

**功能**：设备用来向用户标识应用的单行描述。

**平台行为**：

- **Android**：标题显示在任务管理器的应用快照上方（用户按下"最近应用"按钮时显示）
- **iOS**：此值不能使用，使用 `Info.plist` 中的 `CFBundleDisplayName`（如果存在），否则使用 `CFBundleName`
- **Web**：用作页面标题，显示在浏览器的打开标签列表中

**本地化**：要提供本地化的标题，使用 `onGenerateTitle`。

### onGenerateTitle

```dart 849:862:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.onGenerateTitle}
  /// If non-null this callback function is called to produce the app's
  /// title string, otherwise [title] is used.
  ///
  /// The [onGenerateTitle] `context` parameter includes the [WidgetsApp]'s
  /// [Localizations] widget so that this callback can be used to produce a
  /// localized title.
  ///
  /// This callback function must not return null.
  ///
  /// The [onGenerateTitle] callback is called each time the [WidgetsApp]
  /// rebuilds.
  /// {@endtemplate}
  final GenerateAppTitle? onGenerateTitle;
```

**功能**：如果非 `null`，此回调函数用于生成应用的标题字符串，否则使用 `title`。

**特点**：

- **本地化支持**：`context` 参数包含 `WidgetsApp` 的 `Localizations` widget，因此可以生成本地化的标题
- **非空要求**：回调函数不能返回 `null`
- **重建时调用**：每次 `WidgetsApp` 重建时都会调用此回调

**使用场景**：当需要根据当前语言环境动态生成标题时。

## textStyle

```dart 864:865:packages/flutter/lib/src/widgets/app.dart
  /// The default text style for [Text] in the application.
  final TextStyle? textStyle;
```

**功能**：应用中 `Text` widget 的默认文本样式。

**用途**：为应用中的所有 `Text` widget 提供统一的默认样式基础。

**实现**：通过 `DefaultTextStyle` widget 实现，包装整个应用。

## color

```dart 867:874:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.color}
  /// The primary color to use for the application in the operating system
  /// interface.
  ///
  /// For example, on Android this is the color used for the application in the
  /// application switcher.
  /// {@endtemplate}
  final Color color;
```

**功能**：应用在操作系统界面中使用的主色。

**用途**：例如，在 Android 上，这是应用切换器中应用使用的颜色。

**要求**：此属性是必需的（`required`）。

## 调试相关属性

### showPerformanceOverlay

```dart 1027:1032:packages/flutter/lib/src/widgets/app.dart
  /// Turns on a performance overlay.
  ///
  /// See also:
  ///
  ///  * <https://flutter.dev/to/performance-overlay>
  final bool showPerformanceOverlay;
```

**功能**：打开性能覆盖层。

**用途**：显示应用的性能指标，包括帧率、GPU 使用情况等，用于性能分析和优化。

### showSemanticsDebugger

```dart 1034:1036:packages/flutter/lib/src/widgets/app.dart
  /// Turns on an overlay that shows the accessibility information
  /// reported by the framework.
  final bool showSemanticsDebugger;
```

**功能**：打开显示框架报告的无障碍信息的覆盖层。

**用途**：用于调试无障碍功能，可视化语义树信息。

### debugShowWidgetInspector

```dart 1038:1043:packages/flutter/lib/src/widgets/app.dart
  /// Turns on an overlay that enables inspecting the widget tree.
  ///
  /// The inspector is only available in debug mode as it depends on
  /// [RenderObject.debugDescribeChildren] which should not be called outside of
  /// debug mode.
  final bool debugShowWidgetInspector;
```

**功能**：打开启用检查 widget 树的覆盖层。

**限制**：检查器仅在调试模式下可用，因为它依赖于 `RenderObject.debugDescribeChildren`，不应在调试模式外调用。

**用途**：用于在设备上直接检查 widget 树，选择 widget 并查看其属性。

### Widget Inspector 相关构建器

#### exitWidgetSelectionButtonBuilder

```dart 1045:1050:packages/flutter/lib/src/widgets/app.dart
  /// Builds the widget the [WidgetInspector] uses to exit selection mode.
  ///
  /// This lets [MaterialApp] and [CupertinoApp] use an appropriately styled
  /// button for their design systems without requiring [WidgetInspector] to
  /// depend on the Material or Cupertino packages.
  final ExitWidgetSelectionButtonBuilder? exitWidgetSelectionButtonBuilder;
```

**功能**：构建 `WidgetInspector` 用于退出选择模式的 widget。

**设计目的**：允许 `MaterialApp` 和 `CupertinoApp` 使用适合其设计系统的按钮样式，而不需要 `WidgetInspector` 依赖 Material 或 Cupertino 包。

#### moveExitWidgetSelectionButtonBuilder

```dart 1052:1058:packages/flutter/lib/src/widgets/app.dart
  /// Builds the widget the [WidgetInspector] uses to move the exit selection
  /// mode button.
  ///
  /// This lets [MaterialApp] and [CupertinoApp] use an appropriately styled
  /// button for their design systems without requiring [WidgetInspector] to
  /// depend on the Material or Cupertino packages.
  final MoveExitWidgetSelectionButtonBuilder? moveExitWidgetSelectionButtonBuilder;
```

**功能**：构建 `WidgetInspector` 用于移动退出选择模式按钮的 widget。

**设计目的**：与 `exitWidgetSelectionButtonBuilder` 类似，用于自定义按钮位置。

#### tapBehaviorButtonBuilder

```dart 1060:1066:packages/flutter/lib/src/widgets/app.dart
  /// Builds the widget the [WidgetInspector] uses to change the default
  /// behavior when tapping on widgets in the app.
  ///
  /// This lets [MaterialApp] and [CupertinoApp] use an appropriately styled
  /// button for their design systems without requiring [WidgetInspector] to
  /// depend on the Material or Cupertino packages.
  final TapBehaviorButtonBuilder? tapBehaviorButtonBuilder;
```

**功能**：构建 `WidgetInspector` 用于更改点击 widget 时默认行为的 widget。

**设计目的**：允许自定义 Widget Inspector 的交互按钮样式。

### debugShowCheckedModeBanner

```dart 1068:1083:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.debugShowCheckedModeBanner}
  /// Turns on a little "DEBUG" banner in debug mode to indicate
  /// that the app is in debug mode. This is on by default (in
  /// debug mode), to turn it off, set the constructor argument to
  /// false. In release mode this has no effect.
  ///
  /// To get this banner in your application if you're not using
  /// WidgetsApp, include a [CheckedModeBanner] widget in your app.
  ///
  /// This banner is intended to deter people from complaining that your
  /// app is slow when it's in debug mode. In debug mode, Flutter
  /// enables a large number of expensive diagnostics to aid in
  /// development, and so performance in debug mode is not
  /// representative of what will happen in release mode.
  /// {@endtemplate}
  final bool debugShowCheckedModeBanner;
```

**功能**：在调试模式下打开一个小的 "DEBUG" 横幅，指示应用处于调试模式。

**默认值**：在调试模式下默认为 `true`，要关闭它，将构造函数参数设置为 `false`。

**设计目的**：防止人们在调试模式下抱怨应用速度慢。在调试模式下，Flutter 启用了大量昂贵的诊断功能以辅助开发，因此调试模式下的性能不能代表发布模式下的性能。

## shortcuts

```dart 1085:1129:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.shortcuts}
  /// The default map of keyboard shortcuts to intents for the application.
  ///
  /// By default, this is set to [WidgetsApp.defaultShortcuts].
  ///
  /// Passing this will not replace [DefaultTextEditingShortcuts]. These can be
  /// overridden by using a [Shortcuts] widget lower in the widget tree.
  /// {@endtemplate}
  ///
  /// {@tool snippet}
  /// This example shows how to add a single shortcut for
  /// [LogicalKeyboardKey.select] to the default shortcuts without needing to
  /// add your own [Shortcuts] widget.
  ///
  /// Alternatively, you could insert a [Shortcuts] widget with just the mapping
  /// you want to add between the [WidgetsApp] and its child and get the same
  /// effect.
  ///
  /// ```dart
  /// Widget build(BuildContext context) {
  ///   return WidgetsApp(
  ///     shortcuts: <ShortcutActivator, Intent>{
  ///       ... WidgetsApp.defaultShortcuts,
  ///       const SingleActivator(LogicalKeyboardKey.select): const ActivateIntent(),
  ///     },
  ///     color: const Color(0xFFFF0000),
  ///     builder: (BuildContext context, Widget? child) {
  ///       return const Placeholder();
  ///     },
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// {@template flutter.widgets.widgetsApp.shortcuts.seeAlso}
  /// See also:
  ///
  ///  * [SingleActivator], which defines shortcut key combination of a single
  ///    key and modifiers, such as "Delete" or "Control+C".
  ///  * The [Shortcuts] widget, which defines a keyboard mapping.
  ///  * The [Actions] widget, which defines the mapping from intent to action.
  ///  * The [Intent] and [Action] classes, which allow definition of new
  ///    actions.
  /// {@endtemplate}
  final Map<ShortcutActivator, Intent>? shortcuts;
```

**功能**：应用的键盘快捷键到意图的默认映射。

**默认值**：默认设置为 `WidgetsApp.defaultShortcuts`。

**注意事项**：

- 传递此参数不会替换 `DefaultTextEditingShortcuts`
- 可以通过在 widget 树中较低位置使用 `Shortcuts` widget 来覆盖这些快捷键

**使用场景**：定义应用的全局键盘快捷键，如 `Ctrl+C`、`Ctrl+V` 等。

## actions

```dart 1131:1183:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.actions}
  /// The default map of intent keys to actions for the application.
  ///
  /// By default, this is the output of [WidgetsApp.defaultActions], called with
  /// [defaultTargetPlatform]. Specifying [actions] for an app overrides the
  /// default, so if you wish to modify the default [actions], you can call
  /// [WidgetsApp.defaultActions] and modify the resulting map, passing it as
  /// the [actions] for this app. You may also add to the bindings, or override
  /// specific bindings for a widget subtree, by adding your own [Actions]
  /// widget.
  /// {@endtemplate}
  ///
  /// {@tool snippet}
  /// This example shows how to add a single action handling an
  /// [ActivateAction] to the default actions without needing to
  /// add your own [Actions] widget.
  ///
  /// Alternatively, you could insert an [Actions] widget with just the mapping
  /// you want to add between the [WidgetsApp] and its child and get the same
  /// effect.
  ///
  /// ```dart
  /// Widget build(BuildContext context) {
  ///   return WidgetsApp(
  ///     actions: <Type, Action<Intent>>{
  ///       ... WidgetsApp.defaultActions,
  ///       ActivateAction: CallbackAction<Intent>(
  ///         onInvoke: (Intent intent) {
  ///           // Do something here...
  ///           return null;
  ///         },
  ///       ),
  ///     },
  ///     color: const Color(0xFFFF0000),
  ///     builder: (BuildContext context, Widget? child) {
  ///       return const Placeholder();
  ///     },
  ///   );
  /// }
  /// ```
  /// {@end-tool}
  ///
  /// {@template flutter.widgets.widgetsApp.actions.seeAlso}
  /// See also:
  ///  * The [shortcuts] parameter, which defines the default set of shortcuts
  ///    for the application.
  ///  * The [Shortcuts] widget, which defines a keyboard mapping.
  ///  * The [Actions] widget, which defines the mapping from intent to action.
  ///  * The [Intent] and [Action] classes, which allow definition of new
  ///    actions.
  /// {@endtemplate}
  final Map<Type, Action<Intent>>? actions;
```

**功能**：应用的意图键到操作的默认映射。

**默认值**：默认是使用 `defaultTargetPlatform` 调用 `WidgetsApp.defaultActions` 的输出。

**覆盖行为**：为应用指定 `actions` 会覆盖默认值，因此如果要修改默认 `actions`，可以调用 `WidgetsApp.defaultActions` 并修改结果映射。

**扩展方式**：可以通过在 widget 树中添加自己的 `Actions` widget 来添加绑定或覆盖特定绑定。

**与 shortcuts 的关系**：`shortcuts` 定义快捷键到意图的映射，`actions` 定义意图到操作的映射。

## restorationScopeId

```dart 1185:1201:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.restorationScopeId}
  /// The identifier to use for state restoration of this app.
  ///
  /// Providing a restoration ID inserts a [RootRestorationScope] into the
  /// widget hierarchy, which enables state restoration for descendant widgets.
  ///
  /// Providing a restoration ID also enables the [Navigator] or [Router] built
  /// by the [WidgetsApp] to restore its state (i.e. to restore the history
  /// stack of active [Route]s). See the documentation on [Navigator] for more
  /// details around state restoration of [Route]s.
  ///
  /// See also:
  ///
  ///  * [RestorationManager], which explains how state restoration works in
  ///    Flutter.
  /// {@endtemplate}
  final String? restorationScopeId;
```

**功能**：用于此应用状态恢复的标识符。

**作用**：

1. **启用状态恢复**：提供恢复 ID 会在 widget 层次结构中插入 `RootRestorationScope`，为子 widget 启用状态恢复
2. **路由状态恢复**：提供恢复 ID 还使 `WidgetsApp` 构建的 `Navigator` 或 `Router` 能够恢复其状态（即恢复活动 `Route` 的历史堆栈）

**使用场景**：当应用被系统终止后重新启动时，可以恢复应用的状态，包括路由堆栈。

## useInheritedMediaQuery（已废弃）

```dart 1203:1214:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.useInheritedMediaQuery}
  /// Deprecated. This setting is now ignored.
  ///
  /// The widget never introduces its own [MediaQuery]; the [View] widget takes
  /// care of that.
  /// {@endtemplate}
  @Deprecated(
    'This setting is now ignored. '
    'WidgetsApp never introduces its own MediaQuery; the View widget takes care of that. '
    'This feature was deprecated after v3.7.0-29.0.pre.',
  )
  final bool useInheritedMediaQuery;
```

**状态**：已废弃，此设置现在被忽略。

**原因**：`WidgetsApp` 从不引入自己的 `MediaQuery`；`View` widget 会处理这个问题。

## 属性分类总结

### 应用信息

- `title`：应用标题
- `onGenerateTitle`：动态生成标题的回调
- `color`：应用主色

### 样式

- `textStyle`：默认文本样式

### 调试工具

- `showPerformanceOverlay`：性能覆盖层
- `showSemanticsDebugger`：无障碍调试器
- `debugShowWidgetInspector`：Widget 检查器
- `debugShowCheckedModeBanner`：调试模式横幅
- `exitWidgetSelectionButtonBuilder`：退出选择模式按钮构建器
- `moveExitWidgetSelectionButtonBuilder`：移动退出选择模式按钮构建器
- `tapBehaviorButtonBuilder`：点击行为按钮构建器

### 交互

- `shortcuts`：键盘快捷键映射
- `actions`：意图到操作的映射

### 状态管理

- `restorationScopeId`：状态恢复标识符

## 使用示例

### 基本配置

```dart
WidgetsApp(
  title: 'My App',
  color: Colors.blue,
  textStyle: TextStyle(fontSize: 16),
  home: MyHomePage(),
  pageRouteBuilder: (settings, builder) => MaterialPageRoute(
    settings: settings,
    builder: builder,
  ),
)
```

### 本地化标题

```dart
WidgetsApp(
  onGenerateTitle: (context) {
    return Localizations.of(context).appTitle;
  },
  color: Colors.blue,
  // ... 其他配置
)
```

### 自定义快捷键

```dart
WidgetsApp(
  shortcuts: <ShortcutActivator, Intent>{
    ... WidgetsApp.defaultShortcuts,
    SingleActivator(LogicalKeyboardKey.keyS, control: true): SaveIntent(),
  },
  actions: <Type, Action<Intent>>{
    ... WidgetsApp.defaultActions,
    SaveIntent: CallbackAction<SaveIntent>(
      onInvoke: (SaveIntent intent) {
        // 执行保存操作
        return null;
      },
    ),
  },
  color: Colors.blue,
  // ... 其他配置
)
```

### 启用状态恢复

```dart
WidgetsApp(
  restorationScopeId: 'app',
  home: MyHomePage(),
  pageRouteBuilder: (settings, builder) => MaterialPageRoute(
    settings: settings,
    builder: builder,
  ),
  color: Colors.blue,
)
```

## 总结

第六部分详细介绍了 `WidgetsApp` 的其他配置属性：

1. **应用信息**：`title`、`onGenerateTitle`、`color`
2. **样式**：`textStyle`
3. **调试工具**：性能覆盖层、无障碍调试器、Widget 检查器等
4. **交互**：`shortcuts`、`actions`
5. **状态管理**：`restorationScopeId`

这些属性为应用提供了完整的配置选项，从基本的外观到高级的调试和交互功能。合理配置这些属性可以大大提升应用的开发体验和用户体验。
