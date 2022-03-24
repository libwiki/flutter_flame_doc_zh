# 粒子

Flame 提供了一个基本但强大且可扩展的粒子系统。该系统的核心概念是 `Particle` 类，其行为与 `ParticleComponent`.

a 的最基本用法 `Particle` 和 `FlameGame` 如下所示：

```dart
import 'package:flame/components.dart';

// ... 

game.add(
  // Wrapping a Particle with ParticleComponent
  // which maps Component lifecycle hooks to Particle ones
  // and embeds a trigger for removing the component.
  ParticleComponent(
    CircleParticle(),
  ),
);
```

使用时 `Particle` 有一个习惯 `Game` 实施，请确保 `update` 和 `render` 在每个游戏循环滴答期间调用方法。

实现所需粒子效果的主要方法：
* 现有行为的组成。
* 使用行为链（只是第一个的语法糖）。
* 使用 `计算粒子`.

组合的工作方式与 Flutter 小部件的工作方式相似，通过定义从上到下的效果。链接允许通过定义从下到上的行为更流畅地表达相同的组合树。计算粒子又将行为的实现完全委托给你的代码。任何方法都可以在需要时与现有行为结合使用。

```dart
Random rnd = Random();

Vector2 randomVector2() => (Vector2.random(rnd) - Vector2.random(rnd)) * 200;

// Composition.
// 
// Defining a particle effect as a set of nested behaviors from top to bottom, one within another: 
// ParticleComponent 
//   > ComposedParticle 
//     > AcceleratedParticle 
//       > CircleParticle
game.add(
  ParticleComponent(
    Particle.generate(
      count: 10,
      generator: (i) => AcceleratedParticle(
        acceleration: randomVector2(),
        child: CircleParticle(
          paint: Paint()..color = Colors.red,
        ),
      ),
    ),
  ),
);

// Chaining.
// 
// Expresses the same behavior as above, but with a more fluent API.
// Only Particles with SingleChildParticle mixin can be used as chainable behaviors.
game.add(
  ParticleComponent(
    Particle.generate(
      count: 10,
      generator: (i) => pt.CircleParticle(paint: Paint()..color = Colors.red)
    )
  )
);

// Computed Particle.
// 
// All the behaviors are defined explicitly. Offers greater flexibility
// compared to built-in behaviors.
game.add(
  ParticleComponent(
      Particle.generate(
        count: 10,
        generator: (i) {
          Vector2 position = Vector2.zero();
          Vector2 speed = Vector2.zero();
          final acceleration = randomVector2();
          final paint = Paint()..color = Colors.red;
          
          return ComputedParticle(
            renderer: (canvas, _) {
              speed += acceleration;
              position += speed;
              canvas.drawCircle(Offset(position.x, position.y), 1, paint);
            }
        );
      }
    )
  )
);
```

你可以找到更多关于如何在各种组合中使用不同内置粒子的示例 [here](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/rendering/particles_example.dart).


## 生命周期

所有人共同的行为 `Particle`s 是他们都接受一个 `lifespan` 争论。该值用于使 `ParticleComponent` 一旦其内部删除自己 `Particle` 已经到了生命的尽头。时间范围内 `Particle` 本身是使用火焰跟踪的 `Timer` 班级。它可以配置一个 `double`，以秒表示（精度为微秒），将其传递给相应的 `Particle` 构造函数。

```dart
Particle(lifespan: .2); // will live for 200ms.
Particle(lifespan: 4); // will live for 4s.
```

也可以重置一个 `Particle` 的使用寿命通过使用 `setLifespan` 方法，它也接受一个 `double` 秒。

```dart
final particle = Particle(lifespan: 2);

// ... after some time.
particle.setLifespan(2) // will live for another 2s.
```

在其生命周期中，一个 `Particle` 跟踪它的存在时间并通过 `progress`getter，它返回一个介于 `0.0` 和 `1.0`.该值的使用方式与 `value` 的财产 `AnimationController`Flutter 中的类。

```dart
final particle = Particle(lifespan: 2.0);

game.add(ParticleComponent(particle));

// Will print values from 0 to 1 with step of .1: 0, 0.1, 0.2 ... 0.9, 1.0.
Timer.periodic(duration * .1, () => print(particle.progress));
```

这 `lifespan` 被传递给给定的所有后代 `Particle`，如果它支持任何嵌套行为。

## 内置粒子

带有一些内置的火焰船 `Particle` 行为：
* 这 `TranslatedParticle` 翻译它的 `child` 由给定 `Vector2`
* 这 `运动粒子` 移动它的 `child` 两个预定义之间 `Vector2`, 支持 `Curve`
* 这 `加速粒子` 允许基于物理的基本效果，例如重力或速度阻尼
* 这 `圆形粒子` 呈现各种形状和大小的圆圈
* 这 `精灵粒子` 渲染火焰 `Sprite` 在一个 `Particle` 影响
* 这 `图像粒子` 渲染 *飞镖：ui*`Image` 在一个 `Particle` 影响
* 这 `组件粒子` 渲染火焰 `Component` 在一个 `Particle` 影响
* 这 `耀斑粒子` 在 a 中渲染 Flare 动画 `Particle` 影响

提供了更多关于如何一起使用这些行为的示例 [here](https://github.com/flame-engine/flame/blob/main/examples/lib/stories/utils/particles.dart).所有的实现都可以在 [particles](https://github.com/flame-engine/flame/tree/main/packages/flame/lib/src/particles)Flame 存储库中的文件夹。

简单地翻译底层 `Particle` 到指定的 `Vector2` 渲染内 `Canvas`.不改变或改变其位置，考虑使用 `MovingParticle` 要么 `AcceleratedParticle` 需要改变位置的地方。

简单地翻译底层 `Particle` 到指定的 `Vector2` 渲染内 `Canvas`.不改变或改变其位置，考虑使用 `MovingParticle` 要么 `AcceleratedParticle` 需要改变位置的地方。翻译过来也可以达到同样的效果 `Canvas` 层。

```dart
game.add(
  ParticleComponent(
    TranslatedParticle(
      // Will translate the child Particle effect to the center of game canvas.
      offset: game.size / 2,
      child: Particle(),
    ),
  ),
);
```

## MovingParticle

移动孩子 `Particle` 在。。之间 `from` 和 `to``Vector2`s 在其生命周期内。支持 `Curve` 通过 `CurvedParticle`.

```dart
game.add(
  ParticleComponent(
    MovingParticle(
      // Will move from corner to corner of the game canvas.
      from: Vector2.zero(),
      to: game.size,
      child: CircleParticle(
        radius: 2.0,
        paint: Paint()..color = Colors.red,
      ),
    ),
  ),
);
```

## AcceleratedParticle

一个基本的物理粒子，允许你指定它的初始 `position`,`speed` 和 `acceleration` 并让 `update` 循环做剩下的。所有三个都指定为 `Vector2`s，你可以将其视为向量。它特别适用于基于物理的“爆发”，但不限于此。单位 `Vector2` 值为 _逻辑像素/秒_.所以速度为 `Vector2(0, 100)` 会动一个孩子 `Particle` 游戏时间每秒通过设备的 100 个逻辑像素。

```dart
final rnd = Random();
Vector2 randomVector2() => (Vector2.random(rnd) - Vector2.random(rnd)) * 100;

game.add(
  ParticleComponent(
    AcceleratedParticle(
      // Will fire off in the center of game canvas
      position: game.canvasSize/2,
      // With random initial speed of Vector2(-100..100, 0..-100)
      speed: Vector2(rnd.nextDouble() * 200 - 100, -rnd.nextDouble() * 100),
      // Accelerating downwards, simulating "gravity"
      // speed: Vector2(0, 100),
      child: CircleParticle(
        radius: 2.0,
        paint: Paint()..color = Colors.red,
      ),
    ),
  ),
);
```

## CircleParticle

一种 `Particle` 它呈现一个给定的圆圈 `Paint` 在通过的零偏移量处 `Canvas`.配合使用 `TranslatedParticle`,`MovingParticle` 要么 `AcceleratedParticle` 以达到理想的定位。

```dart
game.add(
  ParticleComponent(
    CircleParticle(
      radius: game.size.x / 2,
      paint: Paint()..color = Colors.red.withOpacity(.5),
    ),
  ),
);
```

## SpriteParticle

允许你嵌入 `Sprite` 进入你的粒子效果。

```dart
game.add(
  ParticleComponent(
    SpriteParticle(
      sprite: Sprite('sprite.png'),
      size: Vector2(64, 64),
    ),
  ),
);
```

## ImageParticle

给定的渲染 `dart:ui` 粒子树中的图像。

```dart
// During game initialisation
await Flame.images.loadAll(const [
  'image.png',
]);

// ...

// Somewhere during the game loop
final image = await Flame.images.load('image.png');

game.add(
  ParticleComponent(
    ImageParticle(
      size: Vector2.all(24),
      image: image,
    );
  ),
);
```

## 动画粒子

一种 `Particle` 其中嵌入了一个 `Animation`.默认情况下，对齐 `Animation` 的 `stepTime` 以便在播放期间充分发挥 `Particle` 寿命。可以用 `alignAnimationTime` 争论。

```dart
final spritesheet = SpriteSheet(
  image: yourSpriteSheetImage,
  srcSize: Vector2.all(16.0),
);

game.add(
  ParticleComponent(
    AnimationParticle(
      animation: spritesheet.createAnimation(0, stepTime: 0.1),
    );
  ),
);
```

## ComponentParticle

这 `Particle` 允许你嵌入 `Component` 在粒子效果中。这 `Component` 可以有自己的 `update` 生命周期，并且可以在不同的效果树中重复使用。如果你唯一需要的是为某个实例添加一些动态 `Component`，请考虑将其添加到 `game` 直接，没有 `Particle` 在中间。

```dart
final longLivingRect = RectComponent();

game.add(
  ParticleComponent(
    ComponentParticle(
      component: longLivingRect
    );
  ),
);

class RectComponent extends Component {
  void render(Canvas c) {
    c.drawRect(
      Rect.fromCenter(center: Offset.zero, width: 100, height: 100), 
      Paint()..color = Colors.red
    );
  }

  void update(double dt) {
    /// Will be called by parent [Particle]
  }
}
```

## FlareParticle

要在 Flame 中使用 Flare，请使用 [`flame_flare`](https://github.com/flame-engine/flame_flare) 包裹。

它将提供一个名为 `FlareParticle` 这是一个容器 `FlareActorAnimation`，它传播 `update` 和 `render` 方法给它的孩子。

```dart
import 'package:flame_flare/flame_flare.dart';

// Within your game or component's `onLoad` method
const flareSize = 32.0;
final flareAnimation = FlareActorAnimation('assets/sparkle.flr');
flareAnimation.width = flareSize; 
flareAnimation.height = flareSize;

// Somewhere in game
game.add(
  ParticleComponent(
    FlareParticle(flare: flareAnimation),
  ),
);
```

## ComputedParticle

一种 `Particle` 在以下情况下可以为你提供帮助：
* 默认行为是不够的
* 复杂效果优化
* 自定义缓动

创建后，它将所有渲染委托给提供的 `ParticleRenderDelegate` 在每一帧上调用它来执行必要的计算并向 `Canvas`.

```dart
game.add(
  ParticleComponent(
    // Renders a circle which gradually changes its color and size during the particle lifespan.
    ComputedParticle(
      renderer: (canvas, particle) => canvas.drawCircle(
        Offset.zero,
        particle.progress * 10,
        Paint()
          ..color = Color.lerp(
            Colors.red,
            Colors.blue,
            particle.progress,
          ),
      ),
    ),
  ),
)
```

## 嵌套行为

Flame 的粒子实现遵循与 Flutter 小部件相同的极端组合模式。这是通过在每个粒子中封装小块行为，然后将这些行为嵌套在一起以实现所需的视觉效果来实现的。

允许的两个实体 `Particle`s 相互嵌套是：`SingleChildParticle` 混合和 `ComposedParticle` 班级。

一种 `SingleChildParticle` 可以帮助你创建 `Particles` 具有自定义行为。例如，在每一帧中随机定位其子节点：

这 `SingleChildParticle` 可以帮助你创建 `Particles` 具有自定义行为。

例如，在每一帧中随机定位它的孩子：

```dart
var rnd = Random();

class GlitchParticle extends Particle with SingleChildParticle {
  @override
  Particle child;

  GlitchParticle({
    @required this.child,
    double lifespan,
  }) : super(lifespan: lifespan);

  @override
  render(Canvas canvas)  {
    canvas.save();
    canvas.translate(rnd.nextDouble() * 100, rnd.nextDouble() * 100);

    // Will also render the child
    super.render();

    canvas.restore();
  }
}
```

这 `ComposedParticle` 既可以作为独立的，也可以在现有的 `Particle` 树。
