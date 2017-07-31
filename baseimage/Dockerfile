FROM phusion/baseimage:0.9.22

# Set basic command
CMD /sbin/my_init

# Set APT mirror & install apps
RUN sed -i 's/archive.ubuntu.com/cn.archive.ubuntu.com/g' /etc/apt/sources.list
RUN apt update &&                                            \
    apt upgrade -y -o Dpkg::Options::="--force-confold" &&   \
    apt install -y                                           \
        iputils-ping      # ping                             \
        dnsutils          # dig/nslookup/etc                 \
        silversearcher-ag #                                  \
        wget              #                                  \
        htop              #                                  \
        tzdata            #
RUN apt clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set the locale and timezone
RUN echo "Asia/Shanghai"     > /etc/timezone   && \
    echo "zh_CN.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Essential env vars
ENV TERM=xterm \
    LANG=zh_CN.UTF-8 \
    LANGUAGE=zh_CN.UTF-8 \
    LC_ALL=zh_CN.UTF-8

# Customize bash shell
ADD bashrc_customized /root/.bashrc_customized
RUN echo "source /root/.bashrc_customized" >> /root/.bashrc


