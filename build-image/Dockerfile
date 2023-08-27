FROM ubuntu:20.04

WORKDIR /root

RUN export TZ=Europe/Berlin

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

# common packages
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
    ca-certificates curl file \
    build-essential \
    git clang curl libssl-dev llvm libudev-dev cmake \
    autoconf automake autotools-dev libtool xutils-dev && \
    rm -rf /var/lib/apt/lists/*

ENV SSL_VERSION=1.0.2u

RUN curl https://www.openssl.org/source/openssl-$SSL_VERSION.tar.gz -O && \
    tar -xzf openssl-$SSL_VERSION.tar.gz && \
    cd openssl-$SSL_VERSION && ./config && make depend && make install && \
    cd .. && rm -rf openssl-$SSL_VERSION*

ENV OPENSSL_LIB_DIR=/usr/local/ssl/lib \
    OPENSSL_INCLUDE_DIR=/usr/local/ssl/include \
    OPENSSL_STATIC=1

# install toolchain
RUN curl https://sh.rustup.rs -sSf | \
    sh -s -- --default-toolchain stable -y

ENV PATH=/root/.cargo/bin:$PATH

RUN rustup default stable
RUN rustup update
RUN rustup update nightly
RUN rustup target add wasm32-unknown-unknown --toolchain nightly
RUN cargo install --git https://github.com/paritytech/cachepot
RUN rustup toolchain add nightly-aarch64-apple-darwin

RUN rustup component add rust-src --toolchain nightly-aarch64-apple-darwin && \
    apt-get update && \
    apt-get install wget -y && \
    wget https://github.com/WebAssembly/binaryen/releases/download/version_105/binaryen-version_105-arm64-macos.tar.gz && \
    tar -xvzf binaryen-version_105-arm64-macos.tar.gz && \
    cp binaryen-version_105/bin/* /bin/ && \
    cp binaryen-version_105/include/* /usr/include/ && \
    cp binaryen-version_105/lib/* /usr/lib/ && \
    cargo install cargo-dylint dylint-link && \
    cargo install --version 0.15.0 cargo-contract
