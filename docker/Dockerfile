FROM rust:bookworm AS builder
WORKDIR /build

# Railway က docker/ ဖိုဒါထဲကနေ Build လုပ်ရင် အပြင်က Rust ဖိုင်တွေကို ဆွဲယူနိုင်အောင် လမ်းကြောင်းပြင်ထားပါသည်
COPY ../Cargo.toml ../Cargo.lock ./
COPY ../src/ ./src/
COPY ../data/ ./data/
RUN cargo build --release

FROM debian:bookworm-slim

# ပြုပြင်ချက် (၁) - Railway Network Timeout ကျော်ဖြတ်ရန် Cloudflare Mirror သို့ ပြောင်းလဲခြင်း
RUN sed -i 's/deb.debian.org/mirrors.cloudflare.com/g' /etc/apt/sources.list.d/debian.sources || \
    sed -i 's/deb.debian.org/mirrors.cloudflare.com/g' /etc/apt/sources.list

# ပြုပြင်ချက် (၂) - IPv4 ကို အတင်းသုံးပြီး Retry (၃) ကြိမ်ဖြင့် libcap-bin ကိုပါ တစ်ပါတည်း Install လုပ်ခြင်း
RUN apt-get -o Acquire::ForceIPv4=true -o Acquire::Retries=3 update && \
    apt-get -o Acquire::ForceIPv4=true -o Acquire::Retries=3 install -y --no-install-recommends \
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

# ပြုပြင်ချက် (၃) - Root မဟုတ်သော်လည်း Raw Socket သုံးခွင့်ရအောင် အာဏာပေးအပ်ခြင်း
RUN setcap cap_net_raw=+ep /usr/local/bin/sni-spoof-rs

RUN mkdir -p /etc/sni-spoof-rs /etc/xray

# Dockerfile နှင့် တစ်ဖိုဒါတည်းရှိသော entrypoint.sh ကို တိုက်ရိုက်ဆွဲယူခြင်း
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENV FAKE_SNI=security.vercel.com
ENV PROXY_PORT=1080
ENV SOCKS_PORT=1081

EXPOSE 1080 1081

ENTRYPOINT ["/entrypoint.sh"]

