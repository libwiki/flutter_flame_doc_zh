# 声音的

播放音频对于任何游戏都是必不可少的，所以我们让它变得简单！

首先你必须添加 [flame_audio](https://github.com/flame-engine/flame_audio) 到你的依赖列表中 `pubspec.yaml` 文件：

```yaml
dependencies:
  flame_audio: <VERSION>
```

最新版本可以在 [pub.dev](https://pub.dev/packages/flame_audio/install).

安装后 `flame_audio` 包你可以在你的资产部分添加音频文件 `pubspec.yaml` 文件。确保音频文件存在于你提供的路径中。

的默认目录 `FlameAudio` 是 `assets/audio`（可以更改）和 `AudioPool` 默认目录是 `assets/audio/sfx`.

对于以下示例，你的 `pubspec.yaml` 文件需要包含如下内容：

```yaml
flutter:
  assets:
    - assets/audio/explosion.mp3
    - assets/audio/music.mp3
```

（默认目录为 `FlameAudio` 是 `assets/audio`（可以更改）和 `AudioPool` 默认目录是 `assets/audio/sfx`.)

然后，你可以使用以下方法：

```dart
import 'package:flame_audio/flame_audio.dart';

// For shorter reused audio clips, like sound effects
FlameAudio.play('explosion.mp3');

// For looping an audio file
FlameAudio.loop('music.mp3');

// For playing a longer audio file
FlameAudio.playLongAudio('music.mp3');

// For looping a longer audio file
FlameAudio.loopLongAudio('music.mp3');

// For background music that should be paused/played when the pausing/resuming the game
FlameAudio.bgm.play('music.mp3');
```

之间的区别 `play/loop` 和 `playLongAudio/loopLongAudio` 就是它 `play/loop` 利用优化的功能，允许声音在迭代之间没有间隙地循环，并且几乎不会发生游戏帧率下降。你应该尽可能选择前一种方法。

`playLongAudio/loopLongAudio` 允许播放任何长度的音频，但它们确实会造成帧率下降，并且循环播放的音频在迭代之间会有一个小的间隙。

你可以使用 [the `Bgm` class](bgm.md)（通过 `FlameAudio.bgm`) 播放循环背景音乐曲目。这 `Bgm` 类让 Flame 在游戏进入后台或返回前台时自动管理背景音乐曲目的暂停和恢复。

我们推荐的一些跨设备工作的文件格式是：MP3、OGG 和 WAV。

这个桥库（flame_audio）使用 [audioplayers](https://github.com/luanpotter/audioplayer) 为了允许同时播放多个声音（在游戏中至关重要）。你可以查看链接以获得更深入的解释。

最后，你可以预加载音频。音频需要在第一次被请求时存储在内存中；因此，第一次播放每个 mp3 时，你可能会遇到延迟。为了预加载你的音频，只需使用：

```dart
await FlameAudio.audioCache.load('explosion.mp3');
```

你可以在游戏的开头加载所有音频 `onLoad` 方法，让他们总是玩得很顺利。要加载多个音频文件，请使用 `loadAll` 方法：

```dart
await FlameAudio.audioCache.loadAll(['explosion.mp3', 'music.mp3'])
```

最后，你可以使用 `clear` 删除已加载到缓存中的文件的方法：

```dart
FlameAudio.audioCache.clear('explosion.mp3');
```

还有一个 `clearCache` 方法，清除整个缓存。

例如，如果你的游戏有多个关卡并且每个关卡都有不同的声音和音乐集，这可能会很有用。

两种加载方法都返回一个 `Future` 为了 `File` 已加载。

都开 `play` 和 `loop` 你可以传递一个额外的可选双参数，`volume` （默认为 `1.0`）。

这俩 `play` 和 `loop` 方法返回一个实例 `AudioPlayer` 来自 [audioplayers](https://github.com/luanpotter/audioplayer)lib，它允许你停止、暂停和配置其他参数。
