#
# Docker image configuration for dsbuilder configured with a locally available
# mariadb database and driver added to wildfly.
#
FROM teiidkomodo/dsbuilder:vanilla-1.0
MAINTAINER Teiid-Komodo Community <http://teiiddesigner.jboss.org>

#
# Env variables
#
ENV BIN_DIR /usr/local/bin
ENV MARIADB_DIR /opt/mariadb
ENV JBOSS_HOME /opt/jboss/wildfly
ENV MARIADB_PASS 'admin'

#
# Change to user root
#
USER root

#
# Install mariadb-server using yum
#
RUN yum -y install --setopt=tsflags=nodocs epel-release && \ 
    yum -y install --setopt=tsflags=nodocs mariadb-server bind-utils pwgen psmisc hostname && \
    yum -y erase vim-minimal && \
    yum -y update && yum clean all

#
# Fix permissions to allow for running on openshift
#
ADD https://raw.githubusercontent.com/CentOS/CentOS-Dockerfiles/master/mariadb/centos7/fix-permissions.sh ${BIN_DIR}/
RUN chmod +x ${BIN_DIR}/fix-permissions.sh && \
    ${BIN_DIR}/fix-permissions.sh /var/lib/mysql/   && \
    ${BIN_DIR}/fix-permissions.sh /var/log/mariadb/ && \
    ${BIN_DIR}/fix-permissions.sh /var/run/

#
# Copy mariadb exec script and set permissions
#
COPY mariadb/mariadb-exec.sh ${BIN_DIR}/
RUN chmod +x ${BIN_DIR}/mariadb-exec.sh

#
# Copy mariadb additional options to /etc
#
COPY mariadb/etc/my.cnf.d/* /etc/my.cnf.d/

#
# Copy wildfly exec script and set permissions
#
COPY wildfly-exec.sh ${BIN_DIR}/
RUN chmod +x ${BIN_DIR}/wildfly-exec.sh

#
# ADD and COPY always create files as root. Need to call
# chown to have them owned by the jboss user
#
RUN chown -R jboss:jboss ${JBOSS_HOME}/standalone

#
# Expose the mariadb port, which should be in addition to the wildfly ports
#
EXPOSE 3306

#
# The default entrypoint for this container
#
ENTRYPOINT ${BIN_DIR}/mariadb-exec.sh && ${BIN_DIR}/wildfly-exec.sh
