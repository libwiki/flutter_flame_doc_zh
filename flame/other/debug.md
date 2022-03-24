# 调试功能

## FPS 计数器

火焰提供 `FPSCounter` 用于记录 fps 的 mixin；这个 mixin 可以应用于任何扩展自 `Game`.应用后，你可以使用 `fps` 方法，如下例所示。

```dart
class MyGame extends FlameGame with FPSCounter {
  static final fpsTextConfig = TextConfig(color: BasicPalette.white.color);

  @override
  void render(Canvas canvas) {
    super.render(canvas);
    final fpsCount = fps(120); // The average FPS for the last 120 microseconds.
    fpsTextConfig.render(canvas, fpsCount.toString(), Vector2(0, 50));
  }
}
```

## 火焰游戏特色

Flame 提供了一些调试功能 `FlameGame` 班级。这些功能在启用时 `debugMode` 属性设置为 `true`（或被覆盖为 `true`）。什么时候 `debugMode` 启用时，每个 `PositionComponent` 将以其边界大小呈现，并将其位置写在屏幕上。这样，你可以直观地验证组件的边界和位置。

查看调试功能的工作示例 `FlameGame`，检查这个 [example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/components/debug_example.dart).
