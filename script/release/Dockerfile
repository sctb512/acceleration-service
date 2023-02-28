FROM golang:1.18.7 AS build

ARG CONTAINERD_VERSION=1.6.18
ARG NYDUS_VERSION=v2.1.0

ARG CONFIG_FILE="misc/config/config.yaml"

ENV GOPROXY=https://goproxy.cn

# Install acceld
COPY ./ /accel
RUN make -C /accel install-check-tools
RUN make -C /accel check
RUN make -C /accel

# Install containerd
RUN wget https://github.com/containerd/containerd/releases/download/v$CONTAINERD_VERSION/containerd-$CONTAINERD_VERSION-linux-amd64.tar.gz
RUN wget https://github.com/containerd/containerd/releases/download/$CONTAINERD_VERSION/containerd-$CONTAINERD_VERSION-linux-amd64.tar.gz.sha256sum
RUN sha256sum -c containerd-$CONTAINERD_VERSION-linux-amd64.tar.gz.sha256sum
RUN mkdir -p /usr/local/bin/ && tar xzvf containerd-$CONTAINERD_VERSION-linux-amd64.tar.gz -C /usr/local/
RUN ls -l /usr/local/bin/

# Install nydus component
RUN wget https://github.com/dragonflyoss/image-service/releases/download/$NYDUS_VERSION/nydus-static-$NYDUS_VERSION-linux-amd64.tgz
RUN wget https://github.com/dragonflyoss/image-service/releases/download/$NYDUS_VERSION/nydus-static-$NYDUS_VERSION-linux-amd64.tgz.sha256sum
RUN sha256sum -c nydus-static-$NYDUS_VERSION-linux-amd64.tgz.sha256sum
RUN tar xzvf nydus-static-$NYDUS_VERSION-linux-amd64.tgz && mv nydus-static/nydus-image /usr/local/bin/.

# Build acceld image
FROM photon:4.0
COPY --from=build /accel/acceld /accel/accelctl /usr/local/bin/containerd /usr/local/bin/nydus-image /usr/local/bin/
RUN mkdir /etc/containerd && /usr/local/bin/containerd config default > /etc/containerd/config.toml
COPY ./script/release/entrypoint.sh /entrypoint.sh
COPY $CONFIG_FILE "/config.yaml"

ENTRYPOINT [ "/entrypoint.sh", "/config.yaml" ]
