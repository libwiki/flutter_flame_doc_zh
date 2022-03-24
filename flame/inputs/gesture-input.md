# 手势输入

这包括手势输入文档，即鼠标和触摸指针。

对于其他输入文档，另请参阅：

- [Keyboard Input](keyboard-input.md): 用于击键
- [Other Inputs](other-inputs.md): 用于操纵杆、游戏手柄等。


## 介绍

里面 `package:flame/gestures.dart` 你可以找到一整套 `mixin`s 可以包含在你的游戏类实例中，以便能够接收触摸输入事件。你可以在下面看到这些的完整列表 `mixin`s 及其方法：


## 触摸和鼠标检测器
```
- TapDetector
  - onTap
  - onTapCancel
  - onTapDown
  - onTapUp

- SecondaryTapDetector
  - onSecondaryTapDown
  - onSecondaryTapUp
  - onSecondaryTapCancel

- DoubleTapDetector
  - onDoubleTap

- LongPressDetector
  - onLongPress
  - onLongPressStart
  - onLongPressMoveUpdate
  - onLongPressUp
  - onLongPressEnd

- VerticalDragDetector
  - onVerticalDragDown
  - onVerticalDragStart
  - onVerticalDragUpdate
  - onVerticalDragEnd
  - onVerticalDragCancel

- HorizontalDragDetector
  - onHorizontalDragDown
  - onHorizontalDragStart
  - onHorizontalDragUpdate
  - onHorizontalDragEnd
  - onHorizontalDragCancel

- ForcePressDetector
  - onForcePressStart
  - onForcePressPeak
  - onForcePressUpdate
  - onForcePressEnd

- PanDetector
  - onPanDown
  - onPanStart
  - onPanUpdate
  - onPanEnd
  - onPanCancel

- ScaleDetector
  - onScaleStart
  - onScaleUpdate
  - onScaleEnd

 - MultiTouchTapDetector
  - onTap
  - onTapCancel
  - onTapDown
  - onTapUp

 - MultiTouchDragDetector
  - onReceiveDrag
```

仅鼠标事件

```
 - MouseMovementDetector
  - onMouseMove
 - ScrollDetector
  - onScroll
```


不能将高级检测器 (`MultiTouch*`) 与同类基本检测器混合使用，因为高级检测器将 *总是赢得手势竞技场* 并且基本的探测器永远不会被触发。因此，例如，你不能同时使用两者 `MultiTouchTapDetector` 和 `PanDetector` 一起，因为后者不会触发任何事件（对此也有断言）。

Flame 的 GestureApi 由 Flutter 的 Gesture Widgets 提供，包括 [手势检测器小部件](https://api.flutter.dev/flutter/小部件s/GestureDetector-class.html),[RawGestureDetector 小部件](https://api.flutter.dev/flutter/widgets/RawGestureDetector-class.html) 和 [MouseRegion 小部件](https://api.flutter.dev/flutter/widgets/MouseRegion-class.html)，你还可以阅读更多关于 Flutter 的手势 [here](https://api.flutter.dev/flutter/gestures/gestures-library.html).


## PanDetector 和 ScaleDetector

如果你添加一个 `PanDetector` 连同一个 `ScaleDetector`Flutter 会提示你一个非常神秘的断言：

```
Having both a pan gesture recognizer and a scale gesture recognizer is redundant; scale is a
superset of pan.

Just use the scale gesture recognizer.
```

这可能看起来很奇怪，但是 `onScaleUpdate` 不仅在应该更改比例时触发，而且对于所有平移/拖动事件也触发。因此，如果你需要同时使用这两个检测器，则必须在内部处理它们的两个逻辑 `onScaleUpdate`(+`onScaleStart` 和 `onScaleEnd`）。

例如，如果你想在平移事件上移动相机并在缩放事件上缩放，你可以执行以下操作：

```dart
  late double startZoom;

  @override
  void onScaleStart(_) {
    startZoom = camera.zoom;
  }

  @override
  void onScaleUpdate(ScaleUpdateInfo info) {
    final currentScale = info.scale.global;
    if (!currentScale.isIdentity()) {
      camera.zoom = startZoom * currentScale.y;
    } else {
      camera.translateBy(-info.delta.game);
      camera.snap();
    }
  }
```

在上面的例子中，平移事件被处理 `info.delta` 和规模事件 `info.scale`，尽管它们在理论上都来自潜在的规模事件。

这也可以在 [zoom example](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/camera_and_viewport/zoom_example.dart).


## 鼠标光标

也可以更改当前鼠标光标显示在 `GameWidget` 地区。为此，可以在内部使用以下代码 `Game` 班级

```dart
mouseCursor.value = SystemMouseCursors.move;
```

已经初始化 `GameWidget` 使用自定义光标，`mouseCursor` 可以使用属性

```dart
GameWidget(
  game: MouseCursorGame(),
  mouseCursor: SystemMouseCursors.move,
);
```


## 事件坐标系

在有位置的事件上，例如 `Tap*` 要么 `Drag`，你会注意到 `eventPosition` 属性包括 3 个字段：`游戏` ,`widget` 和 `全球的`.你将在下面找到关于它们中的每一个的简要说明。


### global

考虑到整个屏幕，事件发生的位置，与 `globalPosition` 在 Flutter 的原生事件中。


### widget

事件发生的位置相对于 `GameWidget` 位置和大小，同 `localPosition` 在 Flutter 的原生事件中。


### game

事件发生的位置相对于 `GameWidget` 以及游戏应用于游戏的任何转换（例如相机）。如果游戏没有任何变换，这将等于 `widget` 属性。


## 例子

```dart
class MyGame extends Game with TapDetector {
  // Other methods omitted

  @override
  bool onTapDown(TapDownInfo info) {
    print("Player tap down on ${info.eventPosition.game}");
    return true;
  }

  @override
  bool onTapUp(TapUpInfo info) {
    print("Player tap up on ${info.eventPosition.game}");
    return true;
  }
}
```

你还可以查看更完整的示例 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/controls/).


## 可点击、可拖动和可悬停组件

任何组件衍生自 `Component`（大多数组件）可以添加 `Tappable`， 这 `Draggable`, 和/或 `Hoverable`mixins 来处理组件上的点击、拖动和悬停。

所有被覆盖的方法都返回一个布尔值来控制是否应该将事件进一步向下传递到它下面的组件。所以说你只希望你的顶部可见组件接收点击，而不是它下面的，然后你的 `onTapDown`,`onTapUp` 和 `onTapCancel` 实现应该返回 `false` 如果你希望事件通过下面的更多组件，那么你应该返回 `true`.

如果你的组件有子组件，这同样适用，然后事件首先发送到子树中的叶子，然后进一步向下传递，直到方法返回 `false`.


### 可点击组件

通过添加 `HasTappables` 混入你的游戏，并使用混入 `Tappable` 在你的组件上，你可以在组件上覆盖以下方法：

```dart
bool onTapCancel();
bool onTapDown(TapDownInfo info);
bool onTapUp(TapUpInfo info);
```

最小组件示例：

```dart
import 'package:flame/components.dart';

class TappableComponent extends PositionComponent with Tappable {

  // update and render omitted

  @override
  bool onTapUp(TapUpInfo info) {
    print("tap up");
    return true;
  }

  @override
  bool onTapDown(TapDownInfo info) {
    print("tap down");
    return true;
  }

  @override
  bool onTapCancel() {
    print("tap cancel");
    return true;
  }
}

class MyGame extends FlameGame with HasTappables {
  MyGame() {
    add(TappableComponent());
  }
}
```

**笔记**：`HasTappables` 在引擎盖下使用高级手势检测器，正如本页进一步解释的那样，它不应该与基本检测器一起使用。

识别是否一个 `Tappable` 添加到游戏中处理了一个事件，`handled` 可以在事件中将字段设置为 true 可以在游戏类中的相应方法中进行检查，或者如果让事件继续传播，则可以在链的下方进行检查。

在下面的示例中，可以看到它是如何与 `onTapDown`, 同样的技术也可以应用于 `onTapUp`.

```dart
class MyComponent extends PositionComponent with Tappable{
  @override
  bool onTapDown(TapDownInfo info) {
    info.handled = true;
    return true;
  } 
}

class MyGame extends FlameGame with HasTappables {
  @override
  void onTapDown(int pointerId, TapDownInfo info) {
    if(info.handled) {
      // Do something if a child handled the event
    }
  }
}
```


### 可拖动组件

就像与 `Tappable`, Flame 提供了一个 mixin`Draggable` .

通过添加 `HasDraggables`mixin 到你的游戏中，并使用 mixin`Draggable` 在你的组件上，它们可以覆盖在你的组件上启用易于使用的拖动 api 的简单方法。

```dart
  bool onDragStart(DragStartInfo info);
  bool onDragUpdate(DragUpdateInfo info);
  bool onDragEnd(DragEndInfo info);
  bool onDragCancel();
```

请注意，所有事件都采用唯一生成的指针 id，因此如果需要，你可以区分不同的同时拖动。

提供的默认实现 `Draggable` 将已经检查：

- 在拖动开始时，组件仅在位置在其边界内时接收事件；保持
跟踪 pointerId。
- 在处理更新/结束/取消时，组件仅在 pointerId 为
跟踪（无论位置如何）。
- 在结束/取消时，停止跟踪 pointerId。

最小组件示例（此示例忽略 pointerId，因此如果你尝试多次拖动，它将无法正常工作）：

```dart
import 'package:flame/components.dart';

class DraggableComponent extends PositionComponent with Draggable {

  // update and render omitted

  Vector2? dragDeltaPosition;
  bool get isDragging => dragDeltaPosition != null;

  bool onDragStart(DragStartInfo startPosition) {
    dragDeltaPosition = startPosition.eventPosition.game - position;
    return false;
  }

  @override
  bool onDragUpdate(DragUpdateInfo event) {
    if (isDragging) {
      final localCoords = event.eventPosition.game;
      position = localCoords - dragDeltaPosition!;
    }
    return false;
  }

  @override
  bool onDragEnd(DragEndInfo event) {
    dragDeltaPosition = null;
    return false;
  }

  @override
  bool onDragCancel() {
    dragDeltaPosition = null;
    return false;
  }
}

class MyGame extends FlameGame with HasDraggables {
  MyGame() {
    add(DraggableComponent());
  }
}
```

识别是否一个 `Draggable` 添加到游戏中处理了一个事件，`handled` 可以在事件中将字段设置为 true 可以在游戏类中的相应方法中进行检查，或者如果让事件继续传播，则可以在链的下方进行检查。

在下面的示例中，可以看到它是如何与 `onDragStart`, 同样的技术也可以应用于 `onDragUpdate` 和 `onDragEnd`.

```dart
class MyComponent extends PositionComponent with Draggable {
 @override
 bool onDragStart(DragStartInfo info) {
   info.handled = true;
   return true;
 }
}

class MyGame extends FlameGame with HasDraggables {
  @override
  void onDragStart(int pointerId, DragStartInfo info) {
    if (info.handled) {
      // Do something if a child handled the event
    }
  }
}
```


### 可悬停组件

就像其他的一样，这个 mixin 允许你的组件轻松连接以监听悬停状态和事件。

通过添加 `HasHoverables`mixin 到你的基础游戏中，并使用 mixin`Hoverable` 在你的组件上，他们得到一个 `isHovered` 字段和几个方法（`onHoverStart`,`onHoverEnd` ) 如果你想收听事件，你可以覆盖它。

```dart
  bool isHovered = false;
  bool onHoverEnter(PointerHoverInfo info) {
    print("hover enter");
    return true;
  }
  bool onHoverLeave(PointerHoverInfo info) {
   print("hover leave");
   return true;
  }
```

提供的事件信息来自触发操作（进入或离开）的鼠标移动。当鼠标移动保持在内部或外部时，不会触发任何事件，并且不会传播这些鼠标移动事件。只有当状态改变时才会触发处理程序。

识别是否一个 `Hoverable` 添加到游戏中处理了一个事件，`handled` 可以在事件中将字段设置为 true 可以在游戏类中的相应方法中进行检查，或者如果让事件继续传播，则可以在链的下方进行检查。

在下面的示例中，可以看到它是如何与 `onHoverEnter`, 同样的技术也可以应用于 `onHoverLeave`.

```dart
class MyComponent extends PositionComponent with Hoverable {
  @override
  bool onHoverEnter(PointerHoverInfo info) {
    info.handled = true;
    return true;
  }
}

class MyGame extends FlameGame with HasHoverables {
  @override
  void onHoverEnter(PointerHoverInfo info) {
    if (info.handled) {
      // Do something if a child handled the event
    }
  }
}
```

### 手势命中框

这 `GestureHitboxes`mixin 用于更准确地识别你顶部的手势 `Component`s。假设你有一块相当圆的石头作为 `SpriteComponent` 例如，你不想注册位于未显示岩石的图像角落中的输入，因为 `PositionComponent` 默认为矩形。然后你可以使用 `GestureHitboxes`mixin 来定义一个更准确的圆形或多边形（或其他形状），输入应该在其中，以便在组件上注册事件。

你可以将新的命中框添加到具有 `GestureHitboxes`mixin 就像它们在下面添加一样 `Collidable` 例子。

有关如何定义 hitbox 的更多信息，请参见 [碰撞检测](../collision_detection.md#shapehitbox) 文档。

可以看到一个如何使用它的例子 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/input/gesture_hitboxes_example).
