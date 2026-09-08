FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install --yes --no-install-recommends \
        ca-certificates \
        curl \
        python3 \
        sudo \
    && rm -rf /var/lib/apt/lists/*

CMD ["sleep", "infinity"]
