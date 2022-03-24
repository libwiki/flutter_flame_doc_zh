# 图层

图层允许你按上下文对渲染进行分组，并允许你预渲染事物。例如，这可以渲染在内存中变化不大的游戏部分，例如背景。通过这样做，你将释放处理能力来处理需要在每个游戏时间进行渲染的更多动态内容。

Flame 上有两种类型的图层：
 - `动态层`: 对于正在移动或变化的事物。
 - `预渲染层`: 对于静态的东西。

## DynamicLayer

动态图层是每次在画布上绘制时渲染的图层。顾名思义，它适用于动态内容，最适用于对具有相同上下文的对象进行分组渲染。

使用示例：
```dart
class GameLayer extends DynamicLayer {
  final MyGame game;

  GameLayer(this.game);

  @override
  void drawLayer() {
    game.playerSprite.render(
      canvas,
      position: game.playerPosition,
    );
    game.enemySprite.render(
      canvas,
      position: game.enemyPosition,
    );
  }
}

class MyGame extends Game {
  // Other methods omitted...

  @override
  void render(Canvas canvas) {
    gameLayer.render(canvas); // x and y can be provided as optional position arguments
  }
}
```

## PreRenderedLayer

预渲染层只渲染一次，缓存在内存中，然后在游戏画布上复制。它们对于缓存在游戏过程中不会改变的内容很有用，例如背景。

使用示例：

```dart
class BackgroundLayer extends PreRenderedLayer {
  final Sprite sprite;

  BackgroundLayer(this.sprite);

  @override
  void drawLayer() {
    sprite.render(
      canvas,
      position: Vector2(50, 200),
    );
  }
}

class MyGame extends Game {
  // Other methods omitted...

  @override
  void render(Canvas canvas) {
    backgroundLayer.render(canvas); // x and y can be provided as optional position arguments
  }
}
```

## 层处理器

Flame 还提供了一种在图层上添加处理器的方法，这是在整个图层上添加效果的方法。目前，开箱即用，只有 `ShadowProcessor` 可用时，此处理器会在你的图层上呈现背景阴影。

要将处理器添加到你的层，只需将它们添加到层 `preProcessors` 要么 `postProcessors` 列表。例如：

```dart
// Works the same for both DynamicLayer and PreRenderedLayer
class BackgroundLayer extends PreRenderedLayer {
  final Sprite sprite;

  BackgroundLayer(this.sprite) {
    preProcessors.add(ShadowProcessor());
  }

  @override
  void drawLayer() { /* omitted */ }

  // ...
```

可以通过扩展来创建自定义处理器 `LayerProcessor` 班级。

你可以检查图层的工作示例 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/rendering/layers.dart).
