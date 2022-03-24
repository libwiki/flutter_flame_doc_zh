# 键盘输入

这包括键盘输入的文档。

对于其他输入文档，另请参阅：

- [Gesture Input](gesture-input.md)：用于鼠标和触摸指针手势
- [Other Inputs](other-inputs.md): 用于操纵杆、游戏手柄等。

## 介绍

Flame 上的键盘 API 依赖于 [Flutter 的焦点小部件](https://api.flutter.dev/flutter/widgets/Focus-class.html).

要自定义焦点行为，请参阅 [控制焦点](#controlling-focus).

游戏可以通过两种方式对击键做出反应；在游戏级别和组件级别。对于每个我们都有一个可以添加到 `Game` 要么 `Component` 班级。

### 在游戏关卡中接收键盘事件

做一个 `Game` 对击键敏感的子类，将其与 `KeyboardEvents`.

之后，就可以覆盖一个 `onKeyEvent` 方法。

这个方法接收两个参数，第一个是 [`RawKeyEvent`](https://api.flutter.dev/flutter/services/RawKeyEvent-class.html) 首先触发回调。第二个是当前按下的一组 [` 逻辑键盘键 `](https://api.flutter.dev/flutter/widgets/KeyEventResult-class.html).

返回值为一个 [`KeyEventResult`](https://api.flutter.dev/flutter/widgets/KeyEventResult-class.html).

`KeyEventResult.handled` 将告诉框架击键已在 Flame 内部解决，并跳过任何其他键盘处理程序小部件 `GameWidget`.

`KeyEventResult.ignored` 将告诉框架继续在任何其他键盘处理程序小部件中测试此事件，除了 `GameWidget`.如果事件没有被任何处理程序解析，框架将触发 `SystemSoundType.alert`.

`KeyEventResult.skipRemainingHandlers` 非常相似 `.ignored`，除了将跳过任何其他处理程序小部件并直接播放警报声音的事实。

最小的例子：

```dart
class MyGame extends FlameGame with KeyboardEvents {
  // ...
  @override
  KeyEventResult onKeyEvent(
    RawKeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    final isKeyDown = event is RawKeyDownEvent;

    final isSpace = keysPressed.contains(LogicalKeyboardKey.space);

    if (isSpace && isKeyDown) {
      if (keysPressed.contains(LogicalKeyboardKey.altLeft) ||
          keysPressed.contains(LogicalKeyboardKey.altRight)) {
        this.shootHarder();
      } else {
        this.shoot();
      }
      return KeyEventResult.handled;
    }
    return KeyEventResult.ignored;
  }
}
```

### 在组件级别接收键盘事件

要在组件中直接接收键盘事件，有 mixin`KeyboardHandler` .

类似于 `Tappable` 和 `Draggable`,`KeyboardHandler` 可以混入任何子类 `Component`.

KeyboardHandlers 只能添加到与 `HasKeyboardHandlerComponents`.

> ⚠️注意：如果 `HasKeyboardHandlerComponents` 已使用，你必须删除 `KeyboardEvents`
> 从游戏混合列表中避免冲突。

申请后 `KeyboardHandler`，将有可能覆盖一个 `onKeyEvent` 方法。

该方法接收两个参数。首先是 [`RawKeyEvent`](https://api.flutter.dev/flutter/services/RawKeyEvent-class.html) 这首先触发了回调。第二个是当前按下的一组 [` 逻辑键盘键 `](https://api.flutter.dev/flutter/widgets/KeyEventResult-class.html)s。

返回值应该是 `true` 允许关键事件在其他组件之间持续传播。要不允许任何其他组件接收事件，请返回 `false`.

### 控制焦点

在小部件级别，可以使用 [`FocusNode`](https://api.flutter.dev/flutter/widgets/FocusNode-class.html) 控制游戏是否聚焦的 API。

`GameWidget` 有一个可选的 `focusNode` 允许从外部控制其焦点的参数。

默认 `GameWidget` 有它的 `autofocus` 设置为 true，这意味着一旦安装它就会聚焦。要覆盖该行为，请设置 `autofocus` 为假。

有关更完整的示例，请参见 [here](https://github.com/flame-engine/flame/tree/main/examples/lib/stories/input/keyboard.dart).
