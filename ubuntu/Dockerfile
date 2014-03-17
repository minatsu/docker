FROM ubuntu:saucy

ENV DEBIAN_FRONTEND noninteractive
RUN sed 's/main$/main universe/' -i /etc/apt/sources.list
RUN apt-get update
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
  libapache2-mod-php5 \
  php-apc \
  php5-json \
  php5-cli \
  php5-mysql \
  php5-gd \
  php5-mcrypt \
  php5-curl

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
## apache2
RUN echo '[program:apache2]' >> /etc/supervisord.conf
RUN echo 'command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"' >> /etc/supervisord.conf
RUN echo '' >> /etc/supervisord.conf

EXPOSE 22 80 3306

CMD ["/usr/bin/supervisord"]
