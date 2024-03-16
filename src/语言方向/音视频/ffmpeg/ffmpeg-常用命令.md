[toc]

---

# 前言

- [官网](https://ffmpeg.org/ffmpeg.html#Stream-specifiers-1)

# 选项

## 主要选项

- `-f fmt (input/output)` 指定输入或者输出文件格式。常规可省略而使用依据扩展名的自动指定，但一些选项需要强制明确设定。
- `-i filename （input）` 指定输入文件。
- `-y （global）` 默认自动覆盖输出文件，而不再询问确认。
- `-n （global）` 不覆盖输出文件，如果输出文件已经存在则立即退出。
- `-stream_loop number (input)` 设置输入流循环的次数。循环0表示无循环，循环-1表示无限循环。 
- `-recast_media (global)` 允许强制使用与解码器检测到的或指定的媒体类型不同的解码器。用于解码作为数据流混合的媒体数据。 
- `-t duration（input/output）` 限制输入/输出的时间。如果是在 -i 前面，就是限定从输入中读取多少时间的数据；如果是用于限定输出文件，则表示写入多少时间数据后就停止。duration可以是以秒为单位的数值或者 hh:mm:ss[.xxx] 格式的时间值。 **注意** -to 和 -t 是互斥的，-t 有更高优先级。
- `-to position (output)` 只写入position时间后就停止，position可以是以秒为单位的数值或者 hh:mm:ss[.xxx]格式的时间值。 **注意** -to 和 -t 是互斥的，-t 有更高优先级。
- `-ss position (input/output)`
  当在 -i 前，表示定位输入文件到position指定的位置。注意可能一些格式是不支持精确定位的，所以ffmpeg可能是定位到最接近position（在之前）的可定位点。position可以是以秒为单位的数值或者 hh:mm:ss[.xxx] 格式的时间值。
- `-itsoffset offset (input)` 设置输入文件的时间偏移。offset 必须采用时间持续的方式指定，即可以有-号的时间值（以秒为单位的数值或者 hh:mm:ss[.xxx] 格式的时间值）。偏移会附加到输入文件的时间码上，意味着所指定的流会以时间码+偏移量作为最终输出时间码。
- `-timestamp date (output)` 设置在容器中记录时间戳。
- `-codec[:stream_specifier] codec (input/output,per-stream)` 为特定的文件选择编/解码模式，对于输出文件就是编码器，对于输入或者某个流就是解码器。选项参数中 codec 是编解码器的名字，或者是 copy（仅对输出文件）则意味着流数据直接复制而不再编码。

## 视频选项

- `-vframes number (output)` 设置输出文件的帧数，是 -frames:v 的别名。
- `-r[:stream_specifier] fps (input/output,per-stream)` 设置帧率（一种Hz值，缩写或者分数值）。
- `-s[:stream_specifier] size (input/output,per-stream)` 设置帧的尺寸。
  - `-s 1024x720` 指定输入视频的分辨率为1024x720，1024是宽度，720是高度
- `-vn (output)` 禁止输出视频。
- `-vcodec codec (output)` 设置视频编码器，这是 -codec:v 的一个别名。
- `aspect[:stream_specifier] aspect (output,per-stream)` 指定视频的纵横比（长宽显示比例）。aspect 是一个浮点数字符串或者num:den格式字符串(其值就是num/den)，例如"4:3","16:9","1.3333"以及"1.7777"都是常用参数值。
- `-b:v 640k `  设置输出视频的码率为640kbps

## 音频选项

- `-aframes number (output)` 设置 number 音频帧输出，是 -frames:a 的别名。
- `-ar[:stream_specifier] freq (input/output,per-stream)` 设置音频采样率。默认是输出同于输入。对于输入进行设置，仅仅通道是真实的设备或者raw数据分离出并映射的通道才有效。对于输出则可以强制设置音频量化的采用率。
- `-aq q (output)` 设置音频品质(编码指定为VBR)，它是 -q:a 的别名。
- `-ac[:stream_specifier] channels (input/output,per-stream)` 设置音频通道数。默认输出会有输入相同的音频通道。对于输入进行设置，仅仅通道是真实的设备或者raw数据分离出并映射的通道才有效。
- `-an (output)` 禁止输出音频。
- `-acode codec (input/output)` 设置音频解码/编码的编/解码器，是 -codec:a 的别名。

## 字幕选项

- `-scodec codec（input/output）` 设置字幕解码器，是 -codec:s 的别名。
- `-sn (output)` 禁止输出字幕。
- `-fix_sub_duration` 修正字幕持续时间。
- `-canvas_size size` 设置字幕渲染区域的尺寸（位置）。

## 转码优化

- **-preset**的参数主要调节编码速度和质量的平衡，有ultrafast（转码速度最快，视频往往也最模糊）、superfast、veryfast、[faster](https://so.csdn.net/so/search?q=faster&spm=1001.2101.3001.7020)、fast、medium、slow、slower、veryslow、placebo这10个选项，从快到慢。 
-  



# 命令实例

## 转化格式

```shell
ffmpeg -i input_test.mp4 -vn -acodec copy output_test.flv
ffmpeg -i input_test.aac -vn -acodec copy output_test.mp3
```

## 抽取画面中的音频

```shell
ffmpeg -i input_test.mp4 -vn -y -acodec copy output_test.aac
ffmpeg -i input_test.mp4 -vn -y -acodec copy output_test.mp3
ffmpeg -i input_test.mp4 -acodec copy -vn output_test.mp3
```

## 抽取画面中的视频

```shell
ffmpeg -i input_test.mp4 -vcodec copy -an output_test.avi
ffmpeg -i input_test.mp4 -vcodec copy -an output_test.mp4
```

##  音频+视频合成 

- `ffmpeg -i input_test_1.mp4 -i input_test_2.mp3 -vcodec copy -acodec copy output_test.mp4`
- `ffmpeg -i input_test_1.mp4 -itsoffset 10 -i input_test_2.mp3 -vcodec copy -acodec copy output_test.mp4`
- `ffmpeg -ss 20 -t 5 -i input_test_1.mp4 -i input_test_2.aac -vcodec copy -acodec copy output_test.mp4`
  音乐持续播放，视频只播放5秒
- `ffmpeg -ss 20 -t 5 -i input_test_1.mp3 -i input_test_2.mp4 -vcodec copy -acodec copy output_test.mp4`
  视频持续播放，音乐只播放5秒

##  音频+音频合成 

> 格式：ffmpeg -i INPUT1 -i INPUT2 -i INPUT3 -filter_complex  amix=inputs=3:duration=first:dropout_transition=3 OUTPUT

- `ffmpeg -i input_test_1.mp3 -i input_test_2.mp3 -filter_complex amix=inputs=2:duration=shortest output_test.mp3`
- `ffmpeg -i input_test_1.mp3 -i input_test_2.mp3 -filter_complex amix=inputs=2:duration=longest output_test.mp3`
- `ffmpeg –i input_test_1.mp3 –i input_test_2.mp3 –vcodec copy –acodec copy output_test.mp3`


## 视频分离成图片

- `ffmpeg -i input_test.mp4 -r 1 -f image2 output_image-%03d.jpeg`

- 截图图片的一帧

  ```shell
  ffmpeg -ss [时间戳] -i [输入视频文件] -frames:v 1 -update 1 -q:v 2 [输出图像文件]
  ```

  - `-ss [时间戳]`：指定要截取哪一帧的时间，格式为 `HH:MM:SS.sss`。例如，要截取第 5 秒的帧，可以使用 `00:00:05`。
  - `-i [输入视频文件]`：要截取帧的视频文件的路径。
  - `-vframes 1/-frames:v 1`：指定要截取的帧数。在这里，我们只截取一帧，所以设置为 1。
  - `-q:v 2`：指定输出图像的质量。数值越低，质量越高。在这里，我们使用 2，这是一个常用的中等质量设置。

## 图片合成视频

- `ffmpeg -f image2 -i output_image-%03d.jpeg output_test.mp4` 

## 改变音量大小

- `ffmpeg -i input_test.mp3 -af 'volume=0.5' output_test.mp3`

## 音效淡入淡出效果

- `ffmpeg -i input_test.mp3 -filter_complex afade=t=in:ss=0:d=4 output_test.mp3`
  淡入效果：把 input_test.mp3 文件的前5s做一个淡入淡出效果，输出到 output_test.mp3
  文件中
- `ffmpeg -i input_test.mp3 -filter_complex afade=t=out:st=20:d=6 output_test.mp3`
  淡出效果：将 input_test.mp3 文件从20s开始，做6s的淡出效果，输出到 output_test.mp3 文件中

## 截取音频

- `ffmpeg -ss 10 -i input_test.mp3 -to 20 -vcodec copy -acodec copy output_test.mp3`
- `ffmpeg -ss 10 -i input_test.mp3 -t 5 -vcodec copy -acodec copy output_test.mp3`
- `ffmpeg -i input_test.mp3 -c copy -t 10 -output_ts_offset 120 output_test.mp3`

## 容器时长获取

- `ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 -i input_test.mp3`

## 网络资源下载

- `ffmpeg -i https://xxx.xxx.xxxxxx -c copy -f mp3 output_test.mp3`


## 播放音频视频

- `ffplay input_test.mp3`


## 图片生成gif动图

- `ffmpeg -i input_image_%03d.png -r 5 output_test.gif`

## 抽取PCM数据

- `ffmpeg -i input_test.mp4 -vn -ar 44100 -ac 2 -f s16le output_test.pcm`

## 视频转码

> 参考：
>
> - https://zhuanlan.zhihu.com/p/675879599

- 转 `H.264` 编码

  ```shell
  ffmpeg -y -i [input.video] -c:v h264 -crf 23 -preset slow [output.video]
  ```

  - `-c:v`指定编码器。（也可以使用 libx264 编码器）这是一种高效率的H.264视频编码器
  -  `-pixel_format yuv420p`: 指定输入视频的像素格式为 yuv420p。这是一种常见的 YUV 格式，其中 Y 分量是完全采样的，而 U 和 V 分量则以 2x2 的子样本进行采样。 
  -  `-crf 23`: 指定编码质量因子为 23。CRF（Constant Rate Factor）是一种基于质量的编码控制方法，值越小表示更高的质量和比特率。常用的范围是 18-28。 