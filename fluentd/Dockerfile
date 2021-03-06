FROM centos:centos7

MAINTAINER OpenShift Development <dev@lists.openshift.redhat.com>

ENV HOME=/opt/app-root/src \
  PATH=/opt/app-root/src/bin:/opt/app-root/bin:$PATH \
  RUBY_VERSION=2.0 \
  FLUENTD_VERSION=0.12.29 \
  GEM_HOME=/opt/app-root/src

LABEL io.k8s.description="Fluentd container for collecting of docker container logs" \
  io.k8s.display-name="Fluentd ${FLUENTD_VERSION}" \
  io.openshift.expose-services="9200:http, 9300:http" \
  io.openshift.tags="logging,elk,fluentd"

# activesupport version 5.x requires ruby 2.2
# iproute needed for ip command to get ip addresses
RUN rpmkeys --import file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 && \
    yum install -y --setopt=tsflags=nodocs \
      gcc-c++ \
      ruby \
      ruby-devel \
      libcurl-devel \
      make \
      iproute && \
    yum clean all
RUN mkdir -p ${HOME} && \
    gem install -N --conservative --minimal-deps \
      fluentd:${FLUENTD_VERSION} \
      'activesupport:<5' \
      fluent-plugin-kubernetes_metadata_filter \
      fluent-plugin-elasticsearch \
      fluent-plugin-systemd \
      systemd-journal \
      fluent-plugin-rewrite-tag-filter \
      fluent-plugin-secure-forward

ADD configs.d/ /etc/fluent/configs.d/
ADD run.sh generate_throttle_configs.rb ${HOME}/
ADD filter-common-data-model.rb /etc/fluent/plugin/

RUN mkdir -p /etc/fluent/configs.d/{dynamic,user} && \
    chmod 777 /etc/fluent/configs.d/dynamic && \
    ln -s /etc/fluent/configs.d/user/fluent.conf /etc/fluent/fluent.conf

WORKDIR ${HOME}
USER 0
CMD ["sh", "run.sh"]
