# DatePickerDialog 输入模式管理与选择器实现

## 输入模式切换逻辑

```dart 518:534:packages/flutter/lib/src/material/date_picker.dart
  void _handleEntryModeToggle() {
    setState(() {
      switch (_entryMode.value) {
        case DatePickerEntryMode.calendar:
          _autovalidateMode.value = AutovalidateMode.disabled;
          _entryMode.value = DatePickerEntryMode.input;
          _handleOnDatePickerModeChange();
        case DatePickerEntryMode.input:
          _formKey.currentState!.save();
          _entryMode.value = DatePickerEntryMode.calendar;
          _handleOnDatePickerModeChange();
        case DatePickerEntryMode.calendarOnly:
        case DatePickerEntryMode.inputOnly:
          assert(false, 'Can not change entry mode from ${_entryMode.value}');
      }
    });
  }
```

### 模式切换规则

- **日历模式 → 输入模式**:
  - 禁用自动验证（避免切换时的错误提示）
  - 切换到输入模式
  - 触发模式变更回调

- **输入模式 → 日历模式**:
  - 保存表单数据（确保输入的值被应用）
  - 切换到日历模式
  - 触发模式变更回调

- **单一模式**: `calendarOnly` 和 `inputOnly` 模式不允许切换，会抛出断言错误

### 设计意图

1. **状态一致性**: 从输入模式切换回日历模式时保存表单，确保数据不丢失
2. **用户体验**: 从日历切换到输入模式时禁用验证，避免不必要的错误提示
3. **约束完整性**: 单一模式确保组件按预期使用，不会意外切换

## 日期变更处理

```dart 536:538:packages/flutter/lib/src/material/date_picker.dart
  void _handleDateChanged(DateTime date) {
    setState(() => _selectedDate.value = date);
  }
```

简单直接的日期更新逻辑，通过 `setState` 触发UI重新构建。

## 对话框尺寸策略

```dart 540:556:packages/flutter/lib/src/material/date_picker.dart
  Size _dialogSize(BuildContext context) {
    final bool useMaterial3 = Theme.of(context).useMaterial3;
    final bool isCalendar = switch (_entryMode.value) {
      DatePickerEntryMode.calendar || DatePickerEntryMode.calendarOnly => true,
      DatePickerEntryMode.input || DatePickerEntryMode.inputOnly => false,
    };
    final Orientation orientation = MediaQuery.orientationOf(context);

    return switch ((isCalendar, orientation)) {
      (true, Orientation.portrait) when useMaterial3 => _calendarPortraitDialogSizeM3,
      (false, Orientation.portrait) when useMaterial3 => _inputPortraitDialogSizeM3,
      (true, Orientation.portrait) => _calendarPortraitDialogSizeM2,
      (false, Orientation.portrait) => _inputPortraitDialogSizeM2,
      (true, Orientation.landscape) => _calendarLandscapeDialogSize,
      (false, Orientation.landscape) => _inputLandscapeDialogSize,
    };
  }
```

### 尺寸策略说明

根据三个维度确定对话框尺寸：

1. **Material版本**: M2 vs M3 有不同的尺寸标准
2. **模式**: 日历模式 vs 输入模式 需要不同的空间
3. **屏幕方向**: 纵向 vs 横向 有不同的布局要求

## 表单快捷键

```dart 558:561:packages/flutter/lib/src/material/date_picker.dart
  static const Map<ShortcutActivator, Intent> _formShortcutMap = <ShortcutActivator, Intent>{
    // Pressing enter on the field will move focus to the next field or control.
    SingleActivator(LogicalKeyboardKey.enter): NextFocusIntent(),
  };
```

提供键盘导航支持，按Enter键可以在表单字段间移动焦点。

## 操作按钮（Actions）

```dart 598:629:packages/flutter/lib/src/material/date_picker.dart
    final Widget actions = ConstrainedBox(
      constraints: const BoxConstraints(minHeight: 52.0),
      child: MediaQuery.withClampedTextScaling(
        maxScaleFactor: isLandscapeOrientation ? 1.6 : _kMaxTextScaleFactor,
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 8),
          child: Align(
            alignment: AlignmentDirectional.centerEnd,
            child: OverflowBar(
              spacing: 8,
              children: <Widget>[
                TextButton(
                  style: datePickerTheme.cancelButtonStyle ?? defaults.cancelButtonStyle,
                  onPressed: _handleCancel,
                  child: Text(
                    widget.cancelText ??
                        (useMaterial3
                            ? localizations.cancelButtonLabel
                            : localizations.cancelButtonLabel.toUpperCase()),
                  ),
                ),
                TextButton(
                  style: datePickerTheme.confirmButtonStyle ?? defaults.confirmButtonStyle,
                  onPressed: _handleOk,
                  child: Text(widget.confirmText ?? localizations.okButtonLabel),
                ),
              ],
            ),
          ),
        ),
      ),
    );
```

### 按钮特性

1. **布局**: 使用 `OverflowBar` 确保按钮始终可见，必要时可换行
2. **文本**: 根据Material版本决定是否大写（M2大写，M3保持原样）
3. **样式**: 支持主题自定义，通过 `DatePickerThemeData` 配置
4. **文本缩放**: 使用 `MediaQuery.withClampedTextScaling` 防止过度缩放

## 日历日期选择器

```dart 631:643:packages/flutter/lib/src/material/date_picker.dart
    CalendarDatePicker calendarDatePicker() {
      return CalendarDatePicker(
        calendarDelegate: widget.calendarDelegate,
        key: _calendarPickerKey,
        initialDate: _selectedDate.value,
        firstDate: widget.firstDate,
        lastDate: widget.lastDate,
        currentDate: widget.currentDate,
        onDateChanged: _handleDateChanged,
        selectableDayPredicate: widget.selectableDayPredicate,
        initialCalendarMode: widget.initialCalendarMode,
      );
    }
```

### 参数传递

- **calendarDelegate**: 传递日期处理逻辑
- **key**: 用于组件访问，可能用于滚动控制
- **日期范围**: 传递 firstDate、lastDate、currentDate
- **选中状态**: initialDate 和 onDateChanged 回调
- **可选性控制**: selectableDayPredicate
- **初始模式**: initialCalendarMode（日/年视图）

## 输入日期选择器

```dart 645:686:packages/flutter/lib/src/material/date_picker.dart
    Form inputDatePicker() {
      return Form(
        key: _formKey,
        autovalidateMode: _autovalidateMode.value,
        child: SizedBox(
          height: orientation == Orientation.portrait
              ? _inputFormPortraitHeight
              : _inputFormLandscapeHeight,
          child: Padding(
            padding: const EdgeInsets.symmetric(horizontal: 24),
            child: Shortcuts(
              shortcuts: _formShortcutMap,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Flexible(
                    child: MediaQuery.withClampedTextScaling(
                      maxScaleFactor: 2.0,
                      child: InputDatePickerFormField(
                        calendarDelegate: widget.calendarDelegate,
                        initialDate: _selectedDate.value,
                        firstDate: widget.firstDate,
                        lastDate: widget.lastDate,
                        onDateSubmitted: _handleDateChanged,
                        onDateSaved: _handleDateChanged,
                        selectableDayPredicate: widget.selectableDayPredicate,
                        errorFormatText: widget.errorFormatText,
                        errorInvalidText: widget.errorInvalidText,
                        fieldHintText: widget.fieldHintText,
                        fieldLabelText: widget.fieldLabelText,
                        keyboardType: widget.keyboardType,
                        autofocus: true,
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ),
      );
    }
```

### 输入模式特性

1. **表单管理**: 使用 `Form` 和 `GlobalKey<FormState>` 进行验证和保存
2. **自动验证**: 根据 `_autovalidateMode` 控制验证时机
3. **尺寸适配**: 根据屏幕方向使用不同的高度
4. **键盘支持**: 集成快捷键映射，支持Enter键导航
5. **文本缩放**: 允许更大的缩放因子（2.0）以适应输入需求
6. **自动聚焦**: `autofocus: true` 确保输入框获得焦点

### InputDatePickerFormField 参数

- **日期处理**: calendarDelegate、日期范围、回调函数
- **错误处理**: 自定义错误文本（格式错误、无效日期）
- **UI定制**: 提示文本、标签文本、键盘类型
- **验证**: selectableDayPredicate 确保输入日期有效

## 模式选择器与按钮

```dart 688:718:packages/flutter/lib/src/material/date_picker.dart
    final Widget picker;
    final Widget? entryModeButton;
    switch (_entryMode.value) {
      case DatePickerEntryMode.calendar:
        picker = calendarDatePicker();
        entryModeButton = IconButton(
          icon:
              widget.switchToInputEntryModeIcon ??
              Icon(useMaterial3 ? Icons.edit_outlined : Icons.edit),
          color: headerForegroundColor,
          tooltip: localizations.inputDateModeButtonLabel,
          onPressed: _handleEntryModeToggle,
        );

      case DatePickerEntryMode.calendarOnly:
        picker = calendarDatePicker();
        entryModeButton = null;

      case DatePickerEntryMode.input:
        picker = inputDatePicker();
        entryModeButton = IconButton(
          icon: widget.switchToCalendarEntryModeIcon ?? const Icon(Icons.calendar_today),
          color: headerForegroundColor,
          tooltip: localizations.calendarModeButtonLabel,
          onPressed: _handleEntryModeToggle,
        );

      case DatePickerEntryMode.inputOnly:
        picker = inputDatePicker();
        entryModeButton = null;
    }
```

### 模式按钮特性

1. **图标选择**:
   - 日历模式：显示编辑图标（M3使用 `edit_outlined`，M2使用 `edit`）
   - 输入模式：显示日历图标（`calendar_today`）

2. **可定制性**: 支持通过 widget 参数自定义图标

3. **单一模式**: `calendarOnly` 和 `inputOnly` 不显示切换按钮

4. **无障碍**: 提供工具提示文本，支持屏幕阅读器

## 头部组件

```dart 720:733:packages/flutter/lib/src/material/date_picker.dart
    final Widget header = _DatePickerHeader(
      helpText:
          widget.helpText ??
          (useMaterial3
              ? localizations.datePickerHelpText
              : localizations.datePickerHelpText.toUpperCase()),
      titleText: _selectedDate.value == null
          ? ''
          : widget.calendarDelegate.formatMediumDate(_selectedDate.value!, localizations),
      titleStyle: headlineStyle,
      orientation: orientation,
      isShort: orientation == Orientation.landscape,
      entryModeButton: entryModeButton,
    );
```

### 头部特性

1. **帮助文本**: 支持自定义，否则使用本地化文本（M3保持原样，M2大写）
2. **标题文本**: 显示格式化的选中日期，无选中时为空
3. **样式适配**: 根据模式和方向调整文本样式
4. **按钮集成**: 嵌入模式切换按钮

## Material 2/3 差异

### 按钮文本

- M2: 取消和确认按钮文本大写
- M3: 保持原始大小写

### 图标选择

- M2: 使用 `Icons.edit`
- M3: 使用 `Icons.edit_outlined`

### 帮助文本

- M2: 大写显示
- M3: 保持原始大小写

### 对话框尺寸

- M2/M3 有不同的尺寸常量
- 横向输入模式在 M3 中没有专门规范

## 设计模式

### 组合模式

- 将复杂的日期选择逻辑分解为两个专门的选择器组件
- 通过统一的接口（参数传递和回调）保持一致性

### 状态驱动UI

- UI完全由 `_entryMode.value` 驱动
- 每个模式都有对应的选择器和按钮配置

### 渐进式增强

- 基础功能（单一模式）作为基础
- 高级功能（模式切换）作为可选增强

## 维护注意事项

1. **图标更新**: Material 3 图标更新时需要同步图标选择逻辑
2. **本地化**: 确保所有本地化键都有对应的翻译
3. **键盘导航**: 测试各种键盘交互场景的可用性
4. **无障碍**: 验证屏幕阅读器对所有交互元素的正确识别
