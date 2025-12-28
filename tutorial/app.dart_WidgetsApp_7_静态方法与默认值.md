# WidgetsApp 静态方法与默认值详解

## 概述

`WidgetsApp` 提供了多个静态方法和静态常量，用于控制调试工具的显示、提供默认的快捷键和操作映射。这些静态成员为应用提供了平台特定的默认行为和调试支持。

## 调试工具覆盖

### showPerformanceOverlayOverride

```dart 1216:1219:packages/flutter/lib/src/widgets/app.dart
  /// If true, forces the performance overlay to be visible in all instances.
  ///
  /// Used by the `showPerformanceOverlay` VM service extension.
  static bool showPerformanceOverlayOverride = false;
```

**功能**：如果为 `true`，强制在所有实例中显示性能覆盖层。

**用途**：由 `showPerformanceOverlay` VM 服务扩展使用，允许通过调试工具动态控制性能覆盖层的显示。

**使用场景**：在开发过程中，可以通过调试工具或 DevTools 动态开启性能覆盖层，而无需修改代码。

### debugShowWidgetInspectorOverride（已废弃）

```dart 1221:1249:packages/flutter/lib/src/widgets/app.dart
  /// If true, forces the widget inspector to be visible.
  ///
  /// Deprecated.
  /// Use WidgetsBinding.instance.debugShowWidgetInspectorOverrideNotifier.value
  /// instead.
  ///
  /// Overrides the `debugShowWidgetInspector` value set in [WidgetsApp].
  ///
  /// Used by the `debugShowWidgetInspector` debugging extension.
  ///
  /// The inspector allows the selection of a location on your device or emulator
  /// and view what widgets and render objects associated with it. An outline of
  /// the selected widget and some summary information is shown on device and
  /// more detailed information is shown in the IDE or DevTools.
  @Deprecated(
    'Use WidgetsBinding.instance.debugShowWidgetInspectorOverrideNotifier.value instead. '
    'This feature was deprecated after v3.20.0-14.0.pre.',
  )
  static bool get debugShowWidgetInspectorOverride {
    return WidgetsBinding.instance.debugShowWidgetInspectorOverrideNotifier.value;
  }

  @Deprecated(
    'Use WidgetsBinding.instance.debugShowWidgetInspectorOverrideNotifier.value instead. '
    'This feature was deprecated after v3.20.0-14.0.pre.',
  )
  static set debugShowWidgetInspectorOverride(bool value) {
    WidgetsBinding.instance.debugShowWidgetInspectorOverrideNotifier.value = value;
  }
```

**状态**：已废弃，应使用 `WidgetsBinding.instance.debugShowWidgetInspectorOverrideNotifier.value` 代替。

**功能**：如果为 `true`，强制显示 widget 检查器。

**用途**：由 `debugShowWidgetInspector` 调试扩展使用，允许通过调试工具动态控制 widget 检查器的显示。

**迁移**：应使用 `WidgetsBinding.instance.debugShowWidgetInspectorOverrideNotifier.value` 代替。

### debugAllowBannerOverride

```dart 1251:1257:packages/flutter/lib/src/widgets/app.dart
  /// If false, prevents the debug banner from being visible.
  ///
  /// Used by the `debugAllowBanner` VM service extension.
  ///
  /// This is how `flutter run` turns off the banner when you take a screen shot
  /// with "s".
  static bool debugAllowBannerOverride = true;
```

**功能**：如果为 `false`，阻止调试横幅显示。

**用途**：由 `debugAllowBanner` VM 服务扩展使用。

**使用场景**：当使用 `flutter run` 并按下 "s" 键截图时，会自动关闭调试横幅，以便获得干净的截图。

## 默认快捷键

### _defaultShortcuts

```dart 1259:1301:packages/flutter/lib/src/widgets/app.dart
  static const Map<ShortcutActivator, Intent> _defaultShortcuts = <ShortcutActivator, Intent>{
    // Activation
    SingleActivator(LogicalKeyboardKey.enter): ActivateIntent(),
    SingleActivator(LogicalKeyboardKey.numpadEnter): ActivateIntent(),
    SingleActivator(LogicalKeyboardKey.space): ActivateIntent(),
    SingleActivator(LogicalKeyboardKey.gameButtonA): ActivateIntent(),
    SingleActivator(LogicalKeyboardKey.select): ActivateIntent(),

    // Dismissal
    SingleActivator(LogicalKeyboardKey.escape): DismissIntent(),

    // Keyboard traversal.
    SingleActivator(LogicalKeyboardKey.tab): NextFocusIntent(),
    SingleActivator(LogicalKeyboardKey.tab, shift: true): PreviousFocusIntent(),
    SingleActivator(LogicalKeyboardKey.arrowLeft): DirectionalFocusIntent(TraversalDirection.left),
    SingleActivator(LogicalKeyboardKey.arrowRight): DirectionalFocusIntent(
      TraversalDirection.right,
    ),
    SingleActivator(LogicalKeyboardKey.arrowDown): DirectionalFocusIntent(TraversalDirection.down),
    SingleActivator(LogicalKeyboardKey.arrowUp): DirectionalFocusIntent(TraversalDirection.up),

    // Scrolling
    SingleActivator(LogicalKeyboardKey.arrowUp, control: true): ScrollIntent(
      direction: AxisDirection.up,
    ),
    SingleActivator(LogicalKeyboardKey.arrowDown, control: true): ScrollIntent(
      direction: AxisDirection.down,
    ),
    SingleActivator(LogicalKeyboardKey.arrowLeft, control: true): ScrollIntent(
      direction: AxisDirection.left,
    ),
    SingleActivator(LogicalKeyboardKey.arrowRight, control: true): ScrollIntent(
      direction: AxisDirection.right,
    ),
    SingleActivator(LogicalKeyboardKey.pageUp): ScrollIntent(
      direction: AxisDirection.up,
      type: ScrollIncrementType.page,
    ),
    SingleActivator(LogicalKeyboardKey.pageDown): ScrollIntent(
      direction: AxisDirection.down,
      type: ScrollIncrementType.page,
    ),
  };
```

**功能**：默认快捷键映射（用于 Android、Fuchsia、Linux、Windows 平台）。

**快捷键分类**：

1. **激活（Activation）**：
   - `Enter`：激活意图
   - `Numpad Enter`：激活意图
   - `Space`：激活意图
   - `Game Button A`：激活意图
   - `Select`：激活意图

2. **关闭（Dismissal）**：
   - `Escape`：关闭意图

3. **键盘遍历（Keyboard Traversal）**：
   - `Tab`：下一个焦点
   - `Shift + Tab`：上一个焦点
   - `Arrow Left/Right/Up/Down`：方向性焦点移动

4. **滚动（Scrolling）**：
   - `Ctrl + Arrow Up/Down/Left/Right`：滚动
   - `Page Up/Down`：页面滚动

### _defaultWebShortcuts

```dart 1303:1336:packages/flutter/lib/src/widgets/app.dart
  // Default shortcuts for the web platform.
  static const Map<ShortcutActivator, Intent> _defaultWebShortcuts = <ShortcutActivator, Intent>{
    // Activation
    SingleActivator(LogicalKeyboardKey.space): PrioritizedIntents(
      orderedIntents: <Intent>[
        ActivateIntent(),
        ScrollIntent(direction: AxisDirection.down, type: ScrollIncrementType.page),
      ],
    ),
    // On the web, enter activates buttons, but not other controls.
    SingleActivator(LogicalKeyboardKey.enter): ButtonActivateIntent(),
    SingleActivator(LogicalKeyboardKey.numpadEnter): ButtonActivateIntent(),

    // Dismissal
    SingleActivator(LogicalKeyboardKey.escape): DismissIntent(),

    // Keyboard traversal.
    SingleActivator(LogicalKeyboardKey.tab): NextFocusIntent(),
    SingleActivator(LogicalKeyboardKey.tab, shift: true): PreviousFocusIntent(),

    // Scrolling
    SingleActivator(LogicalKeyboardKey.arrowUp): ScrollIntent(direction: AxisDirection.up),
    SingleActivator(LogicalKeyboardKey.arrowDown): ScrollIntent(direction: AxisDirection.down),
    SingleActivator(LogicalKeyboardKey.arrowLeft): ScrollIntent(direction: AxisDirection.left),
    SingleActivator(LogicalKeyboardKey.arrowRight): ScrollIntent(direction: AxisDirection.right),
    SingleActivator(LogicalKeyboardKey.pageUp): ScrollIntent(
      direction: AxisDirection.up,
      type: ScrollIncrementType.page,
    ),
    SingleActivator(LogicalKeyboardKey.pageDown): ScrollIntent(
      direction: AxisDirection.down,
      type: ScrollIncrementType.page,
    ),
  };
```

**功能**：Web 平台的默认快捷键映射。

**与默认快捷键的区别**：

1. **Space 键**：使用 `PrioritizedIntents`，优先激活，如果激活失败则滚动
2. **Enter 键**：使用 `ButtonActivateIntent`，只激活按钮，不激活其他控件（符合 Web 标准）
3. **方向键**：直接用于滚动，不需要 `Ctrl` 修饰键（符合 Web 习惯）

### _defaultAppleOsShortcuts

```dart 1338:1380:packages/flutter/lib/src/widgets/app.dart
  // Default shortcuts for the macOS platform.
  static const Map<ShortcutActivator, Intent>
  _defaultAppleOsShortcuts = <ShortcutActivator, Intent>{
    // Activation
    SingleActivator(LogicalKeyboardKey.enter): ActivateIntent(),
    SingleActivator(LogicalKeyboardKey.numpadEnter): ActivateIntent(),
    SingleActivator(LogicalKeyboardKey.space): ActivateIntent(),

    // Dismissal
    SingleActivator(LogicalKeyboardKey.escape): DismissIntent(),

    // Keyboard traversal
    SingleActivator(LogicalKeyboardKey.tab): NextFocusIntent(),
    SingleActivator(LogicalKeyboardKey.tab, shift: true): PreviousFocusIntent(),
    SingleActivator(LogicalKeyboardKey.arrowLeft): DirectionalFocusIntent(TraversalDirection.left),
    SingleActivator(LogicalKeyboardKey.arrowRight): DirectionalFocusIntent(
      TraversalDirection.right,
    ),
    SingleActivator(LogicalKeyboardKey.arrowDown): DirectionalFocusIntent(TraversalDirection.down),
    SingleActivator(LogicalKeyboardKey.arrowUp): DirectionalFocusIntent(TraversalDirection.up),

    // Scrolling
    SingleActivator(LogicalKeyboardKey.arrowUp, meta: true): ScrollIntent(
      direction: AxisDirection.up,
    ),
    SingleActivator(LogicalKeyboardKey.arrowDown, meta: true): ScrollIntent(
      direction: AxisDirection.down,
    ),
    SingleActivator(LogicalKeyboardKey.arrowLeft, meta: true): ScrollIntent(
      direction: AxisDirection.left,
    ),
    SingleActivator(LogicalKeyboardKey.arrowRight, meta: true): ScrollIntent(
      direction: AxisDirection.right,
    ),
    SingleActivator(LogicalKeyboardKey.pageUp): ScrollIntent(
      direction: AxisDirection.up,
      type: ScrollIncrementType.page,
    ),
    SingleActivator(LogicalKeyboardKey.pageDown): ScrollIntent(
      direction: AxisDirection.down,
      type: ScrollIncrementType.page,
    ),
  };
```

**功能**：macOS 和 iOS 平台的默认快捷键映射。

**与默认快捷键的区别**：

1. **滚动快捷键**：使用 `Meta`（Command）键而不是 `Control` 键，符合 macOS 习惯
2. **其他快捷键**：与默认快捷键基本相同

### defaultShortcuts

```dart 1382:1401:packages/flutter/lib/src/widgets/app.dart
  /// Generates the default shortcut key bindings based on the
  /// [defaultTargetPlatform].
  ///
  /// Used by [WidgetsApp] to assign a default value to [WidgetsApp.shortcuts].
  static Map<ShortcutActivator, Intent> get defaultShortcuts {
    if (kIsWeb) {
      return _defaultWebShortcuts;
    }

    switch (defaultTargetPlatform) {
      case TargetPlatform.android:
      case TargetPlatform.fuchsia:
      case TargetPlatform.linux:
      case TargetPlatform.windows:
        return _defaultShortcuts;
      case TargetPlatform.iOS:
      case TargetPlatform.macOS:
        return _defaultAppleOsShortcuts;
    }
  }
```

**功能**：根据 `defaultTargetPlatform` 生成默认快捷键绑定。

**平台选择逻辑**：

1. **Web**：返回 `_defaultWebShortcuts`
2. **Android/Fuchsia/Linux/Windows**：返回 `_defaultShortcuts`
3. **iOS/macOS**：返回 `_defaultAppleOsShortcuts`

**用途**：由 `WidgetsApp` 使用，为 `WidgetsApp.shortcuts` 分配默认值。

## 默认操作

### defaultActions

```dart 1403:1414:packages/flutter/lib/src/widgets/app.dart
  /// The default value of [WidgetsApp.actions].
  static Map<Type, Action<Intent>> defaultActions = <Type, Action<Intent>>{
    DoNothingIntent: DoNothingAction(),
    DoNothingAndStopPropagationIntent: DoNothingAction(consumesKey: false),
    RequestFocusIntent: RequestFocusAction(),
    NextFocusIntent: NextFocusAction(),
    PreviousFocusIntent: PreviousFocusAction(),
    DirectionalFocusIntent: DirectionalFocusAction(),
    ScrollIntent: ScrollAction(),
    PrioritizedIntents: PrioritizedAction(),
    VoidCallbackIntent: VoidCallbackAction(),
  };
```

**功能**：`WidgetsApp.actions` 的默认值。

**包含的操作**：

1. **DoNothingIntent**：`DoNothingAction()` - 不执行任何操作
2. **DoNothingAndStopPropagationIntent**：`DoNothingAction(consumesKey: false)` - 不执行操作且不消费按键
3. **RequestFocusIntent**：`RequestFocusAction()` - 请求焦点
4. **NextFocusIntent**：`NextFocusAction()` - 下一个焦点
5. **PreviousFocusIntent**：`PreviousFocusAction()` - 上一个焦点
6. **DirectionalFocusIntent**：`DirectionalFocusAction()` - 方向性焦点移动
7. **ScrollIntent**：`ScrollAction()` - 滚动操作
8. **PrioritizedIntents**：`PrioritizedAction()` - 优先级意图处理
9. **VoidCallbackIntent**：`VoidCallbackAction()` - 回调意图

**用途**：这些操作与默认快捷键配合使用，实现标准的键盘交互行为。

## 平台特定的快捷键差异

### 滚动快捷键

| 平台 | 滚动快捷键 |
| --- | --- |
| Android/Linux/Windows | `Ctrl + Arrow` |
| macOS/iOS | `Meta + Arrow` |
| Web | `Arrow`（无修饰键） |

### Enter 键行为

| 平台 | Enter 键行为 |
| --- | --- |
| Android/Linux/Windows/macOS/iOS | `ActivateIntent`（激活所有控件） |
| Web | `ButtonActivateIntent`（只激活按钮） |

### Space 键行为

| 平台 | Space 键行为 |
| --- | --- |
| Android/Linux/Windows/macOS/iOS | `ActivateIntent` |
| Web | `PrioritizedIntents`（优先激活，失败则滚动） |

## 使用示例

### 扩展默认快捷键

```dart
WidgetsApp(
  shortcuts: <ShortcutActivator, Intent>{
    ... WidgetsApp.defaultShortcuts,
    // 添加自定义快捷键
    SingleActivator(LogicalKeyboardKey.keyS, control: true): SaveIntent(),
    SingleActivator(LogicalKeyboardKey.keyO, control: true): OpenIntent(),
  },
  actions: <Type, Action<Intent>>{
    ... WidgetsApp.defaultActions,
    // 添加自定义操作
    SaveIntent: CallbackAction<SaveIntent>(
      onInvoke: (SaveIntent intent) {
        // 执行保存
        return null;
      },
    ),
    OpenIntent: CallbackAction<OpenIntent>(
      onInvoke: (OpenIntent intent) {
        // 执行打开
        return null;
      },
    ),
  },
  color: Colors.blue,
  // ... 其他配置
)
```

### 完全自定义快捷键

```dart
WidgetsApp(
  shortcuts: <ShortcutActivator, Intent>{
    // 完全自定义，不使用默认值
    SingleActivator(LogicalKeyboardKey.enter): CustomActivateIntent(),
    SingleActivator(LogicalKeyboardKey.escape): CustomDismissIntent(),
  },
  actions: <Type, Action<Intent>>{
    ... WidgetsApp.defaultActions,
    CustomActivateIntent: CustomActivateAction(),
    CustomDismissIntent: CustomDismissAction(),
  },
  color: Colors.blue,
  // ... 其他配置
)
```

## 调试工具的使用

### 动态控制性能覆盖层

```dart
// 在调试工具中或通过代码
WidgetsApp.showPerformanceOverlayOverride = true;
// 性能覆盖层将显示在所有 WidgetsApp 实例中
```

### 动态控制调试横幅

```dart
// 在 flutter run 中按 's' 截图时自动执行
WidgetsApp.debugAllowBannerOverride = false;
// 调试横幅将被隐藏
```

## 总结

第七部分详细介绍了 `WidgetsApp` 的静态方法和默认值：

1. **调试工具覆盖**：
   - `showPerformanceOverlayOverride`：强制显示性能覆盖层
   - `debugShowWidgetInspectorOverride`：强制显示 widget 检查器（已废弃）
   - `debugAllowBannerOverride`：控制调试横幅显示

2. **默认快捷键**：
   - `_defaultShortcuts`：Android/Linux/Windows/Fuchsia 平台的默认快捷键
   - `_defaultWebShortcuts`：Web 平台的默认快捷键
   - `_defaultAppleOsShortcuts`：macOS/iOS 平台的默认快捷键
   - `defaultShortcuts`：根据平台自动选择默认快捷键的 getter

3. **默认操作**：`defaultActions` 提供标准的意图到操作的映射

这些静态成员为应用提供了平台特定的默认行为，确保应用在不同平台上都能提供符合平台习惯的键盘交互体验。同时，调试工具覆盖允许开发者在开发过程中动态控制调试工具的显示。
