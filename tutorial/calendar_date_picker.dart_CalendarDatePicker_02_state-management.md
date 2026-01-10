# CalendarDatePicker 状态管理和初始化

## 状态类定义

`_CalendarDatePickerState` 类负责管理 `CalendarDatePicker` 组件的所有状态和交互逻辑：

```dart 202:211:packages/flutter/lib/src/material/calendar_date_picker.dart
class _CalendarDatePickerState extends State<CalendarDatePicker> {
  bool _announcedInitialDate = false;
  late DatePickerMode _mode;
  late DateTime _currentDisplayedMonthDate;
  DateTime? _selectedDate;
  final GlobalKey _monthPickerKey = GlobalKey();
  final GlobalKey _yearPickerKey = GlobalKey();
  late MaterialLocalizations _localizations;
  late TextDirection _textDirection;
```

### 状态变量

- **`_announcedInitialDate`**: 标记是否已宣布初始日期（用于无障碍功能）
- **`_mode`**: 当前显示模式（`DatePickerMode.day` 或 `DatePickerMode.year`）
- **`_currentDisplayedMonthDate`**: 当前显示的月份日期
- **`_selectedDate`**: 当前选中的日期
- **`_monthPickerKey`**: 月选择器的全局键
- **`_yearPickerKey`**: 年选择器的全局键
- **`_localizations`**: Material 本地化资源
- **`_textDirection`**: 文本方向

## 初始化阶段

### initState 方法

组件初始化时设置基础状态：

```dart 212:224:packages/flutter/lib/src/material/calendar_date_picker.dart
  @override
  void initState() {
    super.initState();
    _mode = widget.initialCalendarMode;
    final DateTime currentDisplayedDate = widget.initialDate ?? widget.currentDate;
    _currentDisplayedMonthDate = widget.calendarDelegate.getMonth(
      currentDisplayedDate.year,
      currentDisplayedDate.month,
    );
    if (widget.initialDate != null) {
      _selectedDate = widget.initialDate;
    }
  }
```

初始化逻辑：

1. 设置初始显示模式为 `widget.initialCalendarMode`
2. 确定当前显示的月份（基于 `initialDate` 或 `currentDate`）
3. 如果提供了 `initialDate`，将其设置为选中日期

### didChangeDependencies 方法

依赖变化时进行本地化和无障碍设置：

```dart 226:245:packages/flutter/lib/src/material/calendar_date_picker.dart
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    assert(debugCheckHasMaterial(context));
    assert(debugCheckHasMaterialLocalizations(context));
    assert(debugCheckHasDirectionality(context));
    _localizations = MaterialLocalizations.of(context);
    _textDirection = Directionality.of(context);
    if (!_announcedInitialDate && widget.initialDate != null) {
      assert(_selectedDate != null);
      _announcedInitialDate = true;
      final bool isToday = widget.calendarDelegate.isSameDay(widget.currentDate, _selectedDate);
      final String semanticLabelSuffix = isToday ? ', ${_localizations.currentDateLabel}' : '';
      SemanticsService.sendAnnouncement(
        View.of(context),
        '${_localizations.formatFullDate(_selectedDate!)}$semanticLabelSuffix',
        _textDirection,
      );
    }
  }
```

此方法执行以下操作：

1. **断言检查**：确保必要的 Material 组件和本地化资源可用
2. **资源获取**：
   - 获取 `MaterialLocalizations` 用于格式化日期和文本
   - 获取 `TextDirection` 用于确定文本方向
3. **无障碍公告**：首次显示时宣布选中的日期（仅执行一次）

## 触觉反馈机制

组件提供平台特定的触觉反馈：

```dart 247:258:packages/flutter/lib/src/material/calendar_date_picker.dart
  void _vibrate() {
    switch (Theme.of(context).platform) {
      case TargetPlatform.android:
      case TargetPlatform.fuchsia:
      case TargetPlatform.linux:
      case TargetPlatform.windows:
        HapticFeedback.vibrate();
      case TargetPlatform.iOS:
      case TargetPlatform.macOS:
        break;
    }
  }
```

- **Android/Fuchsia/Linux/Windows**：使用 `HapticFeedback.vibrate()` 提供振动反馈
- **iOS/macOS**：不提供触觉反馈（遵循平台规范）

这个方法在用户进行重要操作（如模式切换、日期选择）时被调用。

## 选择性日期验证

`_isSelectable` 方法用于验证日期是否可以被选择：

```dart 332:334:packages/flutter/lib/src/material/calendar_date_picker.dart
  bool _isSelectable(DateTime date) {
    return widget.selectableDayPredicate?.call(date) ?? true;
  }
```

- 如果提供了 `selectableDayPredicate`，使用它进行验证
- 如果没有提供，默认所有日期都可选择
- 这个方法在处理日期变化时使用，确保只选择有效的日期

## 状态更新的设计原则

组件遵循以下状态管理原则：

1. **不可变配置**：`initialDate` 和 `initialCalendarMode` 的更改不会生效，需要更换 `key` 来重置组件
2. **响应式更新**：通过 `setState` 触发UI重建
3. **依赖管理**：在 `didChangeDependencies` 中处理本地化和方向性变化
4. **无障碍支持**：使用语义服务宣布重要状态变化
5. **平台适配**：根据不同平台提供相应的触觉反馈

这种设计确保了组件的稳定性和可预测性，同时提供了良好的用户体验。
