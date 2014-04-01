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

RUN add-apt-repository -y ppa:chris-lea/node.js
RUN apt-get -y update
RUN apt-get -y install nodejs

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
RUN adduser ubuntu www-data

# polychart
RUN mkdir -p /opt/polychart \
  && cd /opt/polychart \
  && git clone https://github.com/polychart/builder 
RUN sed -i 's/git@github.com:Polychart\/polychart2.git/https:\/\/github.com\/Polychart\/polychart2/g' /opt/polychart/builder/.gitmodules
RUN cd /opt/polychart/builder \
  && git submodule update --init --recursive
RUN cd /opt/polychart/builder \
  && pip install -r requirements \
  && npm install \
  && npm install -g grunt-cli
RUN sed -i 's/--nothreading/0.0.0.0:8000\"\, \"--nothreading/g' /opt/polychart/builder/makeTools/runServerTask.coffee 
RUN sed -i "s/_prompt('First name: ')/args\[0\]/g" /opt/polychart/builder/server/polychart/main/management/commands/createuser.py
RUN sed -i "s/_prompt('Last name: ')/args\[1\]/g" /opt/polychart/builder/server/polychart/main/management/commands/createuser.py
RUN sed -i "s/_prompt('Username: ')/args\[2\]/g" /opt/polychart/builder/server/polychart/main/management/commands/createuser.py
RUN sed -i "s/_prompt('Password: '\, maskInput=True)/args\[3\]/g" /opt/polychart/builder/server/polychart/main/management/commands/createuser.py

RUN cd /opt/polychart/builder \
  && (grunt &) \
  && sleep 10 \
  && killall grunt \
  && sleep 15 \
  && (grunt &) \
  && sleep 10 \
  && python manage.py createuser hoge hoge hoge hoge \
  && sleep 5 \
  && killall grunt

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
## polychart
RUN echo '[program:polychart]' >> /etc/supervisord.conf
RUN echo 'command=/bin/bash -c "cd /opt/polychart/builder && exec grunt"' >> /etc/supervisord.conf
RUN echo '' >> /etc/supervisord.conf

EXPOSE 22 3306 8000

CMD ["/usr/bin/supervisord"]
