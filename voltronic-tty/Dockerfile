FROM debian:trixie

RUN apt update && apt install -y \
        curl \
        git \
        build-essential \
        cmake \
        jq \
        patch \
        mosquitto-clients

# Shared poller sources + default config come from the "common" submodule.
ADD common/sources/ /opt/
ADD common/config/ /etc/inverter/

# Configure the serial port in raw mode so CR terminators are not translated
# by the terminal line discipline (required for reliable Voltronic replies).
COPY serial-raw-mode.patch /tmp/serial-raw-mode.patch
RUN cd /opt && patch -p1 < /tmp/serial-raw-mode.patch && rm /tmp/serial-raw-mode.patch

RUN cd /opt/inverter-cli && \
    mkdir bin && cmake . && make && mv inverter_poller bin/

HEALTHCHECK \
    --interval=30s \
    --timeout=10s \
    --start-period=1m \
    --retries=3 \
  CMD /opt/healthcheck

WORKDIR /opt
ENTRYPOINT ["/bin/bash", "/opt/inverter-mqtt/entrypoint.sh"]
