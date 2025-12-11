# Scrollable 代码讲解

## 概述

`Scrollable` 是滚动交互的核心组件，负责手势识别、滚动位置管理和语义暴露，但不直接决定内容的布局或展示方式。它通常与 `Viewport`/`Sliver` 组合使用，由更高级的 `ScrollView` 家族（如 `ListView`、`GridView`、`CustomScrollView`）封装。

`ScrollableState` 则实现了滚动位置的创建、物理特性解析、手势绑定、滚轮处理、语义更新以及滚动装饰（如滚动条和 Overscroll 指示器）的构建。

## 类定义

```dart 121:151:lib/src/widgets/scrollable.dart
class Scrollable extends StatefulWidget {
  /// Creates a widget that scrolls.
  const Scrollable({
    super.key,
    this.axisDirection = AxisDirection.down,
    this.controller,
    this.physics,
    required this.viewportBuilder,
    this.incrementCalculator,
    this.excludeFromSemantics = false,
    this.semanticChildCount,
    this.dragStartBehavior = DragStartBehavior.start,
    this.restorationId,
    this.scrollBehavior,
    this.clipBehavior = Clip.hardEdge,
    this.hitTestBehavior = HitTestBehavior.opaque,
  }) : assert(semanticChildCount == null || semanticChildCount >= 0);
```

- 继承自 `StatefulWidget`，通过 `ScrollableState` 维护滚动位置和交互状态。
- 仅有一条断言：`semanticChildCount` 需为非负（或空），用于语义树的子节点计数。

## 关键参数与行为

### 滚动方向与控制

```dart 139:174:lib/src/widgets/scrollable.dart
final AxisDirection axisDirection;
final ScrollController? controller;
final ScrollPhysics? physics;
final ScrollBehavior? scrollBehavior;
Axis get axis => axisDirectionToAxis(axisDirection);
```

- `axisDirection`：决定滚动轴向（四个 `AxisDirection`），默认向下。
- `axis`：由 `axisDirection` 推导出 `Axis`（水平/垂直）。
- `controller`：若未提供则内部创建 `_fallbackScrollController`，并负责在 `ScrollableState` 中附着到位置对象。
- `physics` 与 `scrollBehavior`：物理特性优先级为显式 `physics` > `scrollBehavior.getScrollPhysics` > 继承的 `ScrollConfiguration`。仅当物理类型改变或 `ScrollBehavior` 变更时才重建位置（避免频繁重建）。

### 视口构建与键盘增量

```dart 204:230:lib/src/widgets/scrollable.dart
final ViewportBuilder viewportBuilder;
final ScrollIncrementCalculator? incrementCalculator;
```

- `viewportBuilder`：必填，用于创建具体的 `Viewport`（如 `Viewport` 或 `ShrinkWrappingViewport`）并接收 `ViewportOffset`。
- `incrementCalculator`：自定义键盘滚动增量；若为空，`ScrollAction` 使用默认策略（页滚动 80% 视窗、高度/宽度，行滚动 50 逻辑像素）。

### 语义、手势与裁剪

```dart 231:335:lib/src/widgets/scrollable.dart
final bool excludeFromSemantics;
final int? semanticChildCount;
final DragStartBehavior dragStartBehavior;
final String? restorationId;
final Clip clipBehavior;
final HitTestBehavior hitTestBehavior;
```

- `excludeFromSemantics`：是否在语义树暴露滚动动作（如可输入溢出的文本域通常设为 true）。
- `semanticChildCount`：告知语义层子节点数量（列表类视图可自动推断）。
- `dragStartBehavior`：手势起始位置（`start` 默认，或 `down`）。
- `restorationId`：启用状态恢复时用于保存/恢复滚动偏移。
- `clipBehavior`：传递给装饰/viewport 的裁剪方式，默认 `Clip.hardEdge`。
- `hitTestBehavior`：手势命中行为，默认 `opaque` 防止事件穿透。

## 查找与工具方法

### maybeOf / of：定位最近的 Scrollable

```dart 379:453:lib/src/widgets/scrollable.dart
static ScrollableState? maybeOf(BuildContext context, {Axis? axis}) { ... }
static ScrollableState of(BuildContext context, {Axis? axis}) { ... }
```

- 从祖先链查找最近的 `_ScrollableScope`，可通过可选 `axis` 精确到指定轴向的嵌套滚动。
- `maybeOf` 返回可空并建立依赖；`of` 在找不到时断言/抛异常，调试提示包含 Axis 过滤信息。
- 当 widget 自身即为 `Scrollable` 时，查找从其父级开始（不会返回自身）。

### recommendDeferredLoadingForContext：推荐延迟加载

```dart 471:483:lib/src/widgets/scrollable.dart
static bool recommendDeferredLoadingForContext(BuildContext context, {Axis? axis}) { ... }
```

- 不建立依赖，直接沿祖先 `_ScrollableScope` 检查物理层的 `recommendDeferredLoading`（基于当前位置的速度）来判断是否应延迟昂贵的帧内加载。

### ensureVisible：滚动使目标可见

```dart 485:533:lib/src/widgets/scrollable.dart
static Future<void> ensureVisible(
  BuildContext context, {
    double alignment = 0.0,
    Duration duration = Duration.zero,
    Curve curve = Curves.ease,
    ScrollPositionAlignmentPolicy alignmentPolicy = ScrollPositionAlignmentPolicy.explicit,
  }) { ... }
```

- 从给定 `context` 向外层迭代所有包裹的 `Scrollable`，逐层调用 `_performEnsureVisible`。
- 支持对齐策略、动画时长/曲线，若 `duration` 为零或无可滚动项直接返回已完成的 `Future`。
- 嵌套场景下优先让最内层目标 `RenderObject` 可见，再让外层尽可能曝光同一目标，提升用户体验。

## ScrollableState 核心逻辑

```dart 560:576:lib/src/widgets/scrollable.dart
class ScrollableState extends State<Scrollable>
    with TickerProviderStateMixin, RestorationMixin
    implements ScrollContext {
  ScrollPosition get position => _position!;
  ScrollPhysics? get resolvedPhysics => _physics;
```

- 提供 `ScrollContext` 能力（`axisDirection`、`vsync`、`devicePixelRatio`、`notificationContext`、`storageContext` 等），供 `ScrollPosition` 使用。
- 使用 `RestorationMixin` 记录/恢复滚动偏移，`TickerProviderStateMixin` 为物理动画提供 vsync。

### 滚动位置创建与更新

```dart 616:636:lib/src/widgets/scrollable.dart
void _updatePosition() {
  _configuration = widget.scrollBehavior ?? ScrollConfiguration.of(context);
  final ScrollPhysics? physicsFromWidget =
      widget.physics ?? widget.scrollBehavior?.getScrollPhysics(context);
  _physics = _configuration.getScrollPhysics(context);
  _physics = physicsFromWidget?.applyTo(_physics) ?? _physics;

  final ScrollPosition? oldPosition = _position;
  if (oldPosition != null) {
    _effectiveScrollController.detach(oldPosition);
    scheduleMicrotask(oldPosition.dispose);
  }

  _position = _effectiveScrollController.createScrollPosition(_physics!, this, oldPosition);
  _effectiveScrollController.attach(position);
}
```

- 解析物理层顺序：继承配置 → 显式/局部行为叠加 → 显式 physics 最终覆盖。
- 先 detach 旧位置并延后 dispose，避免视口监听尚未解除时立即销毁。
- 始终通过当前有效 controller 创建并重新附着位置。

### 生命周期与控制器切换

- `initState`：若无外部 `controller`，创建 `_fallbackScrollController`。
- `didChangeDependencies`：更新手势设置、设备像素比并重建位置（首次）。
- `_shouldUpdatePosition`：检测 `ScrollBehavior` 变更或物理链类型变更（逐级比较 runtimeType）以及 controller 类型变化。
- `didUpdateWidget`：
  - 处理 controller 更换：旧为 null 时释放 fallback，旧非空则 detach；新为 null 时补建 fallback。
  - 若需要则调用 `_updatePosition()`。
- `dispose`：按来源 detach controller，释放 fallback/position/restorable offset。

### 状态恢复

- `restoreState`：注册 `_persistedScrollOffset` 并在有值时调用 `position.restoreOffset`。
- `saveOffset`：保存偏移并主动刷新 restoration 数据（通常滚动结束后不会立刻触发帧）。

### 手势、拖拽与忽略指针

```dart 775:841:lib/src/widgets/scrollable.dart
void setCanDrag(bool value) { ... }
void setIgnorePointer(bool value) { ... }
```

- 根据 `axis` 构建竖/横向 `GestureRecognizerFactory`，并将物理学配置（最小/最大甩动速度、追踪器、拖拽策略、起始行为、支持设备）传入。
- 禁用拖拽时清空识别器并取消当前拖拽/保持。
- `setIgnorePointer` 通过 `_ignorePointerKey` 定位 `RenderIgnorePointer`，动态切换事件屏蔽。

### 拖拽流程

```dart 862:917:lib/src/widgets/scrollable.dart
void _handleDragDown(DragDownDetails details) { ... }
void _handleDragStart(DragStartDetails details) { ... }
void _handleDragUpdate(DragUpdateDetails details) { ... }
void _handleDragEnd(DragEndDetails details) { ... }
void _handleDragCancel() { ... }
```

- `hold`：在按下时通过 `position.hold` 冻结当前活动。
- `drag`：在开始时通过 `position.drag` 获取 `Drag`，拖拽过程中调用 `update`，结束时 `end`。
- 取消时确保 `hold`/`drag` 均被清理，且处理组件被卸载时的早退。

### 滚轮与指针信号

```dart 920:984:lib/src/widgets/scrollable.dart
double _pointerSignalEventDelta(PointerScrollEvent event) { ... }
void _receivedPointerSignal(PointerSignalEvent event) { ... }
void _handlePointerScroll(PointerEvent event) { ... }
```

- 支持按物理滚轮修饰键翻转轴向（仅鼠标，触摸板不翻转）。
- 将指针增量映射到当前 axis，并考虑 axisDirection 是否反向。
- 若物理学不接受用户偏移则允许平台默认行为；否则注册到 `pointerSignalResolver` 并调用 `position.pointerScroll`。
- 处理 `PointerScrollInertiaCancelEvent` 停止惯性滚动。

### 语义与滚动装饰

- `setSemanticsActions`：通过 `_gestureDetectorKey` 更新手势层的可用语义动作。
- `_handleScrollMetricsNotification`：深度为 0 时刷新滚动语义节点。
- `_buildChrome`：借助 `_configuration` 先包裹 overscroll 指示器，再包裹滚动条，传入 `ScrollableDetails`（方向、controller、裁剪策略）。

### build 流程

```dart 1015:1075:lib/src/widgets/scrollable.dart
Widget build(BuildContext context) {
  Widget result = _ScrollableScope(
    scrollable: this,
    position: position,
    child: Listener(
      onPointerSignal: _receivedPointerSignal,
      child: RawGestureDetector(
        key: _gestureDetectorKey,
        gestures: _gestureRecognizers,
        behavior: widget.hitTestBehavior,
        excludeFromSemantics: widget.excludeFromSemantics,
        child: Semantics(
          explicitChildNodes: !widget.excludeFromSemantics,
          child: IgnorePointer(
            key: _ignorePointerKey,
            ignoring: _shouldIgnorePointer,
            child: widget.viewportBuilder(context, position),
          ),
        ),
      ),
    ),
  );
  ...
}
```

- 结构自内向外：`viewportBuilder` → `IgnorePointer` → `Semantics` → `RawGestureDetector` → `Listener`（指针信号）→ `_ScrollableScope`。
- 若未排除语义，包裹 `_ScrollSemantics` 监听 `ScrollMetricsNotification`，并提供 `semanticChildCount`/`allowImplicitScrolling`。
- 最外层通过 `_buildChrome` 注入 overscroll/Scrollbar。
- 若存在父级 `SelectionRegistrar`，再包裹 `_ScrollableSelectionHandler` 启用滚动选择协同。

### 确保可见的内部实现

```dart 1078:1097:lib/src/widgets/scrollable.dart
_EnsureVisibleResults _performEnsureVisible(
  RenderObject object, {
    double alignment = 0.0,
    Duration duration = Duration.zero,
    Curve curve = Curves.ease,
    ScrollPositionAlignmentPolicy alignmentPolicy = ScrollPositionAlignmentPolicy.explicit,
    RenderObject? targetRenderObject,
  }) {
  final Future<void> ensureVisibleFuture = position.ensureVisible(
    object,
    alignment: alignment,
    duration: duration,
    curve: curve,
    alignmentPolicy: alignmentPolicy,
    targetRenderObject: targetRenderObject,
  );
  return (<Future<void>>[ensureVisibleFuture], this);
}
```

- 直接委托给当前 `position.ensureVisible`，并返回自身用于外层循环继续处理其他祖先 `Scrollable`。

## 总结与使用建议

- `Scrollable` 负责手势/物理/语义/装饰，`Viewport` 负责可视区域，`Sliver` 负责布局；通常通过 `ScrollView` 子类组合使用。
- 自定义行为的入口：
  - 物理与滚动效果：传入自定义 `ScrollPhysics` 或 `ScrollBehavior`
  - 键盘滚动：实现 `incrementCalculator`
  - 语义与命中：使用 `excludeFromSemantics`、`semanticChildCount`、`hitTestBehavior`
  - 状态恢复：提供 `restorationId`
- 嵌套滚动场景可通过 `axis` 参数使用 `maybeOf`/`of`/`recommendDeferredLoadingForContext` 精确选择目标滚动实例，并用 `ensureVisible` 协调多层滚动使元素可见。
