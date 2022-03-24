# 入门！

## 文档目录 

* [Summary.md文件地址](./summary.md)

## 关于火焰

Flame 是一个模块化的 Flutter 游戏引擎，为游戏提供了一整套出路的解决方案。它利用了 Flutter 提供的强大基础设施，但简化了构建项目所需的代码。

它为你提供了一个简单而有效的游戏循环实现，以及你在游戏中可能需要的必要功能。例如;输入、图像、精灵、精灵表、动画、碰撞检测和我们称为火焰组件系统（简称 FCS）的组件系统。

我们还提供扩展 Flame 功能的独立包：
- [flame_audio](https://pub.dev/packages/flame_audio) 它使用 `audioplayers` 包裹。
- [flame_forge2d](https://pub.dev/packages/flame_forge2d) 它使用我们自己的提供物理功能 `Box2D` 港口叫 `Forge2D`.
- [flame_tiled](https://pub.dev/packages/flame_tiled) 它提供了与 [tiled](https://pub.dev/packages/tiled) 包裹。

你可以挑选你想要的任何部分，因为它们都是独立的和模块化的。

引擎及其生态系统正在由社区不断改进，因此请随时联系、打开问题、PR 并提出建议。

如果你想帮助引擎曝光和发展社区，请给我们一颗星。：)

## 安装

通过将以下内容放入你的 pub 包作为你的依赖项 `pubspec.yaml`：

```yaml
dependencies:
  flame: <VERSION>
```

最新版本可以在 [pub.dev](https://pub.dev/packages/flame/install).

然后运行 `pub get` 你已经准备好开始使用它了！

最新版本可以在 [pub.dev](https://pub.dev/packages/flame/install).

## 入门

有一组教程，你可以按照这些教程开始 [教程文件夹](https://github.com/flame-engine/flame/tree/main/tutorials).

所有功能的简单示例可以在 [examples folder](https://github.com/flame-engine/flame/tree/main/examples).

你还可以查看 [很棒的火焰库](https://github.com/flame-engine/awesome-flame#articles--tutorials), 它包含了社区编写的大量优秀教程和文章，帮助你开始使用 Flame。


## 引擎范围之外

游戏有时需要复杂的功能集，具体取决于游戏的全部内容。其中一些功能集超出了火焰引擎生态系统的范围，在本节中你可以找到它们，以及一些可以使用的包/服务的建议：

### 多人游戏（网络代码）

Flame 不捆绑任何网络功能，编写在线多人游戏可能需要这些功能。

如果你正在构建多人游戏，以下是一些包/服务的建议：

 - [Nakama](https://github.com/Allan-Nava/nakama-flutter): Nakama 是一个开源服务器设计
为现代游戏和应用程序提供动力。
 - [Firebase](https://firebase.google.com/): 提供了几十种可以用来写的服务
更简单的多人游戏体验。

### 外部资产

Flame 不捆绑任何帮助程序来从外部资源（外部存储或在线资源）加载资产。

但是大多数 Flame 的 API 都可以从具体的资产实例中加载，例如，`Sprite` s 可以从 `dart:ui`s`Image` instaces，因此用户可以编写自定义代码来从他们需要的任何地方加载图像，然后将其加载到 Flame 的类中。

以下是对 http 客户端包的一些建议：

 - [http](https://pub.dev/packages/http): 用于执行 http 请求的简单封装。
 - [Dio](https://pub.dev/packages/dio)：用于执行 http 请求的流行且强大的软件包。
