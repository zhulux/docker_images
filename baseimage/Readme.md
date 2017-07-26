# 基于 Ubuntu 的 Docker 基础镜像

> 由逐鹿X开发团队基于 [Baseimage-docker](http://phusion.github.io/baseimage-docker/) 裁剪。

Baseimage-docker 是一个特殊的 Docker 镜像，在 Docker 容器内做了配置，并且可以正确使用。它确实是一个 Ubuntu 系统, 除此之外进行了如下修订：

 * 为更加友好的支持 Docker，做了修订。
 * 在 Docker 环境下，作为管理工具特别有用。
 * 在不违反 Docker 哲学的前提下，能够很容易的运行多进程的机制。

可以把它作为自己的基础 Docker 镜像。

<a name="what-are-the-problems-with-the-stock-ubuntu-base-image"></a>
### 原生的 Ubuntu 基础镜像有什么问题呢？          

原生 Ubuntu 不是为了在 Docker 内运行而设计的。它的初始化系统 Upstart，假定运行的环境要么是真实的硬件，要么是虚拟的硬件，而不是在 Docker 容器内。但是在一个 Docker 的容器内，并不需要一个完整的系统，你需要的只是一个很小的系统。但是如果你不是非常熟悉 Unix 的系统模型，想要在 Docker 容器内裁减出最小的系统，会碰到很多难以正确解决的陌生的技术坑。这些坑会引起很多莫名其妙的问题。

Baseimage-docker 让这一切完美。在"内容"部分描述了所有这些修改。

<a name="why-use-baseimage-docker"></a>
### 为什么使用 baseimage-docker？

你自己可以从 Dockerfile 配置一个原生 `ubuntu` 镜像，为什么还要多此一举的使用 baseimage-docker 呢?

 * 配置一个 Docker 友好的基础系统并不是一个简单的任务。如前所述，过程中会碰到很多坑。当你搞定这些坑之后，只不过是又重新发明了一个 baseimage-docker 而已。使用 baseimage-docker 可以免去你这方面需要做的努力。          
 * 减少需要正确编写 Dockerfile 文件的时间。你不用再担心基础系统，可以专注于你自己的技术栈和你的项目。            
 * 减少需要运行 `docker build` 的时间，让你更快的迭代 Dockerfile。         
 * 减少了重新部署的时的下载时间。Docker 只需要在第一次部署的时候下载一次基础镜像。在随后的部署中,只需要改变你下载之后对基础镜像进行修改的部分。

-----------------------------------------

**目录**

 * [镜像里面有什么?](#whats_inside)
   * [概述](#whats_inside_overview)
   * [等等,我认为Docker在一个容器中只能允许运行一个进程?](#docker_single_process)           
   * [Baseimage-docker更侧重于“胖容器”还是“把容器当作虚拟机”？](#fat_containers)            
 * [查看baseimage-docker](#inspecting)
 * [使用baseimage-docker作为基础镜像](#using)
   * [开始](#getting_started)
   * [增加额外的后台进程](#adding_additional_daemons)
   * [容器启动时运行脚本](#running_startup_scripts)
   * [环境变量](#environment_variables)
     * [集中定义自己的环境变量](#envvar_central_definition)
     * [保存环境变量](#envvar_dumps)
     * [修改环境变量](#modifying_envvars)
 * [常见问题](#faq)   

-----------------------------------------

<a name="whats_inside"></a>
## 镜像里面有什么？

<a name="whats_inside_overview"></a>
### 概述

| 模块        | 为什么包含这些？以及备注 |
| ---------------- | ------------------- |
| Ubuntu 16.04 LTS | 基础系统。 |
| 一个**正确**的初始化进程  | *主要文章：[Docker 和 PID 1 僵尸进程回收问题](http://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/)*<br/><br/>根据 Unix 进程模型，[初始化进程](https://en.wikipedia.org/wiki/Init) -- PID 1 -- 继承了所有[孤立的子进程](https://en.wikipedia.org/wiki/Orphan_process)，并且必须[进行回收](https://en.wikipedia.org/wiki/Wait_(system_call))。大多数Docker容器没有一个初始化进程可以正确的完成此操作，随着时间的推移会导致他们的容器出现了大量的[僵尸进程](https://en.wikipedia.org/wiki/Zombie_process)。<br/><br/>而且， `docker stop` 发送 SIGTERM 信号给初始化进程，照理说此信号应该可以停止所有服务。不幸的是由于它们对硬件进行了关闭操作，导致 Docker 内的大多数初始化系统没有正确执行。这会导致进程强行被 SIGKILL 信号关闭，从而丧失了一个正确取消初始化设置的机会。这会导致文件损坏。<br/><br/> Baseimage-docker 配有一个名为 `/sbin/my_init` 的初始化进程来同时正确的完成这些任务。 |
| 修复了 APT 与 Docker 不兼容的问题 | 详情参见：https://github.com/dotcloud/docker/issues/1024 。 |
| syslog-ng | 对于很多服务－包括kernel自身，都需要一个 syslog 后台进程，以便可以正确的将 log 输出到 `/var/log/syslog` 中。如果没有运行 syslog 后台进程，很多重要的信息就会默默的丢失了。<br/><br/>只对本地进行监听。所有 syslog 信息会被转发给 `“docker logs”`。 |
| logrotate | 定期转存和压缩日志。 |
| `vim` | vim 编辑器 |
| cron | 为了保证 cron 任务能够工作，必须运行 cron 后台进程。 |
| [runit](http://smarden.org/runit/) | 替换 Ubuntu 的 Upstart。用于服务监控和管理。比 SysV init 更容易使用，同时当这些服务崩溃之后，支持后台进程自动重启。比 Upstart 更易使用，更加的轻量级。 |
| `setuser` | 使用其它账户运行命令的工具。比 `su` 更容易使用，比使用 `sudo` 有那么一点优势，跟 `chpst` 不同，这个工具需要正确的设置 `$HOME`。像 `/sbin/setuser` 这样。 |
| `ping` | ping 命令 |

Baseimage-docker 非常的轻量级：仅仅占用6MB内存。

<a name="docker_single_process"></a>
### 等等,我认为 Docker 在一个容器中就运行一个进程吗?
绝对不是这样的. 在一个 docker 容器中,运行多个进程也是很好的. 事实上,没有什么技术原因限制你只运行一个进程,运行很多的进程,只会把容器中系统的基本功能搞的更乱,比如 syslog.

Baseimage-docker *鼓励* 通过 runit 来运行多进程.

<a name="inspecting"></a>
## 检测一下 baseimage-docker

要检测镜像,执行下面的命令:

    docker run --rm -t -i zhulux/baseimage:latest /sbin/my_init -- bash -l

你不用手动去下载任何文件.上面的命令会自动从docker仓库下载baseimage-docker镜像.

<a name="using"></a>
## 使用 baseimage-docker 作为基础镜像

<a name="getting_started"></a>
### 入门指南

镜像名字叫 `zhulux/baseimage` ,在 Docker 仓库上也是可用的.

下面的这个是一个 Dockerfile 的模板.

	# 使用 zhulux/baseimage 作为基础镜像,去构建你自己的镜像.
	FROM zhulux/baseimage:lastest

	# 设置正确的环境变量.
	ENV HOME /root

	# 生成 SSH keys,baseimage-docker 不包含任何的key,所以需要你自己生成.你也可以注释掉这句命令,系统在启动过程中,会生成一个.
	RUN /etc/my_init.d/00_regen_ssh_host_keys.sh

	# 初始化 baseimage-docker 系统
	CMD ["/sbin/my_init"]

	# 这里可以放置你自己需要构建的命令

	# 当完成后,清除APT.
	RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


<a name="adding_additional_daemons"></a>
### 增加后台进程

你可以通过 runit 工具向你的镜像中添加后台进程(例如:你自己的某些应用).你需要编写一个运行你需要的后台进程的脚本就可以了,runit 工具会保证它的正常运行,如果进程死掉,runit 也会重启它的.

脚本的名称必须是 `run`,必须是可以运行的,它需要放到 `/etc/service/<NAME>`.

这里有一个例子,向你展示如果运行 memcached 服务的.

	### memcached.sh (确定文件的权限是 `chmod +x`):
	#!/bin/sh
	# `/sbin/setuser memcache` 指定一个 `memcache` 用户来运行命令.如果你忽略了这部分,就会使用 root 用户执行.
	exec /sbin/setuser memcache /usr/bin/memcached >>/var/log/memcached.log 2>&1

	### 在 Dockerfile 中:
    RUN mkdir /etc/service/memcached
    COPY memcached.sh /etc/service/memcached/run

注意脚本必须运行在后台的,**不能让他们进程进行 daemonize/fork**.通常,后台进程会提供一个标志位或者配置文件.

<a name="running_startup_scripts"></a>
### 在容器启动的时候,运行脚本.

baseimage-docker的初始化脚本 `/sbin/my_init` ,在启动的时候进程运行,按照下面的顺序:

 * 如果 `/etc/my_init.d` 存在,则按照字母顺序执行脚本.
 * 如果 `/etc/rc.local` 存在,则执行里面的脚本.

所有的脚本都是正确退出的,例如:退出的 code 是0.如果有任何脚本以非0的 code 退出,启动就会失败.

下面的例子向你展示了怎么添加一个启动脚本.这个脚本很简单的记录的一个系统启动时间,将启动时间记录到 `/tmp/boottime.txt`.

    ### 在 logtime.sh (文件权限chmod +x):
    #!/bin/sh
    date > /tmp/boottime.txt

    ### 在 Dockerfile中:
    RUN mkdir -p /etc/my_init.d
    COPY logtime.sh /etc/my_init.d/logtime.sh


<a name="environment_variables"></a>
### 环境变量

如果你使用 `/sbin/my_init` 作为主容器命令,那么通过 `docker run --env` 或者在 Dockerfile 文件中设置的 `ENV` 环境变量,都会被 `my_init` 读取.

 * 在 Unix 系统中,环境变量都会被子进程给继承.这就意味着,子进程不可能修改环境变量或者修改其他进程的环境变量.
 * 由于上面提到的一点,这里没有一个可以为所有应用和服务集中定义环境的地方.Debian 提供了一个 `/etc/environment` 文件,解决一些问题.
 * 某些服务更改环境变量是为了给子进程使用.Nginx有这样的一个例子:它移除了所有的环境变量,除非你通过 `env` 进行了配置,明确了某些是保留的.如果你部署了任何应用在 Nginx镜像(例如:使用[passenger-docker](https://github.com/phusion/passenger-docker)镜像或者使用 Phusion Passenger 作为你的镜像.),那么你通过 Docker ,你不会看到任何环境变量.


`my_init`提供了一个办法来解决这些问题.

<a name="envvar_central_definition"></a>
#### 集中定义你的环境变量

在启动的时候,在执行[startup scripts](#running_startup_scripts),`my_init` 会从 `/etc/container_environment` 导入环境变量.这个文件夹下面,包含的文件,文件被命名为环境变量的名字.文件内容就是环境变量的值.这个文件夹是因此是一个集中定义你的环境变量的好地方,它会继承到所有启动项目和 Runit 管理的服务中.

给个例子,在你的 Dockerfile 如何定义一个环境变量:

    RUN echo Apachai Hopachai > /etc/container_environment/MY_NAME

你可以按照下面这样验证:

    $ docker run -t -i <YOUR_NAME_IMAGE> /sbin/my_init -- bash -l
    ...
    *** Running bash -l...
    # echo $MY_NAME
    Apachai Hopachai

**换行处理**

如果你观察仔细一点,你会注意到 'echo' 命令,实际上在它是在新行打印出来的.为什么 $MY_NAME 没有包含在一行呢? 因为 `my_init` 在尾部有个换行字符.如果你打算让你的值包含一个新行,你需要增*另外*一个新字符,像这样:

    RUN echo -e "Apachai Hopachai\n" > /etc/container_environment/MY_NAME

<a name="envvar_dumps"></a>
#### 环境变量存储

上面提到集中定义环境变量,它不会从子服务进程改变父服务进程或者重置环境变量.而且, `my_init` 也会很容易的让你查询到原始的环境变量是什么.

在启动的时候, `/etc/container_environment`, `my_init` 中的变量会存储起来,并且导入到环境变量中,例如一下的格式:

 * `/etc/container_environment`
 * `/etc/container_environment.sh`- 一个bash存储的环境变量格式.你可以从这个命令中得到base格式的文件.
 * `/etc/container_environment.json` - 一个json格式存储的环境变量格式.

多种格式可以让你不管采用什么语言 `/apps` 都可以很容易使用环境变量.

这里有个例子,展示怎么使用:

    $ docker run -t -i \
      --env FOO=bar --env HELLO='my beautiful world' \
      phusion/baseimage:<VERSION> /sbin/my_init -- \
      bash -l
    ...
    *** Running bash -l...
    # ls /etc/container_environment
    FOO  HELLO  HOME  HOSTNAME  PATH  TERM  container
    # cat /etc/container_environment/HELLO; echo
    my beautiful world
    # cat /etc/container_environment.json; echo
    {"TERM": "xterm", "container": "lxc", "HOSTNAME": "f45449f06950", "HOME": "/root", "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", "FOO": "bar", "HELLO": "my beautiful world"}
    # source /etc/container_environment.sh
    # echo $HELLO
    my beautiful world

<a name="modifying_envvars"></a>
#### 修改环境变量

通过修改 `/etc/container_environment` 这个文件,很有可能修改了 `my_init` 中的环境变量.之后,每次 `my_init` 启动[启动脚本](#running_startup_scripts),就会重置掉我们自己 `/etc/container_environment` 中的环境变量,也就会导致 `container_environment.sh` 和 `container_environment.json` 重新存储.

但是记住这些:

 * 修改 `container_environment.sh` 和 `container_environment.json` 是没有效果的.
 * Runit 的服务是不能像这样修改环境变量的. `my_init` 运行的时候,只对 `/etc/container_environment` 中的修改是生效的.

<a name="envvar_security"></a>
### 解决Docker没有办法解决的 `/etc/hosts` 的问题

当前是没有办法在docker容器中修改 `/etc/hosts`,这个是因为 [Docker bug 2267](https://github.com/dotcloud/docker/issues/2267).Baseimage-docker 包含了解决这个问题的办法,你必须明白是怎么修改的.

修改的办法包含在系统库中的 `libnss_files.so.2` 文件,这个文件使用 `/etc/workaround-docker-2267/hosts` 来代替系统使用 `/etc/hosts` .如果需要修改 `/etc/hosts`,你只要修改 `/etc/workaround-docker-2267/hosts` 就可以了.

增加这个修改到你的 Dockerfile.下面的命令修改了文件 `libnss_files.so.2`.

    RUN /usr/bin/workaround-docker-2267

(其实你不用在 Dockerfile 文件中运行这个命令,你可以在容器中运行一个 shell 就可以了.)

验证一下它是否生效了,在你的容器中打开一个 shell ,修改`/etc/workaround-docker-2267/hosts`,检查一下是否生效了:

    bash# echo 127.0.0.1 zhulux.dev >> /etc/workaround-docker-2267/hosts
    bash# ping zhulux.dev
    ...should ping 127.0.0.1...

**注意 apt-get 升级:** 如果 Ubuntu 升级,就有可能将 `libnss_files.so.2` 覆盖掉,那么修改就会失效.你必须重新运行 `/usr/bin/workaround-docker-2267`.为了安全一点,你应该在运行 `apt-get upgrade` 之后,运行一下这个命令.

<a name="faq"></a>
## 常见问题

* 为什么没有 `ipconfig` 命令？
  * 试试 `ip r`
* 有哪些需要注意的坑？
  * 默认系统时区为 `中国标准时间（UTC+8）`， LANG的值为`zh_CN.utf8`
