FROM debian:bookworm-slim

# Add bookworm-backports in preparation for downloading newer openldap components, especially sladp
RUN echo "deb http://ftp.debian.org/debian bookworm-backports main" >> /etc/apt/sources.list

# Install locale en_US.UTF-8
RUN apt-get -y update \
    && apt-get -t bookworm-backports install -y --no-install-recommends locales \
    && sed -i -E 's/# (en_US.UTF-8)/\1/' /etc/locale.gen \
    && locale-gen

ENV LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    LC_ALL="en_US.UTF-8" \
    OPENLDAP_PACKAGE_VERSION="2.5.13" \
    DEBIAN_FRONTEND="noninteractive"

ARG LDAP_OPENLDAP_GID
ARG LDAP_OPENLDAP_UID

ARG PQCHECKER_VERSION=2.0.0
ARG PQCHECKER_MD5=c005ce596e97d13e39485e711dcbc7e1


# Add openldap user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
# If explicit uid or gid is given, use it.
RUN if [ -z "${LDAP_OPENLDAP_GID}" ]; then groupadd -g 911 -r openldap; else groupadd -r -g ${LDAP_OPENLDAP_GID} openldap; fi \
    && if [ -z "${LDAP_OPENLDAP_UID}" ]; then useradd -u 911 -r -g openldap openldap; else useradd -r -g openldap -u ${LDAP_OPENLDAP_UID} openldap; fi


# Install OpenLDAP, ldap-utils and ssl-tools from the (backported) baseimage and clean apt-get files
RUN apt-get -y update \
    && apt-get -t bookworm-backports install -y --no-install-recommends \
    ca-certificates \
    curl \
    ldap-utils=${OPENLDAP_PACKAGE_VERSION}\* \
    libsasl2-modules \
    libsasl2-modules-db \
    libsasl2-modules-gssapi-mit \
    libsasl2-modules-ldap \
    libsasl2-modules-otp \
    libsasl2-modules-sql \
    openssl \
    slapd=${OPENLDAP_PACKAGE_VERSION}\* \
    slapd-contrib=${OPENLDAP_PACKAGE_VERSION}\* \
    krb5-kdc-ldap \
    && curl -o pqchecker.deb -SL http://www.meddeb.net/pub/pqchecker/deb/8/pqchecker_${PQCHECKER_VERSION}_amd64.deb \
    && echo "${PQCHECKER_MD5} *pqchecker.deb" | md5sum -c - \
    && dpkg -i pqchecker.deb \
    && rm pqchecker.deb \
    && update-ca-certificates \
    && apt-get remove -y --purge --auto-remove curl ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Expose default ldap and ldaps ports
EXPOSE 389 636

# Put ldap config and database dir in a volume to persist data.
VOLUME ["/etc/ldap/slapd.d", "/var/lib/ldap"]
