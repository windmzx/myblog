---
title: docker镜像
url: 120.html
id: 120
categories:
  - 未分类
  - 计算机杂论
date: 2018-04-06 18:07:05
tags:
- docker
---

酸酸乳

    FROM ubuntu:16.04
    MAINTAINER mzx 2281927774@qq.com
    RUN apt-get update --fix-missing && apt-get install -y git && apt-get clean
    RUN git clone https://github.com/ssrbackup/shadowsocksr /usr
    
    CMD ["python","/usr/shadowsocksr/shadowsocks/local.py","-c","/etc/shadowsocks.json"]

  python环境

    FROM ubuntu:16.04
    MAINTAINER mzx 2281927774@qq.com
    RUN apt-get update --fix-missing && apt-get install -y wget bzip2 ca-
    libglib2.0-0 libxext6 libsm6 libxrender1 git vim fish
    
    RUN echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh
    wget --quiet https://repo.continuum.io/archive/Anaconda3-5.0.1-Li
    /bin/bash ~/anaconda.sh -b -p /opt/conda && \
    rm ~/anaconda.sh
    
    RUN apt-get install -y curl grep sed dpkg && \
    TINI_VERSION=`curl https://github.com/krallin/tini/releases/lates
    curl -L "https://github.com/krallin/tini/releases/download/v${TIN
    dpkg -i tini.deb && \
    rm tini.deb && \
    apt-get clean
    
    ENV PATH /opt/conda/bin:$PATH
    
    ENTRYPOINT [ "/usr/bin/tini", "--" ]
    CMD [ "/bin/bash" ]