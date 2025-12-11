# ListView 代码讲解

## 概述

`ListView` 是最常用的线性滚动列表，继承自 `BoxScrollView`，因此复用了滚动、控制器、物理效果、键盘消退等通用逻辑，只需专注于“线性 sliver 布局”的构建与子项管理。其核心是将不同的构造参数转换为合适的 sliver：`SliverList`、`SliverFixedExtentList`、`SliverPrototypeExtentList` 或 `SliverVariedExtentList`。

## 文档位置

```dart 914:1636:lib/src/widgets/scroll_view.dart
// ListView 源码与文档位置
```

## 构造函数类型与适用场景

1) **显式子节点**：`ListView({ List<Widget> children, ... })`  

- 适合子数目少的静态列表。  
- 立即构建所有子节点，开销与子数量线性相关。

2) **懒加载 builder**：`ListView.builder({ required NullableIndexedWidgetBuilder itemBuilder, int? itemCount, ... })`  

- 适合大量或无限列表。  
- 仅为可见项调用 builder；提供 `itemCount` 以优化最大滚动范围估计。

3) **带分隔符**：`ListView.separated({ required itemBuilder, required separatorBuilder, required itemCount, ... })`  

- 子项与分隔符交替出现：偶数 index 是 item，奇数 index 是 separator。  
- 内部通过 `_computeActualChildCount(itemCount)` 计算实际 sliver 子数量（2 * itemCount - 1）。

4) **完全自定义子模型**：`ListView.custom({ required SliverChildDelegate childrenDelegate, ... })`  

- 允许自定义 `SliverChildDelegate`，例如自定义估算策略或重排支持。  
- 仍可结合 `itemExtent` / `prototypeItem` / `itemExtentBuilder` 以决定主轴尺寸模式。

## 关键属性与断言

```dart 1278:1305:lib/src/widgets/scroll_view.dart
ListView({
  // ...
  this.itemExtent,
  this.itemExtentBuilder,
  this.prototypeItem,
  // ...
}) : assert(
  (itemExtent == null && prototypeItem == null) ||
      (itemExtent == null && itemExtentBuilder == null) ||
      (prototypeItem == null && itemExtentBuilder == null),
  'You can only pass one of itemExtent, prototypeItem and itemExtentBuilder.',
),
```

- `itemExtent` / `itemExtentBuilder` / `prototypeItem` 三者互斥（任意时刻只可设置其中之一）。
- `builder`/`separated` 构造函数会额外校验 `itemCount >= 0` 与 `semanticChildCount <= itemCount`。

### 主轴尺寸决定策略

| 属性            | Sliver 类型                | 适用场景                                                |
|-----------------|----------------------------|---------------------------------------------------------|
| `itemExtent`    | `SliverFixedExtentList`    | 所有子项主轴尺寸固定，最快，滚动估算最准确             |
| `itemExtentBuilder` | `SliverVariedExtentList` | 子项主轴尺寸可变，由回调返回；支持稀疏/不规则高度       |
| `prototypeItem` | `SliverPrototypeExtentList`| 所有子项尺寸等于样板 widget 的尺寸，构建样板一次即可    |
| 未指定          | `SliverList`               | 默认按子项自身尺寸                                      |

### 子模型委托

- `children`（显式列表）→ `SliverChildListDelegate`
- `itemBuilder`（懒加载）→ `SliverChildBuilderDelegate`
- `childrenDelegate`（custom）→ 直接使用调用者提供的委托
- `findChildIndexCallback`：在 builder 模式下支持根据 key 反查 index，便于保持可见项位置或执行跳转。

### 其他重要参数

- `padding`：继承自 `BoxScrollView`，默认自动消费 `MediaQuery` 主轴安全区；传入 `EdgeInsets.zero` 可禁用。
- `addAutomaticKeepAlives` / `addRepaintBoundaries` / `addSemanticIndexes`：对应 `SliverChild*Delegate` 的三个布尔开关，控制保活、绘制分界和语义索引。
- `semanticChildCount`：语义子数目；未提供时默认等于 children 数或 `itemCount`。

## 核心实现：buildChildLayout()

```dart 1610:1623:lib/src/widgets/scroll_view.dart
@override
Widget buildChildLayout(BuildContext context) {
  if (itemExtent != null) {
    return SliverFixedExtentList(delegate: childrenDelegate, itemExtent: itemExtent!);
  } else if (itemExtentBuilder != null) {
    return SliverVariedExtentList(
      delegate: childrenDelegate,
      itemExtentBuilder: itemExtentBuilder!,
    );
  } else if (prototypeItem != null) {
    return SliverPrototypeExtentList(delegate: childrenDelegate, prototypeItem: prototypeItem!);
  }
  return SliverList(delegate: childrenDelegate);
}
```

- 根据主轴尺寸策略选择对应 sliver，保证滚动估算与布局效率：
  - 固定尺寸：`SliverFixedExtentList`，性能最佳。
  - 可变尺寸：`SliverVariedExtentList`，需回调提供尺寸。
  - 样板尺寸：`SliverPrototypeExtentList`，一次测量复用。
  - 默认：`SliverList`。

## 分隔构造的实际 child 数计算

```dart 1631:1634:lib/src/widgets/scroll_view.dart
static int _computeActualChildCount(int itemCount) {
  return math.max(0, itemCount * 2 - 1);
}
```

- 对于 `ListView.separated`，总 sliver 子节点 = item 与 separator 交替数量。  
- 当 `itemCount = 0` 时返回 0，避免负数。

## 垫片与安全区行为

- 文档说明默认会“自动为列表两端添加 padding 来避让 `MediaQuery` 的遮挡”；如不需要，传入 `padding: EdgeInsets.zero` 覆盖。
- 与 `CustomScrollView` 的差异：`CustomScrollView` 不会自动处理 `MediaQuery`，需自行用 `SliverSafeArea`/`SliverPadding` 包裹。

## 使用建议与常见模式

- **小列表**：直接 `ListView(children: [...])`，简单清晰。
- **大/无限列表**：使用 `ListView.builder`，并提供 `itemCount` 以获得准确滚动指标。
- **带分隔符**：用 `ListView.separated` 而非在 item 内部手动添加 divider，可减少重复构建。
- **固定高度项**：优先设置 `itemExtent`，获得最优滚动性能与估算。
- **不规则高度**：使用 `itemExtentBuilder`，按需返回高度；或若高度一致但需测量一次，使用 `prototypeItem`。
- **状态保活**：需要跨滚动保持子项状态时，结合 `AutomaticKeepAliveClientMixin`，并根据需要调整 `addAutomaticKeepAlives`。
- **无界约束**：在 `Column` 等无界主轴布局中使用时，设置 `shrinkWrap: true`（注意性能开销）；或外包 `Expanded`。

## 关联文档

- `ScrollView` 讲解：`scroll_view.dart_ScrollView.md`
- `CustomScrollView` 讲解：`scroll_view.dart_CustomScrollView.md`
- `BoxScrollView` 讲解：`scroll_view.dart_BoxScrollView.md`
