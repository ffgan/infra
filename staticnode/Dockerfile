FROM alpine:latest
ARG VERSION
# One of x64 or arm64 or ppc64le or riscv64
ARG ARCH
# The sha256sum for the node source tarball
ARG SHA256SUM

RUN echo "uname -a"

RUN mkdir -p /build
WORKDIR /build

RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "riscv64" ]; then \
        apk add --no-cache binutils curl g++ gcc gnupg libgcc linux-headers make python3 libstdc++; \
    else \
        apk add --no-cache binutils-gold curl g++ gcc gnupg libgcc linux-headers make python3 libstdc++; \
    fi
RUN mkdir -p /out/bin

RUN curl -O https://nodejs.org/dist/$VERSION/node-$VERSION.tar.gz
RUN echo "$SHA256SUM  node-$VERSION.tar.gz" | sha256sum -c -
RUN tar -zxf node-$VERSION.tar.gz

# Patch OpenSSL configuration for RISC-V to remove -m64 flags
RUN if [ "$ARCH" = "riscv64" ]; then \
        cd node-$VERSION && \
        sed -i 's/-m64//g' deps/openssl/config/Makefile || true; \
    fi

RUN cd node-$VERSION && \
    case "$ARCH" in \
        ppc64le) CPU=ppc64 ;; \
        riscv64) CPU=riscv64 ;; \
        *) CPU="$ARCH" ;; \
    esac && \
    if [ "$ARCH" = "riscv64" ]; then \
        # RISC-V specific configuration
        export CC_host=gcc && \
        export CXX_host=g++ && \
        ./configure \
            --dest-cpu=$CPU \
            --fully-static \
            --openssl-no-asm \
            --without-intl \
            --cross-compiling; \
    else \
        # Standard configuration for other architectures
        ./configure --dest-cpu=$CPU --fully-static; \
    fi && \
    make -j$(nproc)

RUN cp /build/node-$VERSION/LICENSE /out/LICENSE && cp /build/node-$VERSION/out/Release/node /out/bin/node

FROM scratch
COPY --from=0 /out/ /out