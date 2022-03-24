# 文本渲染

Flame 有一些专门的类来帮助你渲染文本。

## 文本渲染器

`TextRenderer` 是 Flame 用来渲染文本的抽象类。Flame 为此提供了一种实现，称为 `文字画` 但是任何人都可以实现这种抽象并创建一种自定义方式来呈现文本。

## TextPaint

一种 `TextPaint` 是 Flame 中文本渲染的内置实现，它是基于 Flutter 的 `TextPainter` 类（因此得名），它可以由样式类配置 `TextStyle` 其中包含呈现文本所需的所有印刷信息；即字体大小和颜色、字体系列等。

示例用法：

```dart
const TextPaint textPaint = TextPaint(
  style: TextStyle(
    fontSize: 48.0,
    fontFamily: 'Awesome Font',
  ),
);
```

注意：有几个包包含该类 `TextStyle`，请确保你导入 `package:flutter/material.dart` 要么 `package:flutter/painting.dart` 如果你还需要导入 `dart:ui` 你需要像这样导入它（因为它包含另一个也被命名的类 `TextStyle`):

```dart
import 'dart:ui' hide TextStyle;
```

一些常见的属性 `TextStyle` 以下是（这里是 [full list](https://api.flutter.dev/flutter/painting/TextStyle-class.html)):

 - `fontFamily`：常用字体，如 Arial（默认），或添加到你的自定义字体
pubspec（见 [here](https://flutter.io/custom-fonts/) 怎么做）。
 - `fontSize`: 字体大小，以 pts 为单位（默认 `24.0`）。
 - `height`: 文本行的高度，字体大小的倍数（默认 `null`）。
 - `color`：颜色，作为 `ui.Color`（默认白色）。

有关颜色以及如何创建颜色的更多信息，请参阅 [颜色和调色板](palette.md) 指导。

创建后 `TextPaint` 你可以使用它的对象 `render` 在画布上绘制字符串的方法：

```dart
textPaint.render(canvas, "Flame is awesome", Vector2(10, 10));
```

如果你想设置文本的锚点，你也可以在渲染调用中执行此操作，可选 `anchor` 范围：

```dart
textPaint.render(canvas, 'Flame is awesome', Vector2(10, 10), anchor: Anchor.topCenter);
```

## 文本组件

Flame 提供了两个文本组件，可以更轻松地在游戏中渲染文本：`文本组件` 和 `文本框组件`.

### TextComponent

`TextComponent` 是一个呈现单行文本的简单组件。

示例用法：

```dart
final style = TextStyle(color: BasicPalette.white.color);
final regular = TextPaint(style: style);

class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    add(TextComponent(text: 'Hello, Flame', textRenderer: regular)
      ..anchor = Anchor.topCenter
      ..x = size.width / 2 // size is a property from game
      ..y = 32.0);
  }
}
```

### TextBoxComponent

`TextBoxComponent` 非常相似 `TextComponent`，但顾名思义，它用于在边界框内渲染文本，根据提供的框大小创建换行符。

你可以决定该框是否应该随着文本的写入而增长，或者是否应该由 `growingBox` 中的变量 `TextBoxConfig`.

如果要更改框的边距，请使用 `margins` 中的变量 `TextBoxConfig`.

示例用法：

```dart
class MyTextBox extends TextBoxComponent {
  MyTextBox(String text) : super(text: text, textRenderer: tiny, boxConfig: TextBoxConfig(timePerChar: 0.05));

  @override
  void drawBackground(Canvas c) {
    Rect rect = Rect.fromLTWH(0, 0, width, height);
    c.drawRect(rect, Paint()..color = Color(0xFFFF00FF));
    c.drawRect(
        rect.deflate(boxConfig.margin),
        BasicPalette.black.Paint()
          ..style = PaintingStyle.stroke,
    );
  }
}
```

这两个组件都在一个示例中展示 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/rendering/text.dart)
