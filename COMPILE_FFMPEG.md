sudo yum install autoconf automake bzip2 bzip2-devel cmake freetype-devel gcc gcc-c++ git libtool make mercurial pkgconfig zlib-devel -y
mkdir ~/ffmpeg_sources

# nasm
cd ~/ffmpeg_sources
curl -O -L https://www.nasm.us/pub/nasm/releasebuilds/2.14.02/nasm-2.14.02.tar.bz2
tar xjvf nasm-2.14.02.tar.bz2
cd nasm-2.14.02
./autogen.sh
./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-shared
make
make install

# yasm
cd ~/ffmpeg_sources
curl -O -L http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
tar xzvf yasm-1.3.0.tar.gz
cd yasm-1.3.0
./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-shared
make
make install

yum install nasm yasm

# libxh264
cd ~/ffmpeg_sources
git clone --depth 1 https://code.videolan.org/videolan/x264
cd x264
PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-static --enable-shared
make
make install

# libx265
cd ~/ffmpeg_sources
git clone https://github.com/videolan/x265
cd ~/ffmpeg_sources/x265/build/linux
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=on ../../source
make
make install

# libfdk_aac
cd ~/ffmpeg_sources
git clone --depth 1 https://github.com/mstorsjo/fdk-aac
cd fdk-aac
autoreconf -fiv
./configure --prefix="$HOME/ffmpeg_build" --enable-shared
make
make install

# libmp3lame
cd ~/ffmpeg_sources
curl -O -L https://downloads.sourceforge.net/project/lame/lame/3.100/lame-3.100.tar.gz
tar xzvf lame-3.100.tar.gz
cd lame-3.100
./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-nasm --enable-shared
make
make install

# libopus
cd ~/ffmpeg_sources
curl -O -L https://archive.mozilla.org/pub/opus/opus-1.3.tar.gz
tar xzvf opus-1.3.tar.gz
cd opus-1.3
./configure --prefix="$HOME/ffmpeg_build" --enable-shared
make
make install

# libvpx
cd ~/ffmpeg_sources
git clone --depth 1 https://chromium.googlesource.com/webm/libvpx.git
cd libvpx
./configure --prefix="$HOME/ffmpeg_build" --enable-shared --disable-examples --disable-unit-tests --enable-vp9-highbitdepth --as=yasm
make
make install

# ffmpeg
cd ~/ffmpeg_sources
# curl -O -L https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2
wget https://ffmpeg.org/releases/ffmpeg-4.4.tar.bz2
tar xjvf ffmpeg-snapshot.tar.bz2
cd ffmpeg-4.4
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$HOME/ffmpeg_build" \
  --enable-shared --disable-static
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include -fPIC" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib -Wl,-Bsymbolic" \
  --extra-libs=-lpthread \
  --extra-libs=-lm \
  --bindir="$HOME/bin" \
  --enable-gpl \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libvpx \
  --enable-libx264 \
  --enable-libx265 \
  --enable-nonfree \
  --enable-shared \
  --enable-pic \
  --disable-doc \
  --disable-encoders \
#   --disable-static

make -j16
make install
hash -d ./