FROM redhat/ubi8-minimal

LABEL name="Percona PostgreSQL Distribution" \
    vendor="Percona" \
    summary="Percona Distribution for PostgreSQL" \
    description="Percona Distribution for PostgreSQL is a collection of tools to assist you in managing your PostgreSQL database system" \
    maintainer="Percona Development <info@percona.com>"

RUN microdnf -y update; \
    microdnf -y install glibc-langpack-en
# platform-python-pip is removed due to CVE-2019-20916, VULNDB-229216
# python3-pip-wheel is required by platform-python
RUN set -ex; \
    microdnf -y update; \
    microdnf -y install glibc-langpack-en platform-python; \
    /usr/libexec/platform-python -m pip install pip --upgrade; \
    microdnf -y remove platform-python-pip; \
    microdnf clean all; \
    rm -rf /var/cache/dnf /var/cache/yum

ENV LC_ALL en_US.utf-8
ENV LANG en_US.utf-8
ARG PG_MAJOR=14

RUN set -ex; \
    export GNUPGHOME="$(mktemp -d)"; \
    gpg --batch --keyserver keyserver.ubuntu.com --recv-keys \
        430BDF5C56E7C94E848EE60C1C4CBDCDCD2EFD2A \
        99DB70FAE1D7CE227FB6488205B555B38483C65D \
        94E279EB8D8F25B21810ADF121EA45AB2F86D6A1 \
        736AF5116D9C40E2AF6B074BF9B9FEE7764429E6; \
    gpg --batch --export --armor 430BDF5C56E7C94E848EE60C1C4CBDCDCD2EFD2A > ${GNUPGHOME}/RPM-GPG-KEY-Percona; \
    gpg --batch --export --armor 99DB70FAE1D7CE227FB6488205B555B38483C65D > ${GNUPGHOME}/RPM-GPG-KEY-centosofficial; \
    gpg --batch --export --armor 94E279EB8D8F25B21810ADF121EA45AB2F86D6A1 > ${GNUPGHOME}/RPM-GPG-KEY-EPEL-8; \
    gpg --batch --export --armor 736AF5116D9C40E2AF6B074BF9B9FEE7764429E6 > ${GNUPGHOME}/RPM-GPG-KEY-CentOS-SIG-Cloud; \
    rpmkeys --import \
        ${GNUPGHOME}/RPM-GPG-KEY-Percona \
        ${GNUPGHOME}/RPM-GPG-KEY-centosofficial \
        ${GNUPGHOME}/RPM-GPG-KEY-EPEL-8 \
        ${GNUPGHOME}/RPM-GPG-KEY-CentOS-SIG-Cloud; \
    microdnf install -y findutils; \
    curl -Lf -o /tmp/epel-release.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm; \
    curl -Lf -o /tmp/percona-release.rpm https://repo.percona.com/yum/percona-release-latest.noarch.rpm; \
    rpmkeys --checksig /tmp/percona-release.rpm; \
    rpmkeys --checksig /tmp/epel-release.rpm; \
    rpm -i /tmp/percona-release.rpm /tmp/epel-release.rpm; \
    rm -rf "$GNUPGHOME" /tmp/percona-release.rpm /tmp/epel-release.rpm; \
    curl -Lf -o /tmp/python3-pyparsing.rpm https://vault.centos.org/8.5.2111/cloud/x86_64/openstack-train/Packages/p/python3-pyparsing-2.4.6-1.el8.noarch.rpm; \
    rpmkeys --checksig /tmp/python3-pyparsing.rpm; \
    rpm -i /tmp/python3-pyparsing.rpm; \
    rm -rf /tmp/python3-pyparsing.rpm; \
    rpm --import /etc/pki/rpm-gpg/PERCONA-PACKAGING-KEY; \
    percona-release enable ppg-${PG_MAJOR} release

RUN set -ex; \
    curl -Lf -o /tmp/c-ares.rpm http://vault.centos.org/centos/8/BaseOS/x86_64/os/Packages/c-ares-1.13.0-5.el8.x86_64.rpm; \
    rpmkeys --checksig /tmp/c-ares.rpm; \
    rpm -i /tmp/c-ares.rpm; \
    rm -rf /tmp/c-ares.rpm; \
    microdnf -y update; \
    microdnf -y install \
        bind-utils \
        gettext \
        hostname \
        perl \
        tar \
        procps-ng \
        nss_wrapper \
        percona-pgbouncer; \
    sed -i 's/enabled=1/enabled=0/' /etc/yum.repos.d/epel*.repo; \
    microdnf -y clean all

RUN set -ex; \
    mkdir -p /opt/crunchy/bin /opt/crunchy/conf /pgconf

COPY bin/pgbouncer /opt/crunchy/bin
COPY bin/common /opt/crunchy/bin
COPY conf/pgbouncer /opt/crunchy/conf
COPY licenses /licenses

RUN set -ex; \
    chown -R 2:0 /opt/crunchy /pgconf; \
    chmod -R g=u /opt/crunchy /pgconf

EXPOSE 6432

VOLUME ["/pgconf"]

# Defines a unique directory name that will be utilized by the nss_wrapper in the UID script
ENV NSS_WRAPPER_SUBDIR="pgbouncer"

ENTRYPOINT ["opt/crunchy/bin/uid_daemon.sh"]

USER 2

CMD ["/opt/crunchy/bin/start.sh"]
