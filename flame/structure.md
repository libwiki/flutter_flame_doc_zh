# 结构

Flame 为你的项目提供了一个包含标准 Flutter 的建议结构 `assets` 除了两个孩子之外的目录：`audio` 和 `images`.

如果使用以下示例代码：
```dart
void main() {
  FlameAudio.play('explosion.mp3');

  Flame.images.load('player.png');
  Flame.images.load('enemy.png');
}
```

Flame 期望在其中找到文件的文件结构是：

```text
.
└── assets
    ├── audio
    │   └── explosion.mp3
    └── images
        ├── enemy.png
        └── player.png
```

你可以选择拆分你的 `audio` 文件夹分为两个子文件夹，一个用于 `music` 一个用于 `sfx`.

不要忘记将这些文件添加到你的 `pubspec.yaml` 文件：

```yaml
flutter:
  assets:
    - assets/audio/explosion.mp3
    - assets/images/player.png
    - assets/images/enemy.png
```

如果要更改此结构，可以使用 `prefix` 参数并创建你自己的实例 `AssetsCache`,`ImagesCache` ,`AudioCache` 和 `SoundPool`s，而不是使用 Flame 提供的全局变量。
