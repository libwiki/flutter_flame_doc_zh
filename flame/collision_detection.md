# 碰撞检测

大多数游戏都需要碰撞检测来检测两个相互交叉的组件并对其采取行动。例如，箭击中敌人或玩家捡起硬币。

在大多数碰撞检测系统中，你使用称为命中框的东西来创建组件的更精确的边界框。在 Flame 中，hitboxes 是组件中可以对碰撞做出反应的区域（并使 [gesture input](inputs/gesture-input.md#gesturehitboxes)） 更准确的。

碰撞检测系统支持三种不同类型的形状，你可以从中构建碰撞盒，这些形状是多边形、矩形和圆形。可以将多个 hitbox 添加到组件以形成可用于检测碰撞或是否包含点的区域，后者对于准确的手势检测非常有用。碰撞检测不处理两个碰撞箱碰撞时应该发生的事情，因此由用户来实现例如两个碰撞时会发生什么 `PositionComponent`s 有相交的碰撞箱。

请注意，内置的碰撞检测系统不会考虑两个碰撞箱之间的碰撞，这两个碰撞箱会相互冲撞，当它们移动非常快或移动非常快时可能会发生这种情况。`update` 以较大的增量时间调用（例如，如果你的应用程序不在前台）。如果你想了解更多有关它的信息，这种行为称为隧道。

另请注意，碰撞检测系统有一个限制，如果你有某些类型的碰撞框祖先的翻转和缩放组合，则它无法正常工作。


## 混合

### 有碰撞检测

如果你想在游戏中使用碰撞检测，你必须添加 `HasCollisionDetection`mixin 到你的游戏中，以便它可以跟踪可能发生碰撞的组件。

例子：

```dart
class MyGame extends FlameGame with HasCollisionDetection {
  // ...
}
```

现在当你添加 `形状命中框`s 到随后添加到游戏中的组件，它们将自动检查碰撞。


### 碰撞回调

要对碰撞做出反应，你应该添加 `CollisionCallbacks` 混合到你的组件中。例子：

```dart
class MyCollidable extends PositionComponent with CollisionCallbacks {
  @override
  void onCollision(Set<Vector2> points, PositionComponent other) {
    if (other is ScreenHitbox) {
      //...
    } else if (other is YourOtherComponent) {
      //...
    }
  }

  @override
  void onCollisionEnd(PositionComponent other) {
    if (other is ScreenHitbox) {
      //...
    } else if (other is YourOtherComponent) {
      //...
    }
  }
}
```

在这个例子中，我们使用 Dart 的 `is` 关键字来检查我们碰撞了什么样的组件。这组点是命中框的边缘相交的地方。

请注意，`onCollision` 方法将在两者上调用 `PositionComponent`s 如果他们都实施了 `onCollision` 方法，以及在两个命中框上。这同样适用于 `onCollisionStart` 和 `onCollisionEnd` 方法，当两个组件和碰撞框开始或停止相互碰撞时调用。

当一个 `PositionComponent`（和碰撞箱）开始与另一个碰撞 `PositionComponent` 两个都 `onCollisionStart` 和 `onCollision` 被调用，所以如果你不需要在碰撞开始时做一些特定的事情，你只需要覆盖 `onCollision`，反之亦然。

如果你想检查与屏幕边缘的碰撞，就像我们在上面的示例中所做的那样，你可以使用预定义的 [屏幕命中框](#screenhitbox) 班级。


## ShapeHitbox

这 `ShapeHitbox`s 是普通组件，因此你将它们添加到要添加命中框的组件中，就像任何其他组件一样：

```dart
class MyComponent extends PositionComponent {
  Future<void> onLoad() async {
    add(RectangleHitbox());
  }
}
```

如果你不向 hitbox 添加任何参数，如上所示，hitbox 将尝试尽可能多地填充其父项。除了让 hitboxes 试图填充它们的父母之外，还有两种方法可以启动 hitboxes，它是使用普通的构造函数，你自己定义 hitbox，大小和位置等。另一种方法是使用 `relative` 构造函数，它根据预期父项的大小定义命中框。

你可以阅读更多关于如何定义不同形状的信息 [ShapeComponents](components.md#shapecomponents) 部分。

请记住，你可以添加尽可能多的 `ShapeHitbox` 如你所愿 `PositionComponent` 组成更复杂的区域。例如，一个带帽子的雪人可以用三个来表示 `圆形命中框`s 和两个 `矩形命中框`s 作为它的帽子。

hitbox 可用于碰撞检测或使手势检测在组件之上更准确，有关后者的更多信息，请参见关于 [GestureHitboxes](inputs/gesture-input.md#gesturehitboxes) 混音。


### 碰撞类型

hitboxes 有一个字段叫做 `collisionType` 它定义了一个 hitbox 何时应该与另一个碰撞。通常你想设置尽可能多的命中框 `CollisionType.passive` 使碰撞检测更高效。默认情况下 `CollisionType` 是 `active`.

这 `CollisionType` 枚举包含以下值：

- `active` 与其他碰撞 `Collidable`s 类型的主动或被动
- `passive` 与其他碰撞 `Collidable` 类型为 active
- `inactive` 不会与其他任何碰撞 `Collidable`s

因此，如果你有不需要检查相互碰撞的碰撞箱，你可以通过设置将它们标记为被动 `collisionType = CollisionType.passive`，例如，这可能是地面组件，或者你的敌人可能不需要检查彼此之间的碰撞，那么它们可以标记为 `passive` 也。

想象一个游戏，有很多子弹，不能相互碰撞，飞向玩家，那么玩家将被设置为 `CollisionType.active` 并且子弹将设置为 `CollisionType.passive`.

然后我们有 `inactive` 在碰撞检测中根本不会检查的类型。例如，如果你在屏幕之外有你目前不关心的组件，但稍后可能会返回查看，因此它们不会完全从游戏中删除，则可以使用此功能。

这些只是你如何使用这些类型的示例，它们会有更多的用例，因此即使你的用例未在此处列出，也不要怀疑使用它们。


### 多边形命中框

需要注意的是，如果要使用碰撞检测或 `containsPoint` 在 `Polygon`，多边形需要是凸的。所以总是使用凸多边形，否则如果你真的不知道你在做什么，你很可能会遇到问题。还应该注意的是，你应该始终以逆时针顺序定义多边形中的顶点。

其他 hitbox 形状没有任何强制构造函数，这是因为它们可以有一个默认值，该默认值是根据它们所连接的可碰撞对象的大小计算得出的，但是由于可以在边界内以无数种方式制作多边形框，你必须在此形状的构造函数中添加定义。

这 `PolygonHitbox` 具有与 [](components.md#polygoncomponent)，请参阅该部分以获取有关这些的文档。


### RectangleHitbox

这 `RectangleHitbox` 具有与 [](components.md#rectanglecomponent)，请参阅该部分以获取有关这些的文档。


### CircleHitbox

这 `CircleHitbox` 具有与 [](components.md#circlecomponent)，请参阅该部分以获取有关这些的文档。


## ScreenHitbox

`ScreenHitbox` 是代表视口/屏幕边缘的组件。如果你添加一个 `ScreenHitbox` 在你的游戏中，你的其他带有碰撞盒的组件会在它们与边缘碰撞时收到通知。它不需要任何参数，它只取决于 `size` 它被添加到的游戏。要添加它，你可以这样做 `add(ScreenHitbox())` 在你的游戏中，如果你不想 `ScreenHitbox` 当有东西与它发生碰撞时，它自己会收到通知。自从 `ScreenHitbox` 有 `CollisionCallbacks`mixin 你可以添加你自己的 `onCollisionCallback`,`onStartCollisionCallback` 和 `onEndCollisionCallback` 如果需要，该对象的功能。


## 复合命中框

在里面 `CompositeHitbox` 你可以添加多个碰撞箱，以便它们模拟成为一个连接的碰撞箱。

例如，如果你想形成一顶帽子，你可能想要使用两个 [](#rectanglehitbox)s 正确地跟随该帽子的边缘，然后你可以将这些碰撞箱添加到此类的一个实例中，并对整个帽子的碰撞做出反应，而不是单独针对每个碰撞箱。


## 宽相

通常你不必担心所使用的广泛阶段系统，因此如果标准实现对你来说足够高效，你可能不必阅读本节。

广泛阶段是计算潜在碰撞的碰撞检测的第一步。计算这些潜在的碰撞比直接检查确切的交叉点要便宜得多，它消除了检查所有碰撞箱的需要，因此避免了 O(n²)。广泛阶段产生一组潜在的碰撞（一组 `CollisionProspect`s)，然后使用这个集合来检查命中框之间的确切交叉点，这有时被称为窄相位。

默认情况下，Flame 的碰撞检测使用扫描和修剪 broadphase 步骤，如果你的游戏需要另一种类型的 broadphase，你可以通过扩展编写自己的 broadphase`Broadphase` 并手动设置应使用的碰撞检测系统。

例如，如果你已经实现了基于四叉树而不是标准扫描和修剪的广泛阶段，那么你将执行以下操作：

```dart
class MyGame extends FlameGame with HasCollisionDetection {
  MyGame() : super() {
    collisionDetection = 
        StandardCollisionDetection(broadphase: QuadTreeBroadphase());
  }
}
```


## 与 Forge2D 的比较

如果你想在游戏中拥有成熟的物理引擎，我们建议你通过添加使用 Forge2D[flame_forge2d](https://github.com/flame-engine/flame_forge2d) 作为依赖。但是，如果你有一个更简单的用例，并且只想检查组件的碰撞并提高手势的准确性，那么 Flame 的内置碰撞检测将为你提供很好的服务。

如果你有以下需求你至少应该考虑使用 [Forge2D](https://github.com/flame-engine/forge2d)：
- 相互作用的现实力量
- 可以与其他物体相互作用的粒子系统
- 身体之间的关节

另一方面，如果你只需要以下一些东西（因为不涉及 Forge2D 更简单），则最好使用 Flame 碰撞检测系统：
- 对一些碰撞的组件采取行动的能力
- 对与屏幕边界冲突的组件进行操作的能力
- 复杂的形状作为组件的碰撞箱，使手势更加准确
- 可以判断组件的哪个部分与某物碰撞的 Hitbox


## 从 v1.0 中的碰撞检测系统迁移

v1.1 中引入的碰撞检测系统比 v1.0 中的更易于使用，效率更高，但在进行这些改进时，必须进行一些重大更改。

不再有一个 `Collidable`mixin，相反，你的游戏会在 `HasCollisionDetection`mixin 已添加到你的游戏中。

要从你的组件所涉及的碰撞中接收回调，你应该添加 `CollisionCallbacks`mixin 到你的组件中，然后覆盖与之前相同的方法。

由于现在的碰撞箱是 `Component`s 你将它们添加到你的组件中 `add` 代替 `addHitbox` 这是以前使用的。


### 名称更改

 - `ScreenCollidable` -> `ScreenHitbox`
 - `HitboxCircle` -> `CircleHitbox`
 - `HitboxRectangle` -> `RectangleHitbox`
 - `HitboxPolygon` -> `PolygonHitbox`
 - `Collidable`->`CollisionCallbacks` （仅当你想要接收回调时才需要）
 - `HasHitboxes`->`GestureHitboxes` （仅当你需要手势的命中框时）
 - `CollidableType` -> `CollisionType`


## 例子

- https://examples.flame-engine.org/#/Collision%20Detection_Circles
- https://examples.flame-engine.org/#/Collision%20Detection_Multiple%20shapes
- https://examples.flame-engine.org/#/Collision%20Detection_Shapes%20without%20components
- https://github.com/flame-engine/flame/tree/main/examples/lib/stories/collision_detection
