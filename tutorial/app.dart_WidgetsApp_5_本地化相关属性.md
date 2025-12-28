# WidgetsApp 本地化相关属性详解

## 概述

`WidgetsApp` 提供了完整的本地化和国际化支持，包括语言环境（Locale）配置、本地化资源委托、语言环境解析回调等。这些属性共同构成了 Flutter 应用的国际化基础设施。

## locale

```dart 876:893:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.locale}
  /// The initial locale for this app's [Localizations] widget is based
  /// on this value.
  ///
  /// If the 'locale' is null then the system's locale value is used.
  ///
  /// The value of [Localizations.locale] will equal this locale if
  /// it matches one of the [supportedLocales]. Otherwise it will be
  /// the first element of [supportedLocales].
  /// {@endtemplate}
  ///
  /// See also:
  ///
  ///  * [localeResolutionCallback], which can override the default
  ///    [supportedLocales] matching algorithm.
  ///  * [localizationsDelegates], which collectively define all of the localized
  ///    resources used by this app.
  final Locale? locale;
```

**功能**：应用的 `Localizations` widget 的初始语言环境基于此值。

**行为**：

1. **系统默认**：如果 `locale` 为 `null`，则使用系统的语言环境值
2. **匹配检查**：如果此 `locale` 匹配 `supportedLocales` 中的一个，`Localizations.locale` 将等于此 `locale`
3. **回退机制**：如果不匹配，将使用 `supportedLocales` 的第一个元素

**使用场景**：用于强制应用使用特定的语言环境，而不是系统默认值。

## localizationsDelegates

```dart 895:901:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.localizationsDelegates}
  /// The delegates for this app's [Localizations] widget.
  ///
  /// The delegates collectively define all of the localized resources
  /// for this application's [Localizations] widget.
  /// {@endtemplate}
  final Iterable<LocalizationsDelegate<dynamic>>? localizationsDelegates;
```

**功能**：应用的 `Localizations` widget 的委托列表。

**作用**：委托共同定义了应用 `Localizations` widget 的所有本地化资源。

**典型委托**：

- `GlobalMaterialLocalizations.delegate`：Material Design 的本地化
- `GlobalCupertinoLocalizations.delegate`：Cupertino 的本地化
- `GlobalWidgetsLocalizations.delegate`：Widgets 的本地化
- 自定义委托：应用特定的本地化资源

**使用示例**：

```dart
localizationsDelegates: [
  GlobalMaterialLocalizations.delegate,
  GlobalWidgetsLocalizations.delegate,
  MyAppLocalizations.delegate,
],
```

## localeListResolutionCallback

```dart 903:940:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.localeListResolutionCallback}
  /// This callback is responsible for choosing the app's locale
  /// when the app is started, and when the user changes the
  /// device's locale.
  ///
  /// When a [localeListResolutionCallback] is provided, Flutter will first
  /// attempt to resolve the locale with the provided
  /// [localeListResolutionCallback]. If the callback or result is null, it will
  /// fallback to trying the [localeResolutionCallback]. If both
  /// [localeResolutionCallback] and [localeListResolutionCallback] are left
  /// null or fail to resolve (return null), basic fallback algorithm will
  /// be used.
  ///
  /// The priority of each available fallback is:
  ///
  ///  1. [localeListResolutionCallback] is attempted.
  ///  2. [localeResolutionCallback] is attempted.
  ///  3. Flutter's basic resolution algorithm, as described in
  ///     [supportedLocales], is attempted last.
  ///
  /// Properly localized projects should provide a more advanced algorithm than
  /// the basic method from [supportedLocales], as it does not implement a
  /// complete algorithm (such as the one defined in
  /// [Unicode TR35](https://unicode.org/reports/tr35/#LanguageMatching))
  /// and is optimized for speed at the detriment of some uncommon edge-cases.
  /// {@endtemplate}
  ///
  /// This callback considers the entire list of preferred locales.
  ///
  /// This algorithm should be able to handle a null or empty list of preferred locales,
  /// which indicates Flutter has not yet received locale information from the platform.
  ///
  /// See also:
  ///
  ///  * [MaterialApp.localeListResolutionCallback], which sets the callback of the
  ///    [WidgetsApp] it creates.
  ///  * [basicLocaleListResolution], the default locale resolution algorithm.
  final LocaleListResolutionCallback? localeListResolutionCallback;
```

**功能**：负责在应用启动时和用户更改设备语言环境时选择应用语言环境的回调。

**解析优先级**：

1. **localeListResolutionCallback**：首先尝试使用此回调
2. **localeResolutionCallback**：如果第一个回调返回 `null`，尝试此回调
3. **基本算法**：如果两个回调都为 `null` 或都返回 `null`，使用 Flutter 的基本解析算法

**优势**：

- **完整列表**：考虑整个首选语言环境列表，而不仅仅是第一个
- **更精确匹配**：可以实现更高级的匹配算法（如 Unicode TR35）
- **边缘情况处理**：可以处理基本算法无法处理的边缘情况

**要求**：算法应该能够处理 `null` 或空的首选语言环境列表，这表示 Flutter 尚未从平台接收语言环境信息。

## localeResolutionCallback

```dart 942:956:packages/flutter/lib/src/widgets/app.dart
  /// {@macro flutter.widgets.widgetsApp.localeListResolutionCallback}
  ///
  /// This callback considers only the default locale, which is the first locale
  /// in the preferred locales list. It is preferred to set [localeListResolutionCallback]
  /// over [localeResolutionCallback] as it provides the full preferred locales list.
  ///
  /// This algorithm should be able to handle a null locale, which indicates
  /// Flutter has not yet received locale information from the platform.
  ///
  /// See also:
  ///
  ///  * [MaterialApp.localeResolutionCallback], which sets the callback of the
  ///    [WidgetsApp] it creates.
  ///  * [basicLocaleListResolution], the default locale resolution algorithm.
  final LocaleResolutionCallback? localeResolutionCallback;
```

**功能**：与 `localeListResolutionCallback` 类似，但只考虑默认语言环境（首选语言环境列表中的第一个）。

**与 localeListResolutionCallback 的区别**：

- **localeListResolutionCallback**：考虑整个首选语言环境列表
- **localeResolutionCallback**：只考虑第一个语言环境

**推荐**：优先使用 `localeListResolutionCallback`，因为它提供了完整的首选语言环境列表。

**要求**：算法应该能够处理 `null` 语言环境，这表示 Flutter 尚未从平台接收语言环境信息。

## supportedLocales

```dart 958:1025:packages/flutter/lib/src/widgets/app.dart
  /// {@template flutter.widgets.widgetsApp.supportedLocales}
  /// The list of locales that this app has been localized for.
  ///
  /// By default only the American English locale is supported. Apps should
  /// configure this list to match the locales they support.
  ///
  /// This list must not null. Its default value is just
  /// `[const Locale('en', 'US')]`.
  ///
  /// The order of the list matters. The default locale resolution algorithm,
  /// [basicLocaleListResolution], attempts to match by the following priority:
  ///
  ///  1. [Locale.languageCode], [Locale.scriptCode], and [Locale.countryCode]
  ///  2. [Locale.languageCode] and [Locale.scriptCode] only
  ///  3. [Locale.languageCode] and [Locale.countryCode] only
  ///  4. [Locale.languageCode] only
  ///  5. [Locale.countryCode] only when all preferred locales fail to match
  ///  6. Returns the first element of [supportedLocales] as a fallback
  ///
  /// When more than one supported locale matches one of these criteria, only
  /// the first matching locale is returned.
  ///
  /// The default locale resolution algorithm can be overridden by providing a
  /// value for [localeListResolutionCallback]. The provided
  /// [basicLocaleListResolution] is optimized for speed and does not implement
  /// a full algorithm (such as the one defined in
  /// [Unicode TR35](https://unicode.org/reports/tr35/#LanguageMatching)) that
  /// takes distances between languages into account.
  ///
  /// When supporting languages with more than one script, it is recommended
  /// to specify the [Locale.scriptCode] explicitly. Locales may also be defined without
  /// [Locale.countryCode] to specify a generic fallback for a particular script.
  ///
  /// A fully supported language with multiple scripts should define a generic language-only
  /// locale (e.g. 'zh'), language+script only locales (e.g. 'zh_Hans' and 'zh_Hant'),
  /// and any language+script+country locales (e.g. 'zh_Hans_CN'). Fully defining all of
  /// these locales as supported is not strictly required but allows for proper locale resolution in
  /// the most number of cases. These locales can be specified with the [Locale.fromSubtags]
  /// constructor:
  ///
  /// ```dart
  /// // Full Chinese support for CN, TW, and HK
  /// supportedLocales: <Locale>[
  ///   const Locale.fromSubtags(languageCode: 'zh'), // generic Chinese 'zh'
  ///   const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans'), // generic simplified Chinese 'zh_Hans'
  ///   const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant'), // generic traditional Chinese 'zh_Hant'
  ///   const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans', countryCode: 'CN'), // 'zh_Hans_CN'
  ///   const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant', countryCode: 'TW'), // 'zh_Hant_TW'
  ///   const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant', countryCode: 'HK'), // 'zh_Hant_HK'
  /// ],
  /// ```
  ///
  /// Omitting some these fallbacks may result in improperly resolved
  /// edge-cases, for example, a simplified Chinese user in Taiwan ('zh_Hans_TW')
  /// may resolve to traditional Chinese if 'zh_Hans' and 'zh_Hans_CN' are
  /// omitted.
  /// {@endtemplate}
  ///
  /// See also:
  ///
  ///  * [MaterialApp.supportedLocales], which sets the `supportedLocales`
  ///    of the [WidgetsApp] it creates.
  ///  * [localeResolutionCallback], an app callback that resolves the app's locale
  ///    when the device's locale changes.
  ///  * [localizationsDelegates], which collectively define all of the localized
  ///    resources used by this app.
  ///  * [basicLocaleListResolution], the default locale resolution algorithm.
  final Iterable<Locale> supportedLocales;
```

**功能**：应用已本地化的语言环境列表。

**默认值**：默认为 `[const Locale('en', 'US')]`（美式英语）。

**重要性**：列表的顺序很重要，因为默认解析算法会按顺序匹配。

### 默认解析算法的匹配优先级

1. **完全匹配**：`languageCode`、`scriptCode` 和 `countryCode` 都匹配
2. **语言+脚本**：`languageCode` 和 `scriptCode` 匹配
3. **语言+国家**：`languageCode` 和 `countryCode` 匹配
4. **仅语言**：只匹配 `languageCode`
5. **仅国家**：当所有首选语言环境都匹配失败时，尝试匹配 `countryCode`
6. **回退**：返回 `supportedLocales` 的第一个元素作为回退

### 多脚本语言支持

对于支持多种脚本的语言（如中文），建议定义完整的语言环境层次结构：

1. **通用语言**：`Locale('zh')` - 通用中文
2. **脚本级别**：`Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans')` - 简体中文
3. **国家级别**：`Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans', countryCode: 'CN')` - 中国大陆简体中文

**完整示例**（中文支持）：

```dart
supportedLocales: <Locale>[
  const Locale.fromSubtags(languageCode: 'zh'), // 通用中文
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans'), // 通用简体中文
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant'), // 通用繁体中文
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans', countryCode: 'CN'), // 中国大陆
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant', countryCode: 'TW'), // 台湾
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant', countryCode: 'HK'), // 香港
],
```

**注意事项**：省略某些回退可能导致边缘情况解析不当。例如，如果省略 `zh_Hans` 和 `zh_Hans_CN`，台湾的简体中文用户（`zh_Hans_TW`）可能被解析为繁体中文。

## 语言环境解析流程

### 完整解析流程

```text
应用启动 / 设备语言环境改变
    ↓
1. 尝试 localeListResolutionCallback
    ↓ (返回 null)
2. 尝试 localeResolutionCallback
    ↓ (返回 null)
3. 使用基本解析算法
    ↓
    a. 完全匹配 (languageCode + scriptCode + countryCode)
    ↓ (未匹配)
    b. 语言+脚本匹配 (languageCode + scriptCode)
    ↓ (未匹配)
    c. 语言+国家匹配 (languageCode + countryCode)
    ↓ (未匹配)
    d. 仅语言匹配 (languageCode)
    ↓ (未匹配)
    e. 仅国家匹配 (countryCode)
    ↓ (未匹配)
    f. 回退到 supportedLocales 的第一个元素
    ↓
确定最终语言环境
```

## 属性之间的关系

### 依赖关系

- `locale` 必须匹配 `supportedLocales` 中的一个，否则使用 `supportedLocales` 的第一个元素
- `localeListResolutionCallback` 和 `localeResolutionCallback` 可以覆盖 `supportedLocales` 的默认匹配算法
- `localizationsDelegates` 提供实际的本地化资源

### 优先级

1. **localeListResolutionCallback**（最高优先级）
2. **localeResolutionCallback**
3. **supportedLocales 的基本算法**（最低优先级）

## 使用示例

### 基本配置

```dart
WidgetsApp(
  supportedLocales: [
    Locale('en', 'US'),
    Locale('zh', 'CN'),
    Locale('ja', 'JP'),
  ],
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
  ],
  color: Colors.blue,
)
```

### 使用自定义解析回调

```dart
WidgetsApp(
  supportedLocales: [
    Locale('en', 'US'),
    Locale('zh', 'CN'),
  ],
  localeListResolutionCallback: (locales, supportedLocales) {
    // 自定义解析逻辑
    for (final locale in locales ?? []) {
      for (final supported in supportedLocales) {
        if (locale.languageCode == supported.languageCode) {
          return supported;
        }
      }
    }
    return supportedLocales.first;
  },
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
  ],
  color: Colors.blue,
)
```

### 完整的中文支持

```dart
WidgetsApp(
  supportedLocales: <Locale>[
    const Locale('en', 'US'),
    const Locale.fromSubtags(languageCode: 'zh'),
    const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans'),
    const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant'),
    const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans', countryCode: 'CN'),
    const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant', countryCode: 'TW'),
    const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant', countryCode: 'HK'),
  ],
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    MyAppLocalizations.delegate,
  ],
  color: Colors.blue,
)
```

## 总结

第五部分详细介绍了 `WidgetsApp` 的所有本地化相关属性：

1. **locale**：应用的初始语言环境
2. **localizationsDelegates**：本地化资源委托列表
3. **localeListResolutionCallback**：考虑完整首选语言环境列表的解析回调（推荐）
4. **localeResolutionCallback**：只考虑第一个语言环境的解析回调
5. **supportedLocales**：应用支持的语言环境列表

这些属性共同构成了完整的国际化支持，允许应用根据用户的语言环境偏好自动选择合适的语言和本地化资源。正确配置这些属性对于多语言应用至关重要。
