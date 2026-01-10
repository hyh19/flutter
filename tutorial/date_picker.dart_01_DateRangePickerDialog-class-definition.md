# DateRangePickerDialog 类定义

## 概述

`DateRangePickerDialog` 是一个 Material 风格的日期范围选择器对话框组件。它提供了两种输入模式：日历模式和文本输入模式，用于让用户选择日期范围。

## 类定义

```dart 1326:1361:packages/flutter/lib/src/material/date_picker.dart
/// A Material-style date range picker dialog.
///
/// It is used internally by [showDateRangePicker] or can be directly pushed
/// onto the [Navigator] stack to enable state restoration. See
/// [showDateRangePicker] for a state restoration app example.
///
/// See also:
///
///  * [showDateRangePicker], which is a way to display the date picker.
class DateRangePickerDialog extends StatefulWidget {
  /// A Material-style date range picker dialog.
  const DateRangePickerDialog({
    super.key,
    this.initialDateRange,
    required this.firstDate,
    required this.lastDate,
    this.currentDate,
    this.initialEntryMode = DatePickerEntryMode.calendar,
    this.helpText,
    this.cancelText,
    this.confirmText,
    this.saveText,
    this.errorInvalidRangeText,
    this.errorFormatText,
    this.errorInvalidText,
    this.fieldStartHintText,
    this.fieldEndHintText,
    this.fieldStartLabelText,
    this.fieldEndLabelText,
    this.keyboardType = TextInputType.datetime,
    this.restorationId,
    this.switchToInputEntryModeIcon,
    this.switchToCalendarEntryModeIcon,
    this.selectableDayPredicate,
    this.calendarDelegate = const GregorianCalendarDelegate(),
  });
```

## 构造函数参数详解

### 核心日期参数

```dart 1363:1386:packages/flutter/lib/src/material/date_picker.dart
  /// The date range that the date range picker starts with when it opens.
  ///
  /// If an initial date range is provided, `initialDateRange.start`
  /// and `initialDateRange.end` must both fall between or on [firstDate] and
  /// [lastDate]. For all of these [DateTime] values, only their dates are
  /// considered. Their time fields are ignored.
  ///
  /// If [initialDateRange] is non-null, then it will be used as the initially
  /// selected date range. If it is provided, `initialDateRange.start` must be
  /// before or on `initialDateRange.end`.
  final DateTimeRange? initialDateRange;

  /// The earliest allowable date on the date range.
  final DateTime firstDate;

  /// The latest allowable date on the date range.
  final DateTime lastDate;

  /// The [currentDate] represents the current day (i.e. today).
  ///
  /// This date will be highlighted in the day grid.
  ///
  /// If `null`, the date of `DateTime.now()` will be used.
  final DateTime? currentDate;
```

- `initialDateRange`: 初始选中的日期范围，必须在 `firstDate` 和 `lastDate` 之间
- `firstDate`: 可选择的最早日期
- `lastDate`: 可选择的最晚日期
- `currentDate`: 当前日期（今天），会在日历中高亮显示

### 界面模式设置

```dart 1388:1395:packages/flutter/lib/src/material/date_picker.dart
  /// The initial date range picker entry mode.
  ///
  /// The date range has two main modes: [DatePickerEntryMode.calendar] (a
  /// scrollable calendar month grid) or [DatePickerEntryMode.input] (two text
  /// input fields) mode.
  ///
  /// It defaults to [DatePickerEntryMode.calendar].
  final DatePickerEntryMode initialEntryMode;
```

`initialEntryMode` 定义了初始的输入模式：

- `DatePickerEntryMode.calendar`: 日历网格模式
- `DatePickerEntryMode.input`: 文本输入字段模式

### 文本标签配置

```dart 1397:1419:packages/flutter/lib/src/material/date_picker.dart
  /// The label on the cancel button for the text input mode.
  ///
  /// If null, the localized value of
  /// [MaterialLocalizations.cancelButtonLabel] is used.
  final String? cancelText;

  /// The label on the "OK" button for the text input mode.
  ///
  /// If null, the localized value of
  /// [MaterialLocalizations.okButtonLabel] is used.
  final String? confirmText;

  /// The label on the save button for the fullscreen calendar mode.
  ///
  /// If null, the localized value of
  /// [MaterialLocalizations.saveButtonLabel] is used.
  final String? saveText;

  /// The label displayed at the top of the dialog.
  ///
  /// If null, the localized value of
  /// [MaterialLocalizations.dateRangePickerHelpText] is used.
  final String? helpText;
```

这些参数用于自定义对话框中的按钮文本和帮助文本，如果为 null 会使用本地化的默认值。

### 错误提示文本

```dart 1421:1438:packages/flutter/lib/src/material/date_picker.dart
  /// The message used when the date range is invalid (e.g. start date is after
  /// end date).
  ///
  /// If null, the localized value of
  /// [MaterialLocalizations.invalidDateRangeLabel] is used.
  final String? errorInvalidRangeText;

  /// The message used when an input text isn't in a proper date format.
  ///
  /// If null, the localized value of
  /// [MaterialLocalizations.invalidDateFormatLabel] is used.
  final String? errorFormatText;

  /// The message used when an input text isn't a selectable date.
  ///
  /// If null, the localized value of
  /// [MaterialLocalizations.dateOutOfRangeLabel] is used.
  final String? errorInvalidText;
```

用于配置各种错误情况下的提示文本。

### 输入字段配置

```dart 1440:1464:packages/flutter/lib/src/material/date_picker.dart
  /// The text used to prompt the user when no text has been entered in the
  /// start field.
  ///
  /// If null, the localized value of
  /// [MaterialLocalizations.dateHelpText] is used.
  final String? fieldStartHintText;

  /// The text used to prompt the user when no text has been entered in the
  /// end field.
  ///
  /// If null, the localized value of [MaterialLocalizations.dateHelpText] is
  /// used.
  final String? fieldEndHintText;

  /// The label for the start date text input field.
  ///
  /// If null, the localized value of [MaterialLocalizations.dateRangeStartLabel]
  /// is used.
  final String? fieldStartLabelText;

  /// The label for the end date text input field.
  ///
  /// If null, the localized value of [MaterialLocalizations.dateRangeEndLabel]
  /// is used.
  final String? fieldEndLabelText;
```

这些参数用于自定义开始和结束日期输入字段的提示文本和标签。

### 其他配置

```dart 1466:1493:packages/flutter/lib/src/material/date_picker.dart
  /// {@macro flutter.material.datePickerDialog}
  final TextInputType keyboardType;

  /// Restoration ID to save and restore the state of the [DateRangePickerDialog].
  ///
  /// If it is non-null, the date range picker will persist and restore the
  /// date range selected on the dialog.
  ///
  /// The state of this widget is persisted in a [RestorationBucket] claimed
  /// from the surrounding [RestorationScope] using the provided restoration ID.
  ///
  /// See also:
  ///
  ///  * [RestorationManager], which explains how state restoration works in
  ///    Flutter.
  final String? restorationId;

  /// {@macro flutter.material.date_picker.switchToInputEntryModeIcon}
  final Icon? switchToInputEntryModeIcon;

  /// {@macro flutter.material.date_picker.switchToCalendarEntryModeIcon}
  final Icon? switchToCalendarEntryModeIcon;

  /// Function to provide full control over which [DateTime] can be selected.
  final SelectableDayForRangePredicate? selectableDayPredicate;

  /// {@macro flutter.material.calendar_date_picker.calendarDelegate}
  final CalendarDelegate<DateTime> calendarDelegate;
```

- `keyboardType`: 键盘类型，默认为 `TextInputType.datetime`
- `restorationId`: 用于状态恢复的 ID
- `switchToInputEntryModeIcon` 和 `switchToCalendarEntryModeIcon`: 模式切换图标
- `selectableDayPredicate`: 自定义日期选择逻辑
- `calendarDelegate`: 日历代理，用于自定义日历行为

## 继承关系

`DateRangePickerDialog` 继承自 `StatefulWidget`，因此需要实现 `createState()` 方法来创建对应的状态类。

```dart 1495:1497:packages/flutter/lib/src/material/date_picker.dart
  @override
  State<DateRangePickerDialog> createState() => _DateRangePickerDialogState();
```

这个类专注于定义对话框的配置和参数，而具体的状态管理和UI构建逻辑在 `_DateRangePickerDialogState` 中实现。
