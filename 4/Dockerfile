FROM registry.access.redhat.com/ubi9/ubi

STOPSIGNAL SIGINT

RUN	dnf -y update && \
    dnf install -y \
      php-cli \
      php-fpm \
      php-mbstring \
      php-pdo \
      php-pgsql \
    && dnf clean all

RUN echo "upload_max_filesize = 128M" >> /etc/php.ini \
    && echo "post_max_size = 128M" >> /etc/php.ini \
    && echo "memory_limit = 1G" >> /etc/php.ini \
    && echo "max_execution_time = 600" >> /etc/php.ini \
    && echo "max_input_vars = 5000" >> /etc/php.ini \
    && echo "variables_order = \"EGPCS\"" >> /etc/php.ini

RUN groupadd -r adminer \
    && useradd -r -g adminer adminer \
    && mkdir -p /var/www/html \
    && mkdir /var/www/html/plugins-enabled \
    && chown -R adminer:adminer /var/www/html

WORKDIR /var/www/html

COPY	*.php /var/www/html/

ENV	ADMINER_VERSION 4.8.1
ENV	ADMINER_DOWNLOAD_SHA256 2fd7e6d8f987b243ab1839249551f62adce19704c47d3d0c8dd9e57ea5b9c6b3
ENV	ADMINER_COMMIT 1f173e18bdf0be29182e0d67989df56eadea4754

RUN buildDeps='git' \
    && dnf -y update \
    && dnf -y install $buildDeps --setopt=install_weak_deps=False \
    && echo "$ADMINER_DOWNLOAD_SHA256  adminer.php" | sha256sum -c - \
    && git clone --recurse-submodules=designs --depth 1 --shallow-submodules --branch "v$ADMINER_VERSION" https://github.com/vrana/adminer.git /tmp/adminer \
    && commit="$(git -C /tmp/adminer/ rev-parse HEAD)" \
    && [ "$commit" = "$ADMINER_COMMIT" ] \
    && cp -r /tmp/adminer/designs/ /tmp/adminer/plugins/ . \
    && rm -rf /tmp/adminer/ \
    && dnf -y remove $buildDeps \
    && dnf clean all

COPY	entrypoint.sh /usr/local/bin/
ENTRYPOINT	[ "entrypoint.sh" ]

USER	adminer
CMD	[ "php", "-S", "[::]:8080", "-t", "/var/www/html" ]

EXPOSE 8080
