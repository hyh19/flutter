# SliverChildListDelegate 代码讲解

## 概述

`SliverChildListDelegate` 使用显式子组件列表提供 sliver 子节点，适合子项数量固定且较小、或子项为编译期常量的场景。相比 `SliverChildBuilderDelegate`，它不按需构建，因此大规模列表性能较弱。

## 构造与核心字段

```dart 633:683:lib/src/widgets/scroll_delegate.dart
class SliverChildListDelegate extends SliverChildDelegate {
  SliverChildListDelegate(
    this.children, {
    this.addAutomaticKeepAlives = true,
    this.addRepaintBoundaries = true,
    this.addSemanticIndexes = true,
    this.semanticIndexCallback = _kDefaultSemanticIndexCallback,
    this.semanticIndexOffset = 0,
  }) : _keyToIndex = <Key?, int>{null: 0};

  const SliverChildListDelegate.fixed(
    this.children, {
    this.addAutomaticKeepAlives = true,
    this.addRepaintBoundaries = true,
    this.addSemanticIndexes = true,
    this.semanticIndexCallback = _kDefaultSemanticIndexCallback,
    this.semanticIndexOffset = 0,
  }) : _keyToIndex = null;

  final bool addAutomaticKeepAlives;
  final bool addRepaintBoundaries;
  final bool addSemanticIndexes;
  final int semanticIndexOffset;
  final SemanticIndexCallback semanticIndexCallback;
  final List<Widget> children;
  final Map<Key?, int>? _keyToIndex;
}
```

- 提供可变构造和 `fixed` 常量构造：后者用于子序列与键稳定的场景，跳过键索引缓存。
- `_keyToIndex`：懒填充的键到索引映射，辅助重排时的状态复用；`fixed` 构造下为 `null`，表示不做动态索引缓存。
- 继承的保活、重绘隔离、语义索引开关与 Builder 版本一致。

## 语义与可访问性

- 与 Builder 版本相同，默认包装 `IndexedSemantics`，支持 `semanticIndexOffset` 跨委托对齐。
- 支持通过 `semanticIndexCallback` 跳过部分子项（如分隔符），保持语义索引连续。

## 键查找与重排

```dart 729:765:lib/src/widgets/scroll_delegate.dart
int? _findChildIndex(Key key) { ... } // 懒填充 _keyToIndex

@override
int? findIndexByKey(Key key) {
  final Key childKey = key is _SaltedValueKey ? key.value : key;
  return _findChildIndex(childKey);
}
```

- 懒加载 `_keyToIndex`：遍历 `children` 直到找到目标键并缓存中间结果，避免每次从头扫描。
- 通过 `_SaltedValueKey` 还原原始键，保持与 `build` 包装逻辑一致。
- 若使用 `fixed` 构造或未设置键，则该映射不可用，返回 `null`。

## build 流程

```dart 767:788:lib/src/widgets/scroll_delegate.dart
@override
Widget? build(BuildContext context, int index) {
  if (index < 0 || index >= children.length) {
    return null;
  }
  Widget child = children[index];
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

- 边界检查超出列表返回 `null`。
- 按需包装重绘隔离、语义索引、保活，并用 `_SaltedValueKey`+`KeyedSubtree` 保持元素身份。

## 其他重写

```dart 790:796:lib/src/widgets/scroll_delegate.dart
@override
int? get estimatedChildCount => children.length;

@override
bool shouldRebuild(covariant SliverChildListDelegate oldDelegate) {
  return children != oldDelegate.children;
}
```

- `estimatedChildCount` 直接返回列表长度，便于滚动范围估算。
- `shouldRebuild`：仅当列表实例发生变更时重建，可避免相同引用导致的重复构建。

## 使用建议

- 列表较大或动态生成时优先使用 `SliverChildBuilderDelegate`，此类适合静态、小量或常量子列表。
- 若列表会变更，务必为子组件提供稳定 `Key` 并避免直接修改原列表引用，修改后创建新 `List`。
- 需要跨委托连续语义索引时设置 `semanticIndexOffset`；仅部分子项需要索引时实现 `semanticIndexCallback`。
- 如子树自管保活或外层已处理语义/重绘，可关闭对应开关以减少包装层级。
