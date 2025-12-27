# SliverChildBuilderDelegate 代码讲解

## 概述

`SliverChildBuilderDelegate` 通过 `NullableIndexedWidgetBuilder` 懒构建 sliver 子节点，常用于 `SliverList`、`SliverGrid` 等。它内置保活、重绘隔离、语义索引包装，并支持键控重排。

## 构造与核心字段

```dart 352:528:lib/src/widgets/scroll_delegate.dart
class SliverChildBuilderDelegate extends SliverChildDelegate {
  const SliverChildBuilderDelegate(
    this.builder, {
    this.findChildIndexCallback,
    this.childCount,
    this.addAutomaticKeepAlives = true,
    this.addRepaintBoundaries = true,
    this.addSemanticIndexes = true,
    this.semanticIndexCallback = _kDefaultSemanticIndexCallback,
    this.semanticIndexOffset = 0,
  });

  final NullableIndexedWidgetBuilder builder;
  final int? childCount;
  final bool addAutomaticKeepAlives;
  final bool addRepaintBoundaries;
  final bool addSemanticIndexes;
  final int semanticIndexOffset;
  final SemanticIndexCallback semanticIndexCallback;
  final ChildIndexGetter? findChildIndexCallback;
}
```

- `builder`：必填，按索引返回子组件，返回 `null` 表示无更多子项。
- `childCount`：可选子项上限；缺省时通过 `builder` 返回 `null` 推断。
- `addAutomaticKeepAlives`：默认插入 `AutomaticKeepAlive` 以支持子树保活。
- `addRepaintBoundaries`：默认插入 `RepaintBoundary`，降低滚动重绘开销。
- `addSemanticIndexes` / `semanticIndexCallback` / `semanticIndexOffset`：生成连续语义索引，便于无障碍阅读器正确报读。
- `findChildIndexCallback`：在重排时根据 `Key` 查找新索引，避免状态丢失。

## 语义与可访问性

源文件注释强调多委托与部分标注场景：

- 多个委托共享同一滚动视口时，使用 `semanticIndexOffset` 补偿前序委托的语义计数，保持全局递增。
- 仅为部分子节点生成索引时（如 `ListView.separated` 的分隔符），通过 `semanticIndexCallback` 为无效节点返回 `null`。

## build 流程

```dart 544:573:lib/src/widgets/scroll_delegate.dart
@override
Widget? build(BuildContext context, int index) {
  if (index < 0 || (childCount != null && index >= childCount!)) {
    return null;
  }
  Widget? child;
  try {
    child = builder(context, index);
  } catch (exception, stackTrace) {
    child = _createErrorWidget(exception, stackTrace);
  }
  if (child == null) {
    return null;
  }
  final Key? key = child.key != null ? _SaltedValueKey(child.key!) : null;
  if (addRepaintBoundaries) {
    child = RepaintBoundary(child: child);
  }
  if (addSemanticIndexes) {
    final int? semanticIndex = semanticIndexCallback(child, index);
    if (semanticIndex != null) {
      child = IndexedSemantics(index: semanticIndex + semanticIndexOffset, child: child);
    }
  }
  if (addAutomaticKeepAlives) {
    child = AutomaticKeepAlive(child: _SelectionKeepAlive(child: child));
  }
  return KeyedSubtree(key: key, child: child);
}
```

- 边界检查：索引越界或超出 `childCount` 直接返回 `null`。
- 异常防护：构建异常时用 `_createErrorWidget` 展示错误，避免列表崩溃。
- 键包装：将原始 `child.key` 转为 `_SaltedValueKey`，便于后续 `findIndexByKey` 识别。
- 重绘隔离：可选 `RepaintBoundary` 降低滚动重绘。
- 语义索引：可选 `IndexedSemantics`，索引由 `semanticIndexCallback` 与偏移组成。
- 保活：可选 `AutomaticKeepAlive`，内部套 `_SelectionKeepAlive` 处理选择保持。
- 最终包装 `KeyedSubtree`，确保重用时保留元素关联。

## 其他重写

```dart 529:580:lib/src/widgets/scroll_delegate.dart
@override
int? findIndexByKey(Key key) { ... } // 使用 findChildIndexCallback

@override
int? get estimatedChildCount => childCount;

@override
bool shouldRebuild(covariant SliverChildBuilderDelegate oldDelegate) => true;
```

- `findIndexByKey`：处理 `_SaltedValueKey` 后转调用户提供的 `findChildIndexCallback`，支持可重排列表状态复用。
- `estimatedChildCount`：直接返回 `childCount`，便于滚动范围估算。
- `shouldRebuild`：始终返回 `true`，意味着替换为新实例时会重建子项；如需更激进的缓存，可自定义子类修改策略。

## 使用建议

- 优先提供 `childCount`，避免无限列表且子项尺寸为零时的内存风险。
- 需要稳定键控时同时提供子项 `Key` 与 `findChildIndexCallback`，保障重排时状态不丢。
- 当子项极轻量且滚动频繁，可关闭 `addRepaintBoundaries` 以减少层次；反之保持默认。
- 只有当外层已提供语义索引时才关闭 `addSemanticIndexes`，否则会影响无障碍朗读。
- 若列表内含可保活子树但已手动管理 keepalive，可关闭 `addAutomaticKeepAlives` 以避免重复包装。
