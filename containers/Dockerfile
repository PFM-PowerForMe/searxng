FROM ghcr.io/searxng/searxng:latest

RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone

COPY containers/searxng/limiter.toml /etc/searxng/limiter.toml
COPY containers/searxng/settings.yml /etc/searxng/settings.yml