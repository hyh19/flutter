# CustomScrollView 代码讲解

## 概述

`CustomScrollView` 允许直接提供 slivers，灵活组合列表、网格、折叠头等效果。它继承自 `ScrollView`，复用滚动手势、视口构建和 controller 逻辑，仅覆盖 `buildSlivers()` 返回传入的 `slivers`。

## 类定义与继承关系

```dart 696:823:lib/src/widgets/scroll_view.dart
class CustomScrollView extends ScrollView {
  const CustomScrollView({ ... });
  final List<Widget> slivers;
  @override
  List<Widget> buildSlivers(BuildContext context) => slivers;
}
```

- 继承 `ScrollView`：保留所有滚动配置（方向、controller、physics、primary 等）
- 核心差异：接收 `slivers` 列表并直接返回给视口

## 构造参数要点

```dart 700:718:lib/src/widgets/scroll_view.dart
const CustomScrollView({
  super.key,
  super.scrollDirection,
  super.reverse,
  super.controller,
  super.primary,
  super.physics,
  super.scrollBehavior,
  super.shrinkWrap,
  super.center,
  super.anchor,
  super.cacheExtent,
  super.paintOrder,
  this.slivers = const <Widget>[],
  super.semanticChildCount,
  super.dragStartBehavior,
  super.keyboardDismissBehavior,
  super.restorationId,
  super.clipBehavior,
  super.hitTestBehavior,
});
```

- **slivers**：必填内容列表，默认空
- **center/anchor**：可选，控制双向增长的锚点；需为 sliver key
- **shrinkWrap**：为 `true` 时使用 `ShrinkWrappingViewport`，性能开销更高
- 其他参数沿用 `ScrollView`（如 `primary`、`physics`、`scrollBehavior` 等）

## slivers 说明与示例

文档内详细解释了 sliver 概念并列举常用组件：

- 列表类：`SliverList`、`SliverFixedExtentList`、`SliverPrototypeExtentList`
- 网格类：`SliverGrid`
- 动画/扩展：`SliverAnimatedList`、`SliverAnimatedGrid`、`SliverPersistentHeader`
- 包装类：`SliverToBoxAdapter`、`SliverPadding`、`SliverFillRemaining` 等

示例（扩展 AppBar + Grid + List）：

```dart
CustomScrollView(
  slivers: <Widget>[
    const SliverAppBar(
      pinned: true,
      expandedHeight: 250.0,
      flexibleSpace: FlexibleSpaceBar(title: Text('Demo')),
    ),
    SliverGrid(
      gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
        maxCrossAxisExtent: 200.0,
        mainAxisSpacing: 10.0,
        crossAxisSpacing: 10.0,
        childAspectRatio: 4.0,
      ),
      delegate: SliverChildBuilderDelegate(
        (BuildContext context, int index) => Container(
          alignment: Alignment.center,
          color: Colors.teal[100 * (index % 9)],
          child: Text('Grid Item $index'),
        ),
        childCount: 20,
      ),
    ),
    SliverFixedExtentList(
      itemExtent: 50.0,
      delegate: SliverChildBuilderDelegate(
        (BuildContext context, int index) => Container(
          alignment: Alignment.center,
          color: Colors.lightBlue[100 * (index % 9)],
          child: Text('List Item $index'),
        ),
      ),
    ),
  ],
);
```

## 关键实现

### buildSlivers

```dart 820:822:lib/src/widgets/scroll_view.dart
@override
List<Widget> buildSlivers(BuildContext context) => slivers;
```

- 直接返回构造传入的 `slivers`
- 其他滚动处理（controller 继承、键盘消退、viewport 选择等）全部由父类 `ScrollView` 完成

## 使用建议与注意事项

- **懒加载**：优先使用 `SliverList`、`SliverGrid` 及其变体，避免一次性构建大量子节点
- **anchor/center**：需要双向增长或固定零偏移位置时为目标 sliver 设置唯一 key
- **physics/primary**：可复用 `ScrollView` 的配置；`primary: true` 时自动继承 `PrimaryScrollController`
- **无界约束**：若放入 `Column` 等无界主轴布局，设置 `shrinkWrap: true`，但注意性能
- **语义信息**：可通过 `semanticChildCount` 与 `IndexedSemantics` 提供无障碍索引

## 关联文档

- `ScrollView` 讲解：`scroll_view.dart_ScrollView.md`
- 官方文档：
  - [CustomScrollView](https://api.flutter.dev/flutter/widgets/CustomScrollView-class.html)
  - [Sliver 家族](https://api.flutter.dev/flutter/widgets/widgets-library.html#slivers)
