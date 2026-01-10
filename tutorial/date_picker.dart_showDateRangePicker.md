# showDateRangePicker 函数详解

## 函数概述

`showDateRangePicker` 是一个用于显示 Material Design 日期范围选择器的全屏模态对话框的函数。该函数允许用户选择一个日期范围，并返回用户选择的日期范围或 null（如果用户取消选择）。

```dart 1165:1285:packages/flutter/lib/src/material/date_picker.dart
Future<DateTimeRange?> showDateRangePicker({
  required BuildContext context,
  DateTimeRange? initialDateRange,
  required DateTime firstDate,
  required DateTime lastDate,
  DateTime? currentDate,
  DatePickerEntryMode initialEntryMode = DatePickerEntryMode.calendar,
  String? helpText,
  String? cancelText,
  String? confirmText,
  String? saveText,
  String? errorFormatText,
  String? errorInvalidText,
  String? errorInvalidRangeText,
  String? fieldStartHintText,
  String? fieldEndHintText,
  String? fieldStartLabelText,
  String? fieldEndLabelText,
  Locale? locale,
  bool barrierDismissible = true,
  Color? barrierColor,
  String? barrierLabel,
  bool useRootNavigator = true,
  RouteSettings? routeSettings,
  TextDirection? textDirection,
  TransitionBuilder? builder,
  Offset? anchorPoint,
  TextInputType keyboardType = TextInputType.datetime,
  final Icon? switchToInputEntryModeIcon,
  final Icon? switchToCalendarEntryModeIcon,
  SelectableDayForRangePredicate? selectableDayPredicate,
  CalendarDelegate<DateTime> calendarDelegate = const GregorianCalendarDelegate(),
}) async {
  // ... 实现代码
}
```

## 参数说明

### 必需参数

#### `context`

- **类型**: `BuildContext`
- **必需**: 是
- **说明**: 构建上下文，用于显示对话框和访问本地化等信息

#### `firstDate`

- **类型**: `DateTime`
- **必需**: 是
- **说明**: 可选择的最早日期。用户无法选择此日期之前的日期

#### `lastDate`

- **类型**: `DateTime`
- **必需**: 是
- **说明**: 可选择的最晚日期。用户无法选择此日期之后的日期

### 日期范围相关参数

#### `initialDateRange`

- **类型**: `DateTimeRange?`
- **默认值**: `null`
- **说明**: 初始选中的日期范围。如果提供，则 `start` 必须在 `end` 之前或等于 `end`

#### `currentDate`

- **类型**: `DateTime?`
- **默认值**: `null`
- **说明**: 表示当前日期（今天），将在日期网格中高亮显示。如果为 null，则使用 `DateTime.now()`

### 显示模式参数

#### `initialEntryMode`

- **类型**: `DatePickerEntryMode`
- **默认值**: `DatePickerEntryMode.calendar`
- **说明**: 初始显示模式
  - `DatePickerEntryMode.calendar`: 显示可滚动的日历月网格
  - `DatePickerEntryMode.input`: 显示两个文本输入字段

#### `keyboardType`

- **类型**: `TextInputType`
- **默认值**: `TextInputType.datetime`
- **说明**: 输入模式下文本字段的键盘类型

### 自定义文本参数

这些参数允许覆盖对话框中各个部分的默认文本：

#### `helpText`

- **类型**: `String?`
- **说明**: 显示在对话框顶部的标签

#### `cancelText`

- **类型**: `String?`
- **说明**: 文本输入模式下取消按钮的标签

#### `confirmText`

- **类型**: `String?`
- **说明**: 文本输入模式下确定按钮的标签

#### `saveText`

- **类型**: `String?`
- **说明**: 全屏日历模式下保存按钮的标签

#### 错误提示文本

- `errorFormatText`: 输入文本不符合正确日期格式时的消息
- `errorInvalidText`: 输入文本不是可选择日期时的消息
- `errorInvalidRangeText`: 日期范围无效时的消息（如开始日期在结束日期之后）

#### 输入字段文本

- `fieldStartHintText`: 开始日期字段为空时显示的提示文本
- `fieldEndHintText`: 结束日期字段为空时显示的提示文本
- `fieldStartLabelText`: 开始日期文本输入字段的标签
- `fieldEndLabelText`: 结束日期文本输入字段的标签

### 本地化和方向参数

#### `locale`

- **类型**: `Locale?`
- **说明**: 设置日期选择器的区域设置。默认为环境提供的本地化

#### `textDirection`

- **类型**: `TextDirection?`
- **说明**: 设置文本方向（LTR 或 RTL）。如果同时提供了 `locale` 和 `textDirection`，则 `textDirection` 会覆盖 `locale` 选择的文本方向

### 对话框外观参数

#### `barrierDismissible`

- **类型**: `bool`
- **默认值**: `true`
- **说明**: 是否可以通过点击背景来关闭对话框

#### `barrierColor`

- **类型**: `Color?`
- **说明**: 背景遮罩的颜色

#### `barrierLabel`

- **类型**: `String?`
- **说明**: 背景遮罩的语义标签

### 导航和路由参数

#### `useRootNavigator`

- **类型**: `bool`
- **默认值**: `true`
- **说明**: 是否使用根导航器显示对话框

#### `routeSettings`

- **类型**: `RouteSettings?`
- **说明**: 路由设置，传递给 `showDialog`

#### `anchorPoint`

- **类型**: `Offset?`
- **说明**: 对话框的锚点位置

### 自定义和回调参数

#### `builder`

- **类型**: `TransitionBuilder?`
- **说明**: 用于包装对话框小部件的构建器，可以添加继承的小部件如 `Theme`

#### `switchToInputEntryModeIcon`

- **类型**: `Icon?`
- **说明**: 切换到输入模式时显示的图标

#### `switchToCalendarEntryModeIcon`

- **类型**: `Icon?`
- **说明**: 切换到日历模式时显示的图标

#### `selectableDayPredicate`

- **类型**: `SelectableDayForRangePredicate?`
- **说明**: 用于确定哪些日期可以选择的回调函数。函数签名：

  ```dart
  bool Function(DateTime day, DateTime? startDate, DateTime? endDate)
  ```

#### `calendarDelegate`

- **类型**: `CalendarDelegate<DateTime>`
- **默认值**: `const GregorianCalendarDelegate()`
- **说明**: 日历委托，用于处理日期相关的逻辑，如公历或自定义日历系统

## 返回值

- **类型**: `Future<DateTimeRange?>`
- **说明**:
  - 返回用户选择的日期范围 (`DateTimeRange`)
  - 如果用户取消对话框，返回 `null`
  - 这是一个异步操作，需要使用 `await` 等待结果

## 实现细节

### 输入验证

函数首先对输入参数进行严格的验证，确保日期范围的合理性：

```dart 1198:1237:packages/flutter/lib/src/material/date_picker.dart
  initialDateRange = initialDateRange == null ? null : calendarDelegate.datesOnly(initialDateRange);
  firstDate = calendarDelegate.dateOnly(firstDate);
  lastDate = calendarDelegate.dateOnly(lastDate);
  assert(
    !lastDate.isBefore(firstDate),
    'lastDate $lastDate must be on or after firstDate $firstDate.',
  );
  assert(
    initialDateRange == null || !initialDateRange.start.isBefore(firstDate),
    "initialDateRange's start date must be on or after firstDate $firstDate.",
  );
  // ... 更多验证断言
```

这些验证包括：

- `lastDate` 必须在 `firstDate` 之后或相等
- 初始日期范围的开始和结束日期必须在 `firstDate` 和 `lastDate` 之间
- 如果提供了 `selectableDayPredicate`，则初始日期必须是可选择的

### 日期处理

使用 `calendarDelegate` 对日期进行处理，只保留日期部分，忽略时间：

```dart 1198:1200:packages/flutter/lib/src/material/date_picker.dart
  initialDateRange = initialDateRange == null ? null : calendarDelegate.datesOnly(initialDateRange);
  firstDate = calendarDelegate.dateOnly(firstDate);
  lastDate = calendarDelegate.dateOnly(lastDate);
```

### 对话框创建

创建 `DateRangePickerDialog` 小部件，并传入所有必要的参数：

```dart 1240:1262:packages/flutter/lib/src/material/date_picker.dart
  Widget dialog = DateRangePickerDialog(
    initialDateRange: initialDateRange,
    firstDate: firstDate,
    lastDate: lastDate,
    // ... 其他参数
  );
```

### 本地化和方向设置

根据提供的参数应用本地化和文本方向：

```dart 1264:1270:packages/flutter/lib/src/material/date_picker.dart
  if (textDirection != null) {
    dialog = Directionality(textDirection: textDirection, child: dialog);
  }

  if (locale != null) {
    dialog = Localizations.override(context: context, locale: locale, child: dialog);
  }
```

### 对话框显示

最终使用 `showDialog` 显示对话框：

```dart 1272:1284:packages/flutter/lib/src/material/date_picker.dart
  return showDialog<DateTimeRange>(
    context: context,
    barrierDismissible: barrierDismissible,
    barrierColor: barrierColor,
    barrierLabel: barrierLabel,
    useRootNavigator: useRootNavigator,
    routeSettings: routeSettings,
    useSafeArea: false,
    builder: (BuildContext context) {
      return builder == null ? dialog : builder(context, dialog);
    },
    anchorPoint: anchorPoint,
  );
```

## 使用示例

### 基本用法

```dart
DateTimeRange? picked = await showDateRangePicker(
  context: context,
  firstDate: DateTime(2020, 1, 1),
  lastDate: DateTime(2030, 12, 31),
);
if (picked != null) {
  print('选择的日期范围: ${picked.start} 到 ${picked.end}');
}
```

### 高级用法

```dart
DateTimeRange? picked = await showDateRangePicker(
  context: context,
  initialDateRange: DateTimeRange(
    start: DateTime.now(),
    end: DateTime.now().add(Duration(days: 7)),
  ),
  firstDate: DateTime(2020, 1, 1),
  lastDate: DateTime(2030, 12, 31),
  currentDate: DateTime.now(),
  initialEntryMode: DatePickerEntryMode.calendar,
  helpText: '选择旅行日期',
  saveText: '确定',
  locale: Locale('zh', 'CN'),
  selectableDayPredicate: (day, start, end) {
    // 只能选择周末
    return day.weekday == DateTime.saturday || day.weekday == DateTime.sunday;
  },
);
```

## 注意事项

1. **日期时间处理**: 所有 `DateTime` 参数只考虑日期部分，时间字段会被忽略
2. **异步操作**: 这是一个异步函数，必须使用 `await` 等待结果
3. **状态恢复**: 此方法不支持状态恢复。要启用状态恢复，需要使用 `Navigator.restorablePush` 和 `DateRangePickerDialog`
4. **输入验证**: 函数会在开发模式下对参数进行严格验证，确保日期范围的合理性
5. **可访问性**: 对话框支持语义标签和键盘导航

## 相关组件

- `showDatePicker`: 显示单日期选择器
- `DateTimeRange`: 表示日期范围的类
- `DateRangePickerDialog`: 实际的日期范围选择器对话框小部件
- `CalendarDelegate`: 处理日历逻辑的委托类
