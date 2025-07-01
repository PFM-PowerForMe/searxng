FROM alpine:edge AS git
RUN apk update && apk add --no-cache \
    python3-dev \
    py3-yaml \
	git
WORKDIR /tmp/
RUN git -c advice.detachedHead=false clone --branch main --depth=1 --recurse-submodules --shallow-submodules https://github.com/pfm-powerforme/searxng.git searxng
WORKDIR /tmp/searxng/
RUN git update-index -q --refresh \
	&& python -m searx.version freeze \
	&& eval "$(python -m searx.version)" \
	&& mkdir -p PFM_VAR \
	&& echo "$(git log -1 --format='%cd' --date=unix -- ./searx/settings.yml)" > PFM_VAR/TIMESTAMP_SETTINGS \
	&& echo "$(git log -1 --format='%cd' --date=unix -- ./container/config/uwsgi.ini)" > PFM_VAR/TIMESTAMP_UWSGI
	

FROM alpine:edge AS builder
### Base Start ###
RUN apk update && apk add --no-cache \
    alpine-base \
    build-base \
    python3-dev \
    py3-pip \
    brotli \
    libffi-dev
ENV PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
    SSL_CERT_FILE="/etc/ssl/certs/ca-certificates.crt" \
    HISTFILE="/dev/null"
WORKDIR /usr/local/searxng/
### Base End ###
COPY --from=git /tmp/searxng/requirements.txt requirements.txt
COPY --from=git /tmp/searxng/searx/ searx/
COPY --from=git /tmp/searxng/PFM_VAR/ PFM_VAR/
RUN --mount=type=cache,id=pip,target=/root/.cache/pip python -m venv venv \
 && . venv/bin/activate \
 && pip install -r requirements.txt \
 && pip install "uwsgi~=2.0"
RUN python -m compileall -q searx \
 && touch -c --date=@$(cat PFM_VAR/TIMESTAMP_SETTINGS) searx/settings.yml \
 && find searx/static \
    \( -name "*.html" -o -name "*.css" -o -name "*.js" -o -name "*.svg" -o -name "*.ttf" -o -name "*.eot" \) \
    -type f -exec gzip -9 -k {} + -exec brotli --best {} +


FROM alpine:edge AS runtime
### Base Start ###
RUN apk update && apk add --no-cache \
    alpine-baselayout \
    ca-certificates-bundle \
    busybox \
    python3 \
    wget \
    libxml2 \
    mailcap
RUN addgroup -g 977 searxng && \
    adduser -D -u 977 -G searxng -s /bin/ash searxng
RUN mkdir -p /usr/local/searxng/ /etc/searxng/ /var/cache/searxng/ && \
    chown searxng:searxng /usr/local/searxng/ /etc/searxng/ /var/cache/searxng/ && \
    chmod 0555 /usr/local/searxng/ && \
    chmod 0755 /etc/searxng/ /var/cache/searxng/
ENV PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
    SSL_CERT_FILE="/etc/ssl/certs/ca-certificates.crt" \
    HISTFILE="/dev/null" \
    CONFIG_PATH="/etc/searxng" \
    DATA_PATH="/var/cache/searxng"
WORKDIR /usr/local/searxng/
### Base End ###
COPY --chown=searxng:searxng --from=builder /usr/local/searxng/venv/ venv/
COPY --chown=searxng:searxng --from=builder /usr/local/searxng/searx/ searx/
COPY --chown=searxng:searxng --from=git /tmp/searxng/container/config/ .template/
COPY --chown=searxng:searxng --from=git /tmp/searxng/container/entrypoint.sh entrypoint.sh
COPY --chown=searxng:searxng --from=git /tmp/searxng/PFM_VAR/ PFM_VAR/
RUN touch -c --date=@$(cat PFM_VAR/TIMESTAMP_UWSGI) .template/uwsgi.ini
COPY searxng/limiter.toml /etc/searxng/limiter.toml
COPY searxng/settings.yml /etc/searxng/settings.yml
ENV SEARXNG_VERSION="Latest" \
	INSTANCE_NAME="SearXNG" \
    AUTOCOMPLETE="" \
    BASE_URL="" \
    BIND_ADDRESS="[::]:8080" \
    SEARXNG_SETTINGS_PATH="$CONFIG_PATH/settings.yml" \
    UWSGI_SETTINGS_PATH="$CONFIG_PATH/uwsgi.ini" \
    UWSGI_WORKERS="%k" \
    UWSGI_THREADS="4"
VOLUME $CONFIG_PATH
VOLUME $DATA_PATH
EXPOSE 8080
ENTRYPOINT ["/usr/local/searxng/entrypoint.sh"]