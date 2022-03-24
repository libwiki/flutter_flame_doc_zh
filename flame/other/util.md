# 实用程序

在此页面上，你可以找到一些实用程序类和方法的文档。

## 设备类

这个类可以从 `Flame.device` 它有一些方法可以用来控制设备的状态，例如你可以改变屏幕方向和设置应用程序是否应该是全屏的。

### `Flame.device.fullScreen()`

调用时，这将禁用所有 `SystemUiOverlay` 使应用程序全屏。在 main 方法中调用时，它会使你的应用全屏显示（没有顶部或底部栏）。

**笔记：** 在 web 上调用时没有效果。

### `Flame.device.setLandscape()`

此方法将整个应用程序（实际上也是游戏）的方向设置为横向，并且根据操作系统和设备设置，应该允许左右横向方向。要将应用程序方向设置为特定方向的横向，请使用 `Flame.device.setLandscapeLeftOnly` 要么 `Flame.device.setLandscapeRightOnly`.

**笔记：** 在 web 上调用时没有效果。

### `Flame.device.setPortrait()`

此方法将整个应用程序（实际上也是游戏）的方向设置为纵向，并且根据操作系统和设备设置，它应该允许上下纵向方向。要将应用程序方向设置为特定方向的纵向，请使用 `Flame.device.setPortraitUpOnly` 要么 `Flame.device.setPortraitDownOnly`.

**笔记：** 在 web 上调用时没有效果。

### `Flame.device.setOrientation()` 和 `Flame.device.setOrientations()`

如果需要对允许的方向进行更精细的控制（无需处理 `SystemChrome` 直接地），`setOrientation` （接受单 `DeviceOrientation` 作为参数）和 `setOrientations`（接受一个 `List<DeviceOrientation>` 对于可能的方向）可以使用。

**笔记：** 在 web 上调用时没有效果。

## 定时器

Flame 提供了一个简单的实用程序类来帮助你处理倒计时和计时器状态更改（如事件）。

倒计时示例：

```dart
import 'dart:ui';

import 'package:flame/game.dart';
import 'package:flame/text_config.dart';
import 'package:flame/timer.dart';
import 'package:flame/vector2.dart';

class MyGame extends Game {
  final TextConfig textConfig = TextConfig(color: const Color(0xFFFFFFFF));
  final countdown = Timer(2);

  @override
  void update(double dt) {
    countdown.update(dt);
    if (countdown.finished) {
      // Prefer the timer callback, but this is better in some cases
    }
  }

  @override
  void render(Canvas canvas) {
    textConfig.render(
      canvas,
      "Countdown: ${countdown.current.toString()}",
      Vector2(10, 100),
    );
  }
}

```

间隔示例：

```dart
import 'dart:ui';

import 'package:flame/game.dart';
import 'package:flame/text_config.dart';
import 'package:flame/timer.dart';
import 'package:flame/vector2.dart';

class MyGame extends Game {
  final TextConfig textConfig = TextConfig(color: const Color(0xFFFFFFFF));
  Timer interval;

  int elapsedSecs = 0;

  MyGame() {
    interval = Timer(
      1,
      onTick: () => elapsedSecs += 1,
      repeat: true,
    );
  }

  @override
  void update(double dt) {
    interval.update(dt);
  }

  @override
  void render(Canvas canvas) {
    textConfig.render(canvas, "Elapsed time: $elapsedSecs", Vector2(10, 150));
  }
}

```

`Timer` 实例也可以在内部使用 `FlameGame` 游戏通过使用 `TimerComponent` 班级。

`TimerComponent` 例子：

```dart
import 'package:flame/timer.dart';
import 'package:flame/components/timer_component.dart';
import 'package:flame/game.dart';

class MyFlameGame extends FlameGame {
  MyFlameGame() {
    add(
      TimerComponent(
        period: 10,
        repeat: true,
        onTick: () => print('10 seconds elapsed'),
      )
    );
  }
}
```

## 扩展

Flame 捆绑了一系列实用程序扩展，这些扩展旨在帮助开发人员提供快捷方式和转换方法，你可以在此处找到这些扩展的摘要。

它们都可以从 `package:flame/extensions.dart`

### 帆布

方法：
 - `scaleVector`： 就像 `canvas scale` 方法，但需要一个 `矢量 2` 作为论据。
 - `translateVector`： 就像 `canvas translate` 方法，但需要一个 `Vector2` 作为论据。
 - `renderPoint`: 在画布上渲染一个点（主要用于调试目的）。
 - `renderAt` 和 `renderRotated`：如果你直接渲染到 `Canvas`, 你可以使用这些
  可以轻松操纵坐标以在正确的位置渲染事物的功能。他们改变 `Canvas` 转换矩阵，但之后重置。

### 颜色

方法：
 - `darken`：将颜色的深浅加深 0 到 1 之间的量。
 - `brighten`：以 0 到 1 的量使颜色的深浅变亮。

工厂：
- `ColorExtension.fromRGBHexString`：从有效的十六进制字符串（例如 #1C1C1C）解析 RGB 颜色。
- `ColorExtension.fromARGBHexString`：从有效的十六进制字符串（例如 #FF1C1C1C）中解析 ARGB 颜色。

### 图片

方法：
 - `pixelsInUint8`: 检索像素数据作为 `Uint8List`， 在里面 `ImageByteFormat.rawRgba`
像素格式，用于图像。
 - `getBounding矩形`: 获取边界矩形 `Image` 作为一个 `Rect`.
 - `size`: 一个大小 `Image` 作为 `Vector2`.
 - `darken`：使每个像素变暗 `Image`0 到 1 之间的数量。
 - `brighten`: 使每个像素变亮 `Image`0 到 1 之间的数量。

### 抵消

方法;
 - `toVector2`;创建一个 `Vector2` 来自 `Offset`.
 - `to尺寸`: 创建一个 `Size` 来自 `Offset`.
 - `toPoint`: 创建一个 `Point` 来自 `Offset`.
 - `toRect`: 创建一个 `Rect` 从 (0,0) 开始，其右下角是 [Offset]。

### Rect

方法：
 - `toOffset`: 创建一个 `Offset` 来自 `Rect`.
 - `toVector2`: 创建一个 `Vector2` 从 (0,0) 开始，到 `Rect`.
 - `containsPoint` 这是否 `Rect` 包含一个 `Vector2` 点与否。
 - `intersectsSegment`;是否由两个组成的段 `Vector2`s 与此相交 `Rect`.
 - `intersectsLineSegment`： 是否 `LineSegment` 相交 `Rect`.
 - `toVertices`：转动四个角 `Rect` 进入列表 `Vector2`.
 - `toMathRectangle`: 转换这个 `Rect` 成一个 `数学.矩形`.
 - `toGeometryRectangle`: 转换这个 `Rect` 成一个 `Rectangle` 来自火焰几何。
 - `transform`: 转换 `Rect` 用一个 `矩阵 4`.

工厂：
 - `RectExtension.getBounds`: 构造一个 `Rect` 表示列表的边界 `Vector2`s。
 - `RectExtension.fromCenter`: 构造一个 `Rect` 从中心点（使用 `Vector2`）。

### math.Rectangle

方法：
 - `toRect`: 转换这个数学 `Rectangle` 进入一个用户界面 `Rect`.

### Size

方法：
 - `toVector2`;创建一个 `Vector2` 来自 `Size`.
 - `toOffset`: 创建一个 `Offset` 来自 `Size`.
 - `toPoint`: 创建一个 `Point` 来自 `Size`.
 - `toRect`: 创建一个 `Rect` 从 (0,0) 开始，大小为 `Size`.

### Vector2

这个类来自 `vector_math` 包，我们在该包提供的基础上提供了一些有用的扩展方法。

方法：
 - `toOffset`: 创建一个 `Offset` 来自 `Vector2`.
 - `toPoint`: 创建一个 `Point` 来自 `Vector2`.
 - `toRect`: 创建一个 `Rect` 从 (0,0) 开始，大小为 `Vector2`.
 - `toPositionedRect`: 创建一个 `Rect` 从 [x, y] 开始 `Vector2` 并且大小为
  这 `Vector2` 争论。
 - `lerp`：线性插值 `Vector2` 朝向另一个 Vector2。
 - `rotate`: 旋转 `Vector2` 以弧度指定的角度，它围绕
  可选定义 `Vector2`，否则围绕中心。
 - `scaleTo`: 改变长度 `Vector2` 到提供的长度，不改变
  方向。
 - `moveToTarget`：将 Vector2 沿目标方向平滑移动给定距离。

工厂：
 - `Vector2Extension.fromInts`： 创建一个 `Vector2` 以整数作为输入。

运营商：
 - `&`: 结合两个 `Vector2`s 组成一个 Rect，原点在左边，大小在
  正确的。
 - `%`: x 和 y 的模/余数分别为两个 `Vector2`s。

### Matrix4

这个类来自 `vector_math` 包裹。我们在已经提供的基础上创建了一些扩展方法 `vector_math`.

方法：
  - `translate2`： 翻译 `Matrix4` 由给定的 `Vector2`.
  - `transform2`： 创建一个新的 `Vector2` 通过转换给定的 `Vector2` 使用 `Matrix4`.
  - `transformed2`: 转换输入 `Vector2` 进入输出 `Vector2`.

吸气剂：
  - `m11`: 第一行第一列。
  - `m12`: 第一行第二列。
  - `m13`: 第一行第三列。
  - `m14`: 第一行第四列。
  - `m21`: 第二行第一列。
  - `m22`: 第二行第二列。
  - `m23`: 第二行第三列。
  - `m24`: 第二行第四列。
  - `m31`: 第三行第一列。
  - `m32`: 第三行第二列。
  - `m33`：第三行第三列。
  - `m34`: 第三行第四列。
  - `m41`: 第四行第一列。
  - `m42`: 第四行第二列。
  - `m43`: 第四行第三列。
  - `m44`: 第四行第四列。

工厂：
 - `Matrix4Extension.scale`: 创建一个缩放 `Matrix4`.要么通过 `Vector4` 要么 `Vector2` 因为它是第一个
    参数，或通过 xyz 双打。
