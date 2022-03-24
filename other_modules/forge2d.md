# Forge2D

我们（Flame 组织）维护了 Box2D 物理引擎的移植版本，我们的版本称为 Forge2D。

如果你想专门为 Flame 使用 Forge2D，你应该使用我们的桥接库 [flame_forge2d](https://github.com/flame-engine/flame/tree/main/packages/flame_forge2d) 如果你只想在 Dart 项目中使用它，你可以使用 [forge2d](https://github.com/flame-engine/forge2d) 图书馆直接。

要在你的游戏中使用它，你只需添加 `flame_forge2d` 到你的 pubspec.yaml，如 [Forge2D example](https://github.com/flame-engine/flame/tree/main/packages/flame_forge2d/example) 并在 pub.dev[安装说明](https://pub.dev/packages/flame_forge2d) .

## Forge2D 游戏

如果你要在项目中使用 Forge2D，最好使用 Forge2D 特定的 `FlameGame` 班级，`Forge2DGame` .

它被称为 `Forge2DGame` 它将控制 Forge2D 的添加和删除 `身体组件s` 以及你的正常组件。

在 `Forge2DGame` 这 `Camera` 默认情况下，缩放级别设置为 10，因此你的组件将比普通 Flame 游戏大得多。这是由于速度限制在 `Forge2D` 世界，如果你使用它，你会很快击中 `zoom = 1.0`.你可以通过调用轻松更改缩放级别 eiter`super(zoom: yourZoom)` 在你的构造函数中，或者做 `game.camera.zoom = yourZoom;` 在稍后阶段。

如果你以前熟悉 Box2D，那么很高兴知道 Box2d 世界的整个概念都映射到 `world` 在里面 `Forge2DGame` 组件和每个 `Body` 应该是一个 `BodyComponent`，并添加到你的 `Forge2DGame`.

例如，你可以有一个 HUD 和其他与物理无关的组件 `Forge2DGame` 的组件列表以及你的物理实体。调用更新时，它将使用 Forge2D 物理引擎正确更新每个孩子。

一个简单的 `Forge2DGame` 实现示例可以在 [examples folder](https://github.com/flame-engine/flame/tree/main/packages/flame_forge2d/example).


## BodyComponent

如果你不需要在你的身体上放一个精灵，你应该使用平原 `BodyComponent`，例如，如果你想要一个圆形、矩形或多边形，但只用 Flutter 绘制 `Paint`.

这 `BodyComponent` 默认情况下有 `debugMode = true`，否则在你创建一个 `Body` 并添加了 `BodyComponent` 到游戏。如果你想关闭它，你可以覆盖 `debugMode` 在组件构造函数中将其设置为 false 或为其分配 false。


## SpriteBody 组件

```{admonition} Deprecated
Add a `SpriteComponent` to a [](#bodycomponent) instead. Will be removed in 0.10.0
```

通常你想在上面渲染一个精灵 `BodyComponent` 你将在你的 `Forge2DGame`.这个组件将处理你的精灵在身体顶部的缩放和定位。


## PositionBodyComponent

```{admonition} Deprecated
Add children to a [](#bodycomponent) instead. Will be removed in 0.10.0
```

Flame 中最常用的类之一是 `PositionComponent`, Flame 中很多常用的组件都是 `PositionComponent`.如果你想放一个 `PositionComponent` 或其任何位于 Forge2D 主体之上的子类，你可以使用 `PositionBodyComponent` 它会，就像 `SpriteBodyComponent`, 处理该组件的旋转、定位和缩放，使其跟随底层 `Body`.


## 联系回调

如果你正在使用 `Forge2DGame` 你可以利用其处理两个联系人之间的方式 `BodyComponent`s。

为你创建身体定义时 `BodyComponent` 确保将 userdata 设置为当前对象，否则将无法检测到碰撞。像这样：
```dart
final bodyDef = BodyDef()
  // To be able to know which component that is involved in a collision
  ..userData = this;
```

现在你必须实现 `ContactCallback` 你可以在其中设置当它们接触时它应该做出反应的两种类型。如果你有两个 `BodyComponent` 命名为 `Ball` 和 `Wall` 并且你想在他们接触时做一些事情，你可以做这样的事情：

```dart
class BallWallCallback extends ContactCallback<Ball, Wall> {
  BallWallCallback();

  @override
  void begin(Ball ball, Wall wall, Contact contact) {
    wall.remove();
  }

  @override
  void end(Ball ball, Wall wall, Contact contact) {}
}
```

然后你只需添加 `BallWallCallback` 给你的 `Forge2DGame`：

```dart
class MyGame extends Forge2DGame {
  MyGame(Forge2DComponent box) : super(box) {
    addContactCallback(BallWallCallback());
  }
}
```

每次 `Ball` 和 `Wall` 取得联系 `begin` 将被调用，并且一旦对象停止接触 `end` 将被调用。

如果你希望一个对象与所有其他实体交互，请将 `BodyComponent` 作为你的参数之一 `ContactCallback` 像这样：

`class BallAnythingCallback implements ContactCallback<Ball, BodyComponent> ...`

一个实现示例可以在 [火焰 Forge2D 示例](https://github.com/flame-engine/flame_forge2d/blob/main/example).


## 视口和相机

`Forge2DGame` 正在使用自己的普通火焰实现 `Viewport` 和 `Camera`, 可以阅读更多关于 [here](../flame/camera_and_viewport.md).

如果你将屏幕视为窗口，而将外部视为 Forge2D 世界，那么 `Viewport` 是你可以透过窗户看到的外面世界的一部分，所以你可以在屏幕上看到的部分。

要移动你在该窗口中看到的内容，请使用 `Camera`，如果你想在 Forge2D 世界中跟踪你的组件之一，或者知道屏幕上的点在世界上的哪个位置，这也非常有用，反之亦然。


### Forge2DCamera.followBodyComponent

和普通的一样 `PositionComponent`s 你可以做 `Forge2DCamera` 跟随 `BodyComponent` 通过调用 `camera.followBodyComponent(...)` 它的工作原理与 [camera.followComponent](../flame/camera_and_viewport.md#camerafollowcomponent).当你想停止关注 `BodyComponent` 你应该打电话 `camera.unfollowBodyComponent`.
