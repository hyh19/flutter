# DatePickerDialog 状态恢复与事件处理

## 状态类定义

`_DatePickerDialogState` 继承自 `State<DatePickerDialog>` 并混入 `RestorationMixin`，提供了完整的状态持久化和恢复功能。

```dart 467:474:packages/flutter/lib/src/material/date_picker.dart
class _DatePickerDialogState extends State<DatePickerDialog> with RestorationMixin {
  late final RestorableDateTimeN _selectedDate = RestorableDateTimeN(widget.initialDate);
  late final _RestorableDatePickerEntryMode _entryMode = _RestorableDatePickerEntryMode(
    widget.initialEntryMode,
  );
  final _RestorableAutovalidateMode _autovalidateMode = _RestorableAutovalidateMode(
    AutovalidateMode.disabled,
  );
```

### Restorable 字段说明

- **`_selectedDate`**: 存储当前选中的日期，使用 `RestorableDateTimeN` 类型，支持空值
- **`_entryMode`**: 存储当前的输入模式（日历/输入），使用自定义的 `_RestorableDatePickerEntryMode`
- **`_autovalidateMode`**: 存储表单验证模式，使用 `_RestorableAutovalidateMode`

这些字段都是 `late final`，意味着它们在第一次访问时初始化，并且一旦初始化就不会改变。

## 状态恢复机制

```dart 476:492:packages/flutter/lib/src/material/date_picker.dart
  @override
  void dispose() {
    _selectedDate.dispose();
    _entryMode.dispose();
    _autovalidateMode.dispose();
    super.dispose();
  }

  @override
  String? get restorationId => widget.restorationId;

  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    registerForRestoration(_selectedDate, 'selected_date');
    registerForRestoration(_autovalidateMode, 'autovalidateMode');
    registerForRestoration(_entryMode, 'calendar_entry_mode');
  }
```

### RestorationMixin 工作原理

1. **`restorationId`**: 返回 widget 的 `restorationId`，只有当不为 null 时才会启用状态恢复
2. **`restoreState`**: 在状态恢复过程中调用，为每个可恢复字段注册恢复键
3. **`dispose`**: 清理所有可恢复字段，确保资源正确释放

### 恢复键说明

- `'selected_date'`: 用于恢复选中的日期
- `'autovalidateMode'`: 用于恢复表单验证模式
- `'calendar_entry_mode'`: 用于恢复输入模式（日历/输入）

## 事件处理方法

```dart 497:516:packages/flutter/lib/src/material/date_picker.dart
  void _handleOk() {
    if (_entryMode.value == DatePickerEntryMode.input ||
        _entryMode.value == DatePickerEntryMode.inputOnly) {
      final FormState form = _formKey.currentState!;
      if (!form.validate()) {
        setState(() => _autovalidateMode.value = AutovalidateMode.always);
        return;
      }
      form.save();
    }
    Navigator.pop(context, _selectedDate.value);
  }

  void _handleCancel() {
    Navigator.pop(context);
  }

  void _handleOnDatePickerModeChange() {
    widget.onDatePickerModeChange?.call(_entryMode.value);
  }
```

### _handleOk() - 确认处理

**执行流程**：

1. 检查当前是否处于输入模式（`input` 或 `inputOnly`）
2. 如果是输入模式，获取表单状态并验证
3. 如果验证失败，设置自动验证模式为 `always` 并返回（不关闭对话框）
4. 如果验证通过或处于日历模式，保存表单数据
5. 使用 `Navigator.pop()` 返回选中的日期

**设计意图**：

- 在输入模式下强制验证，确保用户输入的日期格式正确
- 日历模式下不需要额外验证，因为 UI 已经限制了选择范围
- 失败时不关闭对话框，给用户修正错误的机会

### _handleCancel() - 取消处理

**执行流程**：

1. 直接调用 `Navigator.pop(context)` 关闭对话框
2. 不传递任何返回值（返回 null）

**设计意图**：

- 简单直接的取消操作
- 不保存任何状态变化

### _handleOnDatePickerModeChange() - 模式切换通知

**执行流程**：

1. 调用 widget 的 `onDatePickerModeChange` 回调
2. 传递当前的输入模式值

**设计意图**：

- 允许父组件监听模式切换事件
- 支持保存用户的偏好设置（如记住最后使用的输入模式）

## 辅助字段

```dart 494:495:packages/flutter/lib/src/material/date_picker.dart
  final GlobalKey _calendarPickerKey = GlobalKey();
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
```

- **`_calendarPickerKey`**: 用于访问 `CalendarDatePicker` 组件，可能用于滚动到特定日期
- **`_formKey`**: 用于管理输入模式的表单状态，验证和保存用户输入

## 状态管理设计特点

### 持久化策略

1. **选择性恢复**: 只恢复用户交互产生的重要状态（选中日期、输入模式、验证状态）
2. **类型安全**: 使用专门的 Restorable 类型确保类型安全
3. **资源管理**: 正确实现 dispose 防止内存泄漏

### 事件处理模式

1. **条件验证**: 根据当前模式决定是否需要表单验证
2. **渐进式验证**: 输入模式下使用 `AutovalidateMode.always` 提供即时反馈
3. **回调通知**: 通过回调允许外部监听状态变化

### 错误处理

1. **表单验证**: 在输入模式下验证日期格式和范围
2. **状态保持**: 验证失败时保持对话框打开，让用户修正
3. **用户反馈**: 通过设置 `autovalidateMode` 提供视觉反馈

## 使用注意事项

1. **状态恢复依赖**: 需要在 `RestorationScope` 中使用才能生效
2. **性能考虑**: Restorable 字段的初始化是延迟的，避免不必要的开销
3. **内存管理**: 所有 Restorable 字段都必须在 dispose 中正确清理
4. **模式切换**: 切换模式时会触发回调，但不会自动保存用户偏好
