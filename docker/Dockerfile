ARG ARCH=loong64
FROM ghcr.io/loong64/debian:trixie

ADD dist/* /usr/local/bin/

WORKDIR /var/etcd/
WORKDIR /var/lib/etcd/

EXPOSE 2379 2380

# Define default command.
CMD ["/usr/local/bin/etcd"]
