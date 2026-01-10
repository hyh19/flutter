# CalendarDatePicker 组件构建和布局

## 构建方法概述

`build` 方法负责根据当前状态构建完整的UI布局：

```dart 367:413:packages/flutter/lib/src/material/calendar_date_picker.dart
  @override
  Widget build(BuildContext context) {
    assert(debugCheckHasMaterial(context));
    assert(debugCheckHasMaterialLocalizations(context));
    assert(debugCheckHasDirectionality(context));
    final double textScaleFactor =
        MediaQuery.textScalerOf(
          context,
        ).clamp(maxScaleFactor: _kMaxTextScaleFactor).scale(_fontSizeToScale) /
        _fontSizeToScale;

    // Conform to M3 spec in portrait mode (landscape mode is not specified).
    final Orientation orientation = MediaQuery.orientationOf(context);
    final double maxDayPickerHeight =
        Theme.of(context).useMaterial3 && orientation == Orientation.portrait
        ? _maxDayPickerHeightM3
        : _maxDayPickerHeightM2;

    // Scale the height of the picker area up with larger text. The size of the
    // picker has room for larger text, up until a scale factor of 1.3. After
    // after which, we increase the height to add room for content to continue
    // to scale the text size.
    final double scaledMaxDayPickerHeight = textScaleFactor > 1.3
        ? maxDayPickerHeight + ((_maxDayPickerRowCount + 1) * ((textScaleFactor - 1) * 8))
        : maxDayPickerHeight;
    return Stack(
      children: <Widget>[
        SizedBox(height: _subHeaderHeight + scaledMaxDayPickerHeight, child: _buildPicker()),
        // Put the mode toggle button on top so that it won't be covered up by the _MonthPicker
        MediaQuery.withClampedTextScaling(
          maxScaleFactor: _kModeToggleButtonMaxScaleFactor,
          child: _DatePickerModeToggleButton(
            mode: _mode,
            title: widget.calendarDelegate.formatMonthYear(
              _currentDisplayedMonthDate,
              _localizations,
            ),
            onTitlePressed: () => _handleModeChanged(switch (_mode) {
              DatePickerMode.day => DatePickerMode.year,
              DatePickerMode.year => DatePickerMode.day,
            }),
          ),
        ),
      ],
    );
  }
```

## 响应式布局计算

### 文本缩放因子

组件根据系统文本缩放设置调整布局高度：

```dart 372:376:packages/flutter/lib/src/material/calendar_date_picker.dart
    final double textScaleFactor =
        MediaQuery.textScalerOf(
          context,
        ).clamp(maxScaleFactor: _kMaxTextScaleFactor).scale(_fontSizeToScale) /
        _fontSizeToScale;
```

- 获取当前的文本缩放因子
- 限制最大缩放倍数为 `_kMaxTextScaleFactor`
- 计算相对于基准字体大小的缩放比例

### Material Design 版本适配

根据 Material Design 版本和屏幕方向选择不同的最大高度：

```dart 378:383:packages/flutter/lib/src/material/calendar_date_picker.dart
    // Conform to M3 spec in portrait mode (landscape mode is not specified).
    final Orientation orientation = MediaQuery.orientationOf(context);
    final double maxDayPickerHeight =
        Theme.of(context).useMaterial3 && orientation == Orientation.portrait
        ? _maxDayPickerHeightM3
        : _maxDayPickerHeightM2;
```

- Material Design 3 在纵向模式下使用不同的高度规格
- 横向模式和 Material Design 2 使用相同的规格

### 动态高度调整

根据文本缩放因子动态调整组件高度：

```dart 386:391:packages/flutter/lib/src/material/calendar_date_picker.dart
    // Scale the height of the picker area up with larger text. The size of the
    // picker has room for larger text, up until a scale factor of 1.3. After
    // after which, we increase the height to add room for content to continue
    // to scale the text size.
    final double scaledMaxDayPickerHeight = textScaleFactor > 1.3
        ? maxDayPickerHeight + ((_maxDayPickerRowCount + 1) * ((textScaleFactor - 1) * 8))
        : maxDayPickerHeight;
```

- 文本缩放因子 ≤ 1.3 时使用标准高度
- 超过 1.3 时动态增加高度以容纳更大文本

## 布局结构

组件使用 `Stack` 布局来叠加不同元素：

```dart 392:410:packages/flutter/lib/src/material/calendar_date_picker.dart
    return Stack(
      children: <Widget>[
        SizedBox(height: _subHeaderHeight + scaledMaxDayPickerHeight, child: _buildPicker()),
        // Put the mode toggle button on top so that it won't be covered up by the _MonthPicker
        MediaQuery.withClampedTextScaling(
          maxScaleFactor: _kModeToggleButtonMaxScaleFactor,
          child: _DatePickerModeToggleButton(
            mode: _mode,
            title: widget.calendarDelegate.formatMonthYear(
              _currentDisplayedMonthDate,
              _localizations,
            ),
            onTitlePressed: () => _handleModeChanged(switch (_mode) {
              DatePickerMode.day => DatePickerMode.year,
              DatePickerMode.year => DatePickerMode.day,
            }),
          ),
        ),
      ],
    );
```

### 底层组件

- **`_buildPicker()`**：根据当前模式构建主要的日期选择器内容
- 使用 `SizedBox` 限制高度为头部高度加上日期选择器高度

### 顶层组件

- **`_DatePickerModeToggleButton`**：模式切换按钮，位于顶部
- 使用 `MediaQuery.withClampedTextScaling` 限制文本缩放
- 显示当前月份/年份，并处理点击切换模式

## 选择器构建逻辑

`_buildPicker` 方法根据当前模式返回不同的组件：

```dart 336:365:packages/flutter/lib/src/material/calendar_date_picker.dart
  Widget _buildPicker() {
    switch (_mode) {
      case DatePickerMode.day:
        return _MonthPicker(
          key: _monthPickerKey,
          calendarDelegate: widget.calendarDelegate,
          initialMonth: _currentDisplayedMonthDate,
          currentDate: widget.currentDate,
          firstDate: widget.firstDate,
          lastDate: widget.lastDate,
          selectedDate: _selectedDate,
          onChanged: _handleDayChanged,
          onDisplayedMonthChanged: _handleMonthChanged,
          selectableDayPredicate: widget.selectableDayPredicate,
        );
      case DatePickerMode.year:
        return Padding(
          padding: const EdgeInsets.only(top: _subHeaderHeight),
          child: YearPicker(
            key: _yearPickerKey,
            calendarDelegate: widget.calendarDelegate,
            currentDate: widget.currentDate,
            firstDate: widget.firstDate,
            lastDate: widget.lastDate,
            selectedDate: _currentDisplayedMonthDate,
            onChanged: _handleYearChanged,
          ),
        );
    }
  }
```

### 日视图模式

显示 `_MonthPicker` 组件，包含：

- 月份网格布局
- 日期选择功能
- 月份导航控制

### 年视图模式

显示 `YearPicker` 组件，包含：

- 年份列表
- 年份选择功能
- 顶部填充以避免与模式切换按钮重叠

## 辅助组件

### 模式切换按钮

`_DatePickerModeToggleButton` 是一个内部组件，负责：

- 显示当前显示的月份或年份
- 处理点击事件切换显示模式
- 使用限制的文本缩放以保持UI一致性

### 月选择器

`_MonthPicker` 组件负责：

- 显示月份的日期网格
- 处理日期选择
- 提供月份导航功能

### 年选择器

`YearPicker` 组件负责：

- 显示可选择的年份列表
- 处理年份选择
- 提供年份导航功能

## 布局设计原则

### 无障碍设计

- 所有交互元素都有适当的语义标签
- 状态变化会通过 `SemanticsService` 宣布
- 支持屏幕阅读器导航

### 响应式设计

- 根据屏幕方向调整高度
- 支持文本缩放的无障碍性
- 在不同 Material Design 版本间保持一致性

### 性能优化

- 使用 `GlobalKey` 优化组件重建
- 合理的状态管理避免不必要的重绘
- 延迟加载和按需构建子组件

这种布局设计确保了组件在各种设备和配置下的可靠性和可用性。
