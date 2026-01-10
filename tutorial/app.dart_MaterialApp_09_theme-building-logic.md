# MaterialApp 主题构建逻辑详解

## 概述

`_MaterialAppState` 提供了主题构建的核心逻辑，包括主题选择、主题动画、Material 构建器等。这些方法是 `MaterialApp` 的核心功能，负责将主题系统应用到整个应用。

## _isDarkTheme

```dart 980:984:packages/flutter/lib/src/material/app.dart
  bool _isDarkTheme(BuildContext context) {
    return widget.themeMode == ThemeMode.dark ||
        widget.themeMode == ThemeMode.system &&
            MediaQuery.platformBrightnessOf(context) == Brightness.dark;
  }
```

**功能**：判断当前是否应该使用深色主题。

**判断逻辑**：

1. **ThemeMode.dark**：如果 `themeMode` 为 `ThemeMode.dark`，返回 `true`
2. **ThemeMode.system**：如果 `themeMode` 为 `ThemeMode.system` 且平台亮度为 `Brightness.dark`，返回 `true`
3. **其他情况**：返回 `false`

**用途**：用于 Widget Inspector 按钮构建器，根据当前主题选择按钮的颜色。

## _themeBuilder

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

**功能**：根据主题模式、平台亮度和高对比度设置构建最终的主题。

### 主题选择逻辑

#### 步骤 1：确定主题模式

```dart
final ThemeMode mode = widget.themeMode ?? ThemeMode.system;
```

- 如果 `themeMode` 不为 `null`，使用 `themeMode`
- 否则，默认为 `ThemeMode.system`

#### 步骤 2：确定平台亮度

```dart
final Brightness platformBrightness = MediaQuery.platformBrightnessOf(context);
```

从 `MediaQuery` 获取平台的亮度偏好。

#### 步骤 3：判断是否使用深色主题

```dart
final bool useDarkTheme =
    mode == ThemeMode.dark ||
    (mode == ThemeMode.system && platformBrightness == ui.Brightness.dark);
```

- 如果 `themeMode` 为 `ThemeMode.dark`，使用深色主题
- 如果 `themeMode` 为 `ThemeMode.system` 且平台亮度为深色，使用深色主题

#### 步骤 4：检测高对比度

```dart
final bool highContrast = MediaQuery.highContrastOf(context);
```

从 `MediaQuery` 检测平台是否请求高对比度。

#### 步骤 5：选择主题（按优先级）

1. **高对比度深色主题**：

   ```dart
   if (useDarkTheme && highContrast && widget.highContrastDarkTheme != null) {
     theme = widget.highContrastDarkTheme;
   }
   ```

2. **深色主题**：

   ```dart
   else if (useDarkTheme && widget.darkTheme != null) {
     theme = widget.darkTheme;
   }
   ```

3. **高对比度主题**：

   ```dart
   else if (highContrast && widget.highContrastTheme != null) {
     theme = widget.highContrastTheme;
   }
   ```

4. **默认主题**：

   ```dart
   theme ??= widget.theme ?? ThemeData();
   ```

#### 步骤 6：更新系统 UI 样式

```dart
SystemChrome.setSystemUIOverlayStyle(
  theme.brightness == Brightness.dark ? SystemUiOverlayStyle.light : SystemUiOverlayStyle.dark,
);
```

根据主题的亮度更新系统 UI 样式（状态栏和导航栏）。

- **深色主题**：使用 `SystemUiOverlayStyle.light`（浅色状态栏和导航栏）
- **浅色主题**：使用 `SystemUiOverlayStyle.dark`（深色状态栏和导航栏）

## _materialBuilder

```dart 1010:1059:packages/flutter/lib/src/material/app.dart
  Widget _materialBuilder(BuildContext context, Widget? child) {
    final ThemeData theme = _themeBuilder(context);
    final Color effectiveSelectionColor =
        theme.textSelectionTheme.selectionColor ?? theme.colorScheme.primary.withOpacity(0.40);
    final Color effectiveCursorColor =
        theme.textSelectionTheme.cursorColor ?? theme.colorScheme.primary;

    Widget childWidget = child ?? const SizedBox.shrink();

    if (widget.builder != null) {
      childWidget = Builder(
        builder: (BuildContext context) {
          // Why are we surrounding a builder with a builder?
          //
          // The widget.builder may contain code that invokes
          // Theme.of(), which should return the theme we selected
          // above in AnimatedTheme. However, if we invoke
          // widget.builder() directly as the child of AnimatedTheme
          // then there is no BuildContext separating them, the
          // widget.builder() will not find the theme. Therefore, we
          // surround widget.builder with yet another builder so that
          // a context separates them and Theme.of() correctly
          // resolves to the theme we passed to AnimatedTheme.
          return widget.builder!(context, child);
        },
      );
    }

    childWidget = ScaffoldMessenger(
      key: widget.scaffoldMessengerKey,
      child: DefaultSelectionStyle(
        selectionColor: effectiveSelectionColor,
        cursorColor: effectiveCursorColor,
        child: childWidget,
      ),
    );

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

    return childWidget;
  }
```

**功能**：Material 构建器，将主题、`ScaffoldMessenger`、`DefaultSelectionStyle` 等 Material 功能应用到应用。

### 构建步骤

#### 步骤 1：构建主题

```dart
final ThemeData theme = _themeBuilder(context);
```

调用 `_themeBuilder` 获取最终的主题。

#### 步骤 2：计算文本选择颜色

```dart
final Color effectiveSelectionColor =
    theme.textSelectionTheme.selectionColor ?? theme.colorScheme.primary.withOpacity(0.40);
final Color effectiveCursorColor =
    theme.textSelectionTheme.cursorColor ?? theme.colorScheme.primary;
```

- **选择颜色**：如果 `theme.textSelectionTheme.selectionColor` 不为 `null`，使用它；否则使用 `ColorScheme.primary` 的 0.4 透明度
- **光标颜色**：如果 `theme.textSelectionTheme.cursorColor` 不为 `null`，使用它；否则使用 `ColorScheme.primary`

#### 步骤 3：处理 child

```dart
Widget childWidget = child ?? const SizedBox.shrink();
```

如果 `child` 为 `null`，使用 `SizedBox.shrink()` 作为占位符。

#### 步骤 4：应用用户 builder（如果提供）

```dart
if (widget.builder != null) {
  childWidget = Builder(
    builder: (BuildContext context) {
      return widget.builder!(context, child);
    },
  );
}
```

**为什么需要额外的 Builder？**

文档注释中解释了原因：

- `widget.builder` 可能包含调用 `Theme.of()` 的代码
- `Theme.of()` 应该返回我们在 `AnimatedTheme` 中选择的主题
- 如果直接将 `widget.builder()` 作为 `AnimatedTheme` 的子 widget 调用，它们之间没有 `BuildContext` 分隔
- `widget.builder()` 中的 `Theme.of()` 将无法找到主题
- 因此，我们用另一个 `Builder` 包装 `widget.builder`，以便 `BuildContext` 分隔它们，`Theme.of()` 能够正确解析到我们传递给 `AnimatedTheme` 的主题

#### 步骤 5：应用 ScaffoldMessenger 和 DefaultSelectionStyle

```dart
childWidget = ScaffoldMessenger(
  key: widget.scaffoldMessengerKey,
  child: DefaultSelectionStyle(
    selectionColor: effectiveSelectionColor,
    cursorColor: effectiveCursorColor,
    child: childWidget,
  ),
);
```

- **ScaffoldMessenger**：提供全局的 SnackBar 等 Material 消息功能
- **DefaultSelectionStyle**：为文本选择提供默认样式

#### 步骤 6：应用主题（带动画或不带动画）

```dart
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

- **带动画**：如果 `themeAnimationStyle` 不是 `AnimationStyle.noAnimation`，使用 `AnimatedTheme` 实现主题过渡动画
- **不带动画**：否则，使用普通的 `Theme` widget，主题变化立即生效

**动画参数优先级**：

- `duration`：`themeAnimationStyle?.duration ?? themeAnimationDuration`
- `curve`：`themeAnimationStyle?.curve ?? themeAnimationCurve`

## Widget Inspector 按钮构建器

`_MaterialAppState` 还提供了 Material 风格的 Widget Inspector 按钮构建器：

### _exitWidgetSelectionButtonBuilder

```dart 934:947:packages/flutter/lib/src/material/app.dart
  Widget _exitWidgetSelectionButtonBuilder(
    BuildContext context, {
    required VoidCallback onPressed,
    required String semanticsLabel,
    required GlobalKey key,
  }) {
    return _MaterialInspectorButton.filled(
      onPressed: onPressed,
      semanticsLabel: semanticsLabel,
      icon: Icons.close,
      isDarkTheme: _isDarkTheme(context),
      buttonKey: key,
    );
  }
```

**功能**：构建 Widget Inspector 用于退出选择模式的按钮。

**特点**：

- 使用 `_MaterialInspectorButton.filled` 创建填充样式的按钮
- 图标为 `Icons.close`
- 根据当前主题选择按钮颜色

### _moveExitWidgetSelectionButtonBuilder

```dart 949:961:packages/flutter/lib/src/material/app.dart
  Widget _moveExitWidgetSelectionButtonBuilder(
    BuildContext context, {
    required VoidCallback onPressed,
    required String semanticsLabel,
    bool usesDefaultAlignment = true,
  }) {
    return _MaterialInspectorButton.iconOnly(
      onPressed: onPressed,
      semanticsLabel: semanticsLabel,
      icon: usesDefaultAlignment ? Icons.arrow_right : Icons.arrow_left,
      isDarkTheme: _isDarkTheme(context),
    );
  }
```

**功能**：构建 Widget Inspector 用于移动退出选择模式按钮的按钮。

**特点**：

- 使用 `_MaterialInspectorButton.iconOnly` 创建仅图标样式的按钮
- 根据 `usesDefaultAlignment` 选择 `Icons.arrow_right` 或 `Icons.arrow_left`
- 根据当前主题选择按钮颜色

### _tapBehaviorButtonBuilder

```dart 963:978:packages/flutter/lib/src/material/app.dart
  Widget _tapBehaviorButtonBuilder(
    BuildContext context, {
    required VoidCallback onPressed,
    required String semanticsLabel,
    required bool selectionOnTapEnabled,
  }) {
    return _MaterialInspectorButton.toggle(
      onPressed: onPressed,
      semanticsLabel: semanticsLabel,
      // This unicode icon is also used for the Cupertino-styled button and for
      // DevTools. It should be updated in all 3 places if changed.
      icon: const IconData(0x1F74A),
      isDarkTheme: _isDarkTheme(context),
      toggledOn: selectionOnTapEnabled,
    );
  }
```

**功能**：构建 Widget Inspector 用于更改点击 widget 时默认行为的按钮。

**特点**：

- 使用 `_MaterialInspectorButton.toggle` 创建切换样式的按钮
- 使用 Unicode 图标（0x1F74A，🍊）
- 根据 `selectionOnTapEnabled` 显示切换状态
- 根据当前主题选择按钮颜色

## 构建流程

### 完整构建流程

```text
_materialBuilder(context, child)
    ↓
1. 调用 _themeBuilder 获取主题
    ↓
2. 计算文本选择颜色
    ↓
3. 处理 child（如果为 null，使用 SizedBox.shrink）
    ↓
4. 应用用户 builder（如果提供）
    ↓
5. 应用 ScaffoldMessenger 和 DefaultSelectionStyle
    ↓
6. 应用主题（AnimatedTheme 或 Theme）
    ↓
返回构建的 widget
```

### 主题选择流程

```text
_themeBuilder(context)
    ↓
1. 确定主题模式（themeMode ?? ThemeMode.system）
    ↓
2. 获取平台亮度
    ↓
3. 判断是否使用深色主题
    ↓
4. 检测高对比度
    ↓
5. 按优先级选择主题
    ├── 高对比度深色主题
    ├── 深色主题
    ├── 高对比度主题
    └── 默认主题
    ↓
6. 更新系统 UI 样式
    ↓
返回最终主题
```

## 使用示例

### 基本使用

```dart
MaterialApp(
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  themeMode: ThemeMode.system,
  home: MyHomePage(),
)
```

`_materialBuilder` 会自动：

- 根据系统设置选择主题
- 应用 `ScaffoldMessenger` 和 `DefaultSelectionStyle`
- 使用 `AnimatedTheme` 实现主题过渡

### 自定义 builder

```dart
MaterialApp(
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  builder: (context, child) {
    // 可以访问 Theme.of(context) 获取主题
    return MediaQuery(
      data: MediaQuery.of(context).copyWith(textScaleFactor: 1.2),
      child: child!,
    );
  },
  home: MyHomePage(),
)
```

`_materialBuilder` 会确保 `builder` 中的 `Theme.of(context)` 能够正确解析到主题。

### 禁用主题动画

```dart
MaterialApp(
  theme: ThemeData.light(),
  darkTheme: ThemeData.dark(),
  themeAnimationStyle: AnimationStyle.noAnimation,
  home: MyHomePage(),
)
```

`_materialBuilder` 会使用普通的 `Theme` widget 而不是 `AnimatedTheme`。

## 总结

第九部分详细介绍了 `_MaterialAppState` 的主题构建逻辑：

1. **_isDarkTheme**：判断是否使用深色主题

2. **_themeBuilder**：
   - 根据主题模式、平台亮度和高对比度选择最终主题
   - 更新系统 UI 样式

3. **_materialBuilder**：
   - 构建主题
   - 计算文本选择颜色
   - 应用 `ScaffoldMessenger` 和 `DefaultSelectionStyle`
   - 应用主题（带动画或不带动画）
   - 处理用户 `builder`（确保 `Theme.of()` 正确工作）

4. **Widget Inspector 按钮构建器**：提供 Material 风格的 Widget Inspector 按钮

这些方法共同实现了 `MaterialApp` 的完整主题系统，包括主题选择、主题动画、Material 功能集成等，为应用提供了完整的 Material Design 支持。
