# MaterialApp 主题相关属性详解

## 概述

`MaterialApp` 提供了完整的主题系统，这是 `MaterialApp` 的核心特性之一。主题系统支持浅色主题、深色主题、高对比度主题，以及主题模式切换和主题动画。这些功能使得应用能够根据用户偏好和系统设置自动切换主题，并提供流畅的主题过渡动画。

## theme

```dart 399:416:packages/flutter/lib/src/material/app.dart
  /// Default visual properties, like colors fonts and shapes, for this app's
  /// material widgets.
  ///
  /// A second [darkTheme] [ThemeData] value, which is used to provide a dark
  /// version of the user interface can also be specified. [themeMode] will
  /// control which theme will be used if a [darkTheme] is provided.
  ///
  /// The default value of this property is the value of [ThemeData.light()].
  ///
  /// See also:
  ///
  ///  * [themeMode], which controls which theme to use.
  ///  * [MediaQueryData.platformBrightness], which indicates the platform's
  ///    desired brightness and is used to automatically toggle between [theme]
  ///    and [darkTheme] in [MaterialApp].
  ///  * [ThemeData.brightness], which indicates the [Brightness] of a theme's
  ///  colors.
  final ThemeData? theme;
```

**功能**：应用的默认视觉属性，包括颜色、字体和形状等，用于 Material widget。

**默认值**：默认为 `ThemeData.light()` 的值。

**用途**：

- 定义应用的浅色主题
- 当 `darkTheme` 为 `null` 时，也用作深色模式的回退主题
- 当 `themeMode` 为 `ThemeMode.light` 时，始终使用此主题

**与 themeMode 的关系**：`themeMode` 控制使用哪个主题。如果 `themeMode` 为 `ThemeMode.system`，系统会根据 `MediaQueryData.platformBrightness` 自动在 `theme` 和 `darkTheme` 之间切换。

## darkTheme

```dart 418:438:packages/flutter/lib/src/material/app.dart
  /// The [ThemeData] to use when a 'dark mode' is requested by the system.
  ///
  /// Some host platforms allow the users to select a system-wide 'dark mode',
  /// or the application may want to offer the user the ability to choose a
  /// dark theme just for this application. This is theme that will be used for
  /// such cases. [themeMode] will control which theme will be used.
  ///
  /// This theme should have a [ThemeData.brightness] set to [Brightness.dark].
  ///
  /// Uses [theme] instead when null. Defaults to the value of
  /// [ThemeData.light()] when both [darkTheme] and [theme] are null.
  ///
  /// See also:
  ///
  ///  * [themeMode], which controls which theme to use.
  ///  * [MediaQueryData.platformBrightness], which indicates the platform's
  ///    desired brightness and is used to automatically toggle between [theme]
  ///    and [darkTheme] in [MaterialApp].
  ///  * [ThemeData.brightness], which is typically set to the value of
  ///    [MediaQueryData.platformBrightness].
  final ThemeData? darkTheme;
```

**功能**：当系统请求"深色模式"时使用的 `ThemeData`。

**行为**：

- 如果为 `null`，使用 `theme` 代替
- 如果 `darkTheme` 和 `theme` 都为 `null`，默认使用 `ThemeData.light()` 的值
- 此主题应该将 `ThemeData.brightness` 设置为 `Brightness.dark`

**使用场景**：

- 系统级深色模式：某些平台允许用户选择系统级深色模式
- 应用级深色模式：应用可以允许用户仅为该应用选择深色主题

**与 themeMode 的关系**：`themeMode` 控制使用哪个主题。如果 `themeMode` 为 `ThemeMode.dark`，始终使用 `darkTheme`（如果为 `null`，则回退到 `theme`）。

## highContrastTheme

```dart 440:451:packages/flutter/lib/src/material/app.dart
  /// The [ThemeData] to use when 'high contrast' is requested by the system.
  ///
  /// Some host platforms (for example, iOS) allow the users to increase
  /// contrast through an accessibility setting.
  ///
  /// Uses [theme] instead when null.
  ///
  /// See also:
  ///
  ///  * [MediaQueryData.highContrast], which indicates the platform's
  ///    desire to increase contrast.
  final ThemeData? highContrastTheme;
```

**功能**：当系统请求"高对比度"时使用的 `ThemeData`。

**使用场景**：某些平台（例如 iOS）允许用户通过无障碍设置增加对比度。

**行为**：如果为 `null`，使用 `theme` 代替。

**检测方式**：通过 `MediaQueryData.highContrast` 检测平台是否请求高对比度。

## highContrastDarkTheme

```dart 453:467:packages/flutter/lib/src/material/app.dart
  /// The [ThemeData] to use when a 'dark mode' and 'high contrast' is requested
  /// by the system.
  ///
  /// Some host platforms (for example, iOS) allow the users to increase
  /// contrast through an accessibility setting.
  ///
  /// This theme should have a [ThemeData.brightness] set to [Brightness.dark].
  ///
  /// Uses [darkTheme] instead when null.
  ///
  /// See also:
  ///
  ///  * [MediaQueryData.highContrast], which indicates the platform's
  ///    desire to increase contrast.
  final ThemeData? highContrastDarkTheme;
```

**功能**：当系统同时请求"深色模式"和"高对比度"时使用的 `ThemeData`。

**使用场景**：某些平台（例如 iOS）允许用户通过无障碍设置增加对比度，同时用户可能选择了深色模式。

**行为**：

- 如果为 `null`，使用 `darkTheme` 代替
- 此主题应该将 `ThemeData.brightness` 设置为 `Brightness.dark`

**检测方式**：通过 `MediaQueryData.highContrast` 和 `MediaQueryData.platformBrightness` 检测平台是否同时请求高对比度和深色模式。

## themeMode

```dart 469:493:packages/flutter/lib/src/material/app.dart
  /// Determines which theme will be used by the application if both [theme]
  /// and [darkTheme] are provided.
  ///
  /// If set to [ThemeMode.system], the choice of which theme to use will
  /// be based on the user's system preferences. If the [MediaQuery.platformBrightnessOf]
  /// is [Brightness.light], [theme] will be used. If it is [Brightness.dark],
  /// [darkTheme] will be used (unless it is null, in which case [theme]
  /// will be used.
  ///
  /// If set to [ThemeMode.light] the [theme] will always be used,
  /// regardless of the user's system preference.
  ///
  /// If set to [ThemeMode.dark] the [darkTheme] will be used
  /// regardless of the user's system preference. If [darkTheme] is null
  /// then it will fallback to using [theme].
  ///
  /// The default value is [ThemeMode.system].
  ///
  /// See also:
  ///
  ///  * [theme], which is used when a light mode is selected.
  ///  * [darkTheme], which is used when a dark mode is selected.
  ///  * [ThemeData.brightness], which indicates to various parts of the
  ///    system what kind of theme is being used.
  final ThemeMode? themeMode;
```

**功能**：如果同时提供了 `theme` 和 `darkTheme`，确定应用使用哪个主题。

**默认值**：默认为 `ThemeMode.system`。

### ThemeMode 枚举值

```dart 56:67:packages/flutter/lib/src/material/app.dart
enum ThemeMode {
  /// Use either the light or dark theme based on what the user has selected in
  /// the system settings.
  system,

  /// Always use the light mode regardless of system preference.
  light,

  /// Always use the dark mode (if available) regardless of system preference.
  dark,
}
```

#### ThemeMode.system

- **行为**：根据用户的系统偏好选择主题
- **逻辑**：
  - 如果 `MediaQuery.platformBrightnessOf` 为 `Brightness.light`，使用 `theme`
  - 如果 `MediaQuery.platformBrightnessOf` 为 `Brightness.dark`，使用 `darkTheme`（如果为 `null`，则使用 `theme`）

#### ThemeMode.light

- **行为**：始终使用 `theme`，无论用户的系统偏好如何

#### ThemeMode.dark

- **行为**：始终使用 `darkTheme`，无论用户的系统偏好如何
- **回退**：如果 `darkTheme` 为 `null`，回退到使用 `theme`

## themeAnimationDuration

```dart 495:507:packages/flutter/lib/src/material/app.dart
  /// The duration of animated theme changes.
  ///
  /// When the theme changes (either by the [theme], [darkTheme] or [themeMode]
  /// parameters changing) it is animated to the new theme over time.
  /// The [themeAnimationDuration] determines how long this animation takes.
  ///
  /// To have the theme change immediately, you can set this to [Duration.zero].
  ///
  /// The default is [kThemeAnimationDuration].
  ///
  /// See also:
  ///   [themeAnimationCurve], which defines the curve used for the animation.
  final Duration themeAnimationDuration;
```

**功能**：主题动画变化的时长。

**默认值**：默认为 `kThemeAnimationDuration`。

**行为**：

- 当主题改变时（通过 `theme`、`darkTheme` 或 `themeMode` 参数改变），会在一段时间内动画过渡到新主题
- `themeAnimationDuration` 确定此动画的时长
- 要立即改变主题，可以将此设置为 `Duration.zero`

**与 themeAnimationCurve 的关系**：`themeAnimationCurve` 定义动画使用的曲线。

## themeAnimationCurve

```dart 509:517:packages/flutter/lib/src/material/app.dart
  /// The curve to apply when animating theme changes.
  ///
  /// The default is [Curves.linear].
  ///
  /// This is ignored if [themeAnimationDuration] is [Duration.zero].
  ///
  /// See also:
  ///   [themeAnimationDuration], which defines how long the animation is.
  final Curve themeAnimationCurve;
```

**功能**：主题变化动画时应用的曲线。

**默认值**：默认为 `Curves.linear`。

**行为**：

- 如果 `themeAnimationDuration` 为 `Duration.zero`，此参数会被忽略
- 用于控制主题过渡动画的缓动效果

**常用曲线**：

- `Curves.linear`：线性动画
- `Curves.easeInOut`：缓入缓出
- `Curves.fastOutSlowIn`：快速开始，缓慢结束（Material Design 推荐）

## themeAnimationStyle

```dart 771:791:packages/flutter/lib/src/material/app.dart
  /// Used to override the theme animation curve and duration.
  ///
  /// If [AnimationStyle.duration] is provided, it will be used to override
  /// the theme animation duration in the underlying [AnimatedTheme] widget.
  /// If it is null, then [themeAnimationDuration] will be used. Otherwise,
  /// defaults to 200ms.
  ///
  /// If [AnimationStyle.curve] is provided, it will be used to override
  /// the theme animation curve in the underlying [AnimatedTheme] widget.
  /// If it is null, then [themeAnimationCurve] will be used. Otherwise,
  /// defaults to [Curves.linear].
  ///
  /// To disable the theme animation, use [AnimationStyle.noAnimation].
  ///
  /// {@tool dartpad}
  /// This sample showcases how to override the theme animation curve and
  /// duration in the [MaterialApp] widget using [AnimationStyle].
  ///
  /// ** See code in examples/api/lib/material/app/app.0.dart **
  /// {@end-tool}
  final AnimationStyle? themeAnimationStyle;
```

**功能**：用于覆盖主题动画曲线和时长。

**优先级**：

1. **duration**：
   - 如果 `AnimationStyle.duration` 不为 `null`，使用它覆盖 `themeAnimationDuration`
   - 如果为 `null`，使用 `themeAnimationDuration`
   - 否则，默认为 200ms

2. **curve**：
   - 如果 `AnimationStyle.curve` 不为 `null`，使用它覆盖 `themeAnimationCurve`
   - 如果为 `null`，使用 `themeAnimationCurve`
   - 否则，默认为 `Curves.linear`

**禁用动画**：要禁用主题动画，使用 `AnimationStyle.noAnimation`。

## 主题选择逻辑

`MaterialApp` 的主题选择逻辑在 `_themeBuilder` 方法中实现：

```dart 986:1008:packages/flutter/lib/src/material/app.dart
  ThemeData _themeBuilder(BuildContext context) {
    ThemeData? theme;
    // Resolve which theme to use based on brightness and high contrast.
    final ThemeMode mode = widget.themeMode ?? ThemeMode.system;
    final Brightness platformBrightness = MediaQuery.platformBrightnessOf(context);
    final bool useDarkTheme =
        mode == ThemeMode.dark ||
        (mode == ThemeMode.system && platformBrightness == ui.Brightness.dark);
    final bool highContrast = MediaQuery.highContrastOf(context);
    if (useDarkTheme && highContrast && widget.highContrastDarkTheme != null) {
      theme = widget.highContrastDarkTheme;
    } else if (useDarkTheme && widget.darkTheme != null) {
      theme = widget.darkTheme;
    } else if (highContrast && widget.highContrastTheme != null) {
      theme = widget.highContrastTheme;
    }
    theme ??= widget.theme ?? ThemeData();
    SystemChrome.setSystemUIOverlayStyle(
      theme.brightness == Brightness.dark ? SystemUiOverlayStyle.light : SystemUiOverlayStyle.dark,
    );

    return theme;
  }
```

### 主题选择优先级

1. **高对比度深色主题**：如果 `useDarkTheme` 为 `true` 且 `highContrast` 为 `true` 且 `highContrastDarkTheme` 不为 `null`，使用 `highContrastDarkTheme`
2. **深色主题**：如果 `useDarkTheme` 为 `true` 且 `darkTheme` 不为 `null`，使用 `darkTheme`
3. **高对比度主题**：如果 `highContrast` 为 `true` 且 `highContrastTheme` 不为 `null`，使用 `highContrastTheme`
4. **默认主题**：否则，使用 `theme`，如果 `theme` 也为 `null`，使用 `ThemeData()`

### 系统 UI 样式更新

`_themeBuilder` 还会根据主题的亮度更新系统 UI 样式：

```dart
SystemChrome.setSystemUIOverlayStyle(
  theme.brightness == Brightness.dark ? SystemUiOverlayStyle.light : SystemUiOverlayStyle.dark,
);
```

这确保了状态栏和导航栏的颜色与主题匹配。

## 主题动画

`MaterialApp` 使用 `AnimatedTheme` 实现主题动画：

```dart 1047:1056:packages/flutter/lib/src/material/app.dart
    if (widget.themeAnimationStyle != AnimationStyle.noAnimation) {
      childWidget = AnimatedTheme(
        data: theme,
        duration: widget.themeAnimationStyle?.duration ?? widget.themeAnimationDuration,
        curve: widget.themeAnimationStyle?.curve ?? widget.themeAnimationCurve,
        child: childWidget,
      );
    } else {
      childWidget = Theme(data: theme, child: childWidget);
    }
```

**行为**：

- 如果 `themeAnimationStyle` 不是 `AnimationStyle.noAnimation`，使用 `AnimatedTheme` 实现动画过渡
- 否则，使用普通的 `Theme` widget，主题变化立即生效

## 使用示例

### 基本主题配置

```dart
MaterialApp(
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  themeMode: ThemeMode.system,
  home: MyHomePage(),
)
```

### 自定义主题

```dart
MaterialApp(
  theme: ThemeData(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
    useMaterial3: true,
  ),
  darkTheme: ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.blue,
      brightness: Brightness.dark,
    ),
    useMaterial3: true,
  ),
  themeMode: ThemeMode.system,
  home: MyHomePage(),
)
```

### 高对比度主题支持

```dart
MaterialApp(
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  highContrastTheme: ThemeData.light().copyWith(
    // 增加对比度的配置
  ),
  highContrastDarkTheme: ThemeData.dark().copyWith(
    // 增加对比度的配置
  ),
  themeMode: ThemeMode.system,
  home: MyHomePage(),
)
```

### 自定义主题动画

```dart
MaterialApp(
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  themeMode: ThemeMode.system,
  themeAnimationDuration: Duration(milliseconds: 500),
  themeAnimationCurve: Curves.easeInOut,
  home: MyHomePage(),
)
```

### 使用 AnimationStyle 覆盖动画

```dart
MaterialApp(
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  themeMode: ThemeMode.system,
  themeAnimationStyle: AnimationStyle(
    duration: Duration(milliseconds: 300),
    curve: Curves.fastOutSlowIn,
  ),
  home: MyHomePage(),
)
```

### 禁用主题动画

```dart
MaterialApp(
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  themeMode: ThemeMode.system,
  themeAnimationStyle: AnimationStyle.noAnimation,
  home: MyHomePage(),
)
```

## 主题属性之间的关系

### 依赖关系

- `darkTheme` 依赖于 `theme`（如果为 `null`，使用 `theme`）
- `highContrastTheme` 依赖于 `theme`（如果为 `null`，使用 `theme`）
- `highContrastDarkTheme` 依赖于 `darkTheme`（如果为 `null`，使用 `darkTheme`）
- `themeMode` 控制使用 `theme` 还是 `darkTheme`

### 优先级

1. **高对比度深色主题**（最高优先级）
2. **深色主题**
3. **高对比度主题**
4. **默认主题**（最低优先级）

## 总结

第五部分详细介绍了 `MaterialApp` 的完整主题系统：

1. **基础主题**：`theme`（浅色主题）和 `darkTheme`（深色主题）

2. **无障碍支持**：`highContrastTheme` 和 `highContrastDarkTheme`（高对比度主题）

3. **主题模式**：`themeMode`（system/light/dark）控制主题选择

4. **主题动画**：`themeAnimationDuration`、`themeAnimationCurve` 和 `themeAnimationStyle` 控制主题过渡动画

5. **主题选择逻辑**：根据主题模式、平台亮度和高对比度设置自动选择最合适的主题

6. **系统 UI 集成**：自动更新系统 UI 样式以匹配主题

这个主题系统使得应用能够提供完整的主题支持，包括浅色/深色模式切换、无障碍支持和流畅的主题过渡动画。
