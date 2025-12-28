# ModalRoute 类详解 - 第十一部分：内部实现与 OverlayEntry

## 概述

`ModalRoute` 的内部实现涉及到 `OverlayEntry` 的创建、模态屏障和模态作用域的构建。这些是实现路由显示的核心机制。

## 内部字段定义

```dart 2242:2246:packages/flutter/lib/src/widgets/routes.dart
  // Internals

  final GlobalKey<_ModalScopeState<T>> _scopeKey = GlobalKey<_ModalScopeState<T>>();
  final GlobalKey _subtreeKey = GlobalKey();
  final PageStorageBucket _storageBucket = PageStorageBucket();
```

### _scopeKey

用于定位 `_ModalScope` 的 `State` 对象，用于访问路由的作用域状态。

### _subtreeKey

用于定位路由子树的全局键，通过它可以获取 `subtreeContext`。

### _storageBucket

页面存储桶，用于在路由之间保存和恢复状态（例如滚动位置）。

## 模态屏障构建

### _modalBarrier

```dart 2248:2249:packages/flutter/lib/src/widgets/routes.dart
  // one of the builders
  late OverlayEntry _modalBarrier;
```

`_modalBarrier` 是一个 `OverlayEntry`，用于在 `Overlay` 中显示模态屏障。

### _buildModalBarrier

```dart 2250:2265:packages/flutter/lib/src/widgets/routes.dart
  Widget _buildModalBarrier(BuildContext context) {
    Widget barrier = buildModalBarrier();
    if (filter != null) {
      barrier = BackdropFilter(filter: filter!, child: barrier);
    }
    barrier = IgnorePointer(
      ignoring: !animation!
          .isForwardOrCompleted, // changedInternalState is called when animation.status updates
      child: barrier, // dismissed is possible when doing a manual pop gesture
    );
    if (semanticsDismissible && barrierDismissible) {
      // To be sorted after the _modalScope.
      barrier = Semantics(sortKey: const OrdinalSortKey(1.0), child: barrier);
    }
    return barrier;
  }
```

**功能**：构建模态屏障的 widget 树。

**实现步骤**：

1. **构建基础屏障**：调用 `buildModalBarrier()` 获取基础屏障 widget

2. **应用滤镜**：如果 `filter` 不为 `null`，使用 `BackdropFilter` 包装屏障，实现模糊等效果

3. **处理指针事件**：
   - 使用 `IgnorePointer` 包装屏障
   - `ignoring` 参数：当动画不是前进或完成状态时，忽略指针事件
   - 这允许在手动 Pop 手势时正确处理交互

4. **添加语义支持**：如果 `semanticsDismissible` 和 `barrierDismissible` 都为 `true`，添加 `Semantics` widget 并设置 `sortKey` 为 `OrdinalSortKey(1.0)`（确保在 `_modalScope` 之后排序）

### buildModalBarrier

```dart 2267:2305:packages/flutter/lib/src/widgets/routes.dart
  /// Build the barrier for this [ModalRoute], subclasses can override
  /// this method to create their own barrier with customized features such as
  /// color or accessibility focus size.
  ///
  /// See also:
  /// * [ModalBarrier], which is typically used to build a barrier.
  /// * [ModalBottomSheetRoute], which overrides this method to build a
  ///   customized barrier.
  Widget buildModalBarrier() {
    Widget barrier;
    if (barrierColor != null && barrierColor!.alpha != 0 && !offstage) {
      // changedInternalState is called if barrierColor or offstage updates
      assert(barrierColor != barrierColor!.withOpacity(0.0));
      final Animation<Color?> color = animation!.drive(
        ColorTween(
          begin: barrierColor!.withOpacity(0.0),
          end: barrierColor, // changedInternalState is called if barrierColor updates
        ).chain(
          CurveTween(curve: barrierCurve),
        ), // changedInternalState is called if barrierCurve updates
      );
      barrier = AnimatedModalBarrier(
        color: color,
        dismissible:
            barrierDismissible, // changedInternalState is called if barrierDismissible updates
        semanticsLabel: barrierLabel, // changedInternalState is called if barrierLabel updates
        barrierSemanticsDismissible: semanticsDismissible,
      );
    } else {
      barrier = ModalBarrier(
        dismissible:
            barrierDismissible, // changedInternalState is called if barrierDismissible updates
        semanticsLabel: barrierLabel, // changedInternalState is called if barrierLabel updates
        barrierSemanticsDismissible: semanticsDismissible,
      );
    }

    return barrier;
  }
```

**功能**：构建屏障的基础 widget，子类可以重写此方法来自定义屏障。

**实现逻辑**：

1. **有颜色且不在 offstage**：
   - 创建颜色动画：从透明到 `barrierColor`
   - 应用 `barrierCurve` 曲线
   - 使用 `AnimatedModalBarrier` 实现动画效果

2. **无颜色或在 offstage**：
   - 使用 `ModalBarrier`（无动画）
   - 仍然传递 `barrierDismissible`、`semanticsLabel` 和 `barrierSemanticsDismissible`

**可重写性**：子类可以重写此方法来创建自定义的屏障，例如 `ModalBottomSheetRoute` 就重写了此方法。

## 模态作用域构建

### _modalScopeCache

```dart 2307:2309:packages/flutter/lib/src/widgets/routes.dart
  // We cache the part of the modal scope that doesn't change from frame to
  // frame so that we minimize the amount of building that happens.
  Widget? _modalScopeCache;
```

**功能**：缓存模态作用域 widget，避免每帧都重新构建。

**性能优化**：由于 `_ModalScope` 的内容不会频繁改变，缓存它可以减少不必要的重建。

### _buildModalScope

```dart 2311:2322:packages/flutter/lib/src/widgets/routes.dart
  // one of the builders
  Widget _buildModalScope(BuildContext context) {
    // To be sorted before the _modalBarrier.
    return _modalScopeCache ??= Semantics(
      sortKey: const OrdinalSortKey(0.0),
      child: _ModalScope<T>(
        key: _scopeKey,
        route: this,
        // _ModalScope calls buildTransitions() and buildChild(), defined above
      ),
    );
  }
```

**功能**：构建模态作用域 widget。

**实现**：

1. **使用缓存**：如果 `_modalScopeCache` 已存在，直接返回

2. **创建作用域**：
   - 使用 `Semantics` 包装，设置 `sortKey` 为 `OrdinalSortKey(0.0)`（确保在 `_modalBarrier` 之前排序）
   - 创建 `_ModalScope<T>`，传入 `_scopeKey` 和 `route: this`
   - `_ModalScope` 会调用 `buildTransitions()` 和 `buildChild()`（即 `buildPage`）

3. **缓存结果**：将结果缓存到 `_modalScopeCache`

### _modalScope

```dart 2324:2324:packages/flutter/lib/src/widgets/routes.dart
  late OverlayEntry _modalScope;
```

`_modalScope` 是一个 `OverlayEntry`，用于在 `Overlay` 中显示路由的内容。

## createOverlayEntries

```dart 2326:2336:packages/flutter/lib/src/widgets/routes.dart
  @override
  Iterable<OverlayEntry> createOverlayEntries() {
    return <OverlayEntry>[
      _modalBarrier = OverlayEntry(builder: _buildModalBarrier),
      _modalScope = OverlayEntry(
        builder: _buildModalScope,
        maintainState: maintainState,
        canSizeOverlay: opaque,
      ),
    ];
  }
```

**功能**：创建路由需要的所有 `OverlayEntry`。

**实现**：

返回两个 `OverlayEntry`：

1. **_modalBarrier**：
   - 使用 `_buildModalBarrier` 作为构建器
   - 屏障显示在路由内容后面

2. **_modalScope**：
   - 使用 `_buildModalScope` 作为构建器
   - `maintainState`：使用路由的 `maintainState` 值
   - `canSizeOverlay`：使用路由的 `opaque` 值（如果路由不透明，则可以调整 Overlay 大小）

**顺序重要性**：由于使用了 `OrdinalSortKey`，`_modalScope`（0.0）会显示在 `_modalBarrier`（1.0）之前，即内容显示在屏障上方。

## toString

```dart 2338:2341:packages/flutter/lib/src/widgets/routes.dart
  @override
  String toString() =>
      '${objectRuntimeType(this, 'ModalRoute')}($settings, animation: $_animation)';
```

**功能**：返回路由的字符串表示，用于调试。

**格式**：`ModalRoute(settings, animation: animationValue)`

## Overlay 工作原理

### OverlayEntry 的生命周期

```text
路由安装（install）
    ↓
createOverlayEntries() 创建 OverlayEntry
    ↓
OverlayEntry 被添加到 Overlay
    ↓
OverlayEntry.builder 被调用构建 widget
    ↓
widget 显示在 Overlay 中
    ↓
路由状态改变 → OverlayEntry.markNeedsBuild() → 重新构建
    ↓
路由移除 → OverlayEntry 从 Overlay 中移除
```

### 两个 OverlayEntry 的关系

```text
Overlay（从上到下）
├── _modalScope (sortKey: 0.0)
│   └── _ModalScope
│       └── buildTransitions()
│           └── buildPage() 的内容
│
└── _modalBarrier (sortKey: 1.0)
    └── ModalBarrier / AnimatedModalBarrier
```

**层级关系**：`_modalScope` 在上层，`_modalBarrier` 在下层，屏障作为背景显示。

## 自定义实现示例

### 示例：自定义屏障

```dart
class CustomBarrierRoute extends ModalRoute<void> {
  @override
  Widget buildModalBarrier() {
    // 自定义屏障实现
    return Container(
      color: Colors.black.withOpacity(0.3),
      child: GestureDetector(
        onTap: barrierDismissible ? () => Navigator.pop(context) : null,
        child: Container(
          decoration: BoxDecoration(
            gradient: RadialGradient(
              center: Alignment.center,
              radius: 1.0,
              colors: [
                Colors.transparent,
                Colors.black.withOpacity(0.5),
              ],
            ),
          ),
        ),
      ),
    );
  }

  @override
  Widget buildPage(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation) {
    return Scaffold(
      body: Center(child: Text('Custom Barrier Route')),
    );
  }

  // ... 其他必需的方法实现
}
```

## 性能优化

### 1. _modalScopeCache

缓存 `_ModalScope` widget 避免每帧重建，只有在必要时才重新构建。

### 2. OverlayEntry 的 maintainState

通过 `maintainState` 控制 `OverlayEntry` 的生命周期，可以在不需要时释放资源。

### 3. 条件构建

在 `_buildModalBarrier` 中根据状态条件性地添加滤镜和语义支持，避免不必要的 widget 包装。

## 总结

第十一部分介绍了 `ModalRoute` 的内部实现机制：

1. **内部字段**：`_scopeKey`、`_subtreeKey`、`_storageBucket` 用于状态管理和定位

2. **屏障构建**：
   - `buildModalBarrier()`：构建基础屏障（可重写）
   - `_buildModalBarrier()`：构建完整的屏障 widget 树（包含滤镜、指针处理、语义支持）

3. **作用域构建**：
   - `_buildModalScope()`：构建模态作用域，使用缓存优化性能

4. **OverlayEntry 创建**：`createOverlayEntries()` 创建两个 `OverlayEntry`，分别用于屏障和内容

5. **调试支持**：`toString()` 方法提供路由的字符串表示

这些实现细节共同构成了 `ModalRoute` 的核心功能，确保了路由的正确显示、交互和性能。
