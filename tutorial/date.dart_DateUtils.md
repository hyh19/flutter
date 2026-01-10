# DateUtils 类详解

## 类概述

`DateUtils` 是一个抽象最终类，提供了一系列用于处理日期的静态工具方法。这些方法主要用于 Flutter Material 组件中的日期选择器和日历组件，为日期操作提供了便捷的实用功能。

```dart 246:387:packages/flutter/lib/src/material/date.dart
/// Utility functions for working with dates.
abstract final class DateUtils {
  // ... 所有静态方法
}
```

## 核心功能方法

### 1. dateOnly - 日期标准化

```dart 248:254:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.dateOnly}
/// Returns a [DateTime] with the date of the original, but time set to
/// midnight.
/// {@endtemplate}
static DateTime dateOnly(DateTime date) {
  return DateTime(date.year, date.month, date.day);
}
```

**功能说明**：

- 将传入的 `DateTime` 对象的时间部分重置为午夜（00:00:00.000）
- 只保留年、月、日信息，清除时分秒和毫秒
- 常用于比较日期时忽略时间差异

**使用场景**：

- 日期选择器中只关心日期部分，不关心具体时间
- 比较两个日期是否为同一天
- 数据存储时只保存日期信息

### 2. datesOnly - 日期范围标准化

```dart 256:265:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.datesOnly}
/// Returns a [DateTimeRange] with the dates of the original, but with times
/// set to midnight.
///
/// See also:
///  * [dateOnly], which does the same thing for a single date.
/// {@endtemplate}
static DateTimeRange datesOnly(DateTimeRange range) {
  return DateTimeRange(start: dateOnly(range.start), end: dateOnly(range.end));
}
```

**功能说明**：

- 对日期范围对象执行同样的日期标准化操作
- 分别处理起始日期和结束日期，将它们都重置为午夜
- 返回新的 `DateTimeRange` 对象

**关联方法**：

- 调用了 `dateOnly` 方法处理单个日期

### 3. isSameDay - 日期相同性检查

```dart 267:273:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.isSameDay}
/// Returns true if the two [DateTime] objects have the same day, month, and
/// year, or are both null.
/// {@endtemplate}
static bool isSameDay(DateTime? dateA, DateTime? dateB) {
  return dateA?.year == dateB?.year && dateA?.month == dateB?.month && dateA?.day == dateB?.day;
}
```

**功能说明**：

- 检查两个日期是否为同一天（年月日完全相同）
- 支持 null 值比较：两个 null 值视为相同
- 使用空安全操作符避免空指针异常

**使用场景**：

- 日历组件中标记当前选中的日期
- 检查用户选择的日期是否与特定日期匹配
- 日期过滤和比较逻辑

### 4. isSameMonth - 月份相同性检查

```dart 275:281:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.isSameMonth}
/// Returns true if the two [DateTime] objects have the same month and
/// year, or are both null.
/// {@endtemplate}
static bool isSameMonth(DateTime? dateA, DateTime? dateB) {
  return dateA?.year == dateB?.year && dateA?.month == dateB?.month;
}
```

**功能说明**：

- 检查两个日期是否在同一个月（年和月相同）
- 同样支持 null 值比较
- 比 `isSameDay` 更宽松的比较条件

**使用场景**：

- 月历视图中高亮显示当前月份
- 按月份分组和过滤数据
- 检查日期是否在指定月份范围内

### 5. monthDelta - 计算月份差

```dart 283:298:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.monthDelta}
/// Determines the number of months between two [DateTime] objects.
///
/// For example:
///
/// ```dart
/// DateTime date1 = DateTime(2019, 6, 15);
/// DateTime date2 = DateTime(2020, 1, 15);
/// int delta = DateUtils.monthDelta(date1, date2);
/// ```
///
/// The value for `delta` would be `7`.
/// {@endtemplate}
static int monthDelta(DateTime startDate, DateTime endDate) {
  return (endDate.year - startDate.year) * 12 + endDate.month - startDate.month;
}
```

**功能说明**：

- 计算两个日期之间相差的月份数
- 通过年差乘以12再加上月差来计算总月份差
- 不考虑日期部分，只基于年和月计算

**计算逻辑**：

- `(2020 - 2019) * 12 + (1 - 6) = 1 * 12 + (-5) = 12 - 5 = 7`

**使用场景**：

- 计算时间跨度（如项目周期、租期等）
- 日历导航（向前/向后翻页几个月）
- 相对日期计算

### 6. addMonthsToMonthDate - 月份加法

```dart 300:316:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.addMonthsToMonthDate}
/// Returns a [DateTime] that is [monthDate] with the added number
/// of months and the day set to 1 and time set to midnight.
///
/// For example:
///
/// ```dart
/// DateTime date = DateTime(2019, 1, 15);
/// DateTime futureDate = DateUtils.addMonthsToMonthDate(date, 3);
/// ```
///
/// `date` would be January 15, 2019.
/// `futureDate` would be April 1, 2019 since it adds 3 months.
/// {@endtemplate}
static DateTime addMonthsToMonthDate(DateTime monthDate, int monthsToAdd) {
  return DateTime(monthDate.year, monthDate.month + monthsToAdd);
}
```

**功能说明**：

- 在指定日期基础上添加指定月份数
- 结果日期的日期部分固定为1，时间为午夜
- 主要用于月份级别的日期导航

**注意事项**：

- 添加月份后日期固定为1号，不保留原始日期
- 这与日常的月份加法逻辑不同，更像是"下个月的1号"

### 7. addDaysToDate - 日期加法

```dart 318:324:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.addDaysToDate}
/// Returns a [DateTime] with the added number of days and time set to
/// midnight.
/// {@endtemplate}
static DateTime addDaysToDate(DateTime date, int days) {
  return DateTime(date.year, date.month, date.day + days);
}
```

**功能说明**：

- 在指定日期基础上添加指定天数
- 结果时间重置为午夜
- 最基本的日期算术操作

**使用场景**：

- 计算到期日期
- 日期范围的起始和结束日期
- 简单的日期导航（前一天/后一天）

### 8. firstDayOffset - 日历首日偏移计算

```dart 326:370:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.firstDayOffset}
/// Computes the offset from the first day of the week that the first day of
/// the [month] falls on.
/// ...
/// {@endtemplate}
static int firstDayOffset(int year, int month, MaterialLocalizations localizations) {
  // 0-based day of week for the month and year, with 0 representing Monday.
  final int weekdayFromMonday = DateTime(year, month).weekday - 1;

  // 0-based start of week depending on the locale, with 0 representing Sunday.
  int firstDayOfWeekIndex = localizations.firstDayOfWeekIndex;

  // firstDayOfWeekIndex recomputed to be Monday-based, in order to compare with
  // weekdayFromMonday.
  firstDayOfWeekIndex = (firstDayOfWeekIndex - 1) % 7;

  // Number of days between the first day of week appearing on the calendar,
  // and the day corresponding to the first of the month.
  return (weekdayFromMonday - firstDayOfWeekIndex) % 7;
}
```

**功能说明**：

- 计算指定月份第一天在日历网格中的偏移位置
- 用于确定日历布局中第一天前的空白格子数
- 考虑不同地区的周起始日（周日或周一）

**计算步骤**：

1. 获取指定年月的第一天是星期几（转换为0-6的周一起始索引）
2. 获取本地化的周起始日索引
3. 计算偏移量：`(weekdayFromMonday - firstDayOfWeekIndex) % 7`

**使用场景**：

- 日历组件的网格布局计算
- 确定月份第一天在日历中的位置
- 支持国际化（不同地区周起始日不同）

### 9. getDaysInMonth - 获取月份天数

```dart 372:386:packages/flutter/lib/src/material/date.dart
/// {@template flutter.material.date.getDaysInMonth}
/// Returns the number of days in a month, according to the proleptic
/// Gregorian calendar.
///
/// This applies the leap year logic introduced by the Gregorian reforms of
/// 1582. It will not give valid results for dates prior to that time.
/// {@endtemplate}
static int getDaysInMonth(int year, int month) {
  if (month == DateTime.february) {
    final bool isLeapYear = (year % 4 == 0) && (year % 100 != 0) || (year % 400 == 0);
    return isLeapYear ? 29 : 28;
  }
  const List<int> daysInMonth = <int>[31, -1, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
  return daysInMonth[month - 1];
}
```

**功能说明**：

- 根据公历（格里高利历）计算指定年月的天数
- 正确处理闰年逻辑：四年一闰，百年不闰，四百年再闰
- 使用预定义数组存储各月天数

**闰年判断逻辑**：

```dart
(year % 4 == 0) && (year % 100 != 0) || (year % 400 == 0)
```

**特殊处理**：

- 二月需要特殊判断是否为闰年
- 其他月份使用固定数组：`[31, -1, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]`
- 数组中-1表示二月，由代码动态计算

## 设计特点

### 1. 静态方法设计

- 所有方法都是静态方法，无需实例化
- 提供纯函数式的日期操作接口
- 线程安全，无状态依赖

### 2. 空安全支持

- 大量使用空安全操作符 `?.`
- 支持 null 值比较和处理
- 符合 Dart 的空安全规范

### 3. 国际化考虑

- `firstDayOffset` 方法考虑不同地区的周起始日
- 与 `MaterialLocalizations` 集成
- 支持全球化的日期显示习惯

### 4. 性能优化

- 纯数学计算，无外部依赖
- 预定义常量数组避免重复计算
- 简单的算法逻辑确保高性能

## 使用示例

### 基本日期操作

```dart
// 标准化日期时间
DateTime now = DateTime.now();
DateTime today = DateUtils.dateOnly(now); // 只保留日期部分

// 检查日期相同性
bool sameDay = DateUtils.isSameDay(date1, date2);
bool sameMonth = DateUtils.isSameMonth(date1, date2);

// 日期计算
DateTime nextMonth = DateUtils.addMonthsToMonthDate(today, 1);
DateTime tomorrow = DateUtils.addDaysToDate(today, 1);

// 计算月份差
int monthsDiff = DateUtils.monthDelta(startDate, endDate);
```

### 日历组件集成

```dart
// 获取月份天数
int daysInMonth = DateUtils.getDaysInMonth(2024, 2); // 29 (闰年)

// 计算日历偏移
int offset = DateUtils.firstDayOffset(year, month, localizations);
```

## 总结

`DateUtils` 类是 Flutter Material 日期组件的核心工具类，提供了一套完整的日期操作功能。从基本的日期标准化、比较，到复杂的日历布局计算，都在这个类中得到了实现。

这些方法的设计充分考虑了性能、国际化、空安全等现代编程要求，为上层 UI 组件提供了坚实的基础支持。无论是简单的日期显示，还是复杂的日历交互，都能在这里找到相应的工具方法。
