FROM ubuntu:focal

# non root user (https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user)
ARG USERNAME=dev
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

USER $USERNAME

# All deps ----------------------------------
RUN sudo apt-get update -o Acquire::Retries=30 && sudo DEBIAN_FRONTEND=noninteractive \
    apt-get install -o Acquire::Retries=30 -y --no-install-recommends \
    gcc make gcc-arm-linux-gnueabi binutils-arm-linux-gnueabi \
    gcc-arm-linux-gnueabihf gcc-aarch64-linux-gnu \
    libglew-dev libgtkmm-3.0-dev gdb libssl-dev valgrind curl nano \
    openjdk-11-jdk wget tar unzip lib32stdc++6 \
    lib32z1 git file build-essential python python3 \
    autoconf automake libtool curl make g++ \
    libva-dev libvdpau-dev llvm-dev libclang-dev clang libc6-dev-i386 \
    lldb libasio-dev libmbedtls-dev libssl-dev liblz4-dev liblzo2-dev \
    libboost-all-dev socat iproute2 iptables dnsutils ninja-build net-tools \
    htop tree

ENV OPT_DIR /opt

WORKDIR /tmp

ENV PROJECT_ROOT_DIR="/home/dev/project"

# Android SDK/NDK ----------------------------------
# Attention: ANDROID_NDK_VERSION is used by supernova_app/android/app/build.gradle
ENV ANDROID_HOME=${OPT_DIR}/android-sdk-linux
ENV PATH=${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools:$PATH \
    SDKMANAGER=${ANDROID_HOME}/cmdline-tools/bin/sdkmanager \
    ANDROID_NDK_VERSION=22.1.7171670
ENV ANDROID_NDK_HOME=${ANDROID_HOME}/ndk/${ANDROID_NDK_VERSION} \
    ANDROID_SHA256_HASH="7a00faadc0864f78edd8f4908a629a46d622375cbe2e5814e82934aebecdb622" \
    ANDROID_SDK_URL="https://dl.google.com/android/repository/commandlinetools-linux-7302050_latest.zip" 

RUN set -ex && sudo mkdir ${ANDROID_HOME} && curl -sSL -o android-sdk.zip ${ANDROID_SDK_URL} \
    && echo "${ANDROID_SHA256_HASH} *android-sdk.zip" | shasum -a 256 --check \
    && TEMPD=$(mktemp -d) \
    && unzip android-sdk.zip -d ${TEMPD} && rm android-sdk.zip \
    && sudo mv ${TEMPD}/* ${ANDROID_HOME}/ \
    && rm -rf ${TEMPD} \
    && sudo chown $USERNAME:$USERNAME $ANDROID_HOME -R \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} --list \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "build-tools;25.0.1" \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "build-tools;28.0.3" \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "build-tools;29.0.2" \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "build-tools;30.0.3" \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "platform-tools" \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "platforms;android-25" \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "platforms;android-28" \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "platforms;android-30" \ 
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "extras;android;m2repository" \
    && echo y | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "extras;google;google_play_services" \
    && ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "ndk;${ANDROID_NDK_VERSION}" \
    && ${SDKMANAGER} --sdk_root=${ANDROID_HOME} "cmake;3.18.1" \
    && yes | ${SDKMANAGER} --sdk_root=${ANDROID_HOME} --update \
    && sudo chown root:root $ANDROID_HOME -R

#Flutter installation ----------------------------------
ENV FLUTTER_ROOT="${OPT_DIR}/flutter/development/flutter" \
    FLUTTER_SDK_URL="https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_2.10.4-stable.tar.xz" \
    FLUTTER_SHA256_HASH="7ad769690f1a0fd1e432c7d4c265eba946237aa94241de5b53f92ff31ddc82d2"
ENV PATH="${FLUTTER_ROOT}/bin:${PATH}"

RUN sudo mkdir -p ${OPT_DIR}/flutter/development \
    && TEMPD=$(mktemp -d) \
    && cd $TEMPD \
    && curl --progress-bar -sSL -o flutter_sdk.tar.xz ${FLUTTER_SDK_URL} \
    && echo "${FLUTTER_SHA256_HASH} *flutter_sdk.tar.xz" | shasum -a 256 --check \
    && mkdir flutter && tar -xf flutter_sdk.tar.xz -C flutter --strip-components 1 \
    && rm flutter_sdk.tar.xz \
    && sudo mv * ${OPT_DIR}/flutter/development \
    && cd .. && rm -rf $TEMPD


# Go to https://github.com/rust-lang/docker-rust for updates
ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH \
    RUST_VERSION=1.60.0

RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='3dc5ef50861ee18657f9db2eeb7392f9c2a6c95c90ab41e45ab4ca71476b4338' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='67777ac3bc17277102f2ed73fd5f14c51f4ca5963adadf7f174adf4ebc38747b' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='32a1532f7cef072a667bac53f1a5542c99666c4071af0c9549795bbdb2069ec1' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='e50d1deb99048bc5782a0200aa33e4eea70747d49dffdc9d06812fd22a372515' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.24.3/${rustArch}/rustup-init"; \
    wget "$url"; \
    echo "${rustupSha256} *rustup-init" | sha256sum -c -; \
    chmod +x rustup-init; \
    sudo mkdir -p /usr/local/cargo/bin; \
    sudo mkdir -p /usr/local/rustup; \
    sudo chown $USERNAME:$USERNAME /usr/local/cargo -R; \
    sudo chown $USERNAME:$USERNAME /usr/local/rustup -R; \
    ./rustup-init -y --no-modify-path --profile minimal --default-toolchain $RUST_VERSION --default-host ${rustArch}; \
    rm rustup-init; \
    sudo chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version; \
    cargo --version; \
    rustc --version; \
    sudo chown root:root /usr/local/cargo -R; \
    sudo chown root:root /usr/local/rustup -R 


RUN rustup install stable \
    && rustup target add \
       aarch64-linux-android \
       armv7-linux-androideabi \
       i686-linux-android \
       x86_64-linux-android \
    && rustup component add rust-analysis --toolchain stable-x86_64-unknown-linux-gnu \
    && rustup component add rust-src --toolchain stable-x86_64-unknown-linux-gnu \
    && rustup component add rls --toolchain stable-x86_64-unknown-linux-gnu 

RUN sudo ln -s ${ANDROID_NDK_HOME} /opt/RUST_NDK_TOOLCHAIN

RUN rustup component add clippy && rustup component add rustfmt

RUN mkdir -p /home/dev/project

USER root

WORKDIR /home/dev/project
