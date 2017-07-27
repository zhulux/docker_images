FROM ubuntu:latest
MAINTAINER Zhulux.Inc

# Install dep soft for fluentd
RUN apt-get update && apt-get install -y --no-install-recommends curl ruby-dev sudo libffi-dev
RUN curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-xenial-td-agent2.sh | sh && mkdir -p /var/log/tg-agent \
    && ulimit -n 100000

# Set Env
ENV GEM_HOME /opt/td-agent/embedded/lib/ruby/gems/2.1.0/
ENV GEM_PATH /opt/td-agent/embedded/lib/ruby/gems/2.1.0/
ENV PATH /opt/td-agent/embedded/bin/:$PATH
ENV FLUENTD_CONF="fluentd.conf"

# Add default config file for fluentd & install fluent out kafka plugin 
ADD ./fluentd.conf /etc/td-agent/fluentd.conf
RUN gem install fluent-plugin-kafka --no-ri --no-rdoc

# Clean cache
RUN rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

EXPOSE 24224 5140

CMD td-agent -c /etc/td-agent/${FLUENTD_CONF}

