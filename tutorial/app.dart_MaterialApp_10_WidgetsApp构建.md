# MaterialApp WidgetsApp 构建详解

## 概述

`_MaterialAppState` 的 `_buildWidgetApp` 方法负责构建底层的 `WidgetsApp` 或 `WidgetsApp.router`。这个方法根据是否使用 Router 模式选择不同的构建方式，并将所有配置参数传递给 `WidgetsApp`，同时应用 Material 特有的配置。

## _buildWidgetApp

```dart 1061:1135:packages/flutter/lib/src/material/app.dart
  Widget _buildWidgetApp(BuildContext context) {
    // The color property is always pulled from the light theme, even if dark
    // mode is activated. This was done to simplify the technical details
    // of switching themes and it was deemed acceptable because this color
    // property is only used on old Android OSes to color the app bar in
    // Android's switcher UI.
    //
    // blue is the primary color of the default theme.
    final Color materialColor = widget.color ?? widget.theme?.primaryColor ?? Colors.blue;
    if (_usesRouter) {
      return WidgetsApp.router(
        key: GlobalObjectKey(this),
        routeInformationProvider: widget.routeInformationProvider,
        routeInformationParser: widget.routeInformationParser,
        routerDelegate: widget.routerDelegate,
        routerConfig: widget.routerConfig,
        backButtonDispatcher: widget.backButtonDispatcher,
        onNavigationNotification: widget.onNavigationNotification,
        builder: _materialBuilder,
        title: widget.title,
        onGenerateTitle: widget.onGenerateTitle,
        textStyle: _errorTextStyle,
        color: materialColor,
        locale: widget.locale,
        localizationsDelegates: _localizationsDelegates,
        localeResolutionCallback: widget.localeResolutionCallback,
        localeListResolutionCallback: widget.localeListResolutionCallback,
        supportedLocales: widget.supportedLocales,
        showPerformanceOverlay: widget.showPerformanceOverlay,
        showSemanticsDebugger: widget.showSemanticsDebugger,
        debugShowCheckedModeBanner: widget.debugShowCheckedModeBanner,
        exitWidgetSelectionButtonBuilder: _exitWidgetSelectionButtonBuilder,
        moveExitWidgetSelectionButtonBuilder: _moveExitWidgetSelectionButtonBuilder,
        tapBehaviorButtonBuilder: _tapBehaviorButtonBuilder,
        shortcuts: widget.shortcuts,
        actions: widget.actions,
        restorationScopeId: widget.restorationScopeId,
      );
    }

    return WidgetsApp(
      key: GlobalObjectKey(this),
      navigatorKey: widget.navigatorKey,
      navigatorObservers: widget.navigatorObservers!,
      pageRouteBuilder: <T>(RouteSettings settings, WidgetBuilder builder) {
        return MaterialPageRoute<T>(settings: settings, builder: builder);
      },
      home: widget.home,
      routes: widget.routes!,
      initialRoute: widget.initialRoute,
      onGenerateRoute: widget.onGenerateRoute,
      onGenerateInitialRoutes: widget.onGenerateInitialRoutes,
      onUnknownRoute: widget.onUnknownRoute,
      onNavigationNotification: widget.onNavigationNotification,
      builder: _materialBuilder,
      title: widget.title,
      onGenerateTitle: widget.onGenerateTitle,
      textStyle: _errorTextStyle,
      color: materialColor,
      locale: widget.locale,
      localizationsDelegates: _localizationsDelegates,
      localeResolutionCallback: widget.localeResolutionCallback,
      localeListResolutionCallback: widget.localeListResolutionCallback,
      supportedLocales: widget.supportedLocales,
      showPerformanceOverlay: widget.showPerformanceOverlay,
      showSemanticsDebugger: widget.showSemanticsDebugger,
      debugShowCheckedModeBanner: widget.debugShowCheckedModeBanner,
      exitWidgetSelectionButtonBuilder: _exitWidgetSelectionButtonBuilder,
      moveExitWidgetSelectionButtonBuilder: _moveExitWidgetSelectionButtonBuilder,
      tapBehaviorButtonBuilder: _tapBehaviorButtonBuilder,
      shortcuts: widget.shortcuts,
      actions: widget.actions,
      restorationScopeId: widget.restorationScopeId,
    );
  }
```

**功能**：根据是否使用 Router 模式构建 `WidgetsApp` 或 `WidgetsApp.router`。

## Material 颜色计算

```dart 1069:1069:packages/flutter/lib/src/material/app.dart
    final Color materialColor = widget.color ?? widget.theme?.primaryColor ?? Colors.blue;
```

**功能**：计算传递给 `WidgetsApp` 的 `color` 属性。

**计算逻辑**：

1. **优先使用 color**：如果 `widget.color` 不为 `null`，使用它
2. **回退到主题主色**：否则，如果 `widget.theme` 不为 `null`，使用 `theme.primaryColor`
3. **最终回退**：否则，使用 `Colors.blue`（默认主题的主色）

**设计考虑**：

文档注释中解释了为什么总是从浅色主题中提取颜色：

- `color` 属性总是从浅色主题中提取，即使激活了深色模式也是如此
- 这样做是为了简化主题切换的技术细节
- 这是可以接受的，因为 `color` 属性只在旧版 Android 系统上用于在 Android 的切换器 UI 中为应用栏着色
- `Colors.blue` 是默认主题的主色

## Router 模式构建

```dart 1070:1098:packages/flutter/lib/src/material/app.dart
    if (_usesRouter) {
      return WidgetsApp.router(
        key: GlobalObjectKey(this),
        routeInformationProvider: widget.routeInformationProvider,
        routeInformationParser: widget.routeInformationParser,
        routerDelegate: widget.routerDelegate,
        routerConfig: widget.routerConfig,
        backButtonDispatcher: widget.backButtonDispatcher,
        onNavigationNotification: widget.onNavigationNotification,
        builder: _materialBuilder,
        title: widget.title,
        onGenerateTitle: widget.onGenerateTitle,
        textStyle: _errorTextStyle,
        color: materialColor,
        locale: widget.locale,
        localizationsDelegates: _localizationsDelegates,
        localeResolutionCallback: widget.localeResolutionCallback,
        localeListResolutionCallback: widget.localeListResolutionCallback,
        supportedLocales: widget.supportedLocales,
        showPerformanceOverlay: widget.showPerformanceOverlay,
        showSemanticsDebugger: widget.showSemanticsDebugger,
        debugShowCheckedModeBanner: widget.debugShowCheckedModeBanner,
        exitWidgetSelectionButtonBuilder: _exitWidgetSelectionButtonBuilder,
        moveExitWidgetSelectionButtonBuilder: _moveExitWidgetSelectionButtonBuilder,
        tapBehaviorButtonBuilder: _tapBehaviorButtonBuilder,
        shortcuts: widget.shortcuts,
        actions: widget.actions,
        restorationScopeId: widget.restorationScopeId,
      );
    }
```

**功能**：构建 `WidgetsApp.router`，用于 Router 模式。

**传递的参数**：

1. **Router 相关**：`routeInformationProvider`、`routeInformationParser`、`routerDelegate`、`routerConfig`、`backButtonDispatcher`
2. **导航相关**：`onNavigationNotification`
3. **Material 特有**：
   - `builder: _materialBuilder`：Material 构建器
   - `textStyle: _errorTextStyle`：错误文本样式
   - Widget Inspector 按钮构建器
4. **应用信息**：`title`、`onGenerateTitle`、`color`
5. **本地化**：`locale`、`localizationsDelegates`、`localeResolutionCallback`、`localeListResolutionCallback`、`supportedLocales`
6. **调试工具**：`showPerformanceOverlay`、`showSemanticsDebugger`、`debugShowCheckedModeBanner`
7. **交互**：`shortcuts`、`actions`
8. **状态恢复**：`restorationScopeId`

## Navigator 模式构建

```dart 1101:1134:packages/flutter/lib/src/material/app.dart
    return WidgetsApp(
      key: GlobalObjectKey(this),
      navigatorKey: widget.navigatorKey,
      navigatorObservers: widget.navigatorObservers!,
      pageRouteBuilder: <T>(RouteSettings settings, WidgetBuilder builder) {
        return MaterialPageRoute<T>(settings: settings, builder: builder);
      },
      home: widget.home,
      routes: widget.routes!,
      initialRoute: widget.initialRoute,
      onGenerateRoute: widget.onGenerateRoute,
      onGenerateInitialRoutes: widget.onGenerateInitialRoutes,
      onUnknownRoute: widget.onUnknownRoute,
      onNavigationNotification: widget.onNavigationNotification,
      builder: _materialBuilder,
      title: widget.title,
      onGenerateTitle: widget.onGenerateTitle,
      textStyle: _errorTextStyle,
      color: materialColor,
      locale: widget.locale,
      localizationsDelegates: _localizationsDelegates,
      localeResolutionCallback: widget.localeResolutionCallback,
      localeListResolutionCallback: widget.localeListResolutionCallback,
      supportedLocales: widget.supportedLocales,
      showPerformanceOverlay: widget.showPerformanceOverlay,
      showSemanticsDebugger: widget.showSemanticsDebugger,
      debugShowCheckedModeBanner: widget.debugShowCheckedModeBanner,
      exitWidgetSelectionButtonBuilder: _exitWidgetSelectionButtonBuilder,
      moveExitWidgetSelectionButtonBuilder: _moveExitWidgetSelectionButtonBuilder,
      tapBehaviorButtonBuilder: _tapBehaviorButtonBuilder,
      shortcuts: widget.shortcuts,
      actions: widget.actions,
      restorationScopeId: widget.restorationScopeId,
    );
```

**功能**：构建 `WidgetsApp`，用于 Navigator 模式。

**传递的参数**：

1. **Navigator 相关**：`navigatorKey`、`navigatorObservers`、`home`、`routes`、`initialRoute`、`onGenerateRoute`、`onGenerateInitialRoutes`、`onUnknownRoute`
2. **Material 特有**：
   - `pageRouteBuilder`：自动使用 `MaterialPageRoute`
   - `builder: _materialBuilder`：Material 构建器
   - `textStyle: _errorTextStyle`：错误文本样式
   - Widget Inspector 按钮构建器
3. **其他参数**：与 Router 模式相同

### MaterialPageRoute 自动配置

```dart 1105:1107:packages/flutter/lib/src/material/app.dart
      pageRouteBuilder: <T>(RouteSettings settings, WidgetBuilder builder) {
        return MaterialPageRoute<T>(settings: settings, builder: builder);
      },
```

**功能**：自动使用 `MaterialPageRoute` 作为 `pageRouteBuilder`。

**特点**：

- 开发者不需要手动指定 `pageRouteBuilder`
- `MaterialPageRoute` 提供 Material Design 风格的页面过渡动画
- 支持 Hero 动画

## Material 特有配置

### builder: _materialBuilder

两个模式都使用 `_materialBuilder` 作为 `builder`：

```dart
builder: _materialBuilder,
```

这确保了 Router 模式和 Navigator 模式下的应用都获得完整的 Material Design 功能，包括：

- 主题系统
- `ScaffoldMessenger`
- `DefaultSelectionStyle`
- 主题动画

### textStyle: _errorTextStyle

```dart 38:54:packages/flutter/lib/src/material/app.dart
/// [MaterialApp] uses this [TextStyle] as its [DefaultTextStyle] to encourage
/// developers to be intentional about their [DefaultTextStyle].
///
/// In Material Design, most [Text] widgets are contained in [Material] widgets,
/// which sets a specific [DefaultTextStyle]. If you're seeing text that uses
/// this text style, consider putting your text in a [Material] widget (or
/// another widget that sets a [DefaultTextStyle]).
const TextStyle _errorTextStyle = TextStyle(
  color: Color(0xD0FF0000),
  fontFamily: 'monospace',
  fontSize: 48.0,
  fontWeight: FontWeight.w900,
  decoration: TextDecoration.underline,
  decorationColor: Color(0xFFFFFF00),
  decorationStyle: TextDecorationStyle.double,
  debugLabel: 'fallback style; consider putting your text in a Material',
);
```

**功能**：错误文本样式，用于警告开发者应用没有定义默认文本样式。

**特点**：

- 难看的红/黄文本样式
- 用于提醒开发者将文本放在 `Material` widget 中
- 通常应用的 `Scaffold` 会构建一个 `Material` widget，定义其默认文本样式

### Widget Inspector 按钮构建器

两个模式都传递 Material 风格的 Widget Inspector 按钮构建器：

```dart
exitWidgetSelectionButtonBuilder: _exitWidgetSelectionButtonBuilder,
moveExitWidgetSelectionButtonBuilder: _moveExitWidgetSelectionButtonBuilder,
tapBehaviorButtonBuilder: _tapBehaviorButtonBuilder,
```

这确保了 Widget Inspector 使用 Material 风格的按钮。

### localizationsDelegates: _localizationsDelegates

两个模式都使用 `_localizationsDelegates` getter：

```dart
localizationsDelegates: _localizationsDelegates,
```

这确保了自动包含 Material 和 Cupertino 的本地化委托。

## 两种模式的对比

### 相同点

- 都使用 `_materialBuilder` 作为 `builder`
- 都使用 `_errorTextStyle` 作为 `textStyle`
- 都使用 `_localizationsDelegates` 作为 `localizationsDelegates`
- 都传递 Material 风格的 Widget Inspector 按钮构建器
- 都传递相同的应用信息、本地化、调试、交互等参数

### 不同点

| 特性 | Router 模式 | Navigator 模式 |
| --- | --- | --- |
| **构建的 Widget** | `WidgetsApp.router` | `WidgetsApp` |
| **路由相关参数** | Router 相关参数 | Navigator 相关参数 |
| **pageRouteBuilder** | 不需要 | 自动使用 `MaterialPageRoute` |

## 构建流程

### Router 模式构建流程

```text
_buildWidgetApp(context)
    ↓
计算 materialColor
    ↓
检查 _usesRouter
    ↓ (true)
构建 WidgetsApp.router
    ├── 传递 Router 相关参数
    ├── 传递 _materialBuilder
    ├── 传递 _errorTextStyle
    ├── 传递 _localizationsDelegates
    ├── 传递 Widget Inspector 按钮构建器
    └── 传递其他配置参数
    ↓
返回 WidgetsApp.router
```

### Navigator 模式构建流程

```text
_buildWidgetApp(context)
    ↓
计算 materialColor
    ↓
检查 _usesRouter
    ↓ (false)
构建 WidgetsApp
    ├── 传递 Navigator 相关参数
    ├── 传递 pageRouteBuilder (MaterialPageRoute)
    ├── 传递 _materialBuilder
    ├── 传递 _errorTextStyle
    ├── 传递 _localizationsDelegates
    ├── 传递 Widget Inspector 按钮构建器
    └── 传递其他配置参数
    ↓
返回 WidgetsApp
```

## 使用示例

### Navigator 模式

```dart
MaterialApp(
  home: MyHomePage(),
  routes: {
    '/details': (context) => DetailsPage(),
  },
  theme: ThemeData.light(),
)
```

`_buildWidgetApp` 会构建 `WidgetsApp`，并自动使用 `MaterialPageRoute`。

### Router 模式

```dart
MaterialApp.router(
  routerConfig: routerConfig,
  theme: ThemeData.light(),
)
```

`_buildWidgetApp` 会构建 `WidgetsApp.router`，并传递所有 Router 相关参数。

## 总结

第十部分详细介绍了 `_MaterialAppState` 如何构建底层的 `WidgetsApp`：

1. **Material 颜色计算**：从 `color`、`theme.primaryColor` 或 `Colors.blue` 中计算 `materialColor`

2. **Router 模式构建**：构建 `WidgetsApp.router`，传递所有 Router 相关参数和 Material 特有配置

3. **Navigator 模式构建**：构建 `WidgetsApp`，自动使用 `MaterialPageRoute` 作为 `pageRouteBuilder`

4. **Material 特有配置**：
   - `builder: _materialBuilder`：确保获得完整的 Material Design 功能
   - `textStyle: _errorTextStyle`：错误文本样式警告
   - Widget Inspector 按钮构建器：Material 风格的按钮
   - `localizationsDelegates: _localizationsDelegates`：自动包含 Material 和 Cupertino 委托

5. **参数传递**：将所有配置参数传递给 `WidgetsApp`，确保功能完整

这个方法确保了 `MaterialApp` 能够正确构建底层的 `WidgetsApp`，同时应用所有 Material 特有的配置，使得应用获得完整的 Material Design 支持。
