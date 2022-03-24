# 图片

首先，你必须具有适当的文件夹结构并将文件添加到 `pubspec.yaml` 文件，像这样：

```yaml
flutter:
  assets:
    - assets/images/player.png
    - assets/images/enemy.png
```

图像可以是 Flutter 支持的任何格式，包括：JPEG、WebP、PNG、GIF、动画 GIF、动画 WebP、BMP 和 WBMP。其他格式需要额外的库。例如，SVG 图像可以通过 `flame_svg` 图书馆。


## 加载图像

Flame 捆绑了一个实用程序类，称为 `Images` 这使你可以轻松地将资产目录中的图像加载和缓存到内存中。

Flutter 有一些与图像相关的类型，并将所有内容从本地资产正确转换为 `Image` 可以在 Canvas 上绘制的内容有点复杂。这个类允许你获得一个 `Image` 可以画在 `Canvas` 使用 `drawImageRect` 方法。

它会自动缓存按文件名加载的任何图像，因此你可以安全地多次调用它。

加载和清除缓存的方法有：`load` ,`loadAll` ,`clear` 和 `clearCache`.他们回来了 `Future`s 用于加载图像。在以任何方式使用图像之前，必须等待这些未来。如果你不想立即等待这些期货，你可以启动多个 `load()` 操作，然后使用一次等待所有操作 `Images.ready()` 方法。

要同步检索以前缓存的图像，`fromCache` 可以使用方法。如果之前未加载具有该键的图像，它将引发异常。

要将已加载的图像添加到缓存中，`add` 可以使用方法，你可以设置图像在缓存中应具有的键。

你也可以使用 `ImageExtension.fromPixels()` 在游戏过程中动态创建图像。

为了 `clear` 和 `clearCache`，请注意 `dispose` 为从缓存中删除的每个图像调用，因此请确保以后不要使用该图像。

### 独立使用

它可以通过实例化来手动使用：

```dart
import 'package:flame/images.dart';
final imagesLoader = Images();
Image image = await imagesLoader.load('yourImage.png');
```

但是 Flame 也提供了两种使用这个类的方法，而无需自己实例化它。

### 火焰图像

有一个单例，由 `Flame` 类，可以用作全局图像缓存。

例子：

```dart
import 'package:flame/flame.dart';
import 'package:flame/sprite.dart';

// inside an async context
Image image = await Flame.images.load('player.png');

final playerSprite = Sprite(image);
```

### 游戏图像

这 `Game` 类也提供了一些处理图像加载的实用方法。它捆绑了一个实例 `Images` 类，可用于加载要在游戏期间使用的图像资产。当游戏小部件从小部件树中删除时，游戏将自动释放缓存。

这 `onLoad` 方法从 `Game`class 是加载初始资产的好地方。

例子：

```dart
class MyGame extends Game {

  Sprite player;

  @override
  Future<void> onLoad() async {
    final playerImage = await images.load('player.png'); // Note that you could also use Sprite.load for this
    player = Sprite(playerImage);
  }
}
```

加载的资产也可以在游戏运行时检索 `images.fromCache`， 例如：

```dart
class MyGame extends Game {

  // attributes omitted

  @override
  Future<void> onLoad() async {
    // other loads omitted
    await images.load('bullet.png');
  }

  void shoot() {
    // This is just an example, in your game you probably don't want to instantiate new [Sprite]
    // objects every time you shoot.
    final bulletSprite = Sprite(images.fromCache('bullet.png'));
    _bullets.add(bulletSprite);
  }
}
```

## 雪碧

火焰提供了一个 `Sprite` 表示图像或图像区域的类。

你可以创建一个 `Sprite` 通过提供它 `Image` 以及定义该精灵所代表的图像片段的坐标。

例如，这将创建一个 sprite 表示传递的文件的整个图像：

```dart
final image = await images.load('player.png');
Sprite player = Sprite(image);
```

你还可以指定精灵所在的原始图像中的坐标。这允许你使用精灵表并减少内存中的图像数量，例如：

```dart
final image = await images.load('player.png');
final playerFrame = Sprite(
  image,
  srcPosition: Vector2(32.0, 0),
  srcSize: Vector2(16.0, 16.0),
);
```

默认值为 `(0.0, 0.0)` 为了 `srcPosition` 和 `null` 为了 `srcSize`（意味着它将使用源图像的全宽/高度）。

这 `Sprite` 类有一个渲染方法，允许你将精灵渲染到 `Canvas`：

```dart
final image = await images.load('block.png');
Sprite block = Sprite(image);

// in your render method
block.render(canvas, 16.0, 16.0); //canvas, width, height
```

你必须将大小传递给 render 方法，图像将相应地调整大小。

来自的所有渲染方法 `Sprite` 班级可以收到一个 `Paint` 实例作为可选的命名参数 `overridePaint` 该参数将覆盖当前 `Sprite` 该渲染调用的绘制实例。

`Sprite`s 也可以用作小部件，只需使用 `SpriteWidget` 班级。

可以找到使用 sprite 作为小部件的完整示例 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/widget/sprite_widget.dart).

## 精灵批处理

如果你有一个精灵表（也称为图像图集，这是一个内部包含较小图像的图像），并且想要有效地渲染它 -`SpriteBatch` 为你处理这项工作。

给它图像的文件名，然后添加描述图像各个部分的矩形，除了变换（位置、缩放和旋转）和可选颜色。

你用一个渲染它 `Canvas` 和一个可选的 `Paint`,`BlendMode` 和 `CullRect`.

一种 `SpriteBatchComponent` 也为你提供方便。

查看示例 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/sprites/spritebatch.dart).

## 图像合成

在某些情况下，你可能希望将多个图像合并为一个图像；这就是所谓的 [Compositing](https://en.wikipedia.org/wiki/Compositing).例如，在使用 [SpriteBatch](#spritebatch) 用于优化绘图调用的 API。

对于这样的用例，Flame 带有 `ImageComposition` 班级。这允许你将多个图像添加到新图像上，每个图像都在自己的位置：

```dart
final composition = ImageComposition()
  ..add(image1, Vector2(0, 0))
  ..add(image2, Vector2(64, 0));
  ..add(image3,
    Vector2(128, 0),
    source: Rect.fromLTWH(32, 32, 64, 64),
  );

Image image = await composition.compose();
```

**笔记：** 合成图像很昂贵，我们不建议你每次运行都运行它，因为它会严重影响性能。相反，我们建议你预先渲染你的作品，以便你可以重复使用输出图像。

## SVG

Flame 提供了一个简单的 API 来在你的游戏中渲染 SVG 图像。

Svg 支持由 `flame_svg` 外部包，请务必将其放入你的 pubspec 文件中以使用它。

要使用它，只需导入 `Svg` 类从 `'package:flame_svg/flame_svg.dart'`，并使用以下代码段在画布上呈现它：

```dart
Svg svgInstance = Svg('android.svg');

final position = Vector2(100, 100);
final size = Vector2(300, 300);

svgInstance.renderPosition(canvas, position, size);
```

或使用 [SvgComponent]：

```dart
class MyGame extends FlameGame {
    Future<void> onLoad() async {
      final svgInstance = await Svg.load('android.svg');
      final size = Vector2.all(100);
      final svgComponent = SvgComponent.fromSvg(size, svgInstance);
      svgComponent.x = 100;
      svgComponent.y = 100;

      add(svgComponent);
    }
}
```

## 动画

Animation 类可帮助你创建精灵的循环动画。

你可以通过传递相同大小的精灵列表和 stepTime（即移动到下一帧所需的秒数）来创建它：

```dart
final a = SpriteAnimation.spriteList(sprites, stepTime: 0.02);
```

创建动画后，需要调用它的 `update` 方法并在你的游戏实例上渲染当前帧的精灵。

例子：

```dart
class MyGame extends Game {
  SpriteAnimation a;

  MyGame() {
    a = SpriteAnimation(...);
  }

  void update(double dt) {
    a.update(dt);
  }

  void render(Canvas c) {
    a.getSprite().render(c);
  }
}
```

生成精灵列表的更好选择是使用 `fromFrameData` 构造函数：

```dart
const amountOfFrames = 8;
final a = SpriteAnimation.fromFrameData(
    imageInstance,
    SpriteAnimationFrame.sequenced(
      amount: amountOfFrames,
      textureSize: Vector2(16.0, 16.0),
      stepTime: 0.1,
    ),
);
```

这个构造函数使得创建一个 `Animation` 使用精灵表时非常容易。

在构造函数中，你传递一个图像实例和帧数据，其中包含一些可用于描述动画的参数。检查有关可用构造函数的文档 `SpriteAnimationFrameData` 类查看所有参数。

如果你在动画中使用 Aseprite，Flame 确实为 Aseprite 动画的 JSON 数据提供了一些支持。要使用此功能，你需要导出 Sprite Sheet 的 JSON 数据，并使用类似于以下代码段的内容：

```dart
final image = await images.load('chopper.png');
final jsonData = await assets.readJson('chopper.json');
final animation = SpriteAnimation.fromAsepriteData(image, jsonData);
```

**笔记：** 火焰不支持修剪的精灵表，所以如果你将精灵表导出到这个 kway，它将具有修剪后的大小，而不是精灵的原始大小。

动画，在创建后，有一个更新和渲染方法；后者渲染当前帧，前者滴答内部时钟以更新帧。

动画通常在内部使用 `SpriteAnimationComponent`s，但也可以创建具有多个动画的自定义组件。

可以找到使用动画作为小部件的完整示例 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/widgets/sprite_animation_widget.dart).

## FlareAnimation

Flame 提供了一个简单的封装 [Flare](https://rive.app/#LearnMore) 动画，以便你可以在火焰游戏中使用它们。

检查以下有关如何使用此包装器的代码段：

```dart
class MyGame extends Game {
  FlareAnimation flareAnimation;
  bool loaded = false;

  MyGame() {
    _start();
  }

  void _start() async {
    flareAnimation = await FlareAnimation.load("assets/FLARE_FILE.flr");
    flareAnimation.updateAnimation("ANIMATION_NAME");

    flareAnimation.width = 306;
    flareAnimation.height = 228;

    loaded = true;
  }

  @override
  void render(Canvas canvas) {
    if (loaded) {
      flareAnimation.render(canvas, x: 50, y: 50);
    }
  }

  @override
  void update(double dt) {
    if (loaded) {
      flareAnimation.update(dt);
    }
  }
}
```

FlareAnimations 通常在内部使用 `FlareComponent`s，那样 `FlameGame` 将处理调用 `render` 和 `update` 自动地。

你可以在示例中看到如何将 Flare 与 Flame 一起使用的完整示例 [here](https://github.com/flame-engine/flame_flare/tree/main/example).

## 精灵表

精灵表是大图像，上面有几帧相同的精灵，是组织和保存动画的好方法。Flame 提供了一个非常简单的实用程序类来处理 SpriteSheets，你可以使用它加载你的精灵表图像并从中提取动画。下面是一个非常简单的例子来说明如何使用它：

```dart
import 'package:flame/sprite.dart';

final spritesheet = SpriteSheet(
  image: imageInstance,
  srcSize: Vector2.all(16.0),
);

final animation = spritesheet.createAnimation(0, stepTime: 0.1);
```

现在你可以直接使用动画或在动画组件中使用它。

你还可以使用 `getSprite` 方法：

```dart
spritesheet.getSprite(0, 0) // row, column;
```

你可以看到一个完整的例子 `SpriteSheet` 班级 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/sprites/spritesheet.dart).
