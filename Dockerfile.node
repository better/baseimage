FROM node:12-alpine3.10

LABEL \
maintainer="core-tech@better.com"

COPY rds-combined-ca-bundle.pem redshift-ca-bundle.crt /tmp/

# NOTE: keeping 'make' in the baseimage allows us to use it for builds
# TODO: write a separate "build" image, to keep the image light
RUN apk update                                                                                \
  && apk add --upgrade --no-cache coreutils openssl curl make                                 \
  && curl -o sops -L https://github.com/mozilla/sops/releases/download/3.3.1/sops-3.3.1.linux \
  && sha1sum sops | grep af2fc3d3a29565b0d6a73249136965ffee62892f                             \
  && chmod +x sops                                                                            \
  && mv sops /usr/local/bin                                                                   \
  && mkdir -p /etc/ssl                                                                        \
  && cp /tmp/rds-combined-ca-bundle.pem /tmp/redshift-ca-bundle.crt /etc/ssl

# Database SSL certificates
RUN cd /tmp                                                                                                       \
  && csplit --elide-empty-files --quiet --prefix rds-crt rds-combined-ca-bundle.pem '/-BEGIN CERTIFICATE-/' '{*}' \
  && csplit --elide-empty-files --quiet --prefix rs-crt redshift-ca-bundle.crt '/-BEGIN CERTIFICATE-/' '{*}'      \
  && for c in /tmp/rds-crt* ; do mv /$c /usr/local/share/ca-certificates/aws-rds-ca-$(basename $c).crt  ; done    \
  && for c in /tmp/rs-crt*  ; do mv /$c /usr/local/share/ca-certificates/redshift-ca-$(basename $c).crt ; done    \
  && update-ca-certificates

# Librdkafka for event service dependents TODO: split out / parameterize
RUN apk add --upgrade --no-cache --virtual .build-deps bash build-base linux-headers openssl-dev musl-dev python3 python3-dev \
  && curl -o librdkafka.zip -L https://github.com/edenhill/librdkafka/archive/v1.2.1.zip                                      \
  && echo "8b5e95318b190f40cbcd4a86d6a59dbe57b54a920d8fdf64d9c850bdf05002ca  librdkafka.zip" | sha256sum -c -                 \
  && unzip librdkafka.zip                                                                                                     \
  && rm librdkafka.zip                                                                                                        \
  && cd librdkafka-1.2.1                                                                                                      \
  && ./configure --prefix /usr                                                                                                \
  && make                                                                                                                     \
  && make install                                                                                                             \
  && cd ..                                                                                                                    \
  && rm -rf librdkafka-1.2.1                                                                                                  \
  && apk del .build-deps                                                                                                      \
  && rm -rf /var/cache/apk/*

# Check for CVE-2019-5021
RUN grep -F 'root:!::0:::::' /etc/shadow
