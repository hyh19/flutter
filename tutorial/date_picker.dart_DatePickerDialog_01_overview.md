# DatePickerDialog 概述

## 主要功能

`DatePickerDialog` 是一个 Material 风格的日期选择器对话框组件。它提供了两种交互模式（日历模式和输入模式），支持状态恢复功能，并可以直接推送到导航栈中。

## 类定义与构造器

```dart 310:357:packages/flutter/lib/src/material/date_picker.dart
class DatePickerDialog extends StatefulWidget {
  /// A Material-style date picker dialog.
  DatePickerDialog({
    super.key,
    DateTime? initialDate,
    required DateTime firstDate,
    required DateTime lastDate,
    DateTime? currentDate,
    this.initialEntryMode = DatePickerEntryMode.calendar,
    this.selectableDayPredicate,
    this.cancelText,
    this.confirmText,
    this.helpText,
    this.initialCalendarMode = DatePickerMode.day,
    this.errorFormatText,
    this.errorInvalidText,
    this.fieldHintText,
    this.fieldLabelText,
    this.keyboardType,
    this.restorationId,
    this.onDatePickerModeChange,
    this.switchToInputEntryModeIcon,
    this.switchToCalendarEntryModeIcon,
    this.insetPadding = const EdgeInsets.symmetric(horizontal: 16.0, vertical: 24.0),
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
      initialDate == null || !this.initialDate!.isBefore(this.firstDate),
      'initialDate ${this.initialDate} must be on or after firstDate ${this.firstDate}.',
    );
    assert(
      initialDate == null || !this.initialDate!.isAfter(this.lastDate),
      'initialDate ${this.initialDate} must be on or before lastDate ${this.lastDate}.',
    );
    assert(
      selectableDayPredicate == null ||
          initialDate == null ||
          selectableDayPredicate!(this.initialDate!),
      'Provided initialDate ${this.initialDate} must satisfy provided selectableDayPredicate',
    );
  }
```

### 构造器参数说明

- **key**: Widget 的唯一标识符
- **initialDate**: 初始选中的日期，为空时表示无选中状态
- **firstDate** (必需): 可选择的最早日期
- **lastDate** (必需): 可选择的最晚日期
- **currentDate**: 表示"今天"的日期，用于高亮显示
- **initialEntryMode**: 初始的输入模式，默认为日历模式
- **selectableDayPredicate**: 可选日期判断函数，用于控制哪些日期可以被选择
- **cancelText/confirmText**: 取消和确认按钮的文本
- **helpText**: 头部显示的帮助文本
- **initialCalendarMode**: 日历的初始显示模式（日/年）
- **errorFormatText/errorInvalidText**: 格式错误和无效日期的错误提示文本
- **fieldHintText/fieldLabelText**: 输入框的提示和标签文本
- **keyboardType**: 输入框的键盘类型
- **restorationId**: 状态恢复ID
- **onDatePickerModeChange**: 输入模式切换时的回调
- **switchToInputEntryModeIcon/switchToCalendarEntryModeIcon**: 模式切换图标
- **insetPadding**: 对话框外边距
- **calendarDelegate**: 日历委托，用于处理日期相关逻辑

### 初始化逻辑

构造器中使用了 `calendarDelegate.dateOnly()` 方法来处理日期，确保只保留日期部分（年月日），忽略时间部分。同时通过断言确保：

1. `lastDate` 必须晚于或等于 `firstDate`
2. `initialDate` 必须在 `firstDate` 和 `lastDate` 之间
3. 如果提供了 `selectableDayPredicate`，`initialDate` 必须满足该条件

## 主要字段

```dart 359:461:packages/flutter/lib/src/material/date_picker.dart
  /// The initially selected [DateTime] that the picker should display.
  ///
  /// If this is null, there is no selected date. A date must be selected to
  /// submit the dialog.
  final DateTime? initialDate;

  /// The earliest allowable [DateTime] that the user can select.
  final DateTime firstDate;

  /// The latest allowable [DateTime] that the user can select.
  final DateTime lastDate;

  /// The [DateTime] representing today. It will be highlighted in the day grid.
  final DateTime currentDate;

  /// The initial mode of date entry method for the date picker dialog.
  ///
  /// See [DatePickerEntryMode] for more details on the different data entry
  /// modes available.
  final DatePickerEntryMode initialEntryMode;

  /// Function to provide full control over which [DateTime] can be selected.
  final SelectableDayPredicate? selectableDayPredicate;

  /// The text that is displayed on the cancel button.
  final String? cancelText;

  /// The text that is displayed on the confirm button.
  final String? confirmText;

  /// The text that is displayed at the top of the header.
  ///
  /// This is used to indicate to the user what they are selecting a date for.
  final String? helpText;

  /// The initial display of the calendar picker.
  final DatePickerMode initialCalendarMode;

  /// The error text displayed if the entered date is not in the correct format.
  final String? errorFormatText;

  /// The error text displayed if the date is not valid.
  ///
  /// A date is not valid if it is earlier than [firstDate], later than
  /// [lastDate], or doesn't pass the [selectableDayPredicate].
  final String? errorInvalidText;

  /// The hint text displayed in the [TextField].
  ///
  /// If this is null, it will default to the date format string. For example,
  /// 'mm/dd/yyyy' for en_US.
  final String? fieldHintText;

  /// The label text displayed in the [TextField].
  ///
  /// If this is null, it will default to the words representing the date format
  /// string. For example, 'Month, Day, Year' for en_US.
  final String? fieldLabelText;

  /// {@template flutter.material.datePickerDialog}
  /// The keyboard type of the [TextField].
  ///
  /// If this is null, it will default to [TextInputType.datetime]
  /// {@endtemplate}
  final TextInputType? keyboardType;

  /// Restoration ID to save and restore the state of the [DatePickerDialog].
  ///
  /// If it is non-null, the date picker will persist and restore the
  /// date selected on the dialog.
  ///
  /// The state of this widget is persisted in a [RestorationBucket] claimed
  /// from the surrounding [RestorationScope] using the provided restoration ID.
  ///
  /// See also:
  ///
  ///  * [RestorationManager], which explains how state restoration works in
  ///    Flutter.
  final String? restorationId;

  /// Called when the [DatePickerDialog] is toggled between
  /// [DatePickerEntryMode.calendar],[DatePickerEntryMode.input].
  ///
  /// An example of how this callback might be used is an app that saves the
  /// user's preferred entry mode and uses it to initialize the
  /// `initialEntryMode` parameter the next time the date picker is shown.
  final ValueChanged<DatePickerEntryMode>? onDatePickerModeChange;

  /// {@macro flutter.material.date_picker.switchToInputEntryModeIcon}
  final Icon? switchToInputEntryModeIcon;

  /// {@macro flutter.material.date_picker.switchToCalendarEntryModeIcon}
  final Icon? switchToCalendarEntryModeIcon;

  /// The amount of padding added to [MediaQueryData.viewInsets] on the outside
  /// of the dialog. This defines the minimum space between the screen's edges
  /// and the dialog.
  ///
  /// Defaults to `EdgeInsets.symmetric(horizontal: 16.0, vertical: 24.0)`.
  final EdgeInsets insetPadding;

  /// {@macro flutter.material.calendar_date_picker.calendarDelegate}
  final CalendarDelegate<DateTime> calendarDelegate;
```

## 使用场景与意图

1. **直接使用**: 可以直接推送到导航栈，用于需要状态恢复的复杂场景
2. **内部使用**: 被 `showDatePicker` 函数内部使用，提供标准的日期选择体验
3. **自定义集成**: 通过各种参数可以高度自定义外观和行为
4. **状态恢复**: 通过 `restorationId` 支持应用重启后的状态恢复
5. **多模式支持**: 同时支持日历可视化和文本输入两种交互模式
6. **国际化**: 自动适应不同语言环境和日期格式
7. **可访问性**: 提供完整的语义信息和键盘导航支持

## 设计特点

- **灵活的日期控制**: 通过 `firstDate`、`lastDate` 和 `selectableDayPredicate` 精确控制可选日期范围
- **状态持久化**: 支持完整的状态恢复，防止用户输入丢失
- **响应式布局**: 自动适应不同的屏幕方向和尺寸
- **Material 设计**: 遵循 Material Design 规范，提供一致的用户体验
- **国际化支持**: 自动使用系统语言环境进行本地化
