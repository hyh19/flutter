# DateRangePickerDialog UI 结构

## 概述

`build` 方法是 `_DateRangePickerDialogState` 的核心，负责根据当前的状态和配置构建合适的UI。这个方法根据输入模式（日历模式或文本输入模式）渲染不同的界面，并应用相应的主题和布局。

## build 方法结构

```dart 1605:1611:packages/flutter/lib/src/material/date_picker.dart
  @override
  Widget build(BuildContext context) {
    final ThemeData theme = Theme.of(context);
    final bool useMaterial3 = theme.useMaterial3;
    final Orientation orientation = MediaQuery.orientationOf(context);
    final MaterialLocalizations localizations = MaterialLocalizations.of(context);
    final DatePickerThemeData datePickerTheme = DatePickerTheme.of(context);
    final DatePickerThemeData defaults = DatePickerTheme.defaults(context);
```

首先获取必要的上下文信息：

- `theme`: 当前主题
- `useMaterial3`: 是否使用 Material 3 设计
- `orientation`: 设备方向
- `localizations`: 本地化文本
- `datePickerTheme`: 日期选择器主题
- `defaults`: 默认主题数据

## 模式切换按钮配置

```dart 1620:1622:packages/flutter/lib/src/material/date_picker.dart
    final bool showEntryModeButton =
        _entryMode.value == DatePickerEntryMode.calendar ||
        _entryMode.value == DatePickerEntryMode.input;
```

只有在 `calendar` 或 `input` 模式下才显示模式切换按钮，`calendarOnly` 和 `inputOnly` 模式下不显示。

## 日历模式 UI 构建

```dart 1624:1667:packages/flutter/lib/src/material/date_picker.dart
      case DatePickerEntryMode.calendar:
      case DatePickerEntryMode.calendarOnly:
        contents = _CalendarRangePickerDialog(
          key: _calendarPickerKey,
          calendarDelegate: widget.calendarDelegate,
          selectedStartDate: _selectedStart.value,
          selectedEndDate: _selectedEnd.value,
          firstDate: widget.firstDate,
          lastDate: widget.lastDate,
          selectableDayPredicate: widget.selectableDayPredicate,
          currentDate: widget.currentDate,
          onStartDateChanged: _handleStartDateChanged,
          onEndDateChanged: _handleEndDateChanged,
          onConfirm: _hasSelectedDateRange ? _handleOk : null,
          onCancel: _handleCancel,
          entryModeButton: showEntryModeButton
              ? IconButton(
                  icon:
                      widget.switchToInputEntryModeIcon ??
                      Icon(useMaterial3 ? Icons.edit_outlined : Icons.edit),
                  padding: EdgeInsets.zero,
                  tooltip: localizations.inputDateModeButtonLabel,
                  onPressed: _handleEntryModeToggle,
                )
              : null,
          confirmText:
              widget.saveText ??
              (useMaterial3
                  ? localizations.saveButtonLabel
                  : localizations.saveButtonLabel.toUpperCase()),
          helpText:
              widget.helpText ??
              (useMaterial3
                  ? localizations.dateRangePickerHelpText
                  : localizations.dateRangePickerHelpText.toUpperCase()),
        );
        size = MediaQuery.sizeOf(context);
        insetPadding = EdgeInsets.zero;
        elevation = datePickerTheme.rangePickerElevation ?? defaults.rangePickerElevation!;
        shadowColor = datePickerTheme.rangePickerShadowColor ?? defaults.rangePickerShadowColor!;
        surfaceTintColor =
            datePickerTheme.rangePickerSurfaceTintColor ?? defaults.rangePickerSurfaceTintColor!;
        shape = datePickerTheme.rangePickerShape ?? defaults.rangePickerShape;
```

日历模式使用 `_CalendarRangePickerDialog` 组件：

- **核心功能**：传递所有必要的日期参数和回调函数
- **模式切换按钮**：显示切换到输入模式的图标按钮
- **尺寸设置**：使用全屏尺寸（`MediaQuery.sizeOf(context)`）
- **主题配置**：使用范围选择器特定的主题属性

## 文本输入模式 UI 构建

```dart 1668:1747:packages/flutter/lib/src/material/date_picker.dart
      case DatePickerEntryMode.input:
      case DatePickerEntryMode.inputOnly:
        contents = _InputDateRangePickerDialog(
          calendarDelegate: widget.calendarDelegate,
          selectedStartDate: _selectedStart.value,
          selectedEndDate: _selectedEnd.value,
          currentDate: widget.currentDate,
          picker: SizedBox(
            height: orientation == Orientation.portrait
                ? _inputFormPortraitHeight
                : _inputFormLandscapeHeight,
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 24),
              child: Column(
                children: <Widget>[
                  const Spacer(),
                  _InputDateRangePicker(
                    key: _inputPickerKey,
                    calendarDelegate: widget.calendarDelegate,
                    initialStartDate: _selectedStart.value,
                    initialEndDate: _selectedEnd.value,
                    firstDate: widget.firstDate,
                    lastDate: widget.lastDate,
                    selectableDayPredicate: widget.selectableDayPredicate,
                    onStartDateChanged: _handleStartDateChanged,
                    onEndDateChanged: _handleEndDateChanged,
                    autofocus: true,
                    autovalidate: _autoValidate.value,
                    helpText: widget.helpText,
                    errorInvalidRangeText: widget.errorInvalidRangeText,
                    errorFormatText: widget.errorFormatText,
                    errorInvalidText: widget.errorInvalidText,
                    fieldStartHintText: widget.fieldStartHintText,
                    fieldEndHintText: widget.fieldEndHintText,
                    fieldStartLabelText: widget.fieldStartLabelText,
                    fieldEndLabelText: widget.fieldEndLabelText,
                    keyboardType: widget.keyboardType,
                  ),
                  const Spacer(),
                ],
              ),
            ),
          ),
          onConfirm: _handleOk,
          onCancel: _handleCancel,
          entryModeButton: showEntryModeButton
              ? IconButton(
                  icon: widget.switchToCalendarEntryModeIcon ?? const Icon(Icons.calendar_today),
                  padding: EdgeInsets.zero,
                  tooltip: localizations.calendarModeButtonLabel,
                  onPressed: _handleEntryModeToggle,
                )
              : null,
          confirmText: widget.confirmText ?? localizations.okButtonLabel,
          cancelText:
              widget.cancelText ??
              (useMaterial3
                  ? localizations.cancelButtonLabel
                  : localizations.cancelButtonLabel.toUpperCase()),
          helpText:
              widget.helpText ??
              (useMaterial3
                  ? localizations.dateRangePickerHelpText
                  : localizations.dateRangePickerHelpText.toUpperCase()),
        );
        final DialogThemeData dialogTheme = theme.dialogTheme;
        size = orientation == Orientation.portrait
            ? (useMaterial3 ? _inputPortraitDialogSizeM3 : _inputPortraitDialogSizeM2)
            : _inputRangeLandscapeDialogSize;
        elevation = useMaterial3
            ? datePickerTheme.elevation ?? defaults.elevation!
            : datePickerTheme.elevation ?? dialogTheme.elevation ?? 24;
        shadowColor = datePickerTheme.shadowColor ?? defaults.shadowColor;
        surfaceTintColor = datePickerTheme.surfaceTintColor ?? defaults.surfaceTintColor;
        shape = useMaterial3
            ? datePickerTheme.shape ?? defaults.shape
            : datePickerTheme.shape ?? dialogTheme.shape ?? defaults.shape;

        insetPadding = const EdgeInsets.symmetric(horizontal: 16.0, vertical: 24.0);
```

文本输入模式使用 `_InputDateRangePickerDialog` 组件：

- **布局结构**：
  - 使用 `SizedBox` 控制高度（根据方向调整）
  - 使用 `Padding` 和 `Column` 创建居中的布局
  - `_InputDateRangePicker` 是核心的文本输入组件

- **输入组件配置**：
  - 支持自动聚焦和自动验证
  - 传递所有错误提示文本配置
  - 使用指定的键盘类型

- **尺寸和主题**：
  - 纵向和横向有不同的对话框尺寸
  - 使用标准的对话框主题属性
  - 添加适当的内边距

## Dialog 组件构建

```dart 1749:1772:packages/flutter/lib/src/material/date_picker.dart
    return Dialog(
      insetPadding: insetPadding,
      backgroundColor: datePickerTheme.backgroundColor ?? defaults.backgroundColor,
      elevation: elevation,
      shadowColor: shadowColor,
      surfaceTintColor: surfaceTintColor,
      shape: shape,
      clipBehavior: Clip.antiAlias,
      child: AnimatedContainer(
        width: size.width,
        height: size.height,
        duration: _dialogSizeAnimationDuration,
        curve: Curves.easeIn,
        child: MediaQuery.withClampedTextScaling(
          maxScaleFactor: _kMaxRangeTextScaleFactor,
          child: Builder(
            builder: (BuildContext context) {
              return contents;
            },
          ),
        ),
      ),
    );
```

最终的 Dialog 配置：

- **主题应用**：使用日期选择器主题的背景色、阴影等属性
- **动画过渡**：`AnimatedContainer` 提供平滑的尺寸变化动画
- **文字缩放**：限制最大文字缩放因子以保持UI的一致性
- **内容渲染**：通过 `Builder` 确保正确的上下文传递

## 总结

这个 build 方法实现了：

1. **响应式设计**：根据设备方向和 Material 版本调整UI
2. **模式切换**：在日历和文本输入模式之间无缝切换
3. **主题集成**：充分利用 Flutter 的主题系统
4. **动画效果**：提供平滑的界面过渡
5. **可访问性**：支持本地化和适当的工具提示

通过这种架构，`DateRangePickerDialog` 能够在不同模式和配置下提供一致且高质量的用户体验。
