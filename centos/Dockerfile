#FROM quay.io/mbach/r-base:centos7

FROM centos:centos7

# podman build -t r-base:centos7 -f Dockerfile.centos7 .

RUN yum install -y epel-release \
  && yum update -y \
  && yum -y install R \
  && yum clean all \
  && rm -rf /var/cache/yum

# podman build -t rstudio:centos7 -f Dockerfile.centos7 .

ARG RSTUDIO_VERSION
ENV RSTUDIO_VERSION=${RSTUDIO_VERSION:-1.2.5033}
ARG S6_VERSION
ARG PANDOC_TEMPLATES_VERSION
ENV S6_VERSION=${S6_VERSION:-v1.21.7.0}
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2
ENV PATH=/usr/lib/rstudio-server/bin:$PATH
ENV PANDOC_TEMPLATES_VERSION=${PANDOC_TEMPLATES_VERSION:-2.6}
ENV USER_NAME=rstudio-server

COPY start_studio /usr/bin/start_studio

RUN yum install -y git wget \
  && wget https://download2.rstudio.org/server/centos6/x86_64/rstudio-server-rhel-${RSTUDIO_VERSION}-x86_64.rpm \
  && yum -y install rstudio-server-rhel-${RSTUDIO_VERSION}-x86_64.rpm \
  && rm -f rstudio-server-rhel-${RSTUDIO_VERSION}-x86_64.rpm \
  && yum clean all \
  && rm -rf /var/cache/yum \
  && mkdir -p /home/rstudio-server \
  && chown 'rstudio-server:rstudio-server' /home/rstudio-server \
  && echo 'auth-minimum-user-id=0' >> /etc/rstudio/rserver.conf \
  ## Symlink pandoc & standard pandoc templates for use system-wide
  && ln -s /usr/lib/rstudio-server/bin/pandoc/pandoc /usr/local/bin \
  && ln -s /usr/lib/rstudio-server/bin/pandoc/pandoc-citeproc /usr/local/bin \
  && git clone --recursive --branch ${PANDOC_TEMPLATES_VERSION} https://github.com/jgm/pandoc-templates \
  && mkdir -p /opt/pandoc/templates \
  && cp -r pandoc-templates*/* /opt/pandoc/templates && rm -rf pandoc-templates* \
  && mkdir /root/.pandoc && ln -s /opt/pandoc/templates /root/.pandoc/templates \
  ## use more robust file locking to avoid errors when using shared volumes:
  && echo 'lock-type=advisory' >> /etc/rstudio/file-locks \
  ## configure git not to request password each time
  && git config --system credential.helper 'cache --timeout=3600' \
  && git config --system push.default simple \
  && chgrp -R 0 /usr/lib/rstudio-server/bin \
  && chmod -R g=u /usr/lib/rstudio-server/bin /etc/passwd

EXPOSE 8787


ENTRYPOINT ["/usr/bin/start_studio"]
CMD ["/usr/lib/rstudio-server/bin/rserver","--server-daemonize=0","--www-port=8787"]  
USER 10001