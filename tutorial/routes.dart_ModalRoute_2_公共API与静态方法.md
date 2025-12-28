# ModalRoute 公共API与静态方法详解

## 概述

`ModalRoute` 提供了丰富的静态方法，允许开发者在任何 `BuildContext` 中访问当前最接近的模态路由及其属性。这些方法通过 `InheritedModel` 机制实现了高效的依赖追踪和自动重建。

## 核心静态方法

### `of<T>`

```dart 1273:1291:packages/flutter/lib/src/widgets/routes.dart
  /// Returns the modal route most closely associated with the given context.
  ///
  /// Returns null if the given context is not associated with a modal route.
  ///
  /// {@tool snippet}
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// ModalRoute<int>? route = ModalRoute.of<int>(context);
  /// ```
  /// {@end-tool}
  ///
  /// The given [BuildContext] will be rebuilt if the state of the route changes
  /// while it is visible (specifically, if [isCurrent] or [canPop] change value).
  @optionalTypeArgs
  static ModalRoute<T>? of<T extends Object?>(BuildContext context) {
    return _of<T>(context);
  }

  static ModalRoute<T>? _of<T extends Object?>(BuildContext context, [_ModalRouteAspect? aspect]) {
    return InheritedModel.inheritFrom<_ModalScopeStatus>(context, aspect: aspect)?.route
        as ModalRoute<T>?;
  }
```

**功能**：返回与给定 `context` 最接近的模态路由。

**返回值**：

- 如果 `context` 与模态路由关联，返回对应的 `ModalRoute<T>`
- 否则返回 `null`

**关键特性**：

1. **自动重建**：当路由状态改变时（特别是 `isCurrent` 或 `canPop` 改变时），使用此方法的 widget 会自动重建
2. **泛型支持**：使用 `@optionalTypeArgs` 注解，支持类型推断
3. **实现机制**：通过 `InheritedModel.inheritFrom` 从 widget 树中获取 `_ModalScopeStatus`，然后获取其关联的路由

### isCurrentOf

```dart 1298:1306:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [ModalRoute.isCurrent] for the modal route most closely associated
  /// with the given context.
  ///
  /// Returns null if the given context is not associated with a modal route.
  ///
  /// Use of this method will cause the given [context] to rebuild any time that
  /// the [ModalRoute.isCurrent] property of the ancestor [_ModalScopeStatus] changes.
  static bool? isCurrentOf(BuildContext context) =>
      _of(context, _ModalRouteAspect.isCurrent)?.isCurrent;
```

**功能**：返回最接近模态路由的 `isCurrent` 属性值。

**依赖追踪**：使用此方法会在 `isCurrent` 属性改变时触发 widget 重建。

**使用场景**：判断当前路由是否是栈顶路由（当前激活的路由）。

### canPopOf

```dart 1308:1315:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [ModalRoute.canPop] for the modal route most closely associated
  /// with the given context.
  ///
  /// Returns null if the given context is not associated with a modal route.
  ///
  /// Use of this method will cause the given [context] to rebuild any time that
  /// the [ModalRoute.canPop] property of the ancestor [_ModalScopeStatus] changes.
  static bool? canPopOf(BuildContext context) => _of(context, _ModalRouteAspect.canPop)?.canPop;
```

**功能**：返回最接近模态路由的 `canPop` 属性值。

**依赖追踪**：使用此方法会在 `canPop` 属性改变时触发 widget 重建。

**使用场景**：判断当前路由是否可以被弹出，通常用于控制返回按钮的显示/隐藏。

### settingsOf

```dart 1317:1326:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [ModalRoute.settings] for the modal route most closely associated
  /// with the given context.
  ///
  /// Returns null if the given context is not associated with a modal route.
  ///
  /// Calling this method creates a dependency on the [ModalRoute] associated
  /// with the given [context]. As a result, the widget corresponding to [context]
  /// will be rebuilt whenever the route's [ModalRoute.settings] changes.
  static RouteSettings? settingsOf(BuildContext context) =>
      _of(context, _ModalRouteAspect.settings)?.settings;
```

**功能**：返回最接近模态路由的 `settings` 属性值。

**依赖追踪**：当路由的 `settings` 改变时，使用此方法的 widget 会重建。

**使用场景**：获取路由的配置信息，如路由名称、参数等。

### isActiveOf

```dart 1328:1337:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [ModalRoute.isActive] for the modal route most closely associated
  /// with the given context.
  ///
  /// Returns null if the given context is not associated with a modal route.
  ///
  /// Calling this method creates a dependency on the [ModalRoute] associated
  /// with the given [context]. As a result, the widget corresponding to [context]
  /// will be rebuilt whenever the route's [ModalRoute.isActive] changes.
  static bool? isActiveOf(BuildContext context) =>
      _of(context, _ModalRouteAspect.isActive)?.isActive;
```

**功能**：返回最接近模态路由的 `isActive` 属性值。

**依赖追踪**：当路由的 `isActive` 改变时，使用此方法的 widget 会重建。

**使用场景**：判断路由是否处于活动状态（在导航堆栈中且可见）。

### isFirstOf

```dart 1339:1347:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [ModalRoute.isFirst] for the modal route most closely associated
  /// with the given context.
  ///
  /// Returns null if the given context is not associated with a modal route.
  ///
  /// Calling this method creates a dependency on the [ModalRoute] associated
  /// with the given [context]. As a result, the widget corresponding to [context]
  /// will be rebuilt whenever the route's [ModalRoute.isFirst] changes.
  static bool? isFirstOf(BuildContext context) => _of(context, _ModalRouteAspect.isFirst)?.isFirst;
```

**功能**：返回最接近模态路由的 `isFirst` 属性值。

**依赖追踪**：当路由的 `isFirst` 改变时，使用此方法的 widget 会重建。

**使用场景**：判断路由是否是导航堆栈中的第一个路由（根路由）。

### opaqueOf

```dart 1349:1357:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [ModalRoute.opaque] for the modal route most closely associated
  /// with the given context.
  ///
  /// Returns null if the given context is not associated with a modal route.
  ///
  /// Calling this method creates a dependency on the [ModalRoute] associated
  /// with the given [context]. As a result, the widget corresponding to [context]
  /// will be rebuilt whenever the route's [ModalRoute.opaque] changes.
  static bool? opaqueOf(BuildContext context) => _of(context, _ModalRouteAspect.opaque)?.opaque;
```

**功能**：返回最接近模态路由的 `opaque` 属性值。

**依赖追踪**：当路由的 `opaque` 改变时，使用此方法的 widget 会重建。

**使用场景**：判断路由是否完全不透明（即是否完全遮挡下方的路由）。

### popDispositionOf

```dart 1359:1368:packages/flutter/lib/src/widgets/routes.dart
  /// Returns [ModalRoute.popDisposition] for the modal route most closely associated
  /// with the given context.
  ///
  /// Returns null if the given context is not associated with a modal route.
  ///
  /// Calling this method creates a dependency on the [ModalRoute] associated
  /// with the given [context]. As a result, the widget corresponding to [context]
  /// will be rebuilt whenever the route's [ModalRoute.popDisposition] changes.
  static RoutePopDisposition? popDispositionOf(BuildContext context) =>
      _of(context, _ModalRouteAspect.popDisposition)?.popDisposition;
```

**功能**：返回最接近模态路由的 `popDisposition` 属性值。

**依赖追踪**：当路由的 `popDisposition` 改变时，使用此方法的 widget 会重建。

**使用场景**：获取路由的弹出处置方式，用于判断是否可以弹出路由。

## 依赖追踪机制

所有静态方法都通过 `_ModalRouteAspect` 参数实现细粒度的依赖追踪。`_ModalRouteAspect` 是一个枚举类型，用于指定需要追踪的路由属性的哪个方面。

### _of 方法的 aspect 参数

```dart 1293:1296:packages/flutter/lib/src/widgets/routes.dart
  static ModalRoute<T>? _of<T extends Object?>(BuildContext context, [_ModalRouteAspect? aspect]) {
    return InheritedModel.inheritFrom<_ModalScopeStatus>(context, aspect: aspect)?.route
        as ModalRoute<T>?;
  }
```

`aspect` 参数允许指定只关注路由的特定属性，这样可以：

1. **提高性能**：只在相关属性改变时触发重建
2. **精确控制**：避免不必要的重建
3. **类型安全**：通过枚举确保只追踪有效的属性

## 使用示例

### 示例 1：判断是否可以返回

```dart
Widget build(BuildContext context) {
  final canPop = ModalRoute.of(context)?.canPop ?? false;

  return Scaffold(
    appBar: AppBar(
      leading: canPop
          ? BackButton() // 可以返回时显示返回按钮
          : SizedBox.shrink(), // 不能返回时隐藏
    ),
    body: YourContent(),
  );
}
```

### 示例 2：获取路由设置

```dart
Widget build(BuildContext context) {
  final settings = ModalRoute.of(context)?.settings;
  final routeName = settings?.name;
  final arguments = settings?.arguments;

  return Text('Route: $routeName, Args: $arguments');
}
```

### 示例 3：监听路由状态变化

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 当 isCurrent 改变时，widget 会自动重建
    final isCurrent = ModalRoute.isCurrentOf(context) ?? false;

    return Container(
      color: isCurrent ? Colors.green : Colors.grey,
      child: Text(isCurrent ? 'Current Route' : 'Inactive Route'),
    );
  }
}
```

## 性能考虑

1. **选择性依赖**：使用 `*Of` 方法而不是 `of` 方法可以更精确地指定依赖的属性
2. **避免过度重建**：只在需要时调用这些方法，避免在频繁重建的 widget 中使用
3. **缓存结果**：如果只需要在特定生命周期阶段使用，可以考虑在 `initState` 中获取并缓存

## 总结

第二部分介绍了 `ModalRoute` 的公共 API 和静态方法。这些方法提供了便捷的方式来访问当前路由的状态和属性，并通过 `InheritedModel` 机制实现了高效的依赖追踪和自动重建功能。
