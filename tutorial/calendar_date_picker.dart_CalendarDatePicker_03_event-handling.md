# CalendarDatePicker 事件处理方法

## 模式切换处理

`_handleModeChanged` 方法处理日视图和年视图之间的切换：

```dart 260:272:packages/flutter/lib/src/material/calendar_date_picker.dart
  void _handleModeChanged(DatePickerMode mode) {
    _vibrate();
    setState(() {
      _mode = mode;
      if (_selectedDate case final DateTime selected) {
        final String message = switch (mode) {
          DatePickerMode.day => widget.calendarDelegate.formatMonthYear(selected, _localizations),
          DatePickerMode.year => widget.calendarDelegate.formatYear(selected.year, _localizations),
        };
        SemanticsService.sendAnnouncement(View.of(context), message, _textDirection);
      }
    });
  }
```

处理逻辑：

1. 触发触觉反馈
2. 更新显示模式
3. 使用模式匹配语法检查是否有选中的日期
4. 根据新模式生成相应的无障碍公告消息

## 月份变化处理

`_handleMonthChanged` 方法处理用户导航到不同月份时的逻辑：

```dart 274:282:packages/flutter/lib/src/material/calendar_date_picker.dart
  void _handleMonthChanged(DateTime date) {
    setState(() {
      if (_currentDisplayedMonthDate.year != date.year ||
          _currentDisplayedMonthDate.month != date.month) {
        _currentDisplayedMonthDate = widget.calendarDelegate.getMonth(date.year, date.month);
        widget.onDisplayedMonthChanged?.call(_currentDisplayedMonthDate);
      }
    });
  }
```

处理逻辑：

1. 检查新日期是否与当前显示月份不同
2. 更新 `_currentDisplayedMonthDate`
3. 触发 `onDisplayedMonthChanged` 回调（如果提供）

## 年份选择处理

`_handleYearChanged` 方法处理用户从年视图选择年份的复杂逻辑：

```dart 284:306:packages/flutter/lib/src/material/calendar_date_picker.dart
  void _handleYearChanged(DateTime value) {
    _vibrate();
    setState(() {
      _mode = DatePickerMode.day;
      _handleMonthChanged(value);

      if (_isSelectable(value)) {
        _selectedDate = value;
        widget.onDateChanged(_selectedDate!);
      }
    });
  }
```

这个方法在用户从年选择器选择年份时调用：

1. 触发触觉反馈
2. 切换回日视图模式
3. 更新显示月份
4. 如果新日期可选择，则选中它并触发日期变化回调

值得注意的是，这个方法会自动调整日期以确保有效性：

```dart 287:290:packages/flutter/lib/src/material/calendar_date_picker.dart
    final int daysInMonth = widget.calendarDelegate.getDaysInMonth(value.year, value.month);
    final int preferredDay = math.min(_selectedDate?.day ?? 1, daysInMonth);
    value = widget.calendarDelegate.getDay(value.year, value.month, preferredDay);
```

这段代码确保：

- 获取目标年月的天数
- 选择合适的日期（优先使用之前选中的日期，否则使用1号）
- 确保日期不超过该月的最大天数

## 日期选择处理

`_handleDayChanged` 方法处理用户在日历网格中选择具体日期：

```dart 308:330:packages/flutter/lib/src/material/calendar_date_picker.dart
  void _handleDayChanged(DateTime value) {
    _vibrate();
    setState(() {
      _selectedDate = value;
      widget.onDateChanged(_selectedDate!);
      switch (Theme.of(context).platform) {
        case TargetPlatform.linux:
        case TargetPlatform.macOS:
        case TargetPlatform.windows:
          final bool isToday = widget.calendarDelegate.isSameDay(widget.currentDate, _selectedDate);
          final String semanticLabelSuffix = isToday ? ', ${_localizations.currentDateLabel}' : '';
          SemanticsService.sendAnnouncement(
            View.of(context),
            '${_localizations.selectedDateLabel} ${widget.calendarDelegate.formatFullDate(_selectedDate!, _localizations)}$semanticLabelSuffix',
            _textDirection,
          );
        case TargetPlatform.android:
        case TargetPlatform.iOS:
        case TargetPlatform.fuchsia:
          break;
      }
    });
  }
```

处理逻辑：

1. 触发触觉反馈
2. 更新选中的日期
3. 触发 `onDateChanged` 回调
4. 在桌面平台（Linux/macOS/Windows）上宣布日期选择（移动平台不宣布，因为已经有视觉反馈）

## 事件处理的平台差异

组件根据不同平台采用不同的交互策略：

### 触觉反馈

- **移动平台**（Android、iOS、Fuchsia）：在所有交互时提供触觉反馈
- **桌面平台**（Linux、macOS、Windows）：不提供触觉反馈

### 无障碍公告

- **桌面平台**：详细宣布模式切换和日期选择
- **移动平台**：只在初始化时宣布，避免过多语音干扰

这种差异化处理确保了在不同平台上都能提供最佳的用户体验。

## 日期验证和边界处理

所有日期变化都会经过验证：

```dart 291:305:packages/flutter/lib/src/material/calendar_date_picker.dart
    if (value.isBefore(widget.firstDate)) {
      value = widget.firstDate;
    } else if (value.isAfter(widget.lastDate)) {
      value = widget.lastDate;
    }
```

- 如果选择的日期早于 `firstDate`，自动调整为 `firstDate`
- 如果选择的日期晚于 `lastDate`，自动调整为 `lastDate`
- 只有可选择的日期（通过 `_isSelectable` 验证）才会被真正选中

## 事件处理的设计模式

组件使用以下设计模式来处理用户交互：

1. **统一的状态更新**：所有事件处理都通过 `setState` 更新状态
2. **组合事件处理**：复杂操作（如年份选择）会组合使用多个处理方法
3. **平台适配**：根据平台特性调整反馈机制
4. **无障碍优先**：确保所有交互都能被辅助技术正确理解
5. **数据验证**：在接受用户输入前进行严格的边界和有效性检查

这种设计确保了组件的可靠性和用户体验的一致性。
