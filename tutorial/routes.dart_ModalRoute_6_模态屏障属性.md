# ModalRoute 模态屏障属性详解

## 概述

模态屏障（Modal Barrier）是 `ModalRoute` 的一个重要特性，它是渲染在路由后面的遮罩层，用于阻止用户与下方路由的交互，并通常部分遮挡下方的路由。`ModalRoute` 提供了多个属性来控制屏障的行为和外观。

## barrierDismissible

```dart 1684:1716:packages/flutter/lib/src/widgets/routes.dart
  /// {@template flutter.widgets.ModalRoute.barrierDismissible}
  /// Whether you can dismiss this route by tapping the modal barrier.
  ///
  /// The modal barrier is the scrim that is rendered behind each route, which
  /// generally prevents the user from interacting with the route below the
  /// current route, and normally partially obscures such routes.
  ///
  /// For example, when a dialog is on the screen, the page below the dialog is
  /// usually darkened by the modal barrier.
  ///
  /// If [barrierDismissible] is true, then tapping this barrier, pressing
  /// the escape key on the keyboard, or calling route popping functions
  /// such as [Navigator.pop] will cause the current route to be popped
  /// with null as the value.
  ///
  /// If [barrierDismissible] is false, then tapping the barrier has no effect.
  ///
  /// If this getter would ever start returning a different value,
  /// either [changedInternalState] or [changedExternalState] should
  /// be invoked so that the change can take effect.
  ///
  /// It is safe to use `navigator.context` to look up inherited
  /// widgets here, because the [Navigator] calls
  /// [changedExternalState] whenever its dependencies change, and
  /// [changedExternalState] causes the modal barrier to rebuild.
  ///
  /// See also:
  ///
  ///  * [Navigator.pop], which is used to dismiss the route.
  ///  * [barrierColor], which controls the color of the scrim for this route.
  ///  * [ModalBarrier], the widget that implements this feature.
  /// {@endtemplate}
  bool get barrierDismissible;
```

**功能**：控制是否可以通过点击模态屏障来关闭路由。

**行为**：

- **true**：点击屏障、按 ESC 键或调用 `Navigator.pop` 都会弹出当前路由，返回值为 `null`
- **false**：点击屏障没有任何效果

**状态更新**：如果这个 getter 的返回值会改变，需要调用 `changedInternalState` 或 `changedExternalState` 来使改变生效。

## semanticsDismissible

```dart 1718:1740:packages/flutter/lib/src/widgets/routes.dart
  /// Whether the semantics of the modal barrier are included in the
  /// semantics tree.
  ///
  /// The modal barrier is the scrim that is rendered behind each route, which
  /// generally prevents the user from interacting with the route below the
  /// current route, and normally partially obscures such routes.
  ///
  /// If [semanticsDismissible] is true, then modal barrier semantics are
  /// included in the semantics tree.
  ///
  /// If [semanticsDismissible] is false, then modal barrier semantics are
  /// excluded from the semantics tree and tapping on the modal barrier
  /// has no effect.
  ///
  /// If this getter would ever start returning a different value,
  /// either [changedInternalState] or [changedExternalState] should
  /// be invoked so that the change can take effect.
  ///
  /// It is safe to use `navigator.context` to look up inherited
  /// widgets here, because the [Navigator] calls
  /// [changedExternalState] whenever its dependencies changes, and
  /// [changedExternalState] causes the modal barrier to rebuild.
  bool get semanticsDismissible => true;
```

**功能**：控制模态屏障的语义是否包含在语义树中。

**行为**：

- **true（默认）**：模态屏障的语义包含在语义树中，无障碍工具可以识别
- **false**：模态屏障的语义从语义树中排除，点击屏障也没有效果

**默认值**：`true`

**无障碍支持**：这个属性对于无障碍支持很重要，它决定了屏幕阅读器等工具是否能够识别屏障的交互性。

## barrierColor

```dart 1742:1785:packages/flutter/lib/src/widgets/routes.dart
  /// {@template flutter.widgets.ModalRoute.barrierColor}
  /// The color to use for the modal barrier. If this is null, the barrier will
  /// be transparent.
  ///
  /// The modal barrier is the scrim that is rendered behind each route, which
  /// generally prevents the user from interacting with the route below the
  /// current route, and normally partially obscures such routes.
  ///
  /// For example, when a dialog is on the screen, the page below the dialog is
  /// usually darkened by the modal barrier.
  ///
  /// The color is ignored, and the barrier made invisible, when
  /// [ModalRoute.offstage] is true.
  ///
  /// While the route is animating into position, the color is animated from
  /// transparent to the specified color.
  ///
  /// {@endtemplate}
  ///
  /// If this getter would ever start returning a different color, one
  /// of the [changedInternalState] or [changedExternalState] methods
  /// should be invoked so that the change can take effect.
  ///
  /// It is safe to use `navigator.context` to look up inherited
  /// widgets here, because the [Navigator] calls
  /// [changedExternalState] whenever its dependencies change, and
  /// [changedExternalState] causes the modal barrier to rebuild.
  ///
  /// {@tool snippet}
  ///
  /// For example, to make the barrier color use the theme's
  /// background color, one could say:
  ///
  /// ```dart
  /// Color get barrierColor => Theme.of(navigator.context).colorScheme.surface;
  /// ```
  ///
  /// {@end-tool}
  ///
  /// See also:
  ///
  ///  * [barrierDismissible], which controls the behavior of the barrier when
  ///    tapped.
  ///  * [ModalBarrier], the widget that implements this feature.
  Color? get barrierColor;
```

**功能**：控制模态屏障的颜色。

**行为**：

- **null**：屏障透明（不可见但仍然可能阻止交互）
- **非 null**：屏障使用指定的颜色

**特殊情况**：

- 当 `offstage` 为 `true` 时，颜色被忽略，屏障不可见
- 路由进入时，颜色从透明动画过渡到指定颜色

**使用示例**：

```dart
class CustomDialogRoute extends ModalRoute<void> {
  @override
  Color? get barrierColor => Colors.black54; // 半透明黑色

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Dialog(
      child: Text('Custom Dialog'),
    );
  }

  // ... 其他必需的方法实现
}
```

**主题集成示例**：

```dart
class ThemedDialogRoute extends ModalRoute<void> {
  @override
  Color? get barrierColor => Theme.of(navigator!.context).colorScheme.surface.withOpacity(0.5);

  // ... 其他实现
}
```

## barrierLabel

```dart 1787:1815:packages/flutter/lib/src/widgets/routes.dart
  /// {@template flutter.widgets.ModalRoute.barrierLabel}
  /// The semantic label used for a dismissible barrier.
  ///
  /// If the barrier is dismissible, this label will be read out if
  /// accessibility tools (like VoiceOver on iOS) focus on the barrier.
  ///
  /// The modal barrier is the scrim that is rendered behind each route, which
  /// generally prevents the user from interacting with the route below the
  /// current route, and normally partially obscures such routes.
  ///
  /// For example, when a dialog is on the screen, the page below the dialog is
  /// usually darkened by the modal barrier.
  /// {@endtemplate}
  ///
  /// If this getter would ever start returning a different label,
  /// either [changedInternalState] or [changedExternalState] should
  /// be invoked so that the change can take effect.
  ///
  /// It is safe to use `navigator.context` to look up inherited
  /// widgets here, because the [Navigator] calls
  /// [changedExternalState] whenever its dependencies change, and
  /// [changedExternalState] causes the modal barrier to rebuild.
  ///
  /// See also:
  ///
  ///  * [barrierDismissible], which controls the behavior of the barrier when
  ///    tapped.
  ///  * [ModalBarrier], the widget that implements this feature.
  String? get barrierLabel;
```

**功能**：用于可关闭屏障的语义标签。

**无障碍支持**：如果屏障是可关闭的（`barrierDismissible` 为 `true`），当无障碍工具（如 iOS 的 VoiceOver）聚焦到屏障时，会读出这个标签。

**使用场景**：提供有意义的无障碍标签，帮助视障用户理解屏障的用途。

**示例**：

```dart
class AccessibleDialogRoute extends ModalRoute<void> {
  @override
  bool get barrierDismissible => true;

  @override
  String? get barrierLabel => 'Tap to close dialog';

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Dialog(
      child: Text('Accessible Dialog'),
    );
  }

  // ... 其他必需的方法实现
}
```

## barrierCurve

```dart 1817:1846:packages/flutter/lib/src/widgets/routes.dart
  /// The curve that is used for animating the modal barrier in and out.
  ///
  /// The modal barrier is the scrim that is rendered behind each route, which
  /// generally prevents the user from interacting with the route below the
  /// current route, and normally partially obscures such routes.
  ///
  /// For example, when a dialog is on the screen, the page below the dialog is
  /// usually darkened by the modal barrier.
  ///
  /// While the route is animating into position, the color is animated from
  /// transparent to the specified [barrierColor].
  ///
  /// If this getter would ever start returning a different curve,
  /// either [changedInternalState] or [changedExternalState] should
  /// be invoked so that the change can take effect.
  ///
  /// It is safe to use `navigator.context` to look up inherited
  /// widgets here, because the [Navigator] calls
  /// [changedExternalState] whenever its dependencies change, and
  /// [changedExternalState] causes the modal barrier to rebuild.
  ///
  /// It defaults to [Curves.ease].
  ///
  /// See also:
  ///
  ///  * [barrierColor], which determines the color that the modal transitions
  ///    to.
  ///  * [Curves] for a collection of common curves.
  ///  * [AnimatedModalBarrier], the widget that implements this feature.
  Curve get barrierCurve => Curves.ease;
```

**功能**：用于动画化模态屏障进入和退出时的曲线。

**动画行为**：当路由进入位置时，屏障颜色从透明动画过渡到指定的 `barrierColor`，这个过渡使用 `barrierCurve` 定义的曲线。

**默认值**：`Curves.ease`

**可用的曲线**：可以使用 `Curves` 类中的任何曲线，例如：

- `Curves.ease`（默认）
- `Curves.easeIn`
- `Curves.easeOut`
- `Curves.easeInOut`
- `Curves.linear`
- `Curves.bounceOut`
- 等等

**使用示例**：

```dart
class AnimatedBarrierRoute extends ModalRoute<void> {
  @override
  Color? get barrierColor => Colors.black54;

  @override
  Curve get barrierCurve => Curves.easeOut; // 使用缓出曲线

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Dialog(
      child: Text('Animated Barrier'),
    );
  }

  // ... 其他必需的方法实现
}
```

## 屏障属性总结

### 属性对比表

| 属性 | 类型 | 默认值 | 功能 |
| --- | --- | --- | --- |
| `barrierDismissible` | `bool` | 抽象（必须实现） | 控制是否可以通过点击屏障关闭路由 |
| `semanticsDismissible` | `bool` | `true` | 控制屏障语义是否包含在语义树中 |
| `barrierColor` | `Color?` | 抽象（必须实现） | 控制屏障颜色 |
| `barrierLabel` | `String?` | 抽象（必须实现） | 无障碍标签 |
| `barrierCurve` | `Curve` | `Curves.ease` | 屏障动画曲线 |

### 完整使用示例

```dart
class CustomModalRoute extends ModalRoute<void> {
  @override
  bool get barrierDismissible => true; // 可通过点击屏障关闭

  @override
  bool get semanticsDismissible => true; // 包含在语义树中

  @override
  Color? get barrierColor => Colors.black.withOpacity(0.5); // 半透明黑色

  @override
  String? get barrierLabel => 'Tap to dismiss'; // 无障碍标签

  @override
  Curve get barrierCurve => Curves.easeOut; // 缓出曲线

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Center(
      child: Material(
        child: Container(
          width: 200,
          height: 200,
          color: Colors.white,
          child: Center(
            child: Text('Custom Modal'),
          ),
        ),
      ),
    );
  }

  // ... 其他必需的方法实现
}
```

## 总结

第六部分详细介绍了 `ModalRoute` 的模态屏障相关属性：

1. **barrierDismissible**：控制屏障的可关闭性
2. **semanticsDismissible**：控制屏障的无障碍语义支持
3. **barrierColor**：控制屏障的颜色
4. **barrierLabel**：提供无障碍标签
5. **barrierCurve**：控制屏障动画的曲线

这些属性共同定义了模态屏障的外观、行为和无障碍支持，为创建用户友好的模态路由提供了完整的控制能力。
