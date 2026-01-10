# _InputDateRangePickerDialog 类详解

## 概述

`_InputDateRangePickerDialog` 是一个私有无状态组件类，用于实现 Material Design 日期范围选择器的输入模式对话框。该类专门处理日期范围选择器的界面布局和交互逻辑，支持 Material 2 和 Material 3 设计规范，并能根据设备方向（纵向/横向）自适应调整布局。

## 核心功能

该对话框组件主要负责：

1. **日期范围格式化显示** - 将选中的日期范围转换为用户友好的文本格式
2. **响应式布局** - 根据屏幕方向自动调整对话框布局
3. **Material Design 兼容性** - 支持 M2 和 M3 设计规范
4. **辅助功能支持** - 提供语义标签以支持屏幕阅读器
5. **主题定制** - 支持通过 `DatePickerTheme` 进行样式定制

## 构造函数

```dart 3045:3058:packages/flutter/lib/src/material/date_picker.dart
class _InputDateRangePickerDialog extends StatelessWidget {
  const _InputDateRangePickerDialog({
    required this.selectedStartDate,
    required this.selectedEndDate,
    required this.currentDate,
    required this.picker,
    required this.onConfirm,
    required this.onCancel,
    required this.confirmText,
    required this.cancelText,
    required this.helpText,
    required this.entryModeButton,
    required this.calendarDelegate,
  });
```

### 参数说明

| 参数 | 类型 | 必需 | 说明 |
| --- | --- | --- | ---- |
| `selectedStartDate` | `DateTime?` | 是 | 选中的起始日期 |
| `selectedEndDate` | `DateTime?` | 是 | 选中的结束日期 |
| `currentDate` | `DateTime?` | 是 | 当前日期，用于格式化逻辑 |
| `picker` | `Widget` | 是 | 实际的日期选择器组件 |
| `onConfirm` | `VoidCallback` | 是 | 确认按钮回调 |
| `onCancel` | `VoidCallback` | 是 | 取消按钮回调 |
| `confirmText` | `String?` | 否 | 确认按钮文本，为空时使用本地化默认值 |
| `cancelText` | `String?` | 否 | 取消按钮文本，为空时使用本地化默认值 |
| `helpText` | `String?` | 否 | 帮助文本，为空时使用本地化默认值 |
| `entryModeButton` | `Widget?` | 否 | 模式切换按钮（日历/输入模式切换） |
| `calendarDelegate` | `CalendarDelegate<DateTime>` | 是 | 日历委托，用于日期格式化和本地化 |

## 核心方法

### _formatDateRange 方法

```dart 3072:3083:packages/flutter/lib/src/material/date_picker.dart
String _formatDateRange(BuildContext context, DateTime? start, DateTime? end, DateTime now) {
  final MaterialLocalizations localizations = MaterialLocalizations.of(context);
  final String startText = _formatRangeStartDate(localizations, calendarDelegate, start, end);
  final String endText = _formatRangeEndDate(localizations, calendarDelegate, start, end, now);
  if (start == null || end == null) {
    return localizations.unspecifiedDateRange;
  }
  return switch (Directionality.of(context)) {
    TextDirection.rtl => '$endText – $startText',
    TextDirection.ltr => '$startText – $endText',
  };
}
```

该方法负责将日期范围格式化为用户界面显示的文本：

1. **获取本地化资源** - 使用 `MaterialLocalizations` 获取本地化字符串
2. **格式化起始和结束日期** - 调用辅助函数格式化单个日期
3. **处理未指定日期** - 当起始或结束日期为空时显示默认文本
4. **支持双向文本** - 根据文本方向（LTR/RTL）调整日期顺序

## build 方法详解

### 主题和本地化设置

```dart 3086:3091:packages/flutter/lib/src/material/date_picker.dart
@override
Widget build(BuildContext context) {
  final bool useMaterial3 = Theme.of(context).useMaterial3;
  final MaterialLocalizations localizations = MaterialLocalizations.of(context);
  final Orientation orientation = MediaQuery.orientationOf(context);
  final DatePickerThemeData datePickerTheme = DatePickerTheme.of(context);
  final DatePickerThemeData defaults = DatePickerTheme.defaults(context);
```

构建方法首先获取必要的上下文信息：

- **Material Design 版本** - 确定使用 M2 还是 M3 规范
- **本地化资源** - 获取当前语言环境的文本资源
- **设备方向** - 检测当前是纵向还是横向模式
- **主题数据** - 获取日期选择器主题配置

### 标题样式处理

```dart 3093:3103:packages/flutter/lib/src/material/date_picker.dart
// There's no M3 spec for a landscape layout input (not calendar)
// date range picker. To ensure that the date range displayed in the
// input date range picker's header fits in landscape mode, we override
// the M3 default here.
TextStyle? headlineStyle = (orientation == Orientation.portrait)
    ? datePickerTheme.headerHeadlineStyle ?? defaults.headerHeadlineStyle
    : Theme.of(context).textTheme.headlineSmall;

final Color? headerForegroundColor =
    datePickerTheme.headerForegroundColor ?? defaults.headerForegroundColor;
headlineStyle = headlineStyle?.copyWith(color: headerForegroundColor);
```

这里有一个重要的设计决策：Material 3 规范中没有定义横向输入模式日期范围选择器的布局。为了确保横向模式下日期范围文本能够在头部正确显示，代码覆盖了 M3 的默认样式，使用更小的标题样式。

### 日期文本格式化

```dart 3105:3113:packages/flutter/lib/src/material/date_picker.dart
final String dateText = _formatDateRange(
  context,
  selectedStartDate,
  selectedEndDate,
  currentDate!,
);
final String semanticDateText = selectedStartDate != null && selectedEndDate != null
    ? '${calendarDelegate.formatMediumDate(selectedStartDate!, localizations)} – ${calendarDelegate.formatMediumDate(selectedEndDate!, localizations)}'
    : '';
```

生成两类日期文本：

1. **显示文本** - 用于界面显示的格式化日期范围
2. **语义文本** - 用于辅助功能，提供更详细的日期信息

### 头部组件构建

```dart 3115:3127:packages/flutter/lib/src/material/date_picker.dart
final Widget header = _DatePickerHeader(
  helpText:
      helpText ??
      (useMaterial3
          ? localizations.dateRangePickerHelpText
          : localizations.dateRangePickerHelpText.toUpperCase()),
  titleText: dateText,
  titleSemanticsLabel: semanticDateText,
  titleStyle: headlineStyle,
  orientation: orientation,
  isShort: orientation == Orientation.landscape,
  entryModeButton: entryModeButton,
);
```

头部组件显示帮助文本和当前选中的日期范围。在 Material 2 中，帮助文本会转换为大写格式。

### 操作按钮区域

```dart 3129:3155:packages/flutter/lib/src/material/date_picker.dart
final Widget actions = ConstrainedBox(
  constraints: const BoxConstraints(minHeight: 52.0),
  child: Padding(
    padding: const EdgeInsets.symmetric(horizontal: 8),
    child: Align(
      alignment: AlignmentDirectional.centerEnd,
      child: OverflowBar(
        spacing: 8,
        children: <Widget>[
          TextButton(
            onPressed: onCancel,
            child: Text(
              cancelText ??
                  (useMaterial3
                      ? localizations.cancelButtonLabel
                      : localizations.cancelButtonLabel.toUpperCase()),
          ),
          TextButton(
            onPressed: onConfirm,
            child: Text(confirmText ?? localizations.okButtonLabel),
          ),
        ],
      ),
    ),
  ),
);
```

操作区域包含取消和确认按钮，使用 `OverflowBar` 确保按钮在空间不足时能够正确换行。同样，Material 2 中按钮文本会转换为大写。

## 响应式布局系统

### 文本缩放处理

```dart 3157:3163:packages/flutter/lib/src/material/date_picker.dart
final double textScaleFactor =
    MediaQuery.textScalerOf(
      context,
    ).clamp(maxScaleFactor: _kMaxRangeTextScaleFactor).scale(_fontSizeToScale) /
    _fontSizeToScale;
final Size dialogSize =
    (useMaterial3 ? _inputPortraitDialogSizeM3 : _inputPortraitDialogSizeM2) * textScaleFactor;
```

为了支持无障碍功能，组件会根据用户的文本缩放偏好调整对话框大小，并限制最大缩放因子以确保界面可用性。

### 纵向布局

```dart 3165:3187:packages/flutter/lib/src/material/date_picker.dart
case Orientation.portrait:
  return LayoutBuilder(
    builder: (BuildContext context, BoxConstraints constraints) {
      final Size portraitDialogSize = useMaterial3
          ? _inputPortraitDialogSizeM3
          : _inputPortraitDialogSizeM2;
      // Make sure the portrait dialog can fit the contents comfortably when
      // resized from the landscape dialog.
      final bool isFullyPortrait =
          constraints.maxHeight >= math.min(dialogSize.height, portraitDialogSize.height);

      return Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.stretch,
        // When the portrait dialog does not fit vertically, hide the header.
        children: <Widget>[
          if (isFullyPortrait) header,
          Expanded(child: picker),
          actions,
        ],
      );
    },
  );
```

纵向布局使用 `LayoutBuilder` 动态检测可用空间：

- **空间充足时** - 显示完整的头部、选择器和操作按钮
- **空间不足时** - 隐藏头部以确保选择器和按钮可见

这种设计确保了在不同设备和屏幕尺寸下的可用性。

### 横向布局

```dart 3189:3207:packages/flutter/lib/src/material/date_picker.dart
case Orientation.landscape:
  return Row(
    mainAxisSize: MainAxisSize.min,
    crossAxisAlignment: CrossAxisAlignment.stretch,
    children: <Widget>[
      header,
      Flexible(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            Expanded(child: picker),
            actions,
          ],
        ),
      ),
    ],
  );
```

横向布局将头部放在左侧，右侧放置选择器和操作按钮。这种布局充分利用了横向屏幕的宽高比。

## Material Design 兼容性

该组件完全支持 Material 2 和 Material 3 设计规范：

- **样式差异** - M2 使用大写文本，M3 使用句子大小写
- **尺寸规范** - 使用不同的对话框尺寸常量
- **主题集成** - 通过 `DatePickerTheme` 支持完整的主题定制

## 辅助功能支持

组件特别注重辅助功能：

1. **语义标签** - 为屏幕阅读器提供详细的日期信息
2. **文本方向支持** - 正确处理从右到左的语言布局
3. **文本缩放** - 支持用户偏好的文本大小设置
4. **焦点管理** - 合理的组件顺序和键盘导航

## 使用场景

该组件通常在以下情况下使用：

- 用户调用 `showDateRangePicker()` 并选择输入模式
- 需要精确输入日期范围而非日历选择的场景
- 空间受限或需要紧凑界面的情况

通过精细的布局逻辑和全面的平台支持，该组件为 Flutter 应用提供了高质量的日期范围选择体验。
