FROM alpine:3.9

LABEL \
maintainer="ivan@better.com"

ENV USER_ID=1000 USER=app GOPATH=/go LC_ALL=en_US.utf-8
ENV PATH="${GOPATH}/bin:/usr/local/go/bin:${PATH}" DIR="/home/${USER}"

# System setup
RUN addgroup -g ${USER_ID} ${USER} && adduser -D -G ${USER} -u ${USER_ID} ${USER}

# System dependencies
RUN apk add --no-cache \
  linux-headers coreutils ca-certificates openssl openssl-dev musl-dev g++ python3 python3-dev git curl bash build-base && \
  wget https://github.com/mozilla/sops/releases/download/3.3.1/sops-3.3.1.linux -O sops && \
  sha1sum sops | grep af2fc3d3a29565b0d6a73249136965ffee62892f && \
  chmod +x sops && mv sops /usr/local/bin

# Database SSL certificates
COPY rds-combined-ca-bundle.pem redshift-ca-bundle.crt /tmp/
RUN mkdir -p /etc/ssl && cp /tmp/rds-combined-ca-bundle.pem /tmp/redshift-ca-bundle.crt /etc/ssl

# todo: write a script for this?
RUN cd /tmp && \
  csplit --elide-empty-files --quiet --prefix rds-crt rds-combined-ca-bundle.pem '/-BEGIN CERTIFICATE-/' '{*}' && \
  for c in /tmp/rds-crt*; \
  do mv /$c /usr/local/share/ca-certificates/aws-rds-ca-$(basename $c).crt; done
RUN cd /tmp && \
  csplit --elide-empty-files --quiet --prefix rs-crt redshift-ca-bundle.crt '/-BEGIN CERTIFICATE-/' '{*}' && \
  for c in /tmp/rs-crt*; \
  do mv /$c /usr/local/share/ca-certificates/redshift-ca-$(basename $c).crt; done

RUN update-ca-certificates

# Librdkafka for event service dependents
RUN curl -L https://github.com/edenhill/librdkafka/archive/v1.2.1.zip > librdkafka.zip && \
  echo "8b5e95318b190f40cbcd4a86d6a59dbe57b54a920d8fdf64d9c850bdf05002ca  librdkafka.zip" | sha256sum -c - && \
  unzip librdkafka.zip && \
  rm librdkafka.zip && \
  cd librdkafka-1.2.1 && \
  ./configure --prefix /usr && \
  make && \
  make install

# Application dependencies
RUN python3 -m ensurepip && pip3 install --upgrade pip setuptools

# Check for CVE-2019-5021
RUN grep -F 'root:!::0:::::' /etc/shadow
