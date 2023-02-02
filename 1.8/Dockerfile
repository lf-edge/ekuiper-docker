FROM golang:1.19-alpine AS builder

RUN apk add gcc make git libc-dev zip curl jq bash binutils-gold pkgconfig zeromq-dev && \
    mkdir /lib64 && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2

WORKDIR /go

ENV VSN=1.8.0
RUN curl --silent --show-error -kfL -o ekuiper.tar.gz "https://github.com/lf-edge/ekuiper/archive/refs/tags/${VSN}.tar.gz" && \
    mkdir -p ekuiper && \
    tar -zxf ekuiper.tar.gz -C ekuiper --strip-components 1 && \
    cd ekuiper && \
    make build_with_edgex

FROM alpine:3.16

# Set environment vars
ENV KUIPER_HOME="/kuiper" \
    KUIPER_USER="kuiper" \
    KUIPER_USER_ID="1001" \
    KUIPER__BASIC__CONSOLELOG=true

WORKDIR ${KUIPER_HOME}

RUN apk add sed libzmq

# Set appropriate ownership to allow binary full access to KUIPER_HOME dir
RUN adduser -DH -s /sbin/nologin -u ${KUIPER_USER_ID} ${KUIPER_USER} && \
    chown -Rh ${KUIPER_USER}:${KUIPER_USER} ${KUIPER_HOME} && \
    mkdir -p /usr/local/taos && \
    chown -Rh ${KUIPER_USER}:${KUIPER_USER} /usr/local/taos

# Run the kuiper process under the kuiper user
USER ${KUIPER_USER}

COPY --chown=${KUIPER_USER}:${KUIPER_USER} docker-entrypoint.sh /usr/bin/docker-entrypoint.sh
COPY --chown=${KUIPER_USER}:${KUIPER_USER} --from=builder /go/ekuiper/_build/kuiper-* /kuiper/

VOLUME ["${KUIPER_HOME}/etc", "${KUIPER_HOME}/data", "${KUIPER_HOME}/plugins", "${KUIPER_HOME}/log"]
EXPOSE 9081 20498

ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]

CMD ["./bin/kuiperd"]
