FROM ghcr.io/searxng/searxng:latest

RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone

COPY docker/searxng/limiter.toml /etc/searxng/limiter.toml
COPY docker/searxng/settings.yml /etc/searxng/settings.yml