FROM daocloud.io/zhulux/baseimage:latest
ENV GEM_SOURCE https://rubygems.org
ENV BUNDLER_VERSION 1.15.3

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]
RUN apt-get install -y ruby2.3 ruby2.3-dev && \
    gem install bundler

    
# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
