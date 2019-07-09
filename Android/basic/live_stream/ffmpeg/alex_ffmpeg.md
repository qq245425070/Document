ffmpeg-4.1.3 对应 ndk-r17 以上否则编译会出错的;  

```
#!/bin/bash
NDK=/home/alex/Android/sdk/android-ndk-r14b
ADDI_LDFLAGS="-fPIE -pie"
ADDI_CFLAGS="-fPIE -pie -march=armv7-a -mfloat-abi=softfp -mfpu=neon"
CPU=armv7-a
ARCH=arm
HOST=arm-linux
SYSROOT=$NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot
CROSS_PREFIX=$NDK/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-androideabi21-
PREFIX=$(pwd)/android/$CPU
x264=$(pwd)/x264/android/$CPU

configure()
{
    ./configure \
    --prefix=$PREFIX \
    --disable-encoders \
    --disable-decoders \
    --disable-avdevice \
    --disable-static \
    --disable-doc \
    --disable-ffplay \
    --disable-network \
    --disable-doc \
    --disable-symver \
    --enable-neon \
    --enable-shared \
    --enable-libx264 \
    --enable-gpl \
    --enable-pic \
    --enable-jni \
    --enable-pthreads \
    --enable-mediacodec \
    --enable-encoder=aac \
    --enable-encoder=gif \
    --enable-encoder=libopenjpeg \
    --enable-encoder=libmp3lame \
    --enable-encoder=libwavpack \
    --enable-encoder=libx264 \
    --enable-encoder=mpeg4 \
    --enable-encoder=pcm_s16le \
    --enable-encoder=png \
    --enable-encoder=srt \
    --enable-encoder=subrip \
    --enable-encoder=yuv4 \
    --enable-encoder=text \
    --enable-decoder=aac \
    --enable-decoder=aac_latm \
    --enable-decoder=libopenjpeg \
    --enable-decoder=mp3 \
    --enable-decoder=mpeg4_mediacodec \
    --enable-decoder=pcm_s16le \
    --enable-decoder=flac \
    --enable-decoder=flv \
    --enable-decoder=gif \
    --enable-decoder=png \
    --enable-decoder=srt \
    --enable-decoder=xsub \
    --enable-decoder=yuv4 \
    --enable-decoder=vp8_mediacodec \
    --enable-decoder=h264_mediacodec \
    --enable-decoder=hevc_mediacodec \
    --enable-ffmpeg \
    --enable-bsf=aac_adtstoasc \
    --enable-bsf=h264_mp4toannexb \
    --enable-bsf=hevc_mp4toannexb \
    --enable-bsf=mpeg4_unpack_bframes \
    --enable-cross-compile \
    --cross-prefix=$CROSS_PREFIX \
    --target-os=android \
    --arch=$ARCH \
    --sysroot=$SYSROOT \
    --extra-cflags="-I$x264/include $ADDI_CFLAGS" \
    --extra-ldflags="-L$x264/lib $ADDI_LDFLAGS"
}

build()
{
    make clean
    
    configure
    make -j4
    make install
}

build


```


