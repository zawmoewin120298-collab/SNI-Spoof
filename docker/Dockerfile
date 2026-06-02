FROM rust:bookworm AS builder
WORKDIR /build
COPY Cargo.toml Cargo.lock ./
COPY src/ src/
COPY data/ data/
RUN cargo build --release

FROM debian:bookworm-slim

# ပြုပြင်ချက် (၁) - ရောဂါအမြစ်ပြတ်ဖို့ libcap-bin ကိုပါ တစ်ခါတည်း Install လုပ်မယ်
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl unzip dnsutils python3 libcap-bin && \
    rm -rf /var/lib/apt/lists/*

RUN ARCH=$(dpkg --print-architecture) && \
    if [ "$ARCH" = "amd64" ]; then XRAY_ARCH="64"; \
    elif [ "$ARCH" = "arm64" ]; then XRAY_ARCH="arm64-v8a"; \
    else echo "unsupported arch: $ARCH" && exit 1; fi && \
    curl -sL "https://github.com/XTLS/Xray-core/releases/latest/download/Xray-linux-${XRAY_ARCH}.zip" -o /tmp/xray.zip && \
    cd /tmp && unzip -o xray.zip xray && chmod +x xray && mv xray /usr/local/bin/ && \
    rm -f /tmp/xray.zip /tmp/*.dat /tmp/LICENSE /tmp/README.md

COPY --from=builder /build/target/release/sni-spoof-rs /usr/local/bin/sni-spoof-rs

# ပြုပြင်ချက် (၂) - Root မဟုတ်တဲ့ Container ထဲမှာ Raw Socket ဖွင့်ခွင့် (CAP_NET_RAW) အာဏာကို binary ဆီ အသေအပ်နှင်းမယ်
RUN setcap cap_net_raw=+ep /usr/local/bin/sni-spoof-rs

RUN mkdir -p /etc/sni-spoof-rs /etc/xray

COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENV FAKE_SNI=security.vercel.com
ENV PROXY_PORT=1080
ENV SOCKS_PORT=1081

EXPOSE 1080 1081

ENTRYPOINT ["/entrypoint.sh"]
