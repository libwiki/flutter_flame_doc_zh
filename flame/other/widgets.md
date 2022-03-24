# Widgets

使用 Flutter 开发游戏的一个优势是能够使用 Flutter 丰富的工具集来构建 UI，Flame 试图通过引入考虑到游戏的小部件来扩展这一点。

在这里，你可以找到 Flame 提供的所有可用小部件。

你还可以查看显示在 [Dashbook](https://github.com/erickzanardo/dashbook) 沙盒 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/widgets)

## 九宫格

九宫格是使用网格精灵绘制的矩形。

网格精灵是一个 3x3 的网格，有 9 个方块，分别代表 4 个角、4 个边和中间。

角以相同的尺寸绘制，边沿侧向拉伸，中间向两边展开。

这 `NineTileBox` 小部件实现了一个 `Container` 使用该标准。这种模式也被实现为 `NineTileBoxComponent` 这样你就可以直接将此功能添加到你的 `FlameGame`.要了解更多信息，请查看组件文档 [here](../components.md#ninetileboxcomponent).

在这里你可以找到如何使用它的示例（不使用 `NineTileBoxComponent`):

```dart
import 'package:flame/widgets/nine_tile_box.dart';

NineTileBox.asset(
    image: image, // dart:ui image instance
    tileSize: 16, // The width/height of the tile on your grid image
    destTileSize: 50, // The dimensions to be used when drawing the tile on the canvas
    child: SomeWidget(), // Any Flutter widget
)
```

## SpriteButton

`SpriteButton` 是一个基于火焰精灵创建按钮的简单小部件。这在尝试创建非默认外观按钮时非常有用。例如，当你通过在图形编辑器中绘制按钮而不是直接在 Flutter 中制作它来更容易地实现你想要的外观时。

如何使用它：

```dart
SpriteButton.asset(
    onPressed: () {
      print('Pressed');
    },
    label: const Text('Sprite Button', style: const TextStyle(color: const Color(0xFF5D275D))),
    sprite: _spriteButton,
    pressedSprite: _pressedSprite,
)
```

## SpriteWidget

`SpriteWidget` 是一个小部件，用于显示 [Sprite](../rendering/images.md#sprite) 在小部件树内。

这是如何使用它：

```dart
SpriteWidget.asset(
    sprite: yourSprite,
    anchor: Anchor.center,
)
```

## SpriteAnimationWidget

`SpriteAnimationWidget` 是一个用于显示的小部件 [精灵动画](../rendering/images.md#animation) 在小部件树内。

这是如何使用它：

```dart
SpriteAnimationWidget.asset(
    animation: _animation,
    playing: true,
    anchor: Anchor.center,
)
```
