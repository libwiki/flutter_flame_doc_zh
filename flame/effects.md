# 效果

效果是一种特殊的组件，可以附加到另一个组件以修改其属性或外观。

例如，假设你正在制作一款带有可收藏道具的游戏。你希望这些道具在地图周围随机生成，然后在一段时间后消失。显然，你可以为加电制作一个精灵组件，然后将该组件放在地图上，但我们可以做得更好！

让我们添加一个 `ScaleEffect` 首次出现加电时，将项目从 0 增加到 100%。添加另一个无限重复交替 `MoveEffect` 为了使物品稍微上下移动。然后添加一个 `OpacityEffect` 这将使该项目“闪烁” 3 次，此效果将具有 30 秒的内置延迟，或者你希望加电保持原位的时间。最后，添加一个 `RemoveEffect` 这将在指定时间后自动从游戏树中删除该项目（你可能希望在 `OpacityEffect`）。

如你所见，通过一些简单的效果，我们将一个简单的没有生命的精灵变成了一个更有趣的项目。更重要的是，它并没有增加代码的复杂性：效果一旦添加，就会自动工作，完成后会自动从游戏树中删除。


## 概述

一个函数 `Effect` 是影响某些组件的属性随时间的变化。为了实现这一目标，`Effect` 必须知道属性的初始值、最终值以及随着时间的推移它应该如何发展。初始值通常由效果自动确定，最终值由用户明确提供，随时间的进展由 `EffectController`s。

Flame 提供了多种效果，你还可以 [create your own](#creating-new-effects).包括以下效果：
- [`MoveEffect.by`](#moveeffectby)
- [`MoveEffect.to`](#moveeffectto)
- [`MoveAlongPathEffect`](#movealongpatheffect)
- [`RotateEffect.by`](#rotateeffectby)
- [`RotateEffect.to`](#rotateeffectto)
- [`ScaleEffect.by`](#scaleeffectby)
- [`ScaleEffect.to`](#scaleeffectto)
- [`SizeEffect.by`](#sizeeffectby)
- [`SizeEffect.to`](#sizeeffectto)
- [`OpacityEffect`](#opacityeffect)
- [`颜色效果`](#coloreffect)
- [` 序列效应 `](#sequenceeffect)
- [`RemoveEffect`](#removeeffect)

一个 `EffectController` 是描述效果应如何随时间演变的对象。如果你认为效果的初始值为 0% 进度，最终值为 100% 进度，那么效果控制器的工作就是将“物理”时间（以秒为单位）映射到“逻辑”时间时间，从 0 变为 1。

Flame 框架也提供了多种效果控制器：
- [` 效果控制器 `](#effectcontroller)
- [`LinearEffectController`](#lineareffectcontroller)
- [`ReverseLinearEffectController`](#reverselineareffectcontroller)
- [`CurvedEffectController`](#curvedeffectcontroller)
- [`ReverseCurvedEffectController`](#reversecurvedeffectcontroller)
- [`PauseEffectController`](#pauseeffectcontroller)
- [` 重复效果控制器 `](#repeatedeffectcontroller)
- [` 无限效果控制器 `](#infiniteeffectcontroller)
- [`SequenceEffectController`](#sequenceeffectcontroller)
- [`SpeedEffectController`](#speedeffectcontroller)
- [` 延迟效果控制器 `](#delayedeffectcontroller)
- [`NoiseEffectController`](#noiseeffectcontroller)
- [`RandomEffectController`](#randomeffectcontroller)
- [`SineEffectController`](#sineeffectcontroller)
- [`ZigzagEffectController`](#zigzageffectcontroller)


## 内置效果

### `Effect`

基地 `Effect` 类不能单独使用（它是抽象的），但它提供了一些由所有其他效果继承的通用功能。这包括：

  - 使用暂停/恢复效果的能力 `effect.pause()` 和 `effect.resume()`.你可以检查效果当前是否暂停使用 `effect.isPaused`.

  - 使用反转效果时间方向的能力 `effect.reverse()`.采用 `effect.isReversed` 检查效果当前是否及时运行。

  - 财产 `removeOnFinish`（默认情况下为 true）将导致效果组件从游戏树中移除并在效果完成后进行垃圾收集。如果你打算在效果完成后重用效果，请将其设置为 false。

  - 可选用户提供 `onFinishCallback`, 将在效果刚刚完成执行但在从游戏中移除之前调用。

  - 这 `reset()` 方法将效果恢复到其原始状态，允许它再次运行。


### `MoveEffect.by`

此效果适用于 `PositionComponent` 并按规定移动它 `offset` 数量。这个偏移量是相对于目标的当前位置的：

```dart
final effect = MoveEffect.by(Vector2(0, -10), EffectController(duration: 0.5));
```

如果组件当前位于 `Vector2(250, 200)`, 那么在效果结束时它的位置将是 `Vector2(250, 190)`.

多个移动效果可以同时应用于一个组件。结果将是所有个体效应的叠加。


### `MoveEffect.to`

这个效果会移动一个 `PositionComponent` 从它的当前位置到指定的目的地点是一条直线。

```dart
final effect = MoveEffect.to(Vector2(100, 500), EffectController(duration: 3));
```

可以但不建议将多个此类效果附加到同一个组件。


### `MoveAlongPathEffect`

这个效果会移动一个 `PositionComponent` 沿着相对于组件当前位置的指定路径。路径可以有非线性段，但必须单独连接。建议从路径开始 `Vector2.zero()` 为了避免组件位置的突然跳跃。

```dart
final effect = MoveAlongPathEffect(
  Path() ..quadraticBezierTo(100, 0, 50, -50),
  EffectController(duration: 1.5),
);
```

可选标志 `absolute: true` 将效果内的路径声明为绝对路径。也就是说，目标将在开始时“跳转”到路径的开头，然后沿着该路径走，就好像它是画布上绘制的曲线一样。

另一面旗帜 `oriented: true` 指示目标不仅沿着曲线移动，而且在曲线在每个点所面对的方向上旋转自身。使用此标志，效果同时变为移动和旋转效果。


### `RotateEffect.by`

将目标相对于其当前方向顺时针旋转指定角度。角度以弧度为单位。例如，以下效果将顺时针旋转目标 90º（=[tau]/4 弧度）：

```dart
final effect = RotateEffect.by(tau/4, EffectController(duration: 2));
```


### `RotateEffect.to`

将目标顺时针旋转到指定角度。例如，以下将旋转目标向东看（0º 是北，90º=[tau]/4 东，180º=tau/2 南，270º=tau*3/4 西）：

```dart
final effect = RotateEffect.to(tau/4, EffectController(duration: 2));
```


### `ScaleEffect.by`

此效果将按指定量改变目标的比例。例如，这将导致组件增长 50% 大：

```dart
final effect = ScaleEffect.by(Vector2.all(1.5), EffectController(duration: 0.3));
```


### `ScaleEffect.to`

这种效果类似于 `ScaleEffect.by`, 但设置目标比例的绝对值。

```dart
final effect = ScaleEffect.to(Vector2.zero(), EffectController(duration: 0.5));
```


### `SizeEffect.by`

此效果将更改目标组件的大小，相对于其当前大小。例如，如果目标有大小 `Vector2(100, 100)`，然后在应用以下效果并运行它的过程后，新的大小将是 `Vector2(120, 50)`：

```dart
final effect = SizeEffect.by(Vector2(20, -50), EffectController(duration: 1));
```

一个大小 `PositionComponent` 不能为负。如果效果试图将大小设置为负值，则大小将被限制为零。

请注意，要使此效果起作用，目标组件必须拥有自己的 `size` 渲染时考虑，并不是所有的都这样做。此外，更改组件的大小不会传播到其子组件（如果有的话）。一个替代方案 `SizeEffect` 是个 `ScaleEffect`，它更普遍地工作并且也可以缩放子组件。


### `SizeEffect.to`

将目标组件的大小更改为指定的大小。目标大小不能为负数：

```dart
final effect = SizeEffect.to(Vector2(120, 120), EffectController(duration: 1));
```


### `OpacityEffect`

此效果将随着时间的推移将目标的不透明度更改为指定的 alpha 值。目前，此效果只能应用于具有 `HasPaint` 混音。如果目标组件使用多个绘制，则效果可以使用 `paintId` 范围。

```dart
final effect = OpacityEffect.to(0.5, EffectController(duration: 0.75));
```

opacity 值为 0 对应一个完全透明的组件， opacity 值为 1 是完全不透明的。便利构造函数 `OpacityEffect.fadeOut()` 和 `OpacityEffect.fadeIn()` 将分别将目标设置为完全透明/完全可见性。


### `SequenceEffect`

此效果可用于一个接一个地运行多个其他效果。成分效应可能有不同的类型。

序列效果也可以是交替的（序列会先前进，后退）；并且还重复一定的预定次数，或无限次。

```dart
final effect = SequenceEffect([
  ScaleEffect.by(1.5, EffectController(duration: 0.2, alternate: true)),
  MoveEffect.by(Vector2(30, -50), EffectController(duration: 0.5)),
  OpacityEffect.to(0, EffectController(duration: 0.3)),
  RemoveEffect(),
]);
```


### `RemoveEffect`

这是一个简单的效果，可以附加到组件上，使其在指定的延迟过后从游戏树中删除：

```dart
final effect = RemoveEffect(delay: 10.0);
```


## ColorEffect

此效果将更改绘制的基础颜色，导致渲染的组件在提供的范围内被提供的颜色着色。

使用示例：

```dart
final effect = ColorEffect(
  const Color(0xFF00FF00),
  const Offset(0.0, 0.8),
  EffectController(duration: 1.5),
);
```

这 `Offset` 参数将确定将应用于组件的“多少”颜色，在此示例中，效果将从 0% 开始，最高可达 80%。

__笔记：__ 由于这个效果是如何实现的，以及 Flutter 的 `ColorFilter` 类有效，此效果不能与其他效果混合 `ColorEffect`s，当组件中添加多个时，只有最后一个才会生效。


## 创建新效果

尽管 Flame 提供了广泛的内置效果，但最终你可能会发现它们是不够的。幸运的是，创建新效果非常简单。

每个效果都扩展了基础 `Effect` 类，可能通过更专业的抽象子类之一，例如 `ComponentEffect<T>` 要么 `Transform2DEffect`.

这 `Effect` 类的构造函数需要一个 `EffectController` 实例作为参数。在大多数情况下，你可能希望从你自己的构造函数中传递该控制器。幸运的是，效果控制器封装了效果实现的大部分复杂性，因此你无需担心重新创建该功能。

最后，你将需要实现一个方法 `apply(double progress)` 将在效果激活时在每个更新滴答时调用。在这种方法中，你应该对效果的目标进行更改。

此外，你可能希望实现回调 `onStart()` 和 `onFinish()` 如果在效果开始或结束时必须采取任何行动。

实施时 `apply()` 我们建议仅使用相对更新的方法。也就是说，通过增加/减少其当前值来更改目标属性，而不是直接将该属性设置为固定值。这样，多个效果将能够作用于同一组件而不会相互干扰。


## 效果控制器

### `EffectController`

基地 `EffectController` 类提供了一个能够创建各种通用控制器的工厂构造函数。构造函数的语法如下：

```dart
EffectController({
    required double duration,
    Curve curve = Curves.linear,
    double? reverseDuration,
    Curve? reverseCurve,
    bool alternate = false,
    double atMaxDuration = 0.0,
    double atMinDuration = 0.0,
    int? repeatCount,
    bool infinite = false,
    double startDelay = 0.0,
});
```

- **` 持续时间 `**-- 效果主要部分的长度，即从 0 到 100% 需要多长时间。此参数不能为负数，但可以为零。如果这是唯一指定的参数，那么效果将在 `duration` 秒。

- **` 曲线 `**-- 如果给定，创建一个非线性效果，根据提供的从 0 增长到 100%[curve](https://api.flutter.dev/flutter/animation/Curves-class.html) .

- **`reverseDuration`**-- 如果提供，向控制器添加一个额外的步骤：在效果从 0 增长到 100% 之后 `duration` 秒，然后它将在整个过程中从 100% 倒退到 0`reverseDuration` 秒。此外，效果将在进度级别 0 处完成（通常效果在进度 1 处完成）。

- **` 反向曲线 `**-- 在效果的“反向”步骤中使用的曲线。如果没有给出，这将默认为 `curve.flipped`.

- **` 替代 `**-- 将此设置为 true 等效于指定 `reverseDuration` 等于 `duration`.如果 `reverseDuration` 已设置，此标志无效。

- **`atMaxDuration`**-- 如果非零，这将在效果达到其最大进度之后和反向阶段之前插入一个暂停。在此期间，效果保持在 100% 的进度。如果没有反向阶段，那么这将只是在效果标记为完成之前的暂停。

- **`atMinDuration`**-- 如果非零，则在反向阶段结束时达到其最低进度 (0) 后插入暂停。在此期间，效果的进度为 0%。如果没有反向阶段，则此暂停仍将插入到“最大”暂停（如果存在）之后，否则将插入到正向阶段之后。此外，效果现在将在进度级别为 0 时完成。

- **` 重复计数 `**-- 如果大于一，将导致效果重复规定的次数。每次迭代将包括前向阶段、最大暂停、反向阶段，然后在最小暂停（跳过未指定的阶段）。

- **` 无限 `**-- 如果为真，效果将无限重复，永远不会完成。这相当于好像 `repeatCount` 被设置为无穷大。
  
- **` 开始延迟 `**-- 在效果开始之前插入额外的等待时间。这个等待时间只执行一次，即使效果是重复的。在此期间效果 `.started` 属性返回 false。效果 `onStart()` 回调将在此等待期结束时执行。

  使用此参数是创建一个接一个（或重叠）执行的效果链的最简单方法。

这个工厂构造函数返回的效果控制器将由下面进一步描述的多个更简单的效果控制器组合而成。如果这个构造函数被证明对你的需求太有限，你总是可以从相同的构建块创建你自己的组合。

除了工厂构造函数，`EffectController` 类定义了所有效果控制器通用的许多属性。这些属性是：

- `.started`-- 如果效果已经开始，则为真。对于大多数效果控制器，此属性始终为真。唯一的例外是 `DelayedEffectController` 在效果处于等待阶段时返回 false。

- `.completed`-- 当效果控制器完成执行时变为真。

- `.progress`-- 效果控制器的当前值，0 到 1 的浮点值。该变量是效果控制器的主要“输出”值。

- `.duration`-- 效果的总持续时间，或 `null` 如果无法确定持续时间（例如，如果持续时间是随机的或无限的）。


### `LinearEffectController`

这是最简单的效果控制器，在指定范围内从 0 线性增长到 1`duration` ：

```dart
final ec = LinearEffectController(3);
```


### `ReverseLinearEffectController`

类似于 `LinearEffectController`，但它的方向相反，并在指定的持续时间内从 1 线性增长到 0：

```dart
final ec = ReverseLinearEffectController(1);
```


### `CurvedEffectController`

此效果控制器在指定范围内从 0 非线性增长到 1`duration` 并按照提供的 `curve`：

```dart
final ec = CurvedEffectController(0.5, Curves.easeOut);
```


### `ReverseCurvedEffectController`

类似于 `CurvedEffectController`, 但控制器在提供的之后从 1 增长到 0`curve` ：

```dart
final ec = ReverseCurvedEffectController(0.5, Curves.bounceInOut);
```


### `PauseEffectController`

此效果控制器在指定的持续时间内将进度保持在恒定值。通常情况下，`progress` 将是 0 或 1：

```dart
final ec = PauseEffectController(1.5, progress: 0);
```


### `RepeatedEffectController`

这是一个复合效果控制器。它需要另一个效果控制器作为孩子，并重复多次，在每个下一个循环开始之前重置。

```dart
final ec = RepeatedEffectController(LinearEffectController(1), 10);
```

子效果控制器不能是无限的。如果孩子是随机的，那么它将在每次迭代中使用新的随机值重新初始化。


### `InfiniteEffectController`

类似于 `RepeatedEffectController`，但无限期地重复其子控制器。

```dart
final ec = InfiniteEffectController(LinearEffectController(1));
```


### `SequenceEffectController`

一个接一个地执行一系列效果控制器。控制器列表不能为空。

```dart
final ec = SequenceEffectController([
  LinearEffectController(1),
  PauseEffectController(0.2),
  ReverseLinearEffectController(1),
]);
```


### `SpeedEffectController`

更改其子效果控制器的持续时间，以便效果以预定义的速度进行。子 EffectController 的初始持续时间无关紧要。子控制器必须是 `DurationEffectController`.

这 `SpeedEffectController` 只能应用于速度概念已明确定义的效果。这种效果必须实施 `MeasurableEffect` 界面。例如，以下效果符合条件：[`MoveEffect.by`](#moveeffectby) ,[`MoveEffect.to`](#moveeffectto) ,[`MoveAlongPathEffect`](#movealongpatheffect) ,[`RotateEffect.by`](#rotateeffectby) ,[`RotateEffect.to`](#rotateeffectto) .

参数 `speed` 以每秒单位为单位，其中“单位”的概念取决于目标效果。例如，对于移动效果，它们指的是行进的距离；对于旋转效果，单位是弧度。

```dart
final ec1 = SpeedEffectController(LinearEffectController(0), speed: 1);
final ec2 = EffectController(speed: 1); // same as ec1
```


### `DelayedEffectController`

在规定后执行其子控制器的效果控制器 `delay`.当控制器执行“延迟”阶段时，效果将被视为“未启动”，即其 `.started` 财产将归还 `false`.

```dart
final ec = DelayedEffectController(LinearEffectController(1), delay: 5);
```


### `NoiseEffectController`

该效果控制器表现出噪声行为，即它在零附近随机振荡。这样的效果控制器可以用来实现各种抖动效果。

```dart
final ec = NoiseEffectController(duration: 0.6, frequency: 10);
```


### `RandomEffectController`

该控制器包装另一个控制器并使其持续时间随机。持续时间的实际值在每次重置时重新生成，这使得该控制器在重复的上下文中特别有用，例如 [](#repeatedeffectcontroller) 要么 [](#infiniteeffectcontroller).

```dart
final ec = RandomEffectController.uniform(
  LinearEffectController(0),  // duration here is irrelevant
  min: 0.5,
  max: 1.5,
);
```

用户有能力控制哪个 `Random` 要使用的源，以及产生的随机持续时间的确切分布。两种分布——`.uniform` 和 `.exponential` 包括在内，任何其他都可以由用户实现。


### `SineEffectController`

表示正弦函数的单个周期的效果控制器。使用它来创建看起来自然的谐波振荡。两个垂直移动效果由 `SineEffectControllers` 与不同的时期，将创建一个[李萨如曲线]。

```dart
final ec = SineEffectController(period: 1);
```


### `ZigzagEffectController`

简单的交替效果控制器。在一个过程中 `period`，此控制器将从 0 到 1，然后到 -1，然后回到 0 线性进行。将其用于振荡效果，其中起始位置应该是振荡的中心，而不是极端（由标准交替 `EffectController`）。

```dart
final ec = ZigzagEffectController(period: 2);
```


## 也可以看看

* [各种效果的例子](https://examples.flame-engine.org/#/).


[tau]: https://en.wikipedia.org/wiki/Tau_(mathematical_constant)
[Lissajous curve]: https://en.wikipedia.org/wiki/Lissajous_curve
