FROM ubuntu:saucy

MAINTAINER minatsu

ENV DEBIAN_FRONTEND noninteractive
RUN sed 's/main$/main universe/' -i /etc/apt/sources.list
RUN apt-get -y update
RUN apt-get -y upgrade
RUN apt-get -y install \
  vim \
  curl \
  git \
  unzip \
  zip \
  openssh-server \
  supervisor \
  language-pack-ja \
  build-essential \
  python-pip \
  mysql-server \
  mysql-client \
  python-software-properties \
  libffi-dev \
  libmysqlclient-dev \
  libpq-dev \
  libxml2-dev \
  libxslt-dev \
  python-setuptools \
  python-virtualenv \
  python2.7-dev \
  software-properties-common \
  socat

RUN mkdir /var/run/sshd
RUN echo "root:hogehoge" | chpasswd
RUN locale-gen en_US en_US.UTF-8 && dpkg-reconfigure locales
RUN locale-gen ja_JP ja_JP.UTF-8 && dpkg-reconfigure locales

# sshd
RUN sed -i 's/.*session.*required.*pam_loginuid.so.*/session optional pam_loginuid.so/g' /etc/pam.d/sshd
RUN /bin/echo -e "LANG=\"ja_JP.UTF-8\"" > /etc/default/local
# mysqld
RUN sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/my.cnf
RUN (mysqld_safe &); sleep 3; mysql --execute="GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'pass' WITH GRANT OPTION;"
RUN (mysqld_safe &); sleep 3; mysql --execute="GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'pass' WITH GRANT OPTION;"

# clean packages
RUN apt-get clean
RUN rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/*

# Add User
RUN adduser --disabled-password --gecos "" ubuntu \
  && echo "ubuntu ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers \
  && echo 'ubuntu:ubuntu' | chpasswd

# openssl
RUN wget -O /tmp/openssl-1.0.1g.tar.gz "http://www.openssl.org/source/openssl-1.0.1g.tar.gz"
RUN cd /opt \
  && tar xvfz /tmp/openssl-1.0.1g.tar.gz \
  && cd openssl-1.0.1g \
  && ./config -DOPENSSL_NO_NEXTPROTONEG \
  && make

# supervisord
RUN echo_supervisord_conf > /etc/supervisord.conf
RUN sed 's/^nodaemon=false/nodaemon=true/' -i /etc/supervisord.conf
## sshd
RUN echo '[program:sshd]' >> /etc/supervisord.conf
RUN echo 'command=/usr/sbin/sshd -D' >> /etc/supervisord.conf
RUN echo '' >> /etc/supervisord.conf
## mysql
RUN echo '[program:mysqld]' >> /etc/supervisord.conf
RUN echo 'command=/usr/bin/mysqld_safe' >> /etc/supervisord.conf
RUN echo '' >> /etc/supervisord.conf
## struts
RUN echo '[program:struts]' >> /etc/supervisord.conf
RUN echo 'command=/bin/bash -c "/opt/apache-tomcat-8.0.5/bin/startup.sh && wait && tail -f /opt/apache-tomcat-8.0.5/logs/catalina.out"' >> /etc/supervisord.conf
RUN echo '' >> /etc/supervisord.conf

EXPOSE 22 3306 8080 4433

CMD ["/usr/bin/supervisord"]

