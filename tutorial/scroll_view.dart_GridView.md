# GridView 代码讲解

## 概述

`GridView` 继承自 `BoxScrollView`，用于展示可滚动的二维网格。它复用 `ScrollView` 的滚动、控制器、物理效果、键盘消退等通用逻辑，并通过 `SliverGrid` 实现网格布局。网格的两大核心输入是：

- `gridDelegate`：定义网格在交叉轴和主轴的排布策略（列数或最大宽度、间距、纵横比等）。
- `childrenDelegate`：定义子节点的生成方式（静态列表或懒加载）。

## 文档位置

```dart 1637:2122:lib/src/widgets/scroll_view.dart
// GridView 源码与文档位置
```

## 构造函数类型与适用场景

1) **GridView**（自定义 delegate + 显式 children）  

- 传入 `gridDelegate`，子节点列表转为 `SliverChildListDelegate`。  
- 适合子数目不大、布局规则由调用者控制。

2) **GridView.builder**（自定义 delegate + 懒加载）  

- 传入 `gridDelegate` 与 `itemBuilder`，可选 `itemCount`。  
- 适合大批量或无限列表；提供 `itemCount` 以获得准确滚动估算。  
- 支持 `findChildIndexCallback` 便于根据 key 定位。

3) **GridView.custom**（完全自定义子模型）  

- 直接提供 `gridDelegate` 与 `childrenDelegate`。  
- 用于高度定制的懒加载或重排场景。

4) **GridView.count**（固定列数的便捷构造）  

- 内部创建 `SliverGridDelegateWithFixedCrossAxisCount`，指定 `crossAxisCount`、`mainAxisSpacing`、`crossAxisSpacing`、`childAspectRatio`。  
- 子节点列表转为 `SliverChildListDelegate`。

5) **GridView.extent**（限制最大交叉轴尺寸的便捷构造）  

- 内部创建 `SliverGridDelegateWithMaxCrossAxisExtent`，指定 `maxCrossAxisExtent` 等参数。  
- 子节点列表转为 `SliverChildListDelegate`。

## 关键参数与断言

- `gridDelegate`：必填，控制网格布局规则，所有构造函数都需要（便捷构造内部自动创建）。  
- `childrenDelegate`：在 `custom` 构造中由调用者提供，其余构造由 `children` 或 `itemBuilder` 自动生成。  
- `itemCount`（builder）：如提供必须 `>= 0`，并能优化最大滚动范围估计。  
- `padding`：继承自 `BoxScrollView`；默认会消费 `MediaQuery` 的主轴安全区，可用 `EdgeInsets.zero` 或 `MediaQuery.removePadding` 取消。
- `addAutomaticKeepAlives` / `addRepaintBoundaries` / `addSemanticIndexes`：映射至 `SliverChild*Delegate`，分别控制保活、绘制分界、语义索引。
- `semanticChildCount`：若未显式提供，默认等于子节点数量或 `itemCount`。

## 子模型与网格委托映射

- `children` 列表 → `SliverChildListDelegate`（对应 GridView、count、extent）。  
- `itemBuilder` 懒加载 → `SliverChildBuilderDelegate`（对应 builder）。  
- `childrenDelegate` 直传 → `GridView.custom`。  
- `gridDelegate` 决定网格排布：  
  - `SliverGridDelegateWithFixedCrossAxisCount`：固定列数。  
  - `SliverGridDelegateWithMaxCrossAxisExtent`：限制最大单元宽度，列数自适应。  
  - 自定义 `SliverGridDelegate`：可实现任意 2D 排布。

## 核心实现：buildChildLayout()

```dart 2117:2120:lib/src/widgets/scroll_view.dart
@override
Widget buildChildLayout(BuildContext context) {
  return SliverGrid(delegate: childrenDelegate, gridDelegate: gridDelegate);
}
```

- `GridView` 只需将子委托与网格委托交给 `SliverGrid`，其余滚动和视口管理由父类完成。  
- 由于继承 `BoxScrollView`，最终会被包装进 `SliverPadding`（如有 padding）并放入 `ScrollView` 的 slivers 列表。

## 与 CustomScrollView 的关系

- `GridView` ≈ `CustomScrollView(slivers: [SliverGrid(...)] + 可选 SliverPadding)`。  
- 需要组合多个 sliver（如折叠 AppBar + 网格 + 列表）时，使用 `CustomScrollView` 更灵活；`GridView` 提供单网格快捷用法。

## 使用建议与常见模式

- **固定列数布局**：用 `GridView.count`，快速指定列数和间距。  
- **自适应列数布局**：用 `GridView.extent`，通过 `maxCrossAxisExtent` 控制单元最大宽度。  
- **大数据/无限网格**：用 `GridView.builder`，并提供 `itemCount` 优化滚动估算；如需按 key 定位，提供 `findChildIndexCallback`。  
- **完全自定义**：用 `GridView.custom`，自定义子委托或网格委托实现特殊排布或估算。  
- **安全区处理**：如需去掉默认安全区 padding，传 `padding: EdgeInsets.zero` 或用 `MediaQuery.removePadding` 包裹。  
- **性能提示**：保持子项构建轻量；在可行时使用 `addRepaintBoundaries`/`addAutomaticKeepAlives` 默认值以减少重复布局与重绘。

## 关联文档

- `ScrollView`：`scroll_view.dart_ScrollView.md`
- `CustomScrollView`：`scroll_view.dart_CustomScrollView.md`
- `BoxScrollView`：`scroll_view.dart_BoxScrollView.md`
- `ListView`：`scroll_view.dart_ListView.md`
