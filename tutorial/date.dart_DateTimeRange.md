# DateTimeRange 类详解

## 概述

`DateTimeRange` 是一个不可变的类，用于封装表示日期范围的起始和结束 `DateTime` 对象。该类提供了日期范围的基本操作和属性，确保了数据的一致性和完整性。

## 类定义

```dart 449:451:packages/flutter/lib/src/material/date.dart
@immutable
@optionalTypeArgs
class DateTimeRange<T extends DateTime> {
```

### 泛型约束

类使用了泛型参数 `T`，并约束 `T` 必须是 `DateTime` 类型或其子类型：

- `@optionalTypeArgs` 注解表示泛型参数是可选的
- `@immutable` 注解确保类的实例一旦创建就不可修改

## 构造函数

```dart 452:453:packages/flutter/lib/src/material/date.dart
  /// Creates a date range for the given start and end [DateTime].
  DateTimeRange({required this.start, required this.end}) : assert(!start.isAfter(end));
```

### 参数说明

- `start`: 必需参数，表示日期范围的开始时间
- `end`: 必需参数，表示日期范围的结束时间

### 断言检查

构造函数包含断言 `assert(!start.isAfter(end))`，确保：

- 开始时间不能晚于结束时间
- 如果违反此规则，在调试模式下会抛出异常

## 属性

### start 属性

```dart 455:456:packages/flutter/lib/src/material/date.dart
  /// The start of the range of dates.
  final T start;
```

- 表示日期范围的起始日期时间
- 被声明为 `final`，确保不可修改

### end 属性

```dart 458:459:packages/flutter/lib/src/material/date.dart
  /// The end of the range of dates.
  final T end;
```

- 表示日期范围的结束日期时间
- 被声明为 `final`，确保不可修改

## 计算属性

### duration 属性

```dart 461:464:packages/flutter/lib/src/material/date.dart
  /// Returns a [Duration] of the time between [start] and [end].
  ///
  /// See [DateTime.difference] for more details.
  Duration get duration => end.difference(start);
```

- 返回开始时间和结束时间之间的持续时间
- 使用 `DateTime.difference()` 方法计算
- 这是一个只读的 getter 属性

## 对象比较

### 相等性判断

```dart 466:472:packages/flutter/lib/src/material/date.dart
  @override
  bool operator ==(Object other) {
    if (other.runtimeType != runtimeType) {
      return false;
    }
    return other is DateTimeRange && other.start == start && other.end == end;
  }
```

- 重写了 `==` 操作符
- 首先检查运行时类型是否相同
- 然后检查 `start` 和 `end` 属性是否都相等

### 哈希码

```dart 474:475:packages/flutter/lib/src/material/date.dart
  @override
  int get hashCode => Object.hash(start, end);
```

- 重写了 `hashCode` getter
- 使用 `Object.hash()` 方法基于 `start` 和 `end` 属性生成哈希码
- 确保相等的对象具有相同的哈希码

## 字符串表示

```dart 477:478:packages/flutter/lib/src/material/date.dart
  @override
  String toString() => '$start - $end';
```

- 重写了 `toString()` 方法
- 返回格式为 "开始时间 - 结束时间" 的字符串
- 便于调试和日志记录

## 使用场景

`DateTimeRange` 类主要用于：

1. **日期范围选择器**: 在 `showDateRangePicker` 中使用，表示用户选择的日期范围
2. **时间段计算**: 通过 `duration` 属性获取时间段长度
3. **日期验证**: 确保开始时间不晚于结束时间

## 设计特点

- **不可变性**: 使用 `@immutable` 注解，确保线程安全
- **类型安全**: 通过泛型约束确保类型正确性
- **断言检查**: 在构造函数中验证数据一致性
- **完整的对象协议**: 重写了 `==`、`hashCode` 和 `toString`

## 示例用法

```dart
// 创建一个日期范围
final range = DateTimeRange(
  start: DateTime(2024, 1, 1),
  end: DateTime(2024, 1, 31),
);

// 获取持续时间
final duration = range.duration; // 30天的持续时间

// 比较日期范围
final anotherRange = DateTimeRange(
  start: DateTime(2024, 1, 1),
  end: DateTime(2024, 1, 31),
);

print(range == anotherRange); // true
print(range.toString()); // 2024-01-01 00:00:00.000 - 2024-01-31 00:00:00.000
```
