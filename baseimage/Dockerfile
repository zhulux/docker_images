FROM phusion/baseimage:0.9.22

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

# Upgrade the Ubunut in the image.
RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confold" && apt-get install iputils-ping -y
RUN chmod 755 /etc/container_environment && chmod 644 /etc/container_environment.sh /etc/container_environment.json && /usr/bin/workaround-docker-2267 && rm -rf /etc/service/sshd /etc/my_init.d/00_regen_ssh_host_keys.sh
# - Set the locale + timezone --------------------------------------------------
RUN echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata && \
    sed -i -e 's/# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="zh_CN.UTF-8"'>/etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=zh_CN.UTF-8
ENV TERM xterm
ENV LANG zh_CN.UTF-8
ENV LANGUAGE zh_CN.UTF-8
ENV LC_ALL zh_CN.UTF-8

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


