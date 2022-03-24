# 循环播放背景音乐

随着 `Bgm` 类，你可以管理与应用程序（或游戏）生命周期状态更改有关的背景音乐曲目的循环。

当应用程序终止或发送到后台时，`Bgm` 将自动暂停当前播放的音乐曲目。同样，当应用程序恢复时，`Bgm` 将恢复背景音乐。还支持手动暂停和恢复你的曲目。

这 `Bgm` 类由处理 [flame_audio](https://github.com/flame-engine/flame_audio) 库，因此你首先必须将其添加到你的依赖项列表中 `pubspec.yaml` 文件：

```yaml
dependencies:
  flame_audio: <VERSION>
```

最新版本可以在 [pub.dev](https://pub.dev/packages/flame_audio/install).

要使此类正常运行，必须通过调用以下方法注册观察者：

```dart
FlameAudio.bgm.initialize();
```

**重要的提示：** 这 `initialize` 函数必须在某个时间点调用 `WidgetsBinding` 类已经存在。最佳做法是将此调用放在游戏的 `onLoad` 方法 `。

如果你已完成背景音乐但仍想保持应用程序/游戏运行，请使用 `dispose` 删除观察者的功能。

```dart
FlameAudio.bgm.dispose();
```

要播放循环背景音乐曲目，请运行：

```dart
import 'package:flame_audio/flame_audio.dart';

FlameAudio.bgm.play('adventure-track.mp3');
```

**笔记：** 这 `Bgm` 类将始终使用的静态实例 `FlameAudio` 用于存储缓存的音乐文件。

你必须具有适当的文件夹结构并将文件添加到 `pubspec.yaml` 文件，如中所述 [火焰音频文档](audio.md).

## 缓存音乐文件

以下函数可用于将音乐文件预加载（和卸载）到缓存中。这些函数只是 `FlameAudio` 同名。

再次，请参阅 [火焰音频文档](audio.md) 如果你需要更多信息。

```dart
import 'package:flame_audio/flame_audio.dart';

FlameAudio.bgm.load('adventure-track.mp3');
FlameAudio.bgm.loadAll([
  'menu.mp3',
  'dungeon.mp3',
]);
FlameAudio.bgm.clear('adventure-track.mp3');
FlameAudio.bgm.clearCache();
```

## 方法

### 玩

这 `play` 函数接受一个 `String` 那应该是指向要播放的音乐文件位置的路径（遵循 Flame Audio 文件夹结构要求）。

你可以传递一个额外的可选 `double` 参数是 `volume`（默认为 `1.0`）。

例子：

```dart
FlameAudio.bgm.play('bgm/boss-fight/level-382.mp3');
```

```dart
FlameAudio.bgm.play('bgm/world-map.mp3', volume: .25);
```

### 停止

要停止当前播放的背景音乐曲目，只需调用 `stop`.

```dart
FlameAudio.bgm.stop();
```

### 暂停和恢复

要手动暂停和恢复背景音乐，你可以使用 `pause` 和 `resume` 职能。

`FlameAudio.bgm` 自动处理暂停和恢复当前播放的背景音乐曲目。手动 `pausing` 当焦点回到应用程序/游戏时，防止应用程序/游戏自动恢复。

```dart
FlameAudio.bgm.pause();
```

```dart
FlameAudio.bgm.resume();
```
