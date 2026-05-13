背景：
我希望使用ffmpeg在power shell中将文件名为Vel0-Vel748制作成一个视频。
这是该软件的版本信息：
“ffmpeg version N-116328-gfa5a605542-20240718 Copyright (c) 2000-2024 the FFmpeg developers
  built with gcc 14.1.0 (crosstool-NG 1.26.0.93_a87bf7f)
  configuration: --prefix=/ffbuild/prefix --pkg-config-flags=--static --pkg-config=pkg-config --cross-prefix=x86_64-w64-mingw32- --arch=x86_64 --target-os=mingw32 --enable-gpl --enable-version3 --disable-debug --disable-w32threads --enable-pthreads --enable-iconv --enable-zlib --enable-libfreetype --enable-libfribidi --enable-gmp --enable-libxml2 --enable-fontconfig --enable-libharfbuzz --enable-libvorbis --enable-opencl --disable-libpulse --enable-libvmaf --disable-libxcb --disable-xlib --enable-amf --enable-libaom --enable-libaribb24 --enable-avisynth --enable-chromaprint --enable-libdav1d --enable-libdavs2 --enable-libdvdread --enable-libdvdnav --disable-libfdk-aac --enable-ffnvcodec --enable-cuda-llvm --enable-frei0r --enable-libgme --enable-libkvazaar --enable-libaribcaption --enable-libass --enable-libbluray --enable-libjxl --enable-libmp3lame --enable-libopus --enable-librist --enable-libssh --enable-libtheora --enable-libvpx --enable-libwebp --enable-lv2 --enable-libvpl --enable-openal --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenh264 --enable-libopenjpeg --enable-libopenmpt --enable-librav1e --enable-librubberband --enable-schannel --enable-sdl2 --enable-libsoxr --enable-libsrt --enable-libsvtav1 --enable-libtwolame --enable-libuavs3d --disable-libdrm --enable-vaapi --enable-libvidstab --enable-vulkan --enable-libshaderc --enable-libplacebo --enable-libx264 --enable-libx265 --enable-libxavs2 --enable-libxvid --enable-libzimg --enable-libzvbi --extra-cflags=-DLIBTWOLAME_STATIC --extra-cxxflags= --extra-libs=-lgomp --extra-ldflags=-pthread --extra-ldexeflags= --cc=x86_64-w64-mingw32-gcc --cxx=x86_64-w64-mingw32-g++ --ar=x86_64-w64-mingw32-gcc-ar --ranlib=x86_64-w64-mingw32-gcc-ranlib --nm=x86_64-w64-mingw32-gcc-nm --extra-version=20240718
  libavutil      59. 28.100 / 59. 28.100
  libavcodec     61. 10.100 / 61. 10.100
  libavformat    61.  5.101 / 61.  5.101
  libavdevice    61.  2.100 / 61.  2.100
  libavfilter    10.  2.102 / 10.  2.102
  libswscale      8.  2.100 /  8.  2.100
  libswresample   5.  2.100 /  5.  2.100
  libpostproc    58.  2.100 / 58.  2.100
Universal media converter
usage: ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...”

由于图片比较大和比较多，我希望隔帧取作为视频，此外，为了降低视频的大小，我希望长度方向裁剪一下，上下可略微裁剪。请给出相应的代码。


Response:
在 PowerShell 中使用 FFmpeg 将图片序列合成视频，并满足你提出的**“隔帧提取”**、**“画面裁剪”**以及**“降低视频大小”**的需求，可以通过一条完整的 FFmpeg 命令来实现。

由于你没有提供图片的扩展名，下面的代码默认你的图片是 `.png` 格式（如果是 `.jpg` 或其他格式，请自行在代码中替换）。

### 核心代码

请在包含这些图片的文件夹中打开 PowerShell，然后运行以下命令：

```powershell
ffmpeg -framerate 30 -i 'Vel%d.png' -vf "framestep=2,setpts=N/FRAME_RATE/TB,crop=iw*0.8:ih*0.9,scale=trunc(iw/2)*2:trunc(ih/2)*2,format=yuv420p" -c:v libx264 -crf 28 output.mp4
```

---

### 代码参数详细解释（你可以根据需求调整）：

#### 1. 输入部分
*   `-framerate 30`：设置输入图片的帧率（假设原本是每秒播放30张图）。
*   `-i 'Vel%d.png'`：读取图片序列。在 PowerShell 中，建议用单引号 `' '` 包裹，防止 `%d` 被识别为变量。`%d` 代表自动匹配从 `Vel0.png` 到 `Vel748.png` 的数字序列。

#### 2. 视频滤镜部分 (`-vf`)
滤镜之间用逗号 `,` 隔开，依次执行：
*   `framestep=2`：**【隔帧提取】**。这会每隔1帧提取1帧（即保留第 0, 2, 4, 6... 帧），直接减少一半的图片处理量。如果你想每隔2帧取一次，可以改成 `framestep=3`。
*   `setpts=N/FRAME_RATE/TB`：修复抽帧后的时间戳，保证视频能以正常的帧率平滑播放。
*   `crop=iw*0.8:ih*0.9`：**【画面裁剪】**。
    *   `iw` 是原图宽度，`ih` 是原图高度。
    *   `iw*0.8` 表示宽度裁剪掉 20%（左右各裁 10%），`ih*0.9` 表示高度裁剪掉 10%（上下各裁 5%）。
    *   **自定义裁剪建议**：如果你想指定裁剪掉具体的像素，比如左右各裁掉 200 像素，上下各裁掉 100 像素，可以改成 `crop=iw-400:ih-200`（FFmpeg 会默认从画面正中心向四周扩展裁剪）。
*   `scale=trunc(iw/2)*2:trunc(ih/2)*2`：**【防止报错】**。H.264 编码器要求视频的宽和高必须是 **偶数**。这个指令会自动将裁剪后的尺寸调整为最近的偶数，防止报错。
*   `format=yuv420p`：强制使用 yuv420p 像素格式，这是为了保证生成的视频能在所有播放器（如网页、手机）上正常播放。

#### 3. 编码与压缩部分（降低视频大小）
*   `-c:v libx264`：使用 H.264 编码器（兼容性最好）。
*   `-crf 28`：**【控制视频大小与画质】**。CRF 是恒定质量控制，范围是 0-51。默认值是 23。**数字越大，视频文件越小，画质越低**。如果你觉得 28 画质太差，可以改回 24-26；如果觉得文件还是太大，可以调高到 30-32。

---

### 备用方案：如果图片太大导致 FFmpeg 内存溢出

如果你的图片单张极大（比如 4K/8K 分辨率），FFmpeg 直接读取 700 多张可能会很慢甚至卡死。你可以利用 PowerShell **先将偶数帧的图片复制到一个新文件夹**，然后再进行合成。

在 PowerShell 中按顺序执行以下命令：

```powershell
# 1. 创建一个存放挑选后图片的文件夹
New-Item -ItemType Directory -Force -Path ".\SelectedFrames"

# 2. 挑选出所有偶数编号的图片并复制过去 (隔帧)
0..748 | Where-Object { $_ % 2 -eq 0 } | ForEach-Object {
    $fileName = "Vel$_.png"
    if (Test-Path $fileName) {
        Copy-Item -Path $fileName -Destination ".\SelectedFrames\"
    }
}

# 3. 进入新文件夹
cd .\SelectedFrames

# 4. 由于编号不连续了，使用 -pattern_type glob 读取并裁剪合成
ffmpeg -framerate 15 -pattern_type glob -i '*.png' -vf "crop=iw*0.8:ih*0.9,scale=trunc(iw/2)*2:trunc(ih/2)*2,format=yuv420p" -c:v libx264 -crf 28 output.mp4
```

*注意：第二种方案的 FFmpeg 命令中我去掉了 `framestep`，并将输入帧率改为了 15（因为图片少了一半）。如果你的 FFmpeg 版本不支持 glob 模式，依然可以使用第一种纯 FFmpeg 的方案。*

要实现**“左边多裁、右边少裁”**（不对称裁剪）以及**“视频加速”**，我们可以通过调整 `crop` 滤镜的**起始坐标**，并修改**抽帧比例**或**播放帧率**来实现。

下面是修改后的 PowerShell 完整代码：

```powershell
ffmpeg -framerate 30 -i 'Vel%d.png' -vf "framestep=4,setpts=N/FRAME_RATE/TB,crop=iw*0.8:ih*0.9:iw*0.15:ih*0.05,scale=trunc(iw/2)*2:trunc(ih/2)*2,format=yuv420p" -c:v libx264 -crf 28 output.mp4
```

---

### 参数调整详细解释：

#### 1. 如何实现“左边多裁，右边少裁”？
在 FFmpeg 中，`crop` 滤镜的完整格式是 `crop=宽度:高度:X坐标:Y坐标`。
代码中我改为了 `crop=iw*0.8:ih*0.9:iw*0.15:ih*0.05`，它的运作原理如下：
*   **宽度 (`iw*0.8`)**：保留原图 80% 的宽度，意味着总共要**裁掉 20%**。
*   **高度 (`ih*0.9`)**：保留原图 90% 的高度，意味着总共要**裁掉 10%**。
*   **X坐标 (`iw*0.15`)**：这是关键！它规定裁剪框从距离左边缘 **15%** 的位置开始。因此，**左边被固定裁掉了 15%**。因为总共要裁 20%，所以**右边自然只裁掉了 5%**，完美实现左多右少。
*   **Y坐标 (`ih*0.05`)**：裁剪框从距离顶部 5% 的位置开始。因此顶部裁掉 5%，底部裁掉 5%（上下对称略微裁剪）。

*(你可以根据需要随意修改上面的小数，比如想左边裁去 20%，右侧裁 10%，那就写 `crop=iw*0.7:ih*0.9:iw*0.20:ih*0.05`)*

#### 2. 如何实现“视频加速”？
因为你的输入是“图片序列”，视频的“快慢”完全取决于图片的消耗速度。有两种方法可以加速，上面提供的代码使用了**方法一**：

*   **方法一：加大跳帧跨度（推荐，代码中使用的就是这种）**
    将原本的 `framestep=2` 改为了 `framestep=4`（每隔 4 帧取 1 帧）。
    *效果*：总图片被抛弃了 3/4，只剩下大约 187 张图。以 30 帧/秒 播放时，只需约 6.2 秒播完。这不仅让**视觉效果加速了一倍**，而且极大地**节省了处理时间并进一步缩小了视频体积**。

*   **方法二：提高播放帧率（如果你的图片间隔动作比较小，想要极致丝滑）**
    如果你不想舍弃那么多图片，又想加速，可以把开头的 `-framerate 30` 提高。
    比如改成 `-framerate 60`。
    代码如下：
    ```powershell
    ffmpeg -framerate 60 -i 'Vel%d.png' -vf "framestep=2,setpts=N/FRAME_RATE/TB,crop=iw*0.8:ih*0.9:iw*0.15:ih*0.05,scale=trunc(iw/2)*2:trunc(ih/2)*2,format=yuv420p" -c:v libx264 -crf 28 output.mp4
    ```
    *效果*：依然保留一半图片（约 374 张），但因为播放器一秒钟播放 60 张，所以同样只用 6.2 秒播完，视频动作变快，且画面非常流畅。缺点是视频体积会比方法一稍微大一点。
