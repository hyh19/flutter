# SliverChildDelegate 代码讲解

## 概述

`SliverChildDelegate` 为 sliver 提供子组件来源，常用于 `SliverList`、`SliverGrid` 等懒加载场景。大多数情况下直接使用现有子类（如 `SliverChildBuilderDelegate`、`SliverChildListDelegate`），很少需要自定义实现。

## 类定义与职责

```dart 135:234:lib/src/widgets/scroll_delegate.dart
abstract class SliverChildDelegate {
  const SliverChildDelegate();

  Widget? build(BuildContext context, int index);
  int? get estimatedChildCount => null;
  double? estimateMaxScrollOffset(
    int firstIndex,
    int lastIndex,
    double leadingScrollOffset,
    double trailingScrollOffset,
  ) => null;
  void didFinishLayout(int firstIndex, int lastIndex) {}
  bool shouldRebuild(covariant SliverChildDelegate oldDelegate);
  int? findIndexByKey(Key key) => null;

  @override
  String toString() {
    final List<String> description = <String>[];
    debugFillDescription(description);
    return '${describeIdentity(this)}(${description.join(", ")})';
  }

  @protected
  @mustCallSuper
  void debugFillDescription(List<String> description) {
    try {
      final int? children = estimatedChildCount;
      if (children != null) {
        description.add('estimated child count: $children');
      }
    } catch (e) {
      description.add('estimated child count: EXCEPTION (${e.runtimeType})');
    }
  }
}
```

- 抽象基类：仅定义接口与默认行为，不直接持有子节点数据。
- 约束：`build` 返回 `null` 后，`estimatedChildCount` 必须返回精确数量以供 `RenderSliverBoxChildManager` 使用。
- `toString` 与 `debugFillDescription` 便于调试，默认输出子数量估计或异常信息。

## 生命周期与状态保持

源文件注释详细阐述子元素创建、销毁及状态保留策略：

- **懒创建/销毁**：仅为视口内可见位置构建元素，滚出视口即销毁其元素、状态与渲染对象。
- **状态保留**：通过两种手段避免销毁造成的状态丢失：
  - 将业务状态上移到列表外部数据模型，子树仅保存瞬时 UI 状态。
  - 使用 `KeepAlive` 或默认插入的 `AutomaticKeepAlive` 包装子树；`AutomaticKeepAliveClientMixin` 可在需要时主动请求保活。
- **多委托场景**：同一 `Viewport` 使用多个委托时，每个委托的首个子节点都会被布局，以便 `estimateMaxScrollOffset` 估算整体滚动范围。

## 核心方法说明

### build

```dart 140:153:lib/src/widgets/scroll_delegate.dart
Widget? build(BuildContext context, int index);
```

- 返回指定索引的子组件，超出范围返回 `null`。
- 子类通常在此包裹 `AutomaticKeepAlive`、`IndexedSemantics`、`RepaintBoundary`，以控制保活、无障碍索引和绘制隔离。
- 如果子列表发生变化，需要提供新的 delegate 并让 `shouldRebuild` 返回 `true`，否则缓存的子组件不会刷新。

### estimatedChildCount

```dart 155:165:lib/src/widgets/scroll_delegate.dart
int? get estimatedChildCount => null;
```

- 为滚动范围估算提供子节点数量；返回 `null` 表示数量未知或无限。
- 一旦 `build` 返回 `null`，此 getter 必须返回精确值，否则滚动边界计算会出错。

### estimateMaxScrollOffset

```dart 167:179:lib/src/widgets/scroll_delegate.dart
double? estimateMaxScrollOffset(
  int firstIndex,
  int lastIndex,
  double leadingScrollOffset,
  double trailingScrollOffset,
) => null;
```

- 可根据首尾已布局子项的偏移，估算全部子项的最大滚动范围。
- 默认 `null`，由调用方按已知子项均值推算；自定义子类可结合精确尺寸或分段数据改进估算，减少跳变。

### didFinishLayout

```dart 181:189:lib/src/widgets/scroll_delegate.dart
void didFinishLayout(int firstIndex, int lastIndex) {}
```

- 布局结束回调，告知本次布局覆盖的首尾索引。
- 适合用于统计曝光、预加载下一页、或调试记录。

### shouldRebuild

```dart 191:200:lib/src/widgets/scroll_delegate.dart
bool shouldRebuild(covariant SliverChildDelegate oldDelegate);
```

- 当新的 delegate 实例替换旧实例时调用。返回 `true` 表示需要重建子组件，`false` 时 `build` 可能被跳过以复用缓存。
- 合理实现可避免不必要的重新构建，提升滚动性能。

### findIndexByKey

```dart 202:211:lib/src/widgets/scroll_delegate.dart
int? findIndexByKey(Key key) => null;
```

- 在重建过程中根据子节点 `Key` 查找其新索引，返回 `null` 表示未找到。
- 若未实现此方法，当子顺序变化时，现有 `RenderObject` 可能无法复用，导致状态丢失；适用于可重排列表（如拖拽排序）。

## 使用建议

- 优先使用现成子类：`SliverChildBuilderDelegate` 适合懒加载构建，`SliverChildListDelegate` 适合已知列表。
- 自定义子类时，确保 `build`、`estimatedChildCount` 与 `shouldRebuild` 语义一致，避免滚动边界或缓存异常。
- 需要稳定键控时实现 `findIndexByKey`，并在子组件上提供稳定的 `Key`。
- 在多委托或双向增长视口中，留意首子节点必定布局的特性，避免首项过大导致首帧开销。
