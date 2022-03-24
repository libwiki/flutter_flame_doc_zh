# 氧

我们（Flame 组织）构建了一个名为 Oxygen 的 ECS（实体组件系统）。

如果你想使用专门用于 Flame 的 Oxygen 作为 FCS（火焰组件系统）的替代品，你应该使用我们的桥接库 [flame_oxygen](https://github.com/flame-engine/flame/tree/main/packages/flame_oxygen) 如果你只想在 Dart 项目中使用它，你可以使用 [oxygen](https://github.com/flame-engine/oxygen) 图书馆直接。

如果你不熟悉 Oxygen，我们建议你阅读它的 [documentation](https://github.com/flame-engine/oxygen/tree/main/doc).

要在你的游戏中使用它，你只需添加 `flame_oxygen` 给你的 `pubspec.yaml`, 可以在 [Oxygen example](https://github.com/flame-engine/flame/tree/main/packages/flame_oxygen/example) 并且在 `pub.dev`[安装说明](https://pub.dev/packages/flame_oxygen).

## OxygenGame（游戏扩展）

如果你要在项目中使用 Oxygen，最好使用 Oxygen 的特定扩展 `Game` 班级。

它被称为 `OxygenGame` 它可以让你完全访问 Oxygen 框架，同时还可以完全访问 Flame 游戏循环。

而不是使用 `onLoad`，就像你习惯使用火焰一样，`OxygenGame` 附带 `init` 方法。该方法在 `onLoad` 但在世界初始化之前，允许你注册组件和系统并执行你通常会执行的任何其他操作 `onLoad`.

一个简单的 `OxygenGame` 实现示例可以在 [example folder](https://github.com/flame-engine/flame/tree/main/packages/flame_oxygen/example).

这 `OxygenGame` 还自带 `createEntity` 自动在实体上添加某些默认组件的方法。这在你使用 [基本系统](#basesystem) 作为你的基地。

## 系统

系统定义了游戏的逻辑。在 FCS 中，你通常会将你的逻辑添加到带有 Oxygen 的组件中，我们为此使用系统。Oxygen 本身完全与平台无关，这意味着它没有渲染循环。它只知道 `execute`, 这是一种等于 `update` 火焰中的方法。

在各个 `execute`Oxygen 会自动调用按顺序注册的所有系统。但是在 Flame 中，我们可以为不同的循环（渲染/更新）设置不同的逻辑。所以在 `flame_oxygen` 我们介绍了 `RenderSystem` 和 `UpdateSystem` 混音。这些 mixin 允许你添加 `render` 方法和 `update` 方法分别到你的自定义系统。有关更多信息，请参阅 [RenderSystem](#mixin-rendersystem) 和 [UpdateSystem](#mixin-updatesystem) 部分。

如果你来自 FCS，你可能会期望你通常从 `位置组件`.如前所述，组件不包含任何类型的逻辑，但为了给你相同的默认功能，我们还创建了一个名为 `BaseSystem`.这个系统的行为几乎与之前的预渲染逻辑相同。`PositionComponent` 在 FCS。你只需要将其子类化到你自己的系统。有关更多信息，请参阅 [BaseSystem](#basesystem) 部分。

系统可以使用 `world.registerSystem` 方法 [OxygenGame](#oxygengame-game-extension).

### 混入 GameRef

这 `GameRef`mixin 允许系统意识到 `OxygenGame` 实例它附加到。这允许轻松访问游戏类上的方法。

```dart
class YourSystem extends System with GameRef<YourGame> {
  @override
  void init() {
    // Access to game using the .game propery
  }

  // ...
}
```

### 混合渲染系统

这 `RenderSystem`mixin 允许为渲染循环注册系统。通过添加一个 `render` 系统的方法，你可以像在 Flame 中一样完全访问画布。

```dart
class SimpleRenderSystem extends System with RenderSystem {
  Query? _query;

  @override
  void init() {
    _query = createQuery([/* Your filters */]);
  }

  void render(Canvas canvas) {
    for (final entity in _query?.entities ?? <Entity>[]) {
      // Render entity based on components
    }
  }
}
```

### mixin 更新系统

这 `MixinSystem`mixin 允许为更新循环注册系统。通过添加一个 `update` 系统的方法，你可以像在 Flame 中一样完全访问增量时间。

```dart
class SimpleUpdateSystem extends System with UpdateSystem {
  Query? _query;

  @override
  void init() {
    _query = createQuery([/* Your filters */]);
  }

  void update(double dt) {
    for (final entity in _query?.entities ?? <Entity>[]) {
      // Update components values
    }
  }
}
```

### BaseSystem

这 `BaseSystem` 是一个抽象类，其逻辑可以与 `PositionComponent` 来自 FCS。这 `BaseSystem` 自动过滤所有具有 `PositionComponent` 和 `尺寸组件` 从 `flame_oxygen`.最重要的是，你可以通过定义一个名为 `filters`.然后使用这些过滤器来过滤你感兴趣的实体。

这 `BaseSystem` 也完全了解游戏实例。你可以使用以下方式访问游戏实例 `game` 财产。这也使你可以访问 `createEntity` 辅助方法 `OxygenGame`.

在每个渲染循环上 `BaseSystem` 将以相同的方式准备你的画布 `PositionComponent` 从 FCS 将（平移、旋转和设置锚点。之后它将调用 `renderEntity` 方法，以便你可以在准备好的画布上为该实体添加自己的渲染逻辑。

以下组件将被检查 `BaseSystem` 准备画布：
- `PositionComponent`（必需的）
- `SizeComponent`（必需的）
- `锚组件`（可选，默认为 `Anchor.topLeft`)
- `角度分量`（可选，默认为 `0`)

```dart
class SimpleBaseSystem extends BaseSystem {
  @override
  List<Filter<Component>> get filters => [];

  @override
  void renderEntity(Canvas canvas, Entity entity) {
    // The canvas is translated, rotated and fully prepared for rendering.
  }
}
```

### 粒子系统

这 `ParticleSystem` 是一个简单的系统，将粒子 API 从火焰带到氧气。这允许你使用 [粒子组件](#particlecomponent) 无需担心它将如何呈现或何时更新它。因为大部分逻辑已经包含在粒子本身中。

只需注册 `ParticleSystem` 和 `粒子组件` 像这样对你的世界：

```dart
world.registerSystem(ParticleSystem());

world.registerComponent<ParticleComponent, Particle>(() => ParticleComponent);
```

你现在可以使用 `ParticleComponent`.有关这方面的更多信息，请参阅 [粒子组件](#particlecomponent) 部分。

## 成分

Oxygen 中的组件与 FCS 中的组件不同，主要是因为它们不包含逻辑，而是只包含数据。这些数据随后被用于定义逻辑的系统中。

为了适应从 FCS 切换到 Oxygen 的人们，我们实施了一些组件来帮助你入门。其中一些组件基于 `PositionComponent` 从 FCS 有。其他的只是围绕某些 Flame API 功能的简单包装，它们通常伴随着你可以使用的预定义系统。

组件可以使用 `world.registerComponent` 方法 [OxygenGame](#oxygengame-game-extension).

### PositionComponent

这 `PositionComponent` 顾名思义是描述实体位置的组件。它默认注册到世界。

使用 OxygenGame 创建定位实体：

```dart
game.createEntity(
  position: Vector2(100, 100),
  size: // ...
);
```

使用 World 创建定位实体：

```dart
world.createEntity()
  ..add<PositionComponent, Vector2>(Vector2(100, 100));
```

### SizeComponent

这 `SizeComponent` 顾名思义是描述实体大小的组件。它默认注册到世界。

使用 OxygenGame 创建一个大小的实体：

```dart
game.createEntity(
  position: // ...
  size: Vector2(50, 50),
);
```

使用 World 创建一个大小的实体：

```dart
world.createEntity()
  ..add<SizeComponent, Vector2>(Vector2(50, 50));
```

### AnchorComponent

这 `AnchorComponent`is 顾名思义是一个描述实体锚点位置的组件。它默认注册到世界。

当你使用 [BaseSystem](#basesystem).但也可以用于你自己的锚定逻辑。

使用 OxygenGame 创建锚定实体：

```dart
game.createEntity(
  position: // ...
  size: // ...
  anchor: Anchor.center,
);
```

使用 World 创建一个锚定实体：

```dart
world.createEntity()
  ..add<AnchorComponent, Anchor>(Anchor.center);
```

### AngleComponent

这 `AngleComponent` 顾名思义是描述实体角度的组件。它默认注册到世界。角度以弧度为单位。

当你使用 [BaseSystem](#basesystem).但也可以用于你自己的角度逻辑。

使用 OxygenGame 创建一个有角度的实体：

```dart
game.createEntity(
  position: // ...
  size: // ...
  angle: 1.570796,
);
```

使用 World 创建有角度的实体：

```dart
world.createEntity()
  ..add<AngleComponent, double>(1.570796);
```

### 翻转组件

这 `FlipComponent` 可用于在 X 或 Y 轴上翻转渲染。它默认注册到世界。

当你使用 [BaseSystem](#basesystem).但也可以用于你自己的翻转逻辑。

使用 OxygenGame 创建一个在其 X 轴上翻转的实体：

```dart
game.createEntity(
  position: // ...
  size: // ...
  flipX: true
);
```

使用 World 创建一个在其 X 轴上翻转的实体：

```dart
world.createEntity()
  ..add<FlipComponent, FlipInit>(FlipInit(flipX: true));
```

### 精灵组件

这 `SpriteComponent` 顾名思义是描述实体精灵的组件。它默认注册到世界。

这允许你将 Sprite 分配给实体。

使用 OxygenGame 创建带有精灵的实体：

```dart
game.createEntity(
  position: // ...
  size: // ...
)..add<SpriteComponent, SpriteInit>(
  SpriteInit(await game.loadSprite('pizza.png')),
);
```

使用 World 创建一个带有精灵的实体：

```dart
world.createEntity()
  ..add<SpriteComponent, SpriteInit>(
    SpriteInit(await game.loadSprite('pizza.png')),
  );
```

### 文本组件

这 `TextComponent`is 顾名思义就是为实体添加文本组件的组件。它默认注册到世界。

这允许你向实体添加文本，并结合 `PositionComponent` 你可以将其用作文本实体。

使用 OxygenGame 创建带有文本的实体：

```dart
game.createEntity(
  position: // ...
  size: // ...
)..add<TextComponent, TextInit>(
  TextInit(
    'Your text',
    config: const TextPaintConfig(),
  ),
);
```

使用 World 创建带有文本的实体：

```dart
world.createEntity()
  ..add<TextComponent, TextInit>(
    TextInit(
      'Your text',
      config: const TextPaintConfig(),
    ),
  );
```

### ParticleComponent

这 `ParticleComponent` 包裹一个 `Particle` 来自火焰。结合 [ParticleSystem](#particlesystem) 你可以轻松地将粒子添加到游戏中，而不必担心如何渲染粒子或何时/如何更新粒子。

使用 OxygenGame 创建一个带有粒子的实体：

```dart
game.createEntity(
  position: // ...
  size: // ...
)..add<ParticleComponent, Particle>(
  // Your Particle.
);
```

使用 World 创建带有粒子的实体：

```dart
world.createEntity()
  ..add<ParticleComponent, Particle>(
    // Your Particle.
  );
```
