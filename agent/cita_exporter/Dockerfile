# compile cita-cli with musl
FROM rust:1.48 AS builder
ARG BUILDPLATFORM

WORKDIR /
RUN wget https://www.musl-libc.org/releases/musl-1.1.19.tar.gz
RUN tar -xzf musl-1.1.19.tar.gz
RUN cd musl-1.1.19/ && ./configure && make && make install

COPY ./cita-cli.git /cita-cli.git
WORKDIR /cita-cli.git/cita-cli/
RUN sh -x -e -c '\
    target="x86_64-unknown-linux-musl"; \
	if [ "$BUILDPLATFORM" = "linux/arm64" ]; then target="aarch64-unknown-linux-musl"; fi; \
	rustup target add "$target"; \
    CC=/usr/local/musl/bin/musl-gcc RUSTFLAGS="-A deprecated" cargo install --target "$target" --path .'

# build the citamon/agent-cita-exporter docker image
FROM ubuntu:18.04 AS final
COPY --from=builder /usr/local/cargo/bin/cita-cli /bin/cita-cli
RUN chmod +x /bin/cita-cli
WORKDIR /config
COPY ./cita_monitor_agent.py ./requirements.txt /config/
RUN apt-get update \
        && apt-get install -y python3 python3-pip libcurl4-openssl-dev \
        && apt-get clean \
        && apt-get autoclean \
        && rm -rf /var/lib/apt/lists/*
RUN pip3 install -r requirements.txt
ENV PORT=1923
CMD "python3" "cita_monitor_agent.py" "$NODE_IP_PORT" "$PORT" "$NODE_DIR"
