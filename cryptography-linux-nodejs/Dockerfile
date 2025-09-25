ARG NODE20_ARCH_RELEASE
ARG NODE24_ARCH_RELEASE
ARG BASE_IMAGE_TAG
FROM ghcr.io/pyca/static-nodejs-${NODE20_ARCH_RELEASE} AS staticnodejs20
FROM ghcr.io/pyca/static-nodejs-${NODE24_ARCH_RELEASE} AS staticnodejs24
FROM ghcr.io/pyca/${BASE_IMAGE_TAG}
LABEL org.opencontainers.image.authors="Python Cryptographic Authority"
WORKDIR /root

COPY --from=staticnodejs20 /out/ /staticnode/20/
COPY --from=staticnodejs24 /out/ /staticnode/24/