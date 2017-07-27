FROM daocloud.io/zhulux/baseimage:latest
ENV GEM_SOURCE https://rubygems.org
ENV BUNDLER_VERSION 1.15.3A

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]
RUN apt-get update && apt-get install software-properties-common && \
    apt-add-repository ppa:brightbox/ruby-ng && apt-get update \
    apt-get install -y ruby2.3 ruby2.3-dev && \
    gem install bundler && \
    apt-get remove --auto-remove -y software-properties-common

    
# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
