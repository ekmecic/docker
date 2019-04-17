FROM alpine:edge

WORKDIR /root

ARG LLVM_FULL_VERSION=8.0.0
ARG CLANG_BINARY=clang-8

COPY $PATCH /root/$PATCH

# install build dependencies

RUN apk update && apk upgrade \
    && apk add --no-cache ca-certificates \
    && rm -rf /var/cache/apk/* \
# install core tools
    && apk add --no-cache libc-dev gcc g++ make \
# install bash for convenience
   && apk add --no-cache bash \
# install git to fetch submodules
   && apk add --no-cache git \
# install buildsystem
   && apk add --no-cache cmake \
# LLVM+clang dependencies
   && apk update && apk add --no-cache python3 ninja \
# download source code
    && wget -q http://releases.llvm.org/$LLVM_FULL_VERSION/llvm-$LLVM_FULL_VERSION.src.tar.xz -O llvm.tar.xz \
    && wget -q http://releases.llvm.org/$LLVM_FULL_VERSION/cfe-$LLVM_FULL_VERSION.src.tar.xz  -O clang.tar.xz \
# extract
    && tar -xf llvm.tar.xz && tar -xf clang.tar.xz \
    && mv llvm-$LLVM_FULL_VERSION.src llvm-src && mv cfe-$LLVM_FULL_VERSION.src llvm-src/tools/clang \
# build
    && mkdir llvm-build && mkdir /opt/llvm \
    && cd llvm-build \
    && cmake -G"Ninja" -DCMAKE_INSTALL_PREFIX="/opt/llvm" -DCMAKE_BUILD_TYPE=Release \
                       -DLLVM_HOST_TRIPLE="x86_64-alpine-linux-musl" \
                       -DLIBCLANG_BUILD_STATIC=ON -DLLVM_BUILD_EXAMPLES=OFF -DLLVM_BUILD_TESTS=OFF \
                        ../llvm-src \
    && cmake --build . \
    && cmake --build . --target install \
    # static libclang library is not installed, so have to do manually
    && find . -name 'libclang.a' -exec cp {} /opt/llvm/lib \; \
    && cd ../ \
# remove build files
    && rm -r llvm-src/ llvm-build/ \
    && rm llvm.tar.xz clang.tar.xz \
# remove shared libraries that were built anyway
    && find /opt/llvm/lib \( -type f -or -type l \) \( -name '*.so' -or -name '*.so.*' \) -delete \
# remove build dependencies
    && apk del -r python3 ninja \

# update PATH
ENV PATH="/opt/llvm/bin:${PATH}"
