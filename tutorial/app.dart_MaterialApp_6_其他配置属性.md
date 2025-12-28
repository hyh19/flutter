# MaterialApp 其他配置属性详解

## 概述

`MaterialApp` 除了路由和主题相关的属性外，还提供了许多其他配置属性，包括应用标题、颜色、本地化、快捷键、操作、调试工具、状态恢复、滚动行为等。这些属性中，一部分继承自 `WidgetsApp`，一部分是 `MaterialApp` 特有的。

## 继承自 WidgetsApp 的属性

### title 与 onGenerateTitle

#### title

```dart 389:392:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.title}
  ///
  /// This value is passed unmodified to [WidgetsApp.title].
  final String? title;
```

**功能**：设备用来向用户标识应用的单行描述，继承自 `WidgetsApp`。

**默认值**：在普通构造函数中默认为空字符串 `''`，在 `router` 构造函数中默认为 `null`。

**MaterialApp 的特殊处理**：此值原样传递给 `WidgetsApp.title`，没有修改。

#### onGenerateTitle

```dart 394:397:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.onGenerateTitle}
  ///
  /// This value is passed unmodified to [WidgetsApp.onGenerateTitle].
  final GenerateAppTitle? onGenerateTitle;
```

**功能**：如果非 `null`，此回调函数用于生成应用的标题字符串，否则使用 `title`，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：此值原样传递给 `WidgetsApp.onGenerateTitle`，没有修改。

### color

```dart 519:520:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.color}
  final Color? color;
```

**功能**：应用在操作系统界面中使用的主色，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：

在 `_buildWidgetApp` 中，`MaterialApp` 会计算 `materialColor`：

```dart 1069:1069:packages/flutter/lib/src/material/app.dart
    final Color materialColor = widget.color ?? widget.theme?.primaryColor ?? Colors.blue;
```

**逻辑**：

1. 如果提供了 `color`，使用 `color`
2. 否则，如果 `theme` 不为 `null`，使用 `theme.primaryColor`
3. 否则，使用 `Colors.blue`（默认主题的主色）

**设计考虑**：`color` 属性总是从浅色主题中提取，即使激活了深色模式也是如此。这样做是为了简化主题切换的技术细节，并且是可以接受的，因为 `color` 属性只在旧版 Android 系统上用于在 Android 的切换器 UI 中为应用栏着色。

### 本地化相关属性

#### locale

```dart 522:523:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.locale}
  final Locale? locale;
```

**功能**：应用的 `Localizations` widget 的初始语言环境，继承自 `WidgetsApp`。

#### localizationsDelegates

```dart 525:626:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.localizationsDelegates}
  ///
  /// Internationalized apps that require translations for one of the locales
  /// listed in [GlobalMaterialLocalizations] should specify this parameter
  /// and list the [supportedLocales] that the application can handle.
  ///
  /// ```dart
  /// // The GlobalMaterialLocalizations and GlobalWidgetsLocalizations
  /// // classes require the following import:
  /// // import 'package:flutter_localizations/flutter_localizations.dart';
  ///
  /// const MaterialApp(
  ///   localizationsDelegates: <LocalizationsDelegate<Object>>[
  ///     // ... app-specific localization delegate(s) here
  ///     GlobalMaterialLocalizations.delegate,
  ///     GlobalWidgetsLocalizations.delegate,
  ///   ],
  ///   supportedLocales: <Locale>[
  ///     Locale('en', 'US'), // English
  ///     Locale('he', 'IL'), // Hebrew
  ///     // ... other locales the app supports
  ///   ],
  ///   // ...
  /// )
  /// ```
  ///
  /// ## Adding localizations for a new locale
  ///
  /// The information that follows applies to the unusual case of an app
  /// adding translations for a language not already supported by
  /// [GlobalMaterialLocalizations].
  ///
  /// Delegates that produce [WidgetsLocalizations] and [MaterialLocalizations]
  /// are included automatically. Apps can provide their own versions of these
  /// localizations by creating implementations of
  /// [LocalizationsDelegate<WidgetsLocalizations>] or
  /// [LocalizationsDelegate<MaterialLocalizations>] whose load methods return
  /// custom versions of [WidgetsLocalizations] or [MaterialLocalizations].
  ///
  /// For example: to add support to [MaterialLocalizations] for a locale it
  /// doesn't already support, say `const Locale('foo', 'BR')`, one first
  /// creates a subclass of [MaterialLocalizations] that provides the
  /// translations:
  ///
  /// ```dart
  /// class FooLocalizations extends MaterialLocalizations {
  ///   FooLocalizations();
  ///   @override
  ///   String get okButtonLabel => 'foo';
  ///   // ...
  ///   // lots of other getters and methods to override!
  /// }
  /// ```
  ///
  /// One must then create a [LocalizationsDelegate] subclass that can provide
  /// an instance of the [MaterialLocalizations] subclass. In this case, this is
  /// essentially just a method that constructs a `FooLocalizations` object. A
  /// [SynchronousFuture] is used here because no asynchronous work takes place
  /// upon "loading" the localizations object.
  ///
  /// ```dart
  /// // continuing from previous example...
  /// class FooLocalizationsDelegate extends LocalizationsDelegate<MaterialLocalizations> {
  ///   const FooLocalizationsDelegate();
  ///   @override
  ///   bool isSupported(Locale locale) {
  ///     return locale == const Locale('foo', 'BR');
  ///   }
  ///   @override
  ///   Future<FooLocalizations> load(Locale locale) {
  ///     assert(locale == const Locale('foo', 'BR'));
  ///     return SynchronousFuture<FooLocalizations>(FooLocalizations());
  ///   }
  ///   @override
  ///   bool shouldReload(FooLocalizationsDelegate old) => false;
  /// }
  /// ```
  ///
  /// Constructing a [MaterialApp] with a `FooLocalizationsDelegate` overrides
  /// the automatically included delegate for [MaterialLocalizations] because
  /// only the first delegate of each [LocalizationsDelegate.type] is used and
  /// the automatically included delegates are added to the end of the app's
  /// [localizationsDelegates] list.
  ///
  /// ```dart
  /// // continuing from previous example...
  /// const MaterialApp(
  ///   localizationsDelegates: <LocalizationsDelegate<Object>>[
  ///     FooLocalizationsDelegate(),
  ///   ],
  ///   // ...
  /// )
  /// ```
  /// See also:
  ///
  ///  * [supportedLocales], which must be specified along with
  ///    [localizationsDelegates].
  ///  * [GlobalMaterialLocalizations], a [localizationsDelegates] value
  ///    which provides material localizations for many languages.
  ///  * The Flutter Internationalization Tutorial,
  ///    <https://flutter.dev/to/internationalization/>.
  final Iterable<LocalizationsDelegate<dynamic>>? localizationsDelegates;
```

**功能**：应用的 `Localizations` widget 的委托列表，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：

`MaterialApp` 会自动添加 Material 和 Cupertino 的本地化委托：

```dart 926:932:packages/flutter/lib/src/material/app.dart
  Iterable<LocalizationsDelegate<dynamic>> get _localizationsDelegates {
    return <LocalizationsDelegate<dynamic>>[
      if (widget.localizationsDelegates != null) ...widget.localizationsDelegates!,
      DefaultMaterialLocalizations.delegate,
      DefaultCupertinoLocalizations.delegate,
    ];
  }
```

**逻辑**：

1. 首先添加用户提供的 `localizationsDelegates`
2. 然后添加 `DefaultMaterialLocalizations.delegate`
3. 最后添加 `DefaultCupertinoLocalizations.delegate`

**设计目的**：只有每个 `LocalizationsDelegate.type` 的第一个委托会被使用，因此用户提供的委托可以覆盖自动包含的委托。

#### localeListResolutionCallback

```dart 628:631:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.localeListResolutionCallback}
  ///
  /// This callback is passed along to the [WidgetsApp] built by this widget.
  final LocaleListResolutionCallback? localeListResolutionCallback;
```

**功能**：负责在应用启动时和用户更改设备语言环境时选择应用语言环境的回调，继承自 `WidgetsApp`。

#### localeResolutionCallback

```dart 633:636:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.LocaleResolutionCallback}
  ///
  /// This callback is passed along to the [WidgetsApp] built by this widget.
  final LocaleResolutionCallback? localeResolutionCallback;
```

**功能**：与 `localeListResolutionCallback` 类似，但只考虑默认语言环境，继承自 `WidgetsApp`。

#### supportedLocales

```dart 638:650:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.supportedLocales}
  ///
  /// It is passed along unmodified to the [WidgetsApp] built by this widget.
  ///
  /// See also:
  ///
  ///  * [localizationsDelegates], which must be specified for localized
  ///    applications.
  ///  * [GlobalMaterialLocalizations], a [localizationsDelegates] value
  ///    which provides material localizations for many languages.
  ///  * The Flutter Internationalization Tutorial,
  ///    <https://flutter.dev/to/internationalization/>.
  final Iterable<Locale> supportedLocales;
```

**功能**：应用已本地化的语言环境列表，继承自 `WidgetsApp`。

### 调试相关属性

#### showPerformanceOverlay

```dart 652:657:packages/flutter/lib/src/material/app.dart
  /// Turns on a performance overlay.
  ///
  /// See also:
  ///
  ///  * <https://flutter.dev/to/performance-overlay>
  final bool showPerformanceOverlay;
```

**功能**：打开性能覆盖层，继承自 `WidgetsApp`。

#### checkerboardRasterCacheImages

```dart 659:660:packages/flutter/lib/src/material/app.dart
  /// Turns on checkerboarding of raster cache images.
  final bool checkerboardRasterCacheImages;
```

**功能**：打开光栅缓存图像的棋盘格显示，继承自 `WidgetsApp`。

#### checkerboardOffscreenLayers

```dart 662:663:packages/flutter/lib/src/material/app.dart
  /// Turns on checkerboarding of layers rendered to offscreen bitmaps.
  final bool checkerboardOffscreenLayers;
```

**功能**：打开渲染到离屏位图的层的棋盘格显示，继承自 `WidgetsApp`。

#### showSemanticsDebugger

```dart 665:667:packages/flutter/lib/src/material/app.dart
  /// Turns on an overlay that shows the accessibility information
  /// reported by the framework.
  final bool showSemanticsDebugger;
```

**功能**：打开显示框架报告的无障碍信息的覆盖层，继承自 `WidgetsApp`。

#### debugShowCheckedModeBanner

```dart 669:670:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.debugShowCheckedModeBanner}
  final bool debugShowCheckedModeBanner;
```

**功能**：在调试模式下打开一个小的 "DEBUG" 横幅，继承自 `WidgetsApp`。

### 交互相关属性

#### shortcuts

```dart 672:698:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.shortcuts}
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
  /// {@macro flutter.widgets.widgetsApp.shortcuts.seeAlso}
  final Map<ShortcutActivator, Intent>? shortcuts;
```

**功能**：应用的键盘快捷键到意图的默认映射，继承自 `WidgetsApp`。

#### actions

```dart 700:731:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.actions}
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
  /// {@macro flutter.widgets.widgetsApp.actions.seeAlso}
  final Map<Type, Action<Intent>>? actions;
```

**功能**：应用的意图键到操作的默认映射，继承自 `WidgetsApp`。

### 其他属性

#### builder

```dart 382:387:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.builder}
  ///
  /// Material specific features such as [showDialog] and [showMenu], and widgets
  /// such as [Tooltip], [PopupMenuButton], also require a [Navigator] to properly
  /// function.
  final TransitionBuilder? builder;
```

**功能**：用于在 `Navigator` 或 `Router` 上方插入 widget，或完全替换 `Navigator`/`Router` 的构建器，继承自 `WidgetsApp`。

**MaterialApp 的特殊处理**：`MaterialApp` 的 `builder` 会在 `_materialBuilder` 之后被调用，这意味着 `builder` 可以访问主题、`ScaffoldMessenger` 等 Material 功能。

#### restorationScopeId

```dart 733:734:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.restorationScopeId}
  final String? restorationScopeId;
```

**功能**：用于此应用状态恢复的标识符，继承自 `WidgetsApp`。

#### useInheritedMediaQuery（已废弃）

```dart 763:769:packages/flutter/lib/src/material/app.dart
  /// {@macro flutter.widgets.widgetsApp.useInheritedMediaQuery}
  @Deprecated(
    'This setting is now ignored. '
    'MaterialApp never introduces its own MediaQuery; the View widget takes care of that. '
    'This feature was deprecated after v3.7.0-29.0.pre.',
  )
  final bool useInheritedMediaQuery;
```

**状态**：已废弃，此设置现在被忽略。

## MaterialApp 特有的属性

### scrollBehavior

```dart 736:751:packages/flutter/lib/src/material/app.dart
  /// {@template flutter.material.materialApp.scrollBehavior}
  /// The default [ScrollBehavior] for the application.
  ///
  /// [ScrollBehavior]s describe how [Scrollable] widgets behave. Providing
  /// a [ScrollBehavior] can set the default [ScrollPhysics] across
  /// an application, and manage [Scrollable] decorations like [Scrollbar]s and
  /// [GlowingOverscrollIndicator]s.
  /// {@endtemplate}
  ///
  /// When null, defaults to [MaterialScrollBehavior].
  ///
  /// See also:
  ///
  ///  * [ScrollConfiguration], which controls how [Scrollable] widgets behave
  ///    in a subtree.
  final ScrollBehavior? scrollBehavior;
```

**功能**：应用的默认 `ScrollBehavior`。

**默认值**：如果为 `null`，默认为 `MaterialScrollBehavior`。

**用途**：

- 设置应用的默认 `ScrollPhysics`
- 管理 `Scrollable` 装饰，如 `Scrollbar` 和 `GlowingOverscrollIndicator`

**MaterialScrollBehavior 的特点**：

- 在 Android 和 Fuchsia 平台上应用 `GlowingOverscrollIndicator`
- 在桌面平台上，如果 `Scrollable` widget 垂直滚动，应用 `Scrollbar`
- 根据 `ThemeData.useMaterial3` 选择使用 `StretchingOverscrollIndicator` 或 `GlowingOverscrollIndicator`

### debugShowMaterialGrid

```dart 753:761:packages/flutter/lib/src/material/app.dart
  /// Turns on a [GridPaper] overlay that paints a baseline grid
  /// Material apps.
  ///
  /// Only available in debug mode.
  ///
  /// See also:
  ///
  ///  * <https://material.io/design/layout/spacing-methods.html>
  final bool debugShowMaterialGrid;
```

**功能**：打开一个 `GridPaper` 覆盖层，绘制 Material 应用的基线网格。

**限制**：仅在调试模式下可用。

**用途**：用于 Material Design 布局调试，帮助开发者遵循 Material Design 的间距规范。

**实现**：在 `build` 方法中，如果 `debugShowMaterialGrid` 为 `true`，会使用 `GridPaper` 包装结果：

```dart 1152:1159:packages/flutter/lib/src/material/app.dart
      if (widget.debugShowMaterialGrid) {
        result = GridPaper(
          color: const Color(0xE0F9BBE0),
          interval: 8.0,
          subdivisions: 1,
          child: result,
        );
      }
```

## 属性分类总结

### 应用信息

- `title`：应用标题
- `onGenerateTitle`：动态生成标题的回调
- `color`：应用主色（MaterialApp 会从主题中提取）

### 本地化

- `locale`：应用的语言环境
- `localizationsDelegates`：本地化资源委托列表（MaterialApp 会自动添加 Material 和 Cupertino 委托）
- `localeListResolutionCallback`：语言环境列表解析回调
- `localeResolutionCallback`：语言环境解析回调
- `supportedLocales`：支持的语言环境列表

### 调试工具

- `showPerformanceOverlay`：性能覆盖层
- `checkerboardRasterCacheImages`：光栅缓存图像棋盘格
- `checkerboardOffscreenLayers`：离屏层棋盘格
- `showSemanticsDebugger`：无障碍调试器
- `debugShowCheckedModeBanner`：调试模式横幅
- `debugShowMaterialGrid`：Material 网格调试工具（MaterialApp 特有）

### 交互

- `shortcuts`：键盘快捷键映射
- `actions`：意图到操作的映射

### 状态管理

- `restorationScopeId`：状态恢复标识符

### Material 特有

- `scrollBehavior`：默认滚动行为（MaterialApp 特有）
- `debugShowMaterialGrid`：Material 网格调试工具（MaterialApp 特有）

## 使用示例

### 基本配置

```dart
MaterialApp(
  title: 'My App',
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  locale: Locale('zh', 'CN'),
  supportedLocales: [
    Locale('en', 'US'),
    Locale('zh', 'CN'),
  ],
  home: MyHomePage(),
)
```

### 本地化配置

```dart
MaterialApp(
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    MyAppLocalizations.delegate,
  ],
  supportedLocales: [
    Locale('en', 'US'),
    Locale('zh', 'CN'),
  ],
  home: MyHomePage(),
)
```

### 自定义滚动行为

```dart
MaterialApp(
  scrollBehavior: MyCustomScrollBehavior(),
  home: MyHomePage(),
)
```

### 启用 Material 网格调试

```dart
MaterialApp(
  debugShowMaterialGrid: true, // 仅在调试模式下有效
  home: MyHomePage(),
)
```

## 总结

第六部分详细介绍了 `MaterialApp` 的其他配置属性：

1. **继承自 WidgetsApp 的属性**：包括应用信息、本地化、调试工具、交互、状态管理等

2. **MaterialApp 的特殊处理**：
   - `color`：从主题中提取主色
   - `localizationsDelegates`：自动添加 Material 和 Cupertino 委托

3. **MaterialApp 特有属性**：
   - `scrollBehavior`：Material 风格的默认滚动行为
   - `debugShowMaterialGrid`：Material 网格调试工具

这些属性为应用提供了完整的配置选项，从基本的外观到高级的调试和交互功能。合理配置这些属性可以大大提升应用的开发体验和用户体验。
