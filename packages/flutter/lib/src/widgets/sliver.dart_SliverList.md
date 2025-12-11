# SliverList 代码讲解

## 概述

`SliverList` 是最常用的线性 sliver，实现懒加载的多子节点排列。它依赖 `SliverChildDelegate` 系列提供子组件，支持无限/大规模列表，并在构造函数中提供常见变体（builder、separated、list）。

## 类定义与基础构造

```dart 167:170:lib/src/widgets/sliver.dart
class SliverList extends SliverMultiBoxAdaptorWidget {
  /// Creates a sliver that places box children in a linear array.
  const SliverList({super.key, required super.delegate});
}
```

- 继承 `SliverMultiBoxAdaptorWidget`，复用多盒子适配器的布局/缓存逻辑。
- 通过 `delegate` 注入子节点来源（通常是 `SliverChildBuilderDelegate`/`SliverChildListDelegate`）。

## builder 变体：懒加载列表

```dart 171:231:lib/src/widgets/sliver.dart
SliverList.builder({
  super.key,
  required NullableIndexedWidgetBuilder itemBuilder,
  ChildIndexGetter? findChildIndexCallback,
  int? itemCount,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  int semanticIndexOffset = 0,
}) : super(
       delegate: SliverChildBuilderDelegate(
         itemBuilder,
         findChildIndexCallback: findChildIndexCallback,
         childCount: itemCount,
         addAutomaticKeepAlives: addAutomaticKeepAlives,
         addRepaintBoundaries: addRepaintBoundaries,
         addSemanticIndexes: addSemanticIndexes,
         semanticIndexOffset: semanticIndexOffset,
       ),
     );
```

- 按需构建子项，适合大列表；`itemCount` 为空时直到 `itemBuilder` 返回 `null` 停止。
- `findChildIndexCallback` 支持重排时的键映射，避免状态丢失。
- 包装参数对应 `SliverChildBuilderDelegate` 的保活、重绘、语义配置。

## separated 变体：带分隔符的懒加载列表

```dart 232:314:lib/src/widgets/sliver.dart
SliverList.separated({
  super.key,
  required NullableIndexedWidgetBuilder itemBuilder,
  ChildIndexGetter? findChildIndexCallback,
  required NullableIndexedWidgetBuilder separatorBuilder,
  int? itemCount,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
}) : super(
       delegate: SliverChildBuilderDelegate(
         (BuildContext context, int index) {
           final int itemIndex = index ~/ 2;
           final Widget? widget;
           if (index.isEven) {
             widget = itemBuilder(context, itemIndex);
           } else {
             widget = separatorBuilder(context, itemIndex);
             assert(() {
               if (widget == null) {
                 throw FlutterError('separatorBuilder cannot return null.');
               }
               return true;
             }());
           }
           return widget;
         },
         findChildIndexCallback: findChildIndexCallback,
         childCount: itemCount == null ? null : math.max(0, itemCount * 2 - 1),
         addAutomaticKeepAlives: addAutomaticKeepAlives,
         addRepaintBoundaries: addRepaintBoundaries,
         addSemanticIndexes: addSemanticIndexes,
         semanticIndexCallback: (Widget _, int index) {
           return index.isEven ? index ~/ 2 : null;
         },
       ),
     );
```

- 通过偶数索引放置 item，奇数索引放置分隔符，子计数为 `itemCount * 2 - 1`。
- 自定义 `semanticIndexCallback` 只为 item 标记语义索引，分隔符返回 `null`。
- 分隔符不允许返回 `null`，断言明确提示。

## list 变体：显式子列表

```dart 316:354:lib/src/widgets/sliver.dart
SliverList.list({
  super.key,
  required List<Widget> children,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
}) : super(
       delegate: SliverChildListDelegate(
         children,
         addAutomaticKeepAlives: addAutomaticKeepAlives,
         addRepaintBoundaries: addRepaintBoundaries,
         addSemanticIndexes: addSemanticIndexes,
       ),
     );
```

- 直接传入子组件列表，适合小型或静态集合；大列表建议使用 builder。
- 保留与 builder 相同的包装开关。

## 渲染与元素

```dart 356:365:lib/src/widgets/sliver.dart
@override
SliverMultiBoxAdaptorElement createElement() =>
    SliverMultiBoxAdaptorElement(this, replaceMovedChildren: true);

@override
RenderSliverList createRenderObject(BuildContext context) {
  final SliverMultiBoxAdaptorElement element = context as SliverMultiBoxAdaptorElement;
  return RenderSliverList(childManager: element);
}
```

- 使用 `SliverMultiBoxAdaptorElement` 管理子节点缓存与复用；`replaceMovedChildren: true` 支持移动子项时更高效的元素替换。
- `RenderSliverList` 负责线性布局与滚动测量，基于 adaptor 提供的子管理器。

## 使用建议

- 大型或无限列表优先使用 `SliverList.builder`，并尽量提供 `itemCount` 以改进滚动范围估算。
- 需要分隔符时使用 `SliverList.separated`，保持语义索引仅标记实际项。
- 静态少量内容可用 `SliverList.list`；若列表会变更，确保子项有稳定 `Key`。
- 保持默认的保活、重绘隔离与语义开关，只有在明确性能/语义需求时再调整。
