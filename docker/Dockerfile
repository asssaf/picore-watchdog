# syntax=docker/dockerfile-upstream:master
FROM debian:stable AS build

ARG ARCH

RUN apt-get update \
        && apt-get install -y build-essential crossbuild-essential-${ARCH} squashfs-tools curl \
        && apt-get clean

ENV CC=${ARCH/armhf/arm-linux-gnueabihf-}
ENV CC=${CC/arm64/aarch64-linux-gnu-}

ARG WATCHDOG_VERSION=5.16

RUN mkdir /work
WORKDIR /work
RUN curl -sLo - "https://sourceforge.net/projects/watchdog/files/watchdog/${WATCHDOG_VERSION}/watchdog-${WATCHDOG_VERSION}.tar.gz/download" | tar xvz

WORKDIR watchdog-${WATCHDOG_VERSION}

ENV CC="${CC}gcc"
RUN ./configure --prefix=/usr/local
RUN make

RUN make DESTDIR=extension -C src install
RUN mv src/extension/usr/local/sbin/watchdog src/extension/usr/local/sbin/wd_watchdog

ARG TCZ=watchdog.tcz
RUN mksquashfs src/extension /${TCZ} -all-root -noappend

WORKDIR /
RUN unsquashfs -lc ${TCZ} | sed 's|^squashfs-root/||' > ${TCZ}.list
RUN md5sum ${TCZ} > ${TCZ}.md5.txt

FROM scratch AS export

ARG TCZ=watchdog.tcz

COPY --from=build /${TCZ} /
COPY --from=build /${TCZ}.list /
COPY --from=build /${TCZ}.md5.txt /
