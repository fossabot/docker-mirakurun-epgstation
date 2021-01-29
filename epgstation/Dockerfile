FROM l3tnun/epgstation:master-debian
EXPOSE 8888

ENV DEV="make gcc git g++ automake curl wget autoconf build-essential libass-dev libfreetype6-dev libsdl1.2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo zlib1g-dev"
ARG NASM_VER="2.14"
ARG YASM_VER="1.3.0"
ARG LAME_VER="3.100"

RUN apt-get update && \
    apt-get -y install $DEV && \
    apt-get -y install yasm libx264-dev libmp3lame-dev libopus-dev libvpx-dev && \
    apt-get -y install libx265-dev libnuma-dev && \
    apt-get -y install libasound2 libass9 libvdpau1 libva-x11-2 libva-drm2 libxcb-shm0 libxcb-xfixes0 libxcb-shape0 libvorbisenc2 libtheora0 && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv BA9EF27F &&\
    echo 'deb http://ppa.launchpad.net/ubuntu-toolchain-r/test/ubuntu bionic main' \
        | tee /etc/apt-get/sources.list.d/ubuntu-toolchain-r-ubuntu-test-bionic.list && \
    echo 'deb http://deb.debian.org/debian buster-backports main' \
       | tee /etc/apt-get/sources.list.d/debian-buster-backports.list && \
    apt-get update && \
    apt-get install -y \
        wget build-essential automake autoconf git libtool libvorbis-dev \
        libass-dev libfreetype6-dev libsdl2-dev libva-dev libvdpau-dev \
        libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev \
        mercurial libnuma-dev texinfo zlib1g-dev \
        cmake qtbase5-dev checkinstall gcc-9 g++-9 curl \
        python3-pip ninja-build \
    &&\
    pip3 install meson

# nasm build
RUN set -xe && \
    DIR=/tmp/nasm && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sL https://www.nasm.us/pub/nasm/releasebuilds/${NASM_VER}/nasm-${NASM_VER}.tar.bz2 | \
    tar -jx --strip-components 1 && \
    ./autogen.sh && \
    ./configure --prefix=/usr/local && \
    make -j$(nproc) && \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

# yasm build
RUN set -xe && \
    DIR=/tmp/yasm && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sL https://www.tortall.net/projects/yasm/releases/yasm-${YASM_VER}.tar.gz | \
    tar -zx --strip-components 1 && \
    ./configure \
      --prefix=/usr/local && \
    make -j$(nproc) && \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

# l-smash build
RUN set -xe && \
    DIR=/tmp/l-smash && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone https://github.com/l-smash/l-smash.git && \
    cd l-smash && \
    ./configure \
      --enable-shared \
      && \
    make && \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

# libx264 build
RUN set -xe && \
    DIR=/tmp/x-264 && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone --depth 1 https://code.videolan.org/videolan/x264.git && \
    cd x264 && \
    ./configure \
      --prefix=/usr/local \
      --enable-shared \
      --enable-pic \
    && \
    make && \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

# libvpx build
RUN set -xe && \
    DIR=/tmp/libvpx && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone --depth 1 https://chromium.googlesource.com/webm/libvpx.git && \
    cd libvpx && \
    ./configure \
      --disable-examples \
      --disable-unit-tests \
      --enable-vp9-highbitdepth \
      --as=yasm \
    && \
    make -j$(nproc) && \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

# libfdk-aac build
RUN set -xe && \
    DIR=/tmp/libfdk-aac && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone --depth 1 https://github.com/mstorsjo/fdk-aac && \
    cd fdk-aac && \
    autoreconf -fiv && \
    ./configure \
    && \
    make -j$(nproc) && \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

# libmp3lame build
RUN set -xe && \
    DIR=/tmp/lame && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    curl -sL https://versaweb.dl.sourceforge.net/project/lame/lame/$(echo ${LAME_VER} | \
    sed -e 's/[^0-9]*\([0-9]*\)[.]\([0-9]*\)[.]\([0-9]*\)\([0-9A-Za-z-]*\)/\1.\2/')/lame-${LAME_VER}.tar.gz | \
    tar -zx --strip-components=1 && \
    ./configure \
      --enable-shared \
      --enable-nasm \
    && \
    make -j$(nproc) && \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

# libopus build
RUN set -xe && \
    DIR=/tmp/libopus && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone --depth 1 https://github.com/xiph/opus && \
    cd opus && \
    ./autogen.sh && \
    ./configure \
      --enable-shared \
    && \
    make -j$(nproc) && \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

#AviSynth+ build
RUN set -xe && \
    DIR=/tmp/AvisynthPlus && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone --depth 1 git://github.com/AviSynth/AviSynthPlus.git && \
    cd AviSynthPlus && \
    mkdir avisynth-build && \
    cd avisynth-build && \
    CC=gcc-9 CXX=gcc-9 LD=gcc-9 cmake ../ -G Ninja && \
    ninja && \
    checkinstall \
      --pkgname=avisynth \
      --pkgversion="$(grep -r \
      Version avs_core/avisynth.pc | cut -f2 -d " ")-$(date --rfc-3339=date | \
      sed 's/-//g')-git" \
      --backup=no \
      --deldoc=yes \
      --delspec=yes \
      --deldesc=yes \
      --strip=yes \
      --stripso=yes \
      --addso=yes \
      --fstrans=no \
      --default ninja install \
      && \
    rm -rf ${DIR} && \
    ldconfig

# FFmpeg
RUN set -xe && \
    DIR=/tmp/ffmpeg && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    sed -Ei 's/^# deb-src /deb-src /' /etc/apt-get/sources.list && \
    apt-get update && \
    apt-get -y build-dep ffmpeg && \
    git clone --depth 1 git://git.ffmpeg.org/ffmpeg.git && \
    cd ffmpeg && \
    ./configure \
      --cc=gcc-9 \
      --cxx=gcc-9 \
      --enable-gpl\
      --enable-version3 \
      --disable-doc \
      --disable-debug \
      --enable-pic \
      --enable-avisynth \
      --enable-libx264 \
      --enable-libfdk-aac \
      --enable-libfreetype \
      --enable-libmp3lame \
      --enable-libopus \
      --enable-libvorbis \
      --enable-libvpx \
      --enable-nonfree \
      && \
    make -j$(nproc) && \
    checkinstall --pkgname=ffmpeg --pkgversion="7:$(git rev-list \
    --count HEAD)-g$(git rev-parse --short HEAD)" --backup=no --deldoc=yes \
    --delspec=yes --deldesc=yes --strip=yes --stripso=yes --addso=yes \
    --fstrans=no --default \
    && \
    rm -rf ${DIR}

# l-smash-source
RUN set -xe && \
    DIR=/tmp/l-smash-source && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone https://github.com/HolyWu/L-SMASH-Works.git && \
    cd L-SMASH-Works/AviSynth && \
    git checkout 72d3eac802eebcfc9080009c1a8d47a747e3a306 &&\
    CC=gcc-9 CXX=gcc-9 LD=gcc-9 LDFLAGS="-Wl,-Bsymbolic" meson build && \
    cd build && \
    ninja -v && \
    ninja install && \
    rm -rf ${DIR} && \
    ldconfig

# delogo for Avisynth+ and Linux
RUN set -xe && \
    DIR=/tmp/delogo && \
    mkdir -p ${DIR} && \
    cd ${DIR} && \
    git clone https://github.com/tobitti0/delogo-AviSynthPlus-Linux.git && \
    cd delogo-AviSynthPlus-Linux/src && \
    make CC=gcc-9 CXX=gcc-9 LD=gcc-9&& \
    make install && \
    rm -rf ${DIR} && \
    ldconfig

# install nodejs
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
    apt-get install -y nodejs

# install EPGStation
RUN cd /usr/local/ && \
    git clone https://github.com/l3tnun/EPGStation.git && \
    cd /usr/local/EPGStation && \
    npm install && \
    npm install googleapis@39 --save && \
    npm install async && \
    npm run build

# join logo scp trial for Linux
ADD JoinLogoScpTrialSetLinux/modules /tmp/modules
RUN set -xe && \
    DIR=/tmp/modules && \
    cd /tmp/modules/chapt-geter_exe/src && \
    make && \
    mv chapt-geter_exe /tmp/modules/join_logo_scp_trial/bin/ && \
    cd /tmp/modules/logoframe/src && \
    make && \
    mv logoframe /tmp/modules/join_logo_scp_trial/bin/ && \
    cd /tmp/modules/join_logo_scp/src && \
    make && \
    mv join_logo_scp /tmp/modules/join_logo_scp_trial/bin/ && \    
    mv /tmp/modules/join_logo_scp_trial /usr/local/EPGStation/join_logo_scp_trial && \
    cd /usr/local/EPGStation/join_logo_scp_trial && \
    rm -rf ${DIR} && \
    npm install && \
    npm link && \
    jlse --help

WORKDIR /usr/local/EPGStation

ENTRYPOINT npm start

