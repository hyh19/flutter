# CalendarDelegate 类详解

## 概述

`CalendarDelegate` 是一个抽象类，用于控制日期选择器中使用的日历系统。它定义了如何在选择器中解释、格式化和导航日期。通过提供自定义实现，可以支持不同的日历系统（如公历、尼泊尔历、伊斯兰历、佛教历等）。

这个抽象类的设计采用了策略模式，使得日期选择器组件可以轻松切换不同的日历系统，而不需要修改核心的UI逻辑。

## 泛型参数

```dart 32:32:packages/flutter/lib/src/material/date.dart
abstract class CalendarDelegate<T extends DateTime> {
```

类使用了泛型参数 `T extends DateTime`，这意味着：

- `T` 必须是 `DateTime` 或其子类
- 这样设计是为了支持不同类型的日期对象，比如标准 `DateTime` 或者自定义的日期类
- 默认实现 `GregorianCalendarDelegate` 使用了 `DateTime` 作为类型参数

## 构造函数

```dart 33:34:packages/flutter/lib/src/material/date.dart
  /// Creates a calendar delegate.
  const CalendarDelegate();
```

构造函数很简单，只是为了支持 `const` 构造。

## 核心日期操作方法

### 当前时间获取

```dart 36:37:packages/flutter/lib/src/material/date.dart
  /// Returns a [DateTime] representing the current date and time.
  T now();
```

`now()` 方法返回当前日期和时间。这是抽象方法，子类需要实现。

### 日期标准化

```dart 39:45:packages/flutter/lib/src/material/date.dart
  /// {@macro flutter.material.date.dateOnly}
  T dateOnly(T date);

  /// {@macro flutter.material.date.datesOnly}
  DateTimeRange<T> datesOnly(DateTimeRange<T> range) {
    return DateTimeRange<T>(start: dateOnly(range.start), end: dateOnly(range.end));
  }
```

- `dateOnly(T date)`：将日期的时间部分设置为午夜，只保留日期信息
- `datesOnly(DateTimeRange<T> range)`：对日期范围应用相同的操作，有默认实现

### 日期比较

```dart 47:55:packages/flutter/lib/src/material/date.dart
  /// {@macro flutter.material.date.isSameDay}
  bool isSameDay(T? dateA, T? dateB) {
    return dateA?.year == dateB?.year && dateA?.month == dateB?.month && dateA?.day == dateB?.day;
  }

  /// {@macro flutter.material.date.isSameMonth}
  bool isSameMonth(T? dateA, T? dateB) {
    return dateA?.year == dateB?.year && dateA?.month == dateB?.month;
  }
```

这两个方法都有默认实现：

- `isSameDay`：比较两个日期是否是同一天（年、月、日都相同）
- `isSameMonth`：比较两个日期是否是同一月（年、月相同）

使用可空类型 `T?`，当参数为 `null` 时会返回 `false`。

## 日期计算方法

### 月份差计算

```dart 57:58:packages/flutter/lib/src/material/date.dart
  /// {@macro flutter.material.date.monthDelta}
  int monthDelta(T startDate, T endDate);
```

计算两个日期之间相差的月份数。这是抽象方法，需要子类实现。

### 月份加减

```dart 60:61:packages/flutter/lib/src/material/date.dart
  /// {@macro flutter.material.date.addMonthsToMonthDate}
  T addMonthsToMonthDate(T monthDate, int monthsToAdd);
```

在给定的月份日期上添加或减去指定的月份数。结果的日期会被设置为该月的第1天。

### 日期加减

```dart 63:64:packages/flutter/lib/src/material/date.dart
  /// {@macro flutter.material.date.addDaysToDate}
  T addDaysToDate(T date, int days);
```

在给定的日期上添加或减去指定的天数。

### 月份首日偏移

```dart 66:67:packages/flutter/lib/src/material/date.dart
  /// {@macro flutter.material.date.firstDayOffset}
  int firstDayOffset(int year, int month, MaterialLocalizations localizations);
```

计算指定年月的第一天在星期中的偏移量。这个值用于日历组件的布局计算。

## 日历结构方法

### 月份天数获取

```dart 69:70:packages/flutter/lib/src/material/date.dart
  /// Returns the number of days in a month, according to the calendar system.
  int getDaysInMonth(int year, int month);
```

返回指定年月的天数。根据不同的日历系统，这个计算可能不同。

### 日期构造

```dart 72:76:packages/flutter/lib/src/material/date.dart
  /// Returns a [DateTime] with the given [year] and [month].
  T getMonth(int year, int month);

  /// Returns a [DateTime] with the given [year], [month], and [day].
  T getDay(int year, int month, int day);
```

这两个方法用于构造特定年月或年月日的日期对象。

## 日期格式化方法

`CalendarDelegate` 提供了多种日期格式化方法，每种都有特定的用途和格式：

### 月份年份格式

```dart 78:82:packages/flutter/lib/src/material/date.dart
  /// Formats the month and the year of the given [date].
  ///
  /// The returned string does not contain the day of the month. This appears
  /// in the date picker invoked using [showDatePicker].
  String formatMonthYear(T date, MaterialLocalizations localizations);
```

格式化月份和年份，不包含日期。在日期选择器的标题中显示。

### 年份格式

```dart 84:87:packages/flutter/lib/src/material/date.dart
  /// Full unabbreviated year format, e.g. 2017 rather than 17.
  String formatYear(int year, MaterialLocalizations localizations) {
    return localizations.formatYear(DateTime(year));
  }
```

完整的年份格式，有默认实现。

### 中等宽度日期格式

```dart 89:98:packages/flutter/lib/src/material/date.dart
  /// Formats the date using a medium-width format.
  ///
  /// Abbreviates month and days of week. This appears in the header of the date
  /// picker invoked using [showDatePicker].
  ///
  /// Examples:
  ///
  /// - US English: Wed, Sep 27
  /// - Russian: ср, сент. 27
  String formatMediumDate(T date, MaterialLocalizations localizations);
```

中等宽度的日期格式，会缩写月份和星期几。在日期选择器的头部显示。

### 短格式月份日期

```dart 100:106:packages/flutter/lib/src/material/date.dart
  /// Formats the month and day of the given [date].
  ///
  /// Examples:
  ///
  /// - US English: Feb 21
  /// - Russian: 21 февр.
  String formatShortMonthDay(T date, MaterialLocalizations localizations);
```

只显示月份和日期的短格式。

### 短格式日期

```dart 108:116:packages/flutter/lib/src/material/date.dart
  /// Formats the date using a short-width format.
  ///
  /// Includes the abbreviation of the month, the day and year.
  ///
  /// Examples:
  ///
  /// - US English: Feb 21, 2019
  /// - Russian: 21 февр. 2019 г.
  String formatShortDate(T date, MaterialLocalizations localizations);
```

短格式的完整日期，包含月份缩写、日和年。

### 长格式日期

```dart 118:127:packages/flutter/lib/src/material/date.dart
  /// Formats day of week, month, day of month and year in a long-width format.
  ///
  /// Does not abbreviate names. Appears in spoken announcements of the date
  /// picker invoked using [showDatePicker], when accessibility mode is on.
  ///
  /// Examples:
  ///
  /// - US English: Wednesday, September 27, 2017
  /// - Russian: Среда, Сентябрь 27, 2017
  String formatFullDate(T date, MaterialLocalizations localizations);
```

长格式的完整日期，不缩写名称。主要用于辅助功能（如屏幕阅读器）。

### 紧凑格式日期

```dart 129:140:packages/flutter/lib/src/material/date.dart
  /// Formats the date in a compact format.
  ///
  /// Usually just the numeric values for the for day, month and year are used.
  ///
  /// Examples:
  ///
  /// - US English: 02/21/2019
  /// - Russian: 21.02.2019
  ///
  /// See also:
  ///   * [parseCompactDate], which will convert a compact date string to a [DateTime].
  String formatCompactDate(T date, MaterialLocalizations localizations);
```

紧凑的数值格式日期。主要用于数据输入。

## 字符串解析方法

### 紧凑日期解析

```dart 142:150:packages/flutter/lib/src/material/date.dart
  /// Converts the given compact date formatted string into a [DateTime].
  ///
  /// The format of the string must be a valid compact date format for the
  /// given locale. If the text doesn't represent a valid date, `null` will be
  /// returned.
  ///
  /// See also:
  ///   * [formatCompactDate], which will convert a [DateTime] into a string in the compact format.
  T? parseCompactDate(String? inputString, MaterialLocalizations localizations);
```

将紧凑格式的日期字符串解析为日期对象。如果字符串无效，返回 `null`。

## 用户界面支持

### 帮助文本

```dart 152:154:packages/flutter/lib/src/material/date.dart
  /// The help text used on an empty [InputDatePickerFormField] to indicate
  /// to the user the date format being asked for.
  String dateHelpText(MaterialLocalizations localizations);
```

为空的日期输入字段提供帮助文本，提示用户期望的日期格式。

## 使用场景

`CalendarDelegate` 的主要用途包括：

1. **多日历系统支持**：通过实现不同的委托，可以支持公历、农历、伊斯兰历等多种日历系统
2. **日期选择器组件**：`CalendarDatePicker` 使用这个委托来管理日期选择逻辑
3. **本地化支持**：通过 `MaterialLocalizations` 参数，支持不同地区的日期格式和习惯
4. **可扩展性**：新的日历系统可以通过继承和实现这个抽象类来添加

## 默认实现

Flutter 提供了 `GregorianCalendarDelegate` 作为默认的公历实现，它实现了所有抽象方法并使用了标准的 `DateTime` 操作。

## 设计模式

这个抽象类的设计体现了以下设计模式：

- **策略模式**：`CalendarDelegate` 定义了日期操作的策略，不同的子类提供不同的实现
- **模板方法模式**：一些方法有默认实现，子类可以选择性地重写
- **桥接模式**：将抽象的日期操作与具体的UI组件分离

这样的设计使得日期选择器组件具有很好的可扩展性和可维护性。
