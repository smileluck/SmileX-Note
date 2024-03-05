[toc]

---

# windows

1. 进入下载网站：https://www.gyan.dev/ffmpeg/builds/
2. 选择版本下载。ffmpeg-6.1.1-essentials_build，这里用的是基础版本
3. 解压压缩包到本地
4. 配置环境变量Path即可
5. CMD 执行 ffmpeg -h 出现提示即安装成功

# Centos

## 扩展包

### libx264

H. 264视频编码器。有关更多信息和使用示例，请参阅[H. 264编码指南](https://trac.ffmpeg.org/wiki/Encode/H.264)。

ffmpeg 配置为 `--enable-gpl` `--enable-libx264.` 

```shell
git clone --branch stable --depth 1 https://code.videolan.org/videolan/x264.git
cd x264
./configure --enable-shared --enable-static
make && make install
```

### libx265

H. 265/HEVC 视频编码器。有关更多信息和使用示例，请参阅[H.265编码指南](https://trac.ffmpeg.org/wiki/Encode/H.265)。

要求将 ffmpeg 配置为 `--enable-gpl` `--enable-libx265.`

```shell
git clone --branch stable --depth 2 https://bitbucket.org/multicoreware/x265_git
cd x265
./configure --enable-shared --enable-static
make && make install
```

### libfdk_aac

AAC 音频编码器。有关更多信息和使用示例，请参阅[AAC 音频编码指南](https://trac.ffmpeg.org/wiki/Encode/AAC)。

要求将 ffmpeg 配置为 `--enable-libfdk_aac` (and `--enable-nonfree` if you also included `--enable-gpl`).

```shell
git clone --depth 1 https://github.com/mstorsjo/fdk-aac
cd fdk-aac
autoreconf -fiv
./configure --disable-shared
make && make install
```

### libmp3lame

MP3音频编码器。

要求将 ffmpeg 配置为 `--enable-libmp3lame`.

```shell
curl -O -L https://archive.mozilla.org/pub/opus/opus-1.3.1.tar.gz
tar xzvf opus-1.3.1.tar.gz
cd opus-1.3.1
./configure --disable-shared
make
make install
```

### libopus

Opus音频解码器和编码器。

要求将 ffmpeg 配置为 `--enable-libopus`.

```shell
curl -O -L https://archive.mozilla.org/pub/opus/opus-1.3.1.tar.gz
tar xzvf opus-1.3.1.tar.gz
cd opus-1.3.1
./configure --disable-shared
make
make install
```

### libvpx

VP8/VP9视频编码器和解码器。更多信息和使用实例，请参见[VP9视频编码指南](https://trac.ffmpeg.org/wiki/Encode/VP9)。

 要求将 ffmpeg 配置为 `--enable-libvpx`.

```shell
git clone --depth 1 https://chromium.googlesource.com/webm/libvpx.git
cd libvpx
./configure --disable-examples --disable-unit-tests --enable-vp9-highbitdepth --as=yasm
make
make install
```



## 编译安装

1. 从git上clone代码到本地

   - clone最新代码

     ```shell
     git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
     ```

   - clone指定tag代码

     ```shell
     git clone -b n6.1.1 --depth=1 https://git.ffmpeg.org/ffmpeg.git ffmpeg
     ```

     - `-b` 后面写上指定 版本标签 , 即 tag, 比如 n6.1.1
     - `--depth` 表示克隆深度, 1 表示只克隆最新的版本. 因为如果项目迭代的版本很多, 克隆会很慢 

   - 上面的办法clone下来的 ./configure 无法执行

2. 从github上下载

   ```shell
   wget https://codeload.github.com/FFmpeg/FFmpeg/tar.gz/refs/tags/n6.1.1
   ```

3. 安装依赖项目

	```shell
	yum -y install gcc automake autoconf libtool make
	```
	
4. 解压并安装

   - /usr/local/lib/pkgconfig 是编译时会自动存放一个.pc文件到该目录下
   
   ```shell
   tar -xzvf FFmpeg-n6.1.1.tar.gz
   cd FFmpeg-n6.1.1.tar.gz
   
   ./configure --prefix=/usr/local/ffmpeg --disable-autodetect --enable-gpl --enable-version3 --pkg-config-flags="--static" --enable-libwebp --enable-libx264

   make
   make install
   ```
   
   

# 异常提示

## nasm/yasm not found or too old. Use --disable-x86asm for a crippled build

- 问题描述

  -  nasm/yasm是汇编编译器，ffmpeg为了提高效率使用了汇编指令，如MMX和SSE等。所以系统中未安装nasm/yasm时，就会报上面错误。 

- 解决办法

  1. 安装yasm

  	- 官网地址：http://yasm.tortall.net/

  	- 下载解压并安装

        ```shell
        wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
        tar -xzvf yasm-1.3.0.tar.gz
        cd yasm-1.3.0 
        ./configure --prefix=/usr/bin
        make && make install
        ```
     
  2. 安装nasm

     ```shell
     wget --no-check-certificate https://www.nasm.us/pub/nasm/releasebuilds/2.16/nasm-2.16.tar.gz
     tar -xvf nasm-2.16.tar.gz
     cd nasm-2.16/
     ./configure --prefix=/usr/bin
     make && sudo make install
     ```

- 完整错误信息

  ```shell
  ./configure --prefix=/usr/local/
  nasm/yasm not found or too old. Use --disable-x86asm for a crippled build.
  
  If you think configure made a mistake, make sure you are using the latest
  version from Git.  If the latest version fails, report the problem to the
  ffmpeg-user@ffmpeg.org mailing list or IRC #ffmpeg on irc.libera.chat.
  Include the log file "ffbuild/config.log" produced by configure as this will help
  solve the problem.
  ```

  

## ERROR: libwebp >= 0.2.0 not found using pkg-config

- 问题描述

  ffmpeg ./configure --enable-libwebp 执行时抛出异常

- 解决办法

  ```shell
  yum install libwebp
  # or
  apt-get install libwebp
  
  # 不能解决,只能编译安装了
  sudo yum groupinstall "Development Tools"
  sudo yum install autoconf automake libtool
  git clone https://github.com/webmproject/libwebp.git
  ./autogen.sh
  ./configure
  make && sudo make install
  
  # 关键在于配置环境变量
  export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig:/usr/local/ssl/lib/pkgconfig
  
  ```

  

