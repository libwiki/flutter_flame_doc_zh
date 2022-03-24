# 平铺

[Tiled](https://www.mapeditor.org/) 是设计关卡和地图的绝佳工具。

Flame 提供了一个包（[flame_tiled](https://github.com/flame-engine/flame_tiled) ) 其中捆绑了一个 [dart package](https://pub.dev/packages/tiled) 这允许你解析 tmx (xml) 文件并访问其中的图块、对象和所有内容。

Flame 还提供了一个简单的 `Tiled` 类及其组件包装器 `TiledComponent`, 用于地图渲染，在屏幕上渲染瓦片并支持旋转和翻转。

在最简单的情况下，可以通过调用从 Tilemap 中检索层：

```
getLayer<ObjectGroup>("myObjectGroupLayer");
getLayer<ImageLayer>("myImageLayer");
getLayer<TileLayer>("myTileLayer");
getLayer<Group>("myGroupLayer");
```

这些方法将返回请求的图层类型，如果不存在则返回 null。

尚不支持其他高级功能，但你可以轻松读取 tmx 的对象和其他功能并添加自定义行为（例如触发器和行走区域的区域、自定义动画对象）。

你可以检查一个工作示例 [here](https://github.com/flame-engine/flame_tiled/tree/main/example).
