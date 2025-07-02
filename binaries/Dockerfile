ARG GOLANG_VERSION=1.23

FROM golang:${GOLANG_VERSION}-trixie AS builder

ARG DEPENDENCIES="\
    dpkg-dev \
    git \
    make \
    pkg-config\
    libseccomp-dev \
    gcc"

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    set -ex \
    && rm -f /etc/apt/apt.conf.d/docker-clean \
    && echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' >/etc/apt/apt.conf.d/keep-cache \
    && apt-get update \
    && apt-get install -y ${DEPENDENCIES}

ARG ETCD_VERSION
ENV ETCD_VERSION=${ETCD_VERSION}

ARG WORKDIR=/opt/etcd

RUN set -ex \
    && git clone -b v${ETCD_VERSION} --depth=1 https://github.com/etcd-io/etcd ${WORKDIR}

ADD *.patch /opt/

WORKDIR ${WORKDIR}

RUN set -ex && \
    if [ -f server/etcdmain/etcd.go ]; then \
        sed -i '/runtime\.GOARCH == "amd64"/{N;/arm64/{N;s/runtime\.GOARCH == "arm64"/runtime.GOARCH == "arm64" ||\
        runtime.GOARCH == "loong64"/}}' server/etcdmain/etcd.go && \
        echo "✅ loong64 添加完成: server/etcdmain/etcd.go"; \
    else \
        echo "⚠️  跳过 loong64 插入，文件不存在：server/etcdmain/etcd.go"; \
    fi;

ENV ETCD_UNSUPPORTED_ARCH=loong64

RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    set -ex \
    && go mod tidy \
    && make build \
    && mkdir dist \
    && cp -f bin/* dist/

FROM debian:trixie-slim

WORKDIR /opt/etcd

COPY --from=builder /opt/etcd/dist /opt/etcd/dist

VOLUME /dist

CMD cp -rf dist/* /dist/
