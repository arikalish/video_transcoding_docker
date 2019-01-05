FROM arikalish/ruby-bionic:latest
LABEL maintainer="Ari Kalish <arikalish@gmail.com>"

ENV GEM_VERSION 0.22.0
ENV HANDBRAKE_VERSION 1.2.0

ENV LOCAL_PATH /data
ENV REMOTE_PATH //remote_path
ENV REMOTE_USER user
ENV REMOTE_PASSWORD password

# install build dependencies to compile ffmpeg from master
RUN set -ex \
  && buildDeps=' \
    autoconf \
    automake \
    build-essential \
    cmake \
    cmake-curses-gui \
    git \
    libass-dev \
    libbz2-dev \
    libfontconfig1-dev \
    libfreetype6-dev \
    libfribidi-dev \
    libharfbuzz-dev \
    libjansson-dev \
    liblzma-dev \
    libmp3lame-dev \
    libogg-dev \
    libopus-dev \
    libsamplerate-dev \
    libspeex-dev \
    libtheora-dev \
    libtool \
    libtool-bin \
    libvpx-dev \
    libvorbis-dev \
    libx264-dev \
    libxml2-dev \
    m4 \
    make \
    nasm \
    patch \
    pkg-config \
    python \
    tar \
    texinfo \
    wget \
    yasm \
    zlib1g-dev \
    mercurial \
    
    unzip \
    mkvtoolnix \
    mp4v2-utils \
    mpv \
    cifs-utils \
    nano \
  ' \
  && apt-get update \
  && apt-get install -y --no-install-recommends $buildDeps \
  && mkdir -p /usr/src/ffmpeg/bin \
  && mkdir -p /usr/src/ffmpeg/build \
  && PATH="/usr/src/ffmpeg/bin:$PATH" \
  && cd /usr/src/ffmpeg

  # NASM 2.14
  RUN wget https://www.nasm.us/pub/nasm/releasebuilds/2.14/nasm-2.14.tar.bz2 \
  && tar xjvf nasm-2.14.tar.bz2 \
  && cd nasm-2.14 \
  && ./autogen.sh \
  && PATH="/usr/src/ffmpeg/bin:$PATH" ./configure --prefix="/usr/src/ffmpeg/build" --bindir="/usr/src/ffmpeg/bin" \
  && PATH="/usr/src/ffmpeg/bin:$PATH" make -j"$(nproc)" \
  && make install \
  && cd /usr/src/ffmpeg \
  && rm -rf nasm-2.14*

  # libx264 stable
  RUN git clone -b stable http://git.videolan.org/git/x264.git x264 \
  && cd x264 \
  && PATH="/usr/src/ffmpeg/bin:$PATH" ./configure --prefix="/usr/src/ffmpeg/build" --bindir="/usr/src/ffmpeg/bin" --enable-static --enable-pic --disable-opencl \
  && PATH="/usr/src/ffmpeg/bin:$PATH" make -j"$(nproc)" \
  && make install \
  && cd /usr/src/ffmpeg \
  && rm -rf x264-snapshot*

  # libfdk-aac 0.1.6
  RUN wget -O fdk-aac.tar.gz https://github.com/mstorsjo/fdk-aac/archive/v0.1.6.tar.gz \
  && tar xzvf fdk-aac.tar.gz \
  && cd fdk-aac* \
  && autoreconf -fiv \
  && ./configure --prefix="/usr/src/ffmpeg/build" --disable-shared \
  && make -j"$(nproc)" \
  && make install \
  && cd /usr/src/ffmpeg \
  && rm -rf fdk-aac*

  # libx265 2.9
  RUN wget -O x265.tar.gz https://bitbucket.org/multicoreware/x265/downloads/x265_2.9.tar.gz \
  && tar xzvf x265.tar.gz \
  && cd x265_2.9/build/linux \
  && PATH="/usr/src/ffmpeg/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="/usr/src/ffmpeg/build" -DENABLE_SHARED:bool=off ../../source \
  && PATH="/usr/src/ffmpeg/bin:$PATH" make -j"$(nproc)" \
  && make install \
  && cd /usr/src/ffmpeg \
  && rm -rf x265*

  # HandbrakeCli Release
  RUN wget https://download2.handbrake.fr/$HANDBRAKE_VERSION/HandBrake-$HANDBRAKE_VERSION-source.tar.bz2 \
  && tar xvjf HandBrake-$HANDBRAKE_VERSION-source.tar.bz2 \
  && cd HandBrake-$HANDBRAKE_VERSION \
  && ./configure --launch-jobs=$(nproc) --enable-fdk --disable-gtk --launch \
  && cd build && make install \
  && cd /usr/src/ffmpeg \
  && rm -rf HandBrake-$HANDBRAKE_VERSION HandBrake-$HANDBRAKE_VERSION-source.tar.bz2
  
  # HandbrakeCli master
  #RUN git clone https://github.com/HandBrake/HandBrake.git \
  #&& cd HandBrake \
  #&& git checkout $HANDBRAKE_VERSION
  #&& ./configure --launch-jobs=$(nproc) --enable-fdk --disable-gtk --launch \
  #&& cd build && make install \
  #&& cd /usr/src/ffmpeg \
  #&& rm -rf HandBrake
  
  # FFmpeg 4.1
  RUN wget -O ffmpeg.tar.gz https://github.com/FFmpeg/FFmpeg/archive/n4.1.tar.gz \
  && tar zxvf ffmpeg.tar.gz \
  && mv FFmpeg* ffmpeg_src \
  && cd ffmpeg_src \
  && PATH="/usr/src/ffmpeg/bin:$PATH" PKG_CONFIG_PATH="/usr/src/ffmpeg/build/lib/pkgconfig" ./configure \
    --prefix="/usr/src/ffmpeg/build" \
    --pkg-config-flags="--static" \
    --extra-cflags="-I/usr/src/ffmpeg/build/include" \
    --extra-ldflags="-L/usr/src/ffmpeg/build/lib" \
    --bindir="/usr/src/ffmpeg/bin" \
    --extra-libs=-lpthread \
    --enable-gpl \
    --enable-libass \
    --enable-libfdk-aac \
    --enable-libfreetype \
    --enable-libmp3lame \
    --enable-libopus \
    --enable-libtheora \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libx264 \
    --enable-libx265 \
    --enable-nonfree \
  && PATH="/usr/src/ffmpeg/bin:$PATH" make -j"$(nproc)" \
  && make install \
  && hash -r \
  && cd / \
  && mv /usr/src/ffmpeg/bin/ff* /usr/local/bin \
  && rm -rf /usr/src/ffmpeg

  WORKDIR /
  
  RUN rm -rf HandBrake* \
  && rm -rf fdk* \
  && rm -rf ffmpeg* \
  && rm -rf nasm* \
  && rm -rf x265* \
  && rm -rf x264*
  
  RUN set -ex \
  # Install application dependencies
  && apt-get purge -y --auto-remove $buildDeps \
  && rm -rf /var/lib/apt/lists/* \
  && gem install video_transcoding -v "$GEM_VERSION"

COPY mount.sh /mount.sh

RUN chmod +x /mount.sh

ENTRYPOINT ["/mount.sh"]
