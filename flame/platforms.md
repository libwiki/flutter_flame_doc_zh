# 支持的平台

由于 Flame 运行在 Flutter 之上，因此其支持的平台取决于 Flutter 支持的平台。

目前，Flame 同时支持移动端和网络端。

## 颤振通道

火焰保持它在稳定通道上的支持。dev、beta 和 master 频道应该可以工作，但我们不支持它们。这意味着在稳定渠道之外发生的问题不是优先事项。

## 火焰网

要在 web 上使用 Flame，你需要确保你的游戏使用 CanvasKit/[Skia](https://skia.org/) 渲染器。这将提高 Web 上的性能，因为它将使用 `canvas` 元素而不是单独的 DOM 元素。

要使用 skia 运行游戏，请使用以下命令：
```console
$ flutter run -d chrome --web-renderer canvaskit
```

要使用 skia 构建用于生产的游戏，请使用以下命令：
```console
$ flutter build web --release --web-renderer canvaskit
```

## 将你的游戏部署到 GitHub Pages

在线部署游戏的一种简单方法是使用 [GitHub Pages](https://pages.github.com/).这是 GitHub 的一项很酷的功能，你可以通过它轻松地从你的存储库托管 Web 内容。

在这里，我们将解释使用 GitHub 页面托管游戏的最简单方法。

首先，让我们创建部署文件所在的分支：

```bash
git checkout -b gh-pages
```

这个分支可以从创建 `main` 或者其他任何地方，都无所谓。在你推动那个分支后回到你的 `main` 分支。

现在你应该添加 [颤动-gh-pages](https://github.com/bluefireteam/flutter-gh-pages) 对你的存储库执行操作，你可以通过创建文件来执行此操作 `gh-pages.yaml` 文件夹下 `.github/workflows`.

```yaml
name: Gh-Pages

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: subosito/flutter-action@v1
      - uses: bluefireteam/flutter-gh-pages@v7
        with:
          baseHref: /NAME_OF_YOUR_REPOSITORY/
          webRenderer: canvaskit
```

一定要改变 `NAME_OF_YOUR_REPOSITORY` 到你的 GitHub 存储库的名称。

现在，每当你把东西推到 `main` 分支，该操作将运行并更新你部署的游戏。

游戏应该在这样的 URL 上可用：`https://YOUR_GITHUB_USERNAME.github.io/YOUR_REPO_NAME/`

### 网络支持

在网络上使用 Flame 时，某些方法可能不起作用。例如 `Flame.device.setOrientation` 和 `Flame.device.fullScreen` 不会在网络上工作，它们可以被调用，但什么都不会发生。

另一个例子：使用预缓存音频 `flame_audio` 由于 Audioplayers 在网络上不支持它，该包也不起作用。这可以通过使用 `http` 包，并请求获取音频文件，这将使浏览器缓存文件，产生与移动设备相同的效果。

如果你想创建实例 `ui.Image` 在网络上，你可以使用我们的 `Flame.images.decodeImageFromPixels` 方法。这包裹了 `decodeImageFromPixels` 来自 `ui` 库，但支持网络平台。如果 `runAsWeb` 参数设置为 `true`（默认设置为 `kIsWeb`) 它将使用内部图像方法对图像进行解码。当。。。的时候 `runAsWeb` 是 `false` 它将使用 `decodeImageFromPixels`，目前在网络上不支持。
