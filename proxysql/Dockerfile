FROM debian:8

ENV PROXYSQL_VERSION=2.0.3
ENV MONITOR_CONFIG_CHANGE=true

RUN apt-get update && \
    apt-get install -y wget mysql-client inotify-tools procps && \
    rm -rf /var/lib/apt/lists/*  

RUN wget https://github.com/sysown/proxysql/releases/download/v${PROXYSQL_VERSION}/proxysql_${PROXYSQL_VERSION}-debian8_amd64.deb \
    && dpkg --install proxysql_${PROXYSQL_VERSION}-debian8_amd64.deb \
    && rm -rf proxysql_${PROXYSQL_VERSION}-debian8_amd64.deb \
    && /etc/init.d/proxysql stop \
    && touch /var/log/proxysql.log

COPY etc/proxysql.cnf /etc/proxysql.cnf

VOLUME /var/lib/proxysql
EXPOSE 6032 6033 6080

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]