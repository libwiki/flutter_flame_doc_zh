# 调色板

在整个游戏过程中，你将需要在很多地方使用颜色。有两个班 `dart:ui` 可以使用的，`Color` 和 `Paint`.

这 `Color`class 以十六进制整数格式表示 ARGB 颜色。所以要创建一个 `Color` 例如，你只需要将颜色作为 ARGB 格式的整数传递。

你可以使用 Dart 的十六进制表示法让它变得非常简单；例如：`0xFF00FF00` 是完全不透明的绿色（“面具”将是 `0xAARRGGBB`）。

**笔记**：前两个十六进制数字用于 Alpha 通道（透明度），与常规（非 A）RGB 不同。前两个数字的 max(FF = 255) 表示完全不透明，而 min (00 = 0) 表示完全透明。

在 Material Flutter 包中有一个 `Colors` 提供常用颜色作为常量的类：

```dart
import 'package:flutter/material.dart' show Colors;

const black = Colors.black;
```

一些更复杂的方法也可能需要 `Paint` 对象，这是一个更完整的结构，允许你配置与笔触、颜色、过滤器和混合相关的方面。但是，通常在使用更复杂的 API 时，你只需要一个 `Paint` 仅代表一种简单纯色的对象。

**笔记：** 我们不建议你创建新的 `Paint` 每次你需要特定的对象时 `Paint`，因为它可能会导致创建许多不必要的对象。更好的方法是定义 `Paint` 在某处对象并重新使用它（但是，请注意 `Paint` 类是可变的，不像 `Color`)，或使用 `Palette` 类来定义你想在游戏中使用的所有颜色。

你可以像这样创建这样的对象：

```dart
Paint green = Paint()..color = const Color(0xFF00FF00);
```

为了帮助你并保持游戏的调色板一致，Flame 添加了 `Palette` 班级。你可以使用它轻松访问两者 `Color` 沙 `Paint`s 在需要的地方，还将你的游戏使用的颜色定义为常量，这样你就不会把它们弄混了。

这 `BasicPalette` 类是调色板外观的一个示例，并添加了黑色和白色作为颜色。因此，要使用黑色或白色，你可以直接从 `BasicPalette`;例如，使用 `color`：

```dart
TextConfig regular = TextConfig(color: BasicPalette.white.color);
```

或使用 `paint`：

```dart
canvas.drawRect(rect, BasicPalette.black.paint);
```

但是，这个想法是你可以创建自己的调色板，遵循 `BasicPalette` 例如，并添加游戏的调色板/方案。然后，你将能够静态访问组件和类中的任何颜色。下面是一个示例 `Palette` 实施，来自示例游戏 [BGUG](https://github.com/bluefireteam/bgug/blob/master/lib/palette.dart)：

```dart
import 'dart:ui';

import 'package:flame/palette.dart';

class Palette {
  static PaletteEntry white = BasicPalette.white;

  static PaletteEntry toastBackground = PaletteEntry(Color(0xFFAC3232));
  static PaletteEntry toastText = PaletteEntry(Color(0xFFDA9A00));

  static PaletteEntry grey = PaletteEntry(Color(0xFF404040));
  static PaletteEntry green = PaletteEntry(Color(0xFF54a286));
}
```

一种 `PaletteEntry` 是一个 `const` 保存颜色信息的类，它具有以下成员：

 - `color`: 返回 `Color` 指定的
 - `paint`: 创建一个新的 `Paint` 与指定的颜色。`Paint` 是一个非 `const` 类，所以这个
  方法实际上每次调用时都会创建一个全新的实例。级联突变是安全的。
