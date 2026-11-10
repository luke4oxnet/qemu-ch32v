# CI image for the EthRelax X7 Relays v2 firmware tests (tools/qemu_test.py):
# this repository's CH32V307 QEMU, plus the broker and test tools.
#
# Build from the repository root:
#   docker build -f ci/x7-test-image/Dockerfile -t x7-test-image .

FROM debian:trixie-slim AS build

RUN apt-get update -qq \
 && apt-get install -y --no-install-recommends \
      bash ca-certificates curl xz-utils patch python3-venv libfdt-dev

COPY . /src
WORKDIR /src
RUN bash build-wch-qemu-en.sh --prefix /opt/qemu-ch32v \
      --build-dir /tmp/qemu-build --downloads-dir /tmp/qemu-dl -j"$(nproc)"

# Record the Debian packages holding the libraries the emulator links to.
RUN ldd /opt/qemu-ch32v/bin/qemu-system-riscv32 \
      | awk '/=> \//{print $3}' | xargs realpath | xargs dpkg -S \
      | cut -d: -f1 | sort -u > /runtime-packages.txt

FROM debian:trixie-slim

COPY --from=build /opt/qemu-ch32v /opt/qemu-ch32v
COPY --from=build /runtime-packages.txt /tmp/runtime-packages.txt

RUN apt-get update -qq \
 && xargs -a /tmp/runtime-packages.txt apt-get install -y --no-install-recommends \
 && apt-get install -y --no-install-recommends \
      ca-certificates git mosquitto openssl python3 python3-paho-mqtt python3-pytest \
 && rm -rf /var/lib/apt/lists/* /tmp/runtime-packages.txt \
 && ln -s /opt/qemu-ch32v/bin/qemu-system-riscv32 /usr/local/bin/qemu-system-riscv32-ch32
