# CalendarDatePicker 类概述和定义

## 概述

`CalendarDatePicker` 是一个 Material Design 风格的日历日期选择器组件，用于显示月份网格并允许用户选择日期。

```dart 74:93:packages/flutter/lib/src/material/calendar_date_picker.dart
/// Displays a grid of days for a given month and allows the user to select a
/// date.
///
/// Days are arranged in a rectangular grid with one column for each day of the
/// week. Controls are provided to change the year and month that the grid is
/// showing.
///
/// The calendar picker widget is rarely used directly. Instead, consider using
/// [showDatePicker], which will create a dialog that uses this as well as
/// provides a text entry option.
///
/// See also:
///
///  * [showDatePicker], which creates a Dialog that contains a
///    [CalendarDatePicker] and provides an optional compact view where the
///    user can enter a date as a line of text.
///  * [showTimePicker], which shows a dialog that contains a Material Design
///    time picker.
///
class CalendarDatePicker extends StatefulWidget {
```

这个组件的主要特点包括：

- 以网格形式显示月份的日期
- 提供年份和月份切换控件
- 支持日期选择和范围限制
- 集成无障碍功能
- 支持自定义日历委托（用于不同日历系统）

## 构造函数和参数

`CalendarDatePicker` 的构造函数接受多个必需和可选参数来配置组件的行为：

```dart 126:159:packages/flutter/lib/src/material/calendar_date_picker.dart
  CalendarDatePicker({
    super.key,
    required DateTime? initialDate,
    required DateTime firstDate,
    required DateTime lastDate,
    DateTime? currentDate,
    required this.onDateChanged,
    this.onDisplayedMonthChanged,
    this.initialCalendarMode = DatePickerMode.day,
    this.selectableDayPredicate,
    this.calendarDelegate = const GregorianCalendarDelegate(),
  }) : initialDate = initialDate == null ? null : calendarDelegate.dateOnly(initialDate),
       firstDate = calendarDelegate.dateOnly(firstDate),
       lastDate = calendarDelegate.dateOnly(lastDate),
       currentDate = calendarDelegate.dateOnly(currentDate ?? calendarDelegate.now()) {
    assert(
      !this.lastDate.isBefore(this.firstDate),
      'lastDate ${this.lastDate} must be on or after firstDate ${this.firstDate}.',
    );
    assert(
      this.initialDate == null || !this.initialDate!.isBefore(this.firstDate),
      'initialDate ${this.initialDate} must be on or after firstDate ${this.firstDate}.',
    );
    assert(
      this.initialDate == null || !this.initialDate!.isAfter(this.lastDate),
      'initialDate ${this.initialDate} must be on or before lastDate ${this.lastDate}.',
    );
    assert(
      selectableDayPredicate == null ||
          this.initialDate == null ||
          selectableDayPredicate!(this.initialDate!),
      'Provided initialDate ${this.initialDate} must satisfy provided selectableDayPredicate.',
    );
  }
```

### 必需参数

- **`initialDate`**: 初始选中的日期，可以为 null
- **`firstDate`**: 用户可以选择的最早日期
- **`lastDate`**: 用户可以选择的最晚日期
- **`onDateChanged`**: 当用户选择新日期时调用的回调函数

### 可选参数

- **`currentDate`**: 表示"今天"的日期，会在日期网格中高亮显示
- **`onDisplayedMonthChanged`**: 当用户导航到新月份时调用的回调
- **`initialCalendarMode`**: 初始显示模式（日视图或年视图）
- **`selectableDayPredicate`**: 自定义函数控制哪些日期可以被选择
- **`calendarDelegate`**: 日历委托，用于支持不同的日历系统

## 主要属性

组件定义了多个 final 属性来存储配置信息：

```dart 161:196:packages/flutter/lib/src/material/calendar_date_picker.dart
  /// The initially selected [DateTime] that the picker should display.
  ///
  /// Subsequently changing this has no effect. To change the selected date,
  /// change the [key] to create a new instance of the [CalendarDatePicker], and
  /// provide that widget the new [initialDate]. This will reset the widget's
  /// interactive state.
  final DateTime? initialDate;

  /// The earliest allowable [DateTime] that the user can select.
  final DateTime firstDate;

  /// The latest allowable [DateTime] that the user can select.
  final DateTime lastDate;

  /// The [DateTime] representing today. It will be highlighted in the day grid.
  final DateTime currentDate;

  /// Called when the user selects a date in the picker.
  final ValueChanged<DateTime> onDateChanged;

  /// Called when the user navigates to a new month/year in the picker.
  final ValueChanged<DateTime>? onDisplayedMonthChanged;

  /// The initial display of the calendar picker.
  ///
  /// Subsequently changing this has no effect. To change the calendar mode,
  /// change the [key] to create a new instance of the [CalendarDatePicker], and
  /// provide that widget a new [initialCalendarMode]. This will reset the
  /// widget's interactive state.
  final DatePickerMode initialCalendarMode;

  /// Function to provide full control over which dates in the calendar can be selected.
  final SelectableDayPredicate? selectableDayPredicate;

  /// {@macro flutter.material.calendar_date_picker.calendarDelegate}
  final CalendarDelegate<DateTime> calendarDelegate;
```

## 日历委托机制

组件支持通过 `calendarDelegate` 参数使用不同的日历系统：

```dart 120:125:packages/flutter/lib/src/material/calendar_date_picker.dart
  /// {@template flutter.material.calendar_date_picker.calendarDelegate}
  /// The [calendarDelegate] controls date interpretation, formatting, and
  /// navigation within the picker. By providing a custom implementation,
  /// you can support alternative calendar systems such as Nepali, Hijri,
  /// Buddhist, and more. Defaults to [GregorianCalendarDelegate].
  /// {@endtemplate}
```

默认使用 `GregorianCalendarDelegate`（公历），但可以通过自定义实现支持：

- 尼泊尔历 (Nepali)
- 伊斯兰历 (Hijri)
- 佛教历 (Buddhist)
- 及其他日历系统

## 状态管理

`CalendarDatePicker` 继承自 `StatefulWidget`，其状态由 `_CalendarDatePickerState` 类管理。

```dart 198:200:packages/flutter/lib/src/material/calendar_date_picker.dart
  @override
  State<CalendarDatePicker> createState() => _CalendarDatePickerState();
```

状态类负责：

- 管理当前显示模式（日/年视图）
- 处理用户交互事件
- 维护选中的日期
- 构建和更新UI组件

## 使用建议

虽然 `CalendarDatePicker` 可以直接使用，但官方建议优先使用 `showDatePicker` 函数：

```dart 81:90:packages/flutter/lib/src/material/calendar_date_picker.dart
/// The calendar picker widget is rarely used directly. Instead, consider using
/// [showDatePicker], which will create a dialog that uses this as well as
/// provides a text entry option.
///
/// See also:
///
///  * [showDatePicker], which creates a Dialog that contains a
///    [CalendarDatePicker] and provides an optional compact view where the
///    user can enter a date as a line of text.
```

`showDatePicker` 提供了更完整的用户体验，包括：

- 对话框容器
- 文本输入选项
- 确认和取消按钮
- 更好的移动端适配
