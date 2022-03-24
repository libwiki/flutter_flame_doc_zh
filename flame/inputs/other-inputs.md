# 其他输入

这包括键盘和鼠标以外的输入法的文档。

对于其他输入文档，另请参阅：

- [Gesture Input](gesture-input.md)：用于鼠标和触摸指针手势
- [Keyboard Input](keyboard-input.md): 用于击键

## 操纵杆

Flame 提供了一个组件，该组件能够创建一个虚拟操纵杆，以便为你的游戏获取输入。要使用此功能，你需要创建一个 `JoystickComponent`，以你想要的方式配置它，并将其添加到你的游戏中。

检查此示例以获得更好的理解：

```dart
class MyGame extends FlameGame with HasDraggables {

  MyGame() {
    joystick.addObserver(player);
    add(player);
    add(joystick);
  }

  @override
  Future<void> onLoad() async {
    super.onLoad();
    final image = await images.load('joystick.png');
    final sheet = SpriteSheet.fromColumnsAndRows(
      image: image,
      columns: 6,
      rows: 1,
    );
    final joystick = JoystickComponent(
      knob: SpriteComponent(
        sprite: sheet.getSpriteById(1),
        size: Vector2.all(100),
      ),
      background: SpriteComponent(
        sprite: sheet.getSpriteById(0),
        size: Vector2.all(150),
      ),
      margin: const EdgeInsets.only(left: 40, bottom: 40),
    );

    final player = Player(joystick);
    add(player);
    add(joystick);
  }
}

class JoystickPlayer extends SpriteComponent with HasGameRef {
  /// Pixels/s
  double maxSpeed = 300.0;

  final JoystickComponent joystick;

  JoystickPlayer(this.joystick)
      : super(
          size: Vector2.all(100.0),
        ) {
    anchor = Anchor.center;
  }

  @override
  Future<void> onLoad() async {
    super.onLoad();
    sprite = await gameRef.loadSprite('layers/player.png');
    position = gameRef.size / 2;
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (joystick.direction != JoystickDirection.idle) {
      position.add(joystick.velocity * maxSpeed * dt);
      angle = joystick.delta.screenAngle();
    }
  }
}
```

所以在这个例子中，我们创建了类 `MyGame` 和 `Player`.`MyGame` 创建一个操纵杆，传递给 `Player` 当它被创建时。在里面 `Player` 我们对操纵杆的当前状态采取行动。

操纵杆有几个字段会根据它所处的状态而变化。这些字段应该用于了解操纵杆的状态：
 - `intensity`: 旋钮从震中拖到边缘的百分比[0.0, 1.0]
  操纵杆（或 `knobRadius` 如果已设置）。
 - `delta`：绝对金额（定义为 `Vector2`) 旋钮被拖离其震中。
 - `velocity`：百分比，表示为 `Vector2`, 以及旋钮当前的方向
  从其基本位置拉到操纵杆的边缘。

如果你想创建与操纵杆配合使用的按钮，请查看 [`HudButton 组件`](#hudbuttoncomponent).

可以找到如何使用它的完整示例 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/input/joystick.dart).并且可以看到运行 [here](https://examples.flame-engine.org/#/Controls_Joystick).

## Hud按钮组件

一种 `HudButtonComponent` 是一个可以定义为边缘到边缘的按钮 `Viewport` 而不是一个职位。需要两个 `PositionComponent`s。`button` 和 `buttonDown`，第一个用于按钮空闲时，第二个用于按下按钮时显示。如果你不想在按下按钮时更改按钮的外观，或者如果你通过 `button` 零件。

顾名思义，这个按钮默认是一个 hud，这意味着即使游戏的摄像头四处移动，它也会在你的屏幕上保持静止。你还可以通过设置将此组件用作非 hud`hudButtonComponent.respectCamera = true;` .

如果你想对按下的按钮（这将是常见的事情）并释放按钮采取行动，你可以传入回调函数作为 `onPressed` 和 `onReleased` 参数，或者你可以扩展组件并覆盖 `onTapDown`,`onTapUp` 和/或 `onTapCancel` 并在那里实现你的逻辑。

## 精灵按钮组件

一种 `SpriteButtonComponent` 是一个由两个定义的按钮 `Sprite`s，一个代表按下按钮的时间，一个代表释放按钮的时间。

## ButtonComponent

一种 `ButtonComponent` 是一个由两个定义的按钮 `PositionComponent`s，一个代表按下按钮的时间，一个代表释放按钮的时间。如果你只想为按钮使用精灵，请使用 [](#spritebuttoncomponent) 取而代之的是，但是如果你想拥有一个 `SpriteAnimationComponent` 作为一个按钮，或任何其他不是纯精灵的东西。

## 游戏手柄

Flame 有一个单独的插件来支持外部游戏控制器（游戏手柄），请查看 [here](https://github.com/flame-engine/flame_gamepad) 了解更多信息。
