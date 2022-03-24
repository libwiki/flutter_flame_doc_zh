# 相机和视口

在 Flutter 上渲染时，使用的常规坐标空间是逻辑像素。这意味着 Flutter 的一个像素不一定是设备上的一个真实像素，因为 [设备像素比](https://api.flutter.dev/flutter/widgets/MediaQueryData/devicePixelRatio.html).当它到达火焰级别时，我们总是认为最基本的级别是逻辑像素，因此所有设备特定的复杂性都被抽象掉了。

但是，这仍然会给你留下任意形状和大小的屏幕。而且很可能你的游戏有某种游戏世界，其具有不映射到屏幕坐标的固有坐标系。Flame 添加了两个不同的概念来帮助转换坐标空间。对于前者，我们有 视口 类。对于后者，我们有 相机 类。

## Viewport

视口试图通过平移和调整画布大小将多个屏幕（或者，更确切地说，游戏小部件）尺寸统一为游戏的单一配置。

这 `Viewport` 接口有多种实现，可以从头开始使用 `Game` 或者，如果你正在使用 `FlameGame` 相反，它已经是内置的（具有默认的无操作视口）。

这些是可供选择的视口（或者你可以自己实现接口以满足你的需求）：

 * `DefaultViewport`：这是默认情况下与任何关联的无操作视口 `FlameGame`.
 * `FixedResolutionViewport`：此视口会转换你的画布，以便从游戏的角度来看，尺寸始终设置为固定的预定义值。这意味着它将尽可能地扩展游戏并在需要时添加黑条。

使用时 `FlameGame`，视口执行的操作会自动执行到每个渲染操作，并且 `size` 游戏中的属性，而不是逻辑部件大小，变成了通过视口看到的大小以及相机的缩放。如果由于某种原因你需要访问原始的真实逻辑像素大小，你可以使用 `canvasSize`.有关每个内容的更深入描述 `Viewport` 它是如何运作的，请检查其类的文档。

## Camera

不像 `Viewport`， 这 `Camera` 是一个更动态的 `Canvas` 转换通常取决于：

 * 与屏幕坐标 1:1 不匹配的世界坐标。
 * 在游戏世界中居中或跟随玩家（如果世界大于屏幕）。
 * 用户控制的放大和缩小。

只有一种相机实现，但它允许许多不同的配置。同样，你可以在你的 `Game` 但它已经包含并连接到 `FlameGame`.

关于相机需要注意的一件重要事情是，由于（与视口不同）它是动态的，因此大多数相机移动不会立即发生。相反，相机具有可配置的速度，并在游戏循环中更新。如果你想立即移动你的相机（例如在游戏开始时的第一个相机设置），你可以使用 `snap` 功能。不过，在游戏期间调用 snap 可能会导致不和谐或不自然的摄像机移动，因此除非需要（例如地图过渡），否则请避免这种情况。仔细检查每种方法的文档，了解有关它如何影响相机移动的更多详细信息。

另一个重要的注意事项是相机在视口之后应用，并且仅应用于非 HUD 组件。所以这里的屏幕尺寸是考虑视口变换后的有效尺寸。

相机可以对画布应用两种类型的变换。第一个也是最复杂的一个是翻译。这可以通过几个因素来应用：

 * 什么都没有：默认情况下，相机不会应用任何转换，因此可以选择使用它。
 * 相对偏移：你可以配置它来决定“相机的中心应该在屏幕上的哪个位置”。默认情况下它是左上角，这意味着居中的坐标或对象将始终位于屏幕的左上角。你可以在游戏过程中平滑地更改相对偏移量（例如，可用于应用对话或物品拾取临时相机过渡）。
 * moveTo：如果你想临时移动你的相机，你可以使用这个方法；它会将相机平滑地过渡到新位置，忽略跟随但尊重相对偏移和世界边界。这可以通过重置 `resetMovement` 如果与跟随一起使用，则被跟随的对象再次开始被考虑。
 * follow：你可以使用这个方法让你的相机持续“跟随”一个物体（例如，一个 `PositionComponent`）。这并不平滑，因为被跟踪对象本身的运动被假定为已经平滑（即如果你的角色传送，相机也将立即传送）。
 * 世界边界：使用跟随时，你可以选择定义世界的边界。如果这样做了，相机将停止跟随/移动，以便不显示越界区域（只要世界大于屏幕）。

最后，相机应用的第二个变换是缩放。这允许动态缩放，它由 `zoom` 场地。没有变焦速度，改变时必须由你控制。这 `zoom` 变量立即应用。

在处理输入事件时，必须将屏幕坐标转换为世界坐标（或者，由于某些原因，你可能想要反过来）。相机提供两个功能，`screenToWorld` 和 `worldToScreen` 在这些坐标空间之间轻松转换。


### Camera.followVector2

立即拍下相机开始跟随 `Vector2`.

这意味着相机将移动，以使位置矢量位于屏幕上的固定位置。该位置由屏幕尺寸的一小部分决定 `relativeOffset` 参数（默认为中心）。这 `worldBounds` 可以选择设置参数以将边界添加到允许相机移动的距离。

例子：

```dart
class MyGame extends FlameGame {
  final someVector = Vector2(100, 100);

  Future<void> onLoad() async {
     camera.followVector2(someVector);
  }
}

```


### Camera.followComponent

立即拍下相机开始跟随 `PositionComponent`.

这意味着相机将移动，以使组件的位置矢量位于屏幕上的固定位置。该位置由屏幕尺寸的一小部分决定 `relativeOffset` 参数（默认为中心）。这 `worldBounds` 可以选择设置参数以将边界添加到允许相机移动的距离。

该组件被其锚点“抓取”（默认为左上角）。因此，例如，如果你希望对象的中心位于固定位置，请将组件锚点设置为中心。

例子：

```dart
class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
     final sprite = await loadSprite('pizza.png');
     final player = SpriteComponent(
       sprite: sprite,
       size: size,
       anchor: Anchor.center,
     );
     add(player);
     
     camera.followComponent(player);
  }
}
```


### 将相机与 Game 类一起使用

如果你不使用 `FlameGame`，而是使用 `Game`mixin，那么你需要自己管理调用某些相机方法。假设我们有以下游戏结构，并且我们想要添加相机功能：

```dart
class YourGame with Game {
  Camera? camera;

  Future<void> onLoad() async {}

  void render(Canvas canvas) {}

  void update(double dt) {}
}
```

我们首先在加载时创建一个新的相机实例并将我们的游戏指定为参考：

```dart
  // ...
  
  Future<void> onLoad() async {
    camera = Camera();

    // This is required for the camera to work.
    camera?.gameRef = this;

    // Not required but recommend to set it now or when you set the follow target.
    camera?.worldBounds = yourWorldBounds;

    // Rest of your on load code.
  }

  // ...
```

相机还可以知道要跟随哪个位置，这是一项可选功能，因为你也可以使用相机进行移动、捕捉或摇晃。

要做到这一点 `Camera` 类为它提供了多种方法，但让我们展示一个最简单的方法，那就是 `followVector2`：

```dart
  // Somewhere in your code.

  camera?.followVector2(
    yourPositionToFollow,
    worldBounds: yourWorldBounds, // Optional to pass, it will overwrite the previous bounds.
  );
```

现在相机已经创建并且它知道世界边界和它应该遵循的位置，它可以用来在渲染方法中翻译画布：

```dart
  // ...

  void render(Canvas canvas) {
    camera?.apply(canvas); // This will apply the camera transformation.

    // Rest of your rendering code.
  }

  // ...
```

剩下要做的就是打电话给 `update` 上的方法 `Camera` 所以它可以顺利地跟随你给定的位置：

```dart
  // ...

  void update(double dt) {
    camera?.update(dt);

    // Rest of your update code.
  }

  // ...
```
