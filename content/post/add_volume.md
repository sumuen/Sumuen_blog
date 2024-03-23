---
title: "Add_volume"
date: 2023-11-10T23:22:15+08:00
---
---
```sh
amixer set Master 10%+
```

# 如何在床上增加电脑的音量，(linux)

今天，11.10，今夜我很难受，本来都打算买平板了，结果，结果我的京东号撸黑了，我是怎么也下不了单，好可恶啊啊啊啊啊

![image-20231110233519901](http://easyimage.muzi.studio/i/2023/11/10/12mcg8p-0.png)

还有就是，对加密货币的判断小失误，今天又亏了十个点，亏掉昨天赚的一半

![image-20231110233604878](http://easyimage.muzi.studio/i/2023/11/10/12mvga2-0.png)

我就在床上好难受，好难受

就趴着

我用spotify mobile 设置成电脑发音，但是初始音量太小了，我好难受，我听不见啊，Spotify的音量条我都拉满了，但是系统的音量太小了

我就灵机一动

我直接连ssh进系统调不就完了，肯定有一个命令是控制音量的

所以我就问gpt

> 在Arch Linux上，您可以通过命令行使用 `amixer` 或 `pactl` 命令来增加音量。
>
> 1. **使用 `amixer`**:
>
>    - `amixer` 是 ALSA（高级 Linux 声音体系结构）的一部分，它允许您控制声卡的音量。
>
>    - 要增加主音量，您可以使用以下命令：
>
>      ```
>      arduinoCopy code
>      amixer set Master 10%+
>      ```
>
>    - 这会将主音量提高 10%。
>
> 2. **使用 `pactl`**:
>
>    - `pactl` 是 PulseAudio 的一个工具，它是一个流行的声音服务器，用于管理和操纵声音流。
>
>    - 要增加音量，使用以下命令：
>
>      ```
>      kotlinCopy code
>      pactl set-sink-volume @DEFAULT_SINK@ +10%
>      ```
>
>    - 这会将默认音频输出设备的音量提高 10%。
>
> 请根据您的音频系统配置选择合适的命令。如果您使用的是PulseAudio（大多数现代桌面环境的默认选择），那么使用 `pactl` 可能更合适。

得到了如下答案，非常简单啊

所以我就用手机Termius连接电脑ssh

我靠

我没开ssh端口防火墙

我似了。。。

下床开了防火墙，终究还是需要下床。。volomevolome