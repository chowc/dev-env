FROM debian:bullseye
ENV USER=deploy
ENV PASSWD=deploy
ENV WORKDIR=projects

# Install dependencies
RUN apt-get -qq update; \
    apt-get install -qqy --no-install-recommends \
        gnupg2 wget ca-certificates \
        autoconf automake cmake dpkg-dev file make patch libc6-dev

# Set repository key
RUN wget -nv -O - https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add -

# Install
RUN echo "deb http://apt.llvm.org/bullseye/ llvm-toolchain-bullseye main" \
        > /etc/apt/sources.list.d/llvm.list; \
    apt-get -qq update && \
    apt-get install -qqy -t llvm-toolchain-bullseye \
        clang clang-tidy clang-format lld llvm && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update

RUN apt-get install -y build-essential
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y cmake sudo ccache
RUN apt-get install -y gdb git net-tools openssh-server

RUN mkdir /var/run/sshd
RUN sed -ri 's/^#PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

RUN apt-get install -y rsync
RUN sed -ri 's/RSYNC_ENABLE=false/RSYNC_ENABLE=true/g' /etc/default/rsync
COPY rsync.conf /etc

RUN apt-get install -y git

RUN mkdir /root/sync

RUN useradd -m ${USER} && yes ${PASSWD} | passwd ${USER}

# 赋予sudo权限并允许无密码sudo
RUN echo ${USER}' ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
#RUN chmod 644 /etc/sudoers

CMD ["/usr/sbin/sshd", "-D"]

USER ${USER}
RUN mkdir -p /home/${USER}/${WORKDIR}/
ENV LC_ALL C.UTF-8