# WidgetsApp 本地化处理详解

## 概述

`_WidgetsAppState` 使用 `LocalizationsResolver` 来管理本地化资源的解析和更新。这个解析器负责根据设备语言环境、应用配置和支持的语言环境列表，确定应用应该使用的语言环境，并加载相应的本地化资源。

## LocalizationsResolver

```dart 1644:1651:packages/flutter/lib/src/widgets/app.dart
  // LOCALIZATION
  late final LocalizationsResolver _localizationsResolver = LocalizationsResolver(
    locale: widget.locale,
    localeListResolutionCallback: widget.localeListResolutionCallback,
    localeResolutionCallback: widget.localeResolutionCallback,
    localizationsDelegates: widget.localizationsDelegates,
    supportedLocales: widget.supportedLocales,
  );
```

**功能**：本地化解析器，负责解析和提供本地化资源。

**初始化**：使用 `late final` 关键字，在首次访问时初始化。

**参数**：

1. **locale**：应用的初始语言环境
2. **localeListResolutionCallback**：语言环境列表解析回调
3. **localeResolutionCallback**：语言环境解析回调
4. **localizationsDelegates**：本地化资源委托列表
5. **supportedLocales**：支持的语言环境列表

**设计目的**：将本地化相关的配置封装到一个解析器中，统一管理本地化资源的解析和提供。

## _updateLocalizations

```dart 1653:1661:packages/flutter/lib/src/widgets/app.dart
  void _updateLocalizations({WidgetsApp? oldWidget}) {
    _localizationsResolver.update(
      locale: widget.locale,
      localeListResolutionCallback: widget.localeListResolutionCallback,
      localeResolutionCallback: widget.localeResolutionCallback,
      supportedLocales: widget.supportedLocales,
      localizationsDelegates: widget.localizationsDelegates,
    );
  }
```

**功能**：更新本地化解析器的配置。

**调用时机**：

- 在 `didUpdateWidget` 中调用，当 `WidgetsApp` 的配置更新时
- 确保本地化配置的变化能够及时反映到解析器中

**更新内容**：

- `locale`：应用的语言环境
- `localeListResolutionCallback`：语言环境列表解析回调
- `localeResolutionCallback`：语言环境解析回调
- `supportedLocales`：支持的语言环境列表
- `localizationsDelegates`：本地化资源委托列表

**设计目的**：当 `WidgetsApp` 的本地化相关配置改变时（例如用户切换语言），及时更新解析器，确保应用使用正确的本地化资源。

## LocalizationsResolver 的工作流程

### 初始化流程

```text
_WidgetsAppState 创建
    ↓
首次访问 _localizationsResolver
    ↓
创建 LocalizationsResolver
    ↓
使用 widget 的本地化配置初始化
    ↓
解析器准备就绪
```

### 更新流程

```text
WidgetsApp 配置改变（如 locale 改变）
    ↓
didUpdateWidget(oldWidget)
    ↓
_updateLocalizations(oldWidget)
    ↓
_localizationsResolver.update(...)
    ↓
解析器使用新配置更新
    ↓
触发 Localizations widget 重建
    ↓
应用使用新的本地化资源
```

### 语言环境解析流程

```text
设备语言环境改变 / 应用启动
    ↓
LocalizationsResolver 解析语言环境
    ↓
1. 尝试 localeListResolutionCallback
    ↓ (返回 null)
2. 尝试 localeResolutionCallback
    ↓ (返回 null)
3. 使用基本解析算法
    ↓
确定最终语言环境
    ↓
加载对应的本地化资源
    ↓
Localizations widget 提供资源
```

## 在 build 方法中的使用

`_localizationsResolver` 在 `build` 方法中被使用，通过 `ListenableBuilder` 监听解析器的变化：

```dart
ListenableBuilder(
  listenable: _localizationsResolver,
  builder: (BuildContext context, _) {
    return Localizations(
      isApplicationLevel: true,
      locale: _localizationsResolver.locale,
      delegates: _localizationsResolver.localizationsDelegates.toList(),
      child: title ?? result,
    );
  },
)
```

**工作机制**：

1. **监听变化**：`ListenableBuilder` 监听 `_localizationsResolver` 的变化
2. **获取语言环境**：从解析器获取当前解析的语言环境
3. **获取委托列表**：从解析器获取本地化资源委托列表
4. **构建 Localizations**：使用解析器提供的信息构建 `Localizations` widget

**优势**：

- **自动重建**：当解析器的状态改变时，`ListenableBuilder` 会自动重建 `Localizations` widget
- **响应式更新**：当设备语言环境改变或配置更新时，应用会自动使用新的本地化资源

## 与 WidgetsApp 属性的关系

### 初始化时的映射

```text
WidgetsApp 属性 → LocalizationsResolver 参数
    ↓
locale → locale
localeListResolutionCallback → localeListResolutionCallback
localeResolutionCallback → localeResolutionCallback
localizationsDelegates → localizationsDelegates
supportedLocales → supportedLocales
```

### 更新时的同步

```text
WidgetsApp 配置改变
    ↓
didUpdateWidget
    ↓
_updateLocalizations
    ↓
同步所有本地化相关属性到解析器
```

## 设计优势

### 1. 封装复杂性

将复杂的本地化解析逻辑封装到 `LocalizationsResolver` 中，`_WidgetsAppState` 只需要调用简单的 `update` 方法。

### 2. 响应式更新

通过 `ListenableBuilder` 实现响应式更新，当语言环境改变时自动重建相关 widget。

### 3. 统一管理

所有本地化相关的配置和状态都通过解析器统一管理，避免分散在多个地方。

### 4. 性能优化

使用 `late final` 延迟初始化，只有在需要时才创建解析器。

## 使用场景

### 场景 1：应用启动

```text
应用启动
    ↓
LocalizationsResolver 初始化
    ↓
根据设备语言环境解析
    ↓
加载对应的本地化资源
    ↓
应用显示本地化内容
```

### 场景 2：用户切换语言

```text
用户切换语言（通过设置）
    ↓
WidgetsApp 的 locale 属性改变
    ↓
didUpdateWidget 被调用
    ↓
_updateLocalizations 更新解析器
    ↓
解析器重新解析语言环境
    ↓
Localizations widget 重建
    ↓
应用显示新的语言
```

### 场景 3：设备语言环境改变

```text
用户在系统设置中改变语言
    ↓
平台通知 Flutter
    ↓
LocalizationsResolver 检测到变化
    ↓
重新解析语言环境
    ↓
Localizations widget 重建
    ↓
应用自动切换到新语言
```

## 与 Localizations widget 的集成

`_localizationsResolver` 提供的信息最终用于构建 `Localizations` widget：

```dart
Localizations(
  isApplicationLevel: true,
  locale: _localizationsResolver.locale,  // 解析后的语言环境
  delegates: _localizationsResolver.localizationsDelegates.toList(),  // 委托列表
  child: ...,
)
```

**关键点**：

- **isApplicationLevel: true**：标记为应用级别的 `Localizations`，影响整个应用
- **locale**：使用解析器解析后的语言环境，而不是直接使用 `widget.locale`
- **delegates**：使用解析器提供的委托列表，确保委托顺序正确

## 总结

第十部分详细介绍了 `_WidgetsAppState` 的本地化处理机制：

1. **LocalizationsResolver**：本地化解析器，封装了语言环境解析和资源管理的复杂性

2. **_updateLocalizations**：更新本地化解析器的配置，响应 `WidgetsApp` 配置变化

3. **响应式更新**：通过 `ListenableBuilder` 实现自动重建，当语言环境改变时自动更新 UI

4. **统一管理**：所有本地化相关的配置和状态都通过解析器统一管理

5. **与 Localizations widget 集成**：解析器提供的信息用于构建 `Localizations` widget，为整个应用提供本地化资源

这个机制确保了应用能够正确响应语言环境变化，自动加载和使用相应的本地化资源，为多语言应用提供了坚实的基础。
