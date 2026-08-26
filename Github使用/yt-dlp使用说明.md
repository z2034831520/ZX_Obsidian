# yt-dlp 使用说明

yt-dlp 是一个命令行音视频下载工具，支持 YouTube 等大量网站。本文记录当前电脑上的安装位置、YouTube 下载方法，以及使用 FFmpeg 对下载结果进行转码的完整流程。

> [!important]
> 只下载自己有权保存的内容，并遵守对应网站的服务条款和版权规定。

## 1. 当前环境

检查日期：2026-08-26。

- yt-dlp 版本：`2026.08.19`
- yt-dlp 程序：`D:\Video Download\yt-dlp\yt-dlp.exe`
- 视频保存目录：`D:\Video Download\YouTube`
- FFmpeg 版本：`8.0-essentials_build`
- FFmpeg 程序：`E:\ffmpeg-8.0-essentials_build\bin\ffmpeg.exe`
- FFprobe 程序：`E:\ffmpeg-8.0-essentials_build\bin\ffprobe.exe`
- Deno：当前未检测到

当前 `ffmpeg` 和 `ffprobe` 已经加入 PATH，可以在 PowerShell 中直接执行。

### 安装 Deno

yt-dlp 官方独立版已经包含 YouTube 所需的 `yt-dlp-ejs`，但完整的 YouTube 支持还需要 JavaScript 运行时，官方推荐 Deno。

安装命令：

```powershell
winget install --id DenoLand.Deno --exact
```

安装完成后重新打开 PowerShell，然后验证：

```powershell
deno --version
```

## 2. 打开 yt-dlp

### 方法一：进入程序目录

```powershell
Set-Location -LiteralPath "D:\Video Download\yt-dlp"
```

后续通过以下形式运行：

```powershell
.\yt-dlp.exe --version
```

### 方法二：使用 PowerShell 变量

不切换目录也可以使用绝对路径：

```powershell
$YtDlp = 'D:\Video Download\yt-dlp\yt-dlp.exe'
& $YtDlp --version
```

关闭 PowerShell 后，该变量会失效。以后重新打开终端时，需要再次定义。

## 3. 下载 YouTube 视频

YouTube 链接应放在英文双引号中，避免链接中的 `&` 被 PowerShell 当成特殊字符。

### 下载单个视频

```powershell
$YtDlp = 'D:\Video Download\yt-dlp\yt-dlp.exe'
& $YtDlp -P "D:\Video Download\YouTube" --no-playlist "YouTube视频链接"
```

没有指定画质时，yt-dlp 会选择可用的最佳视频和音频，并调用 FFmpeg 完成合并。`--no-playlist` 可以防止带播放列表参数的单视频链接意外下载整个列表。

### 优先保存为 MP4

```powershell
& $YtDlp -P "D:\Video Download\YouTube" -t mp4 --no-playlist "YouTube视频链接"
```

`-t mp4` 是 yt-dlp 的 MP4 预设，会优先选择 H.264 视频与 AAC 音频，并在需要时转换封装。它的兼容性通常比默认下载得到的 WebM 更好。

### 只下载 MP3 音频

```powershell
& $YtDlp -P "D:\Video Download\YouTube" -t mp3 --no-playlist "YouTube视频链接"
```

此操作需要 FFmpeg。它不是直接修改扩展名，而是提取并转换音频。

## 4. 选择画质和格式

先查看当前视频提供的格式：

```powershell
& $YtDlp -F "YouTube视频链接"
```

输出中会显示格式编号、分辨率、帧率、编码和是否包含音频。例如，要把编号为 `137` 的视频流与编号为 `140` 的音频流合并，可以执行：

```powershell
& $YtDlp -P "D:\Video Download\YouTube" -f "137+140" --no-playlist "YouTube视频链接"
```

格式编号由网站针对每个视频提供，不同视频的编号和可用格式可能不同，不能把某组编号当作永久固定值。

## 5. 下载播放列表

下载整个播放列表，并按列表顺序编号：

```powershell
& $YtDlp `
  -P "D:\Video Download\YouTube" `
  --yes-playlist `
  -o "%(playlist)s/%(playlist_index)03d - %(title)s.%(ext)s" `
  "YouTube播放列表链接"
```

如果只想下载播放列表链接中当前打开的视频，使用：

```powershell
& $YtDlp -P "D:\Video Download\YouTube" --no-playlist "YouTube视频链接"
```

## 6. 下载字幕

先查看可用字幕及其语言代码：

```powershell
& $YtDlp --list-subs "YouTube视频链接"
```

下载人工字幕和自动字幕：

```powershell
& $YtDlp `
  -P "D:\Video Download\YouTube" `
  --write-subs `
  --write-auto-subs `
  --sub-langs "zh-Hans,zh-Hant,zh.*,en.*" `
  --no-playlist `
  "YouTube视频链接"
```

下载后将字幕嵌入视频：

```powershell
& $YtDlp `
  -P "D:\Video Download\YouTube" `
  --write-subs `
  --write-auto-subs `
  --sub-langs "zh-Hans,zh-Hant,zh.*,en.*" `
  --embed-subs `
  --no-playlist `
  "YouTube视频链接"
```

不是所有视频都提供中文字幕。应先使用 `--list-subs` 查看实际存在的语言代码。

## 7. 下载需要登录的内容

对于自己有权访问、但必须登录才能观看的内容，可以让 yt-dlp 读取本机浏览器的登录 Cookie。

读取 Edge：

```powershell
& $YtDlp --cookies-from-browser edge -P "D:\Video Download\YouTube" --no-playlist "YouTube视频链接"
```

读取 Chrome：

```powershell
& $YtDlp --cookies-from-browser chrome -P "D:\Video Download\YouTube" --no-playlist "YouTube视频链接"
```

如果浏览器 Cookie 数据库被占用，可以先完全关闭对应浏览器再重试。Cookie 包含登录凭据，不要分享 Cookie 文件、包含 Cookie 的调试输出或个人下载日志。

## 8. 设置默认下载目录

可以在 yt-dlp 程序旁边创建配置文件：

```text
D:\Video Download\yt-dlp\yt-dlp.conf
```

推荐的基础内容：

```text
-P "D:/Video Download/YouTube"
--windows-filenames
--no-playlist
```

配置完成后，下载单个视频时可以简化为：

```powershell
Set-Location -LiteralPath "D:\Video Download\yt-dlp"
.\yt-dlp.exe "YouTube视频链接"
```

命令行中明确提供的选项可以覆盖或补充配置。下载播放列表时仍需显式加入 `--yes-playlist`。

## 9. 更新和诊断

更新官方独立版：

```powershell
& $YtDlp -U
```

如果网站更新导致稳定版暂时无法下载，可以切换到官方 nightly 渠道：

```powershell
& $YtDlp --update-to nightly
```

显示版本、依赖和详细诊断信息：

```powershell
& $YtDlp -vU "YouTube视频链接"
```

排错时应先更新 yt-dlp，再检查 FFmpeg、Deno、网络和登录状态。发布日志或诊断输出中可能包含视频地址、文件路径等个人信息，分享前应检查并脱敏。

## 10. 转码前检查视频

进入下载目录：

```powershell
Set-Location -LiteralPath "D:\Video Download\YouTube"
```

查看视频、音频编码和封装信息：

```powershell
ffprobe -hide_banner "原视频.webm"
```

转码前需要区分两种操作：

1. **更换封装**：视频和音频编码保持不变，速度快、没有画质损失，但目标容器不一定支持原编码。
2. **重新编码**：把视频和音频转换为指定编码，耗时更长，可能产生画质损失，但兼容性更好。

仅把 `.webm` 后缀改成 `.mp4` 不属于格式转换。

## 11. 无损更换封装

快速转换成 MKV，不重新编码：

```powershell
ffmpeg -i "原视频.webm" -c copy "更换封装后.mkv"
```

如果原视频本身使用 H.264 视频和 AAC 音频，也可以快速封装成 MP4：

```powershell
ffmpeg -i "原视频.mkv" -c copy -movflags +faststart "更换封装后.mp4"
```

如果出现“目标容器不支持该编码”之类的错误，说明不能只更换封装，需要使用下一节的重新编码命令。

## 12. 转换为通用 MP4

推荐的通用格式是：MP4 容器、H.264 视频、AAC 音频、`yuv420p` 像素格式。

```powershell
ffmpeg `
  -i "原视频.webm" `
  -map 0:v:0 `
  -map "0:a?" `
  -c:v libx264 `
  -preset medium `
  -crf 23 `
  -pix_fmt yuv420p `
  -c:a aac `
  -b:a 192k `
  -movflags +faststart `
  "转换后视频.mp4"
```

主要参数：

- `-map 0:v:0`：选择第一条视频流。
- `-map "0:a?"`：存在音频时选择音频，没有音频也不报错。
- `libx264`：转换为兼容性较好的 H.264。
- `-preset medium`：编码速度和压缩效率之间的平衡。
- `-crf 23`：控制 H.264 画质；数值越小，画质越好、文件越大。
- `yuv420p`：提高电视、手机和旧播放器的兼容性。
- `aac`：转换为常用 AAC 音频。
- `+faststart`：把 MP4 索引移动到文件前部，便于在线播放和快速打开。

常用 CRF 范围：

| 需求 | CRF 建议值 | 特点 |
| --- | ---: | --- |
| 高画质 | 18 | 文件较大 |
| 日常保存 | 23 | 画质和体积平衡 |
| 更小体积 | 26 | 压缩更明显 |

不要反复对同一个有损视频进行多次转码。需要不同格式时，尽量每次都从原始下载文件生成。

## 13. 限制到 1080p

当原视频高于 1080p、希望降低分辨率和文件体积时：

```powershell
ffmpeg `
  -i "原视频.webm" `
  -vf "scale=-2:1080" `
  -c:v libx264 `
  -preset medium `
  -crf 23 `
  -pix_fmt yuv420p `
  -c:a aac `
  -b:a 192k `
  -movflags +faststart `
  "1080p视频.mp4"
```

`-2` 会根据原始宽高比自动计算宽度，并保证编码需要的偶数尺寸。原视频不高于 1080p 时通常没有必要放大。

## 14. 提取音频

提取为 MP3：

```powershell
ffmpeg -i "原视频.mp4" -vn -c:a libmp3lame -b:a 192k "提取音频.mp3"
```

提取为无压缩 WAV：

```powershell
ffmpeg -i "原视频.mp4" -vn -c:a pcm_s16le "提取音频.wav"
```

`-vn` 表示不输出视频流。WAV 文件通常明显大于 MP3。

## 15. 批量把 WebM 转换为 MP4

下面的命令会处理下载目录中的所有 `.webm` 文件，并把同名 MP4 保存在原目录中：

```powershell
Get-ChildItem -LiteralPath "D:\Video Download\YouTube" -File -Filter "*.webm" |
  ForEach-Object {
    $OutputFile = Join-Path $_.DirectoryName ($_.BaseName + ".mp4")
    ffmpeg `
      -n `
      -i $_.FullName `
      -map 0:v:0 `
      -map "0:a?" `
      -c:v libx264 `
      -preset medium `
      -crf 23 `
      -pix_fmt yuv420p `
      -c:a aac `
      -b:a 192k `
      -movflags +faststart `
      $OutputFile
  }
```

`-n` 表示目标文件已经存在时跳过，不覆盖原有 MP4。需要覆盖文件时可以使用 `-y`，但执行前必须确认目标文件确实可以被替换。

## 16. 验证转码结果

转码完成后查看输出文件信息：

```powershell
ffprobe `
  -v error `
  -show_entries "stream=index,codec_type,codec_name,width,height" `
  -show_entries "format=format_name,duration,size" `
  -of default=noprint_wrappers=1 `
  "转换后视频.mp4"
```

还应使用实际播放器检查：

1. 视频能否从头到尾正常播放。
2. 图像和声音是否同步。
3. 分辨率、字幕和音轨是否符合预期。
4. 文件体积是否合理。

确认输出文件没有问题以后，再决定是否删除原始视频。转码命令默认不会删除输入文件。

## 17. 常见问题

### PowerShell 无法识别 yt-dlp

使用完整路径调用：

```powershell
& 'D:\Video Download\yt-dlp\yt-dlp.exe' --version
```

路径包含空格时必须使用引号；通过变量或字符串路径启动程序时，需要在前面使用调用运算符 `&`。

### PowerShell 无法识别 ffmpeg

当前 FFmpeg 的完整路径是：

```text
E:\ffmpeg-8.0-essentials_build\bin\ffmpeg.exe
```

可以先验证：

```powershell
ffmpeg -version
ffprobe -version
```

### 下载结果是 WebM

WebM 是正常的视频容器，不代表下载失败。需要更广泛兼容性时，可以下载时使用 `-t mp4`，或者下载后通过 FFmpeg 转换为 H.264/AAC MP4。

### 提示需要登录或确认不是机器人

先更新 yt-dlp，并确认 Deno 已安装。对于自己有权访问的登录内容，再使用 `--cookies-from-browser edge` 或 `--cookies-from-browser chrome`。

### 转码速度慢

重新编码需要解码和压缩整个视频，耗时取决于视频长度、分辨率和处理器性能。`-c copy` 很快，但只能在目标容器兼容原始编码时使用。

## 18. 推荐操作流程

```text
检查 yt-dlp、FFmpeg 和 Deno
→ 用 --no-playlist 下载单个视频
→ 需要兼容性时优先使用 -t mp4
→ 下载完成后用 ffprobe 检查编码
→ 能兼容则使用 -c copy 更换封装
→ 不兼容则转为 H.264/AAC MP4
→ 用 ffprobe 和播放器验证结果
→ 确认无误后再处理原文件
```

## 参考资料

- [yt-dlp GitHub 仓库](https://github.com/yt-dlp/yt-dlp)
- [yt-dlp 安装说明](https://github.com/yt-dlp/yt-dlp/wiki/Installation)
- [yt-dlp 外部 JavaScript 运行时说明](https://github.com/yt-dlp/yt-dlp/wiki/EJS)
- [yt-dlp 格式选择说明](https://github.com/yt-dlp/yt-dlp#format-selection)
- [FFmpeg 官方文档](https://ffmpeg.org/documentation.html)
