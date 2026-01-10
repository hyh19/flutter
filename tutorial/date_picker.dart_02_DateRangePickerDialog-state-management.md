# DateRangePickerDialog 状态管理

## 概述

`_DateRangePickerDialogState` 是 `DateRangePickerDialog` 的状态类，负责管理对话框的状态、事件处理和状态恢复功能。这个类继承自 `State<DateRangePickerDialog>` 并混入了 `RestorationMixin` 来支持状态恢复。

## 状态变量定义

```dart 1499:1511:packages/flutter/lib/src/material/date_picker.dart
class _DateRangePickerDialogState extends State<DateRangePickerDialog> with RestorationMixin {
  late final _RestorableDatePickerEntryMode _entryMode = _RestorableDatePickerEntryMode(
    widget.initialEntryMode,
  );
  late final RestorableDateTimeN _selectedStart = RestorableDateTimeN(
    widget.initialDateRange?.start,
  );
  late final RestorableDateTimeN _selectedEnd = RestorableDateTimeN(widget.initialDateRange?.end);
  final RestorableBool _autoValidate = RestorableBool(false);
  final GlobalKey _calendarPickerKey = GlobalKey();
  final GlobalKey<_InputDateRangePickerState> _inputPickerKey =
      GlobalKey<_InputDateRangePickerState>();
```

### 状态变量详解

- `_entryMode`: 当前的输入模式（日历模式或文本输入模式），使用可恢复的状态管理
- `_selectedStart`: 选中的开始日期，使用 `RestorableDateTimeN` 支持状态恢复
- `_selectedEnd`: 选中的结束日期，使用 `RestorableDateTimeN` 支持状态恢复
- `_autoValidate`: 是否自动验证输入，用于文本输入模式
- `_calendarPickerKey`: 日历选择器的全局键，用于访问日历组件
- `_inputPickerKey`: 文本输入选择器的全局键，用于访问输入组件

## 状态恢复实现

```dart 1512:1521:packages/flutter/lib/src/material/date_picker.dart
  @override
  String? get restorationId => widget.restorationId;

  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    registerForRestoration(_entryMode, 'entry_mode');
    registerForRestoration(_selectedStart, 'selected_start');
    registerForRestoration(_selectedEnd, 'selected_end');
    registerForRestoration(_autoValidate, 'autovalidate');
  }
```

通过混入 `RestorationMixin` 并实现 `restoreState` 方法，对话框可以保存和恢复用户的选择状态，包括输入模式和选中的日期范围。

## 资源清理

```dart 1523:1530:packages/flutter/lib/src/material/date_picker.dart
  @override
  void dispose() {
    _entryMode.dispose();
    _selectedStart.dispose();
    _selectedEnd.dispose();
    _autoValidate.dispose();
    super.dispose();
  }
```

在组件销毁时清理所有可恢复的状态对象，防止内存泄漏。

## 事件处理方法

### 确认操作 (_handleOk)

```dart 1532:1548:packages/flutter/lib/src/material/date_picker.dart
  void _handleOk() {
    if (_entryMode.value == DatePickerEntryMode.input ||
        _entryMode.value == DatePickerEntryMode.inputOnly) {
      final _InputDateRangePickerState picker = _inputPickerKey.currentState!;
      if (!picker.validate()) {
        setState(() {
          _autoValidate.value = true;
        });
        return;
      }
    }
    final DateTimeRange? selectedRange = _hasSelectedDateRange
        ? DateTimeRange(start: _selectedStart.value!, end: _selectedEnd.value!)
        : null;

    Navigator.pop(context, selectedRange);
  }
```

- 在文本输入模式下，首先验证输入的有效性
- 如果验证失败，启用自动验证并返回
- 如果有选中的日期范围，创建 `DateTimeRange` 对象
- 通过 `Navigator.pop` 返回选中的日期范围

### 取消操作 (_handleCancel)

```dart 1550:1552:packages/flutter/lib/src/material/date_picker.dart
  void _handleCancel() {
    Navigator.pop(context);
  }
```

简单的取消操作，直接关闭对话框而不返回任何值。

### 模式切换 (_handleEntryModeToggle)

```dart 1554:1582:packages/flutter/lib/src/material/date_picker.dart
  void _handleEntryModeToggle() {
    setState(() {
      switch (_entryMode.value) {
        case DatePickerEntryMode.calendar:
          _autoValidate.value = false;
          _entryMode.value = DatePickerEntryMode.input;

        case DatePickerEntryMode.input:
          // Validate the range dates
          if (_selectedStart.value != null &&
              _selectedEnd.value != null &&
              _selectedStart.value!.isAfter(_selectedEnd.value!)) {
            _selectedEnd.value = null;
          }
          if (_selectedStart.value != null && !_isDaySelectable(_selectedStart.value!)) {
            _selectedStart.value = null;
            // With no valid start date, having an end date makes no sense for the UI.
            _selectedEnd.value = null;
          } else if (_selectedEnd.value != null && !_isDaySelectable(_selectedEnd.value!)) {
            _selectedEnd.value = null;
          }
          _entryMode.value = DatePickerEntryMode.calendar;

        case DatePickerEntryMode.calendarOnly:
        case DatePickerEntryMode.inputOnly:
          assert(false, 'Can not change entry mode from $_entryMode');
      }
    });
  }
```

处理日历模式和文本输入模式之间的切换：

- 从日历模式切换到输入模式：禁用自动验证
- 从输入模式切换到日历模式：
  - 验证日期范围的有效性
  - 清除无效的日期选择
  - 确保开始日期在结束日期之前

### 日期选择验证 (_isDaySelectable)

```dart 1584:1592:packages/flutter/lib/src/material/date_picker.dart
  bool _isDaySelectable(DateTime day) {
    if (day.isBefore(widget.firstDate) || day.isAfter(widget.lastDate)) {
      return false;
    }
    if (widget.selectableDayPredicate == null) {
      return true;
    }
    return widget.selectableDayPredicate!(day, _selectedStart.value, _selectedEnd.value);
  }
```

检查给定的日期是否可以被选择：

- 检查日期是否在允许的范围内（firstDate 到 lastDate）
- 如果提供了自定义的选择谓词，调用它进行额外验证

### 日期变更处理

```dart 1594:1600:packages/flutter/lib/src/material/date_picker.dart
  void _handleStartDateChanged(DateTime? date) {
    setState(() => _selectedStart.value = date);
  }

  void _handleEndDateChanged(DateTime? date) {
    setState(() => _selectedEnd.value = date);
  }
```

处理开始和结束日期的变化，更新相应的状态变量。

### 便捷属性

```dart 1602:1602:packages/flutter/lib/src/material/date_picker.dart
  bool get _hasSelectedDateRange => _selectedStart.value != null && _selectedEnd.value != null;
```

检查是否已经选择了完整的日期范围（开始和结束日期都已选择）。

## 总结

这个状态类主要负责：

1. **状态管理**：使用可恢复的状态对象管理输入模式和选中的日期
2. **事件处理**：处理确认、取消和模式切换等用户交互
3. **验证逻辑**：确保选择的日期有效且符合约束条件
4. **状态恢复**：支持应用重启后恢复用户的选择状态

这些方法共同确保了日期范围选择器的可靠性和用户体验的一致性。
