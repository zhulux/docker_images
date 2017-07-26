FROM phusion/baseimage:0.9.22

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

# Upgrade the Ubunut in the image.
RUN sed -i 's/archive.ubuntu.com/cn.archive.ubuntu.com/g' /etc/apt/sources.list
RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confold" && apt-get install iputils-ping tzdata -y
# - Set the locale + timezone --------------------------------------------------
RUN echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure --frontend=noninteractive locales
ENV TERM xterm
ENV LANG zh_CN.UTF-8
ENV LANGUAGE zh_CN.UTF-8
ENV LC_ALL zh_CN.UTF-8

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*



